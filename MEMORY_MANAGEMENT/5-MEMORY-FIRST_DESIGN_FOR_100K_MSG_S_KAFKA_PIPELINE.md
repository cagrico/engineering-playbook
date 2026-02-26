# 100.000 msg/s Kafka Pipeline için Memory-First Tasarım (Pratik Rehber)

Bu doküman, **100k mesaj/s** gibi yüksek throughput’ta bir Kafka consumer (ve genel event pipeline) tasarlarken, belleği “rastgele büyüyen” değil **öngörülebilir (predictable)** hale getirmek için gereken prensipleri ve sayısal düşünme yöntemini anlatır.

---

## 1) Önce problemi doğru kuruyoruz

Kafka tarafında iki kritik gerçek var:

1. **Mesaj akışı süreklidir** (24/7).
2. **Üretim hızı tüketim hızını aşarsa** birikme olur. Birikme = **RAM riski**.

Bu yüzden 100k msg/s sisteminde asıl hedef:

> **Sistemin en kötü durumda bile RAM’i sınırsız büyütmemesi.**
> 

Bunu sağlayan çekirdek fikir:

### “Bounded Everything”

- Bounded concurrency (worker sayısı)
- Bounded queue (iş kuyruğu kapasitesi)
- Bounded batch (batch boyutu)
- Bounded retry (retry sayısı/TTL)
- Bounded in-flight messages (aynı anda işlenen mesaj sayısı)

---

## 2) Temel hesap: Kaç mesaj aynı anda “yaşıyor”?

### 2.1 Concurrency ≈ Throughput × Ortalama işlem süresi

- Throughput: **100.000 msg/s**
- Ortalama işleme süresi: diyelim **5 ms** (0.005 s)
    
    (DB/cache update + decode + validate + routing dahil)
    

**In-flight ≈ 100.000 × 0.005 = 500 mesaj**

Yani, ideal tasarımda aynı anda yaklaşık 500 mesaj sistem içinde “aktif” olur.

Ama işleme süresi 20 ms olursa:

- **100.000 × 0.02 = 2.000 mesaj**

Bu yüzden Kafka pipeline’da “işleme süresi” bellek için direkt belirleyicidir.

---

## 3) Mesaj başına bellek maliyeti (Memory per message)

Bir mesaj işlenirken tipik olarak şunlar oluşur:

- Kafka client’ın verdiği message objesi (metadata + pointers)
- Payload (byte slice)
- Decode edilmiş event struct
- Validation sırasında geçici string/slice/map’ler
- DB driver buffer’ları (özellikle batch kullanmıyorsan)
- Log/metric için formatlanan alanlar

Pratikte iki farklı yaklaşım var:

### A) Mesajı “kopyalamadan” işlemek (ideal)

- Kafka’dan gelen `[]byte` payload’ı referans gibi kullanırsın
- Decode ederken minimum ara obje üretirsin
- Büyük kopyalar çıkmaz

### B) Mesajı her aşamada kopyalamak (kötü)

- Payload kopyalanır
- JSON decode sırasında bir sürü string allocation olur
- Map’lere koyulur, tekrar encode edilir

100k msg/s’te B yaklaşımı RAM + GC’yi patlatır.

---

## 4) Budgeting: In-flight mesaj × mesaj başı bellek

Şimdi “öngörülebilirlik” için bir hedef koyuyoruz.

Örnek hedef:

- In-flight: **1.000 mesaj** (güvenli pay)
- Mesaj başı (aktif processing sırasında): **4 KB** (iyi optimize)
- Total: **~4 MB**

Bu çok güzel.

Ama kötü senaryoda:

- In-flight: 10.000 (backpressure yok)
- Mesaj başı: 10 KB (decode + kopyalama + log)
- Total: **~100 MB** sadece in-flight data

Bir de kuyruk büyürse:

- 1 milyon mesaj birikirse **1M × payload** = direkt felaket.

Buradan çıkan en kritik ders:

> Kafka pipeline’da asıl bellek tehlikesi “in-flight” değil, “unbounded queue / backlog”tur.
> 

---

## 5) Mimari: 100k msg/s için referans pipeline

Aşağıdaki yapı “genel olarak” en sağlıklı yaklaşımdır:

### 5.1 Aşamalar

1. **Poll / Fetch** (Kafka’dan al)
2. **Dispatch** (job queue’ya koy)
3. **Decode & Validate** (minimum allocation)
4. **Process** (DB/cache updates)
5. **Ack / Commit** (offset yönetimi)
6. **Retry / DLQ** (hata yönetimi)

Bu akışta bellek kontrol noktaları:

- Dispatch queue kapasitesi
- Worker sayısı
- Batch boyutu
- Retry mekanizmasının birikmeyi önlemesi

---

## 6) Bounded Worker Pool (Olmazsa olmaz)

### Neden?

Unbounded goroutine açarsan:

- Goroutine sayısı büyür
- Stack + heap artar
- Scheduler yükü artar
- RAM predictability kaybolur

### Prensip

- Worker sayısı sabit: örn **64 / 128**
- Job queue sabit: örn **10.000 job capacity** (hesaplı)

### Queue kapasitesini nasıl seçeriz?

Kuyruk, “mikro burst”leri absorbe eder ama “sonsuz backlog” olmaz.

Örn:

- 100k msg/s
- Queue capacity 10k → **0.1 saniyelik** buffer

Bu, kısa spike’larda faydalı; uzun spike’ta sistem kendini korumalı.

---

## 7) Backpressure planı (asıl kritik)

Backpressure sorusu:

> Worker’lar dolu ve queue full olduğunda ne olacak?
> 

Memory-first cevap:

### Seçenek 1: Poll’u yavaşlat / beklet (tercih)

- Queue dolduysa poll etmeyi durdurursun
- Kafka’nın kendi buffering’ine bırakırsın
- RAM’in şişmez

### Seçenek 2: Mesajı reddet (nadiren)

Kafka consumer’da “reddetmek” genelde yoktur (topic’ten geldiği için). Ama bazı pipeline’larda drop/skip olabilir.

### Seçenek 3: Queue’yu büyüt (yanlış refleks)

Queue’yu büyütmek RAM’i büyütmektir. Sorunu gizler.

> Doğru olan: “Queue dolduysa sistem yavaşlasın ama RAM büyümesin.”
> 

---

## 8) Batch Processing: 100k msg/s’te neredeyse şart

Tek tek DB update yapmak:

- IO call sayısını patlatır
- CPU + allocation + lock contention artar
- Throughput düşer

Batch ile:

- 100 mesaj → 1 DB call
- Allocation overhead amortize olur
- Throughput artar

### Batch size nasıl seçilir?

Trade-off:

- Büyük batch → throughput iyi, latency artabilir
- Küçük batch → latency iyi, throughput düşük

Başlangıç için güvenli aralık:

- 50–500 arası (iş tipine göre)

Memory-first prensip:

- Batch size **üst limitli** olmalı
- Batch bekleme süresi (max wait) olmalı (örn 10–50ms)

---

## 9) Decode/Serialize maliyeti: “Shape” önemli

100k msg/s’te payload formatı belirleyicidir:

- JSON: kolay ama allocation yüksek
- Protobuf/Avro: daha az allocation, daha hızlı
- FlatBuffers gibi: bazı senaryolarda çok hızlı ama karmaşık

Memory-first yaklaşım:

- Eğer event yoğunluğu çok yüksekse ve JSON pahalıysa, binary formatlar ciddi fark yaratır.
- JSON ile kalınacaksa: decode sürecini minimum ara obje ile tasarla (field sayısı, string kopyaları, map’ler vs.)

---

## 10) Retry/DLQ: Bellek açısından nasıl tasarlanır?

Yanlış retry:

- Hatalı mesajlar queue’da birikir
- Aynı mesaj tekrar tekrar denenir
- Sonsuz döngü backlog yaratır

Memory-first retry:

- Retry sayısı sınırlı
- Exponential backoff var
- TTL var
- DLQ’ye yönlendirme var
- Retry queue da **bounded**

Özet:

> Retry mekanizması, sistemi “çökertmek” için değil “kurtarmak” için tasarlanır.
> 

---

## 11) Offset Commit stratejisi: bellekle ilişkisi

Commit stratejisi yanlışsa:

- Mesaj tekrar okunur
- Duplicate processing artar
- Daha çok iş → daha çok allocation → daha çok GC

Genel prensip:

- “İşlenmeden commit etme” (aksi veri kaybı riski)
- “Commit’i çok sık yapma” (overhead)
- Batch commit kullan

(Bu konu doğrulama/garanti modellerine bağlı olduğu için burada “genel prensip” düzeyinde bıraktım.)

---

## 12) “Memory Predictability” için metrik seti

100k msg/s pipeline’da şu metrikler zorunlu:

- In-flight messages
- Queue length
- Worker utilization (busy ratio)
- Processing latency (avg/p95/p99)
- Batch size distribution
- Heap inuse / heap alloc
- Allocation rate (allocs/s)
- GC cycles & pause
- Goroutine count
- Kafka lag (consumer lag)

Memory-first yaklaşımda:

> Queue length + lag artıyorsa bu RAM riski alarmıdır.
> 

---

## 13) Anti-pattern kataloğu (Kafka pipeline özel)

- Unbounded goroutine spawn
- Unbounded queue
- Her mesajda büyük payload kopyalama
- Her mesajda fmt.Sprintf ile string üretmek
- Her mesajda map[string]interface{} kullanmak
- Retry’ı limitsiz yapmak
- “Lag artınca queue büyütelim” refleksi
- Batch olmadan DB’ye vurmak
- Backpressure’sız poll

---

## 14) 100k msg/s için özet tasarım kuralları

1. **Bounded worker pool** kullan
2. **Bounded queue** kullan (kısa spike için)
3. Queue dolunca **poll yavaşlat** (backpressure)
4. **Batch** kullan (IO amortize)
5. Decode/validation’da **allocation’ı minimize et**
6. Retry ve DLQ’yi **bounded** tasarla
7. Commit stratejini tekrar işlemeyi minimize edecek şekilde planla
8. Metriklerle “predictability”yi sürekli doğrula