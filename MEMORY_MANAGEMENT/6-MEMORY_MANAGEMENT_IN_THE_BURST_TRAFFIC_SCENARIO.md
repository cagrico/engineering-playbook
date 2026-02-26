# Burst Traffic Senaryosu

# 100k → 500k msg/s (30 saniye) geldiğinde sistem nasıl “çökmeyip” hayatta kalır?

Bu doküman, Kafka/event pipeline’ında **ani trafik patlaması (burst)** geldiğinde bellek kullanımını **sınırsız büyütmeden** sistemi nasıl tasarlayacağımızı anlatır. Amaç “her şeyi işleyelim” değil; amaç:

> **RAM’i koru, sistemi ayakta tut, toparlanınca yetiş.**
> 

---

## 1) Burst Traffic Nedir?

**Burst**, kısa sürede anormal yüksek hacimde trafik gelmesidir.

Örnek:

- Normal: **100k msg/s**
- Burst: **500k msg/s**
- Süre: **30 saniye**

Bu 30 saniyede toplam gelen mesaj:

- 500k × 30 = **15.000.000 mesaj**

Normal kapasiten 100k msg/s ise, 30 saniyede işleyebileceğin:

- 100k × 30 = **3.000.000 mesaj**

Aradaki fark (biriken backlog):

- 15M – 3M = **12.000.000 mesaj**

İşte bu 12M mesaj “birikecek” ve asıl soru şu:

> Bu birikme RAM’e mi binecek, Kafka’da mı kalacak?
> 

Memory-first yaklaşımın cevabı:

- **RAM’e binmeyecek.**
- Backlog mümkün olduğunca **Kafka tarafında kalacak** (consumer lag olarak).

---

## 2) Burst’te En Büyük Tehlike: Unbounded Buffering

Burst anında yanlış mimari şunu yapar:

- Mesajları sürekli poll eder
- RAM’de bir queue’da biriktirir
- “Sonra işleriz” der

Bu, birkaç saniyede RAM’i öldürür.

### Neden?

Queue büyüdükçe:

- Payload’lar RAM’de tutulur
- GC basıncı patlar
- Tail latency ve CPU spike olur
- En sonunda OOM

Bu yüzden Burst tasarımının **altın kuralı**:

> **RAM, backlog saklamak için kullanılmaz.**
> 

---

## 3) Burst’te Doğru Hedef: “Controlled In-Flight”

Burst’te kontrol etmek istediğimiz şey:

- **In-flight messages**: Aynı anda işlenen mesaj sayısı
- **Queue length**: RAM’de bekleyen mesaj sayısı

Memory-first hedef:

- In-flight sabit bir üst sınırda kalsın
- Queue sabit bir üst sınırda kalsın
- Fazlası Kafka’da lag olarak biriksin

---

## 4) Bounded Queue Tasarımı (Sayısal)

Diyelim pipeline tasarımın:

- Worker sayısı: **128**
- Her worker ortalama 5ms’de 1 mesaj işlesin
    - 1 worker throughput: 200 msg/s
    - 128 worker throughput: 25.600 msg/s

Bu 100k msg/s hedefinin altında; demek ki:

- ya işlem süresi daha düşük olmalı
- ya worker sayısı artmalı
- ya batch kullanılmalı

Burst’te zaten yetişmeyeceğini kabul ediyoruz; ama önemli olan:

> Queue büyüyerek RAM’i patlatmasın.
> 

Örneğin queue capacity:

- **10.000 job**

Bu queue, 500k msg/s’te kaç saniyede dolar?

- 10.000 / 500.000 = **0.02 saniye** (20ms)

Yani burst başladığında queue çok hızlı dolar. Bu kötü değil; iyi.

Çünkü bu bize şunu söyler:

> “Artık RAM’de tutmayacağım, poll’u yavaşlatacağım.”
> 

---

## 5) Poll Control: Burst’te Hayatta Kalmanın Ana Mekanizması

Kafka consumer’ın poll/fetch döngüsü burst’te şöyle davranmalı:

### Kural:

**Queue doluysa → poll durur / yavaşlar**

Bu pratikte şuna benzer:

- Dispatch edemiyorsan daha fazla mesaj çekme
- Mesajlar Kafka’da kalsın (lag artsın)

Bunun sonucu:

- RAM stabil
- GC stabil
- Sistem ölmez
- Lag artar ama bu yönetilebilir

---

## 6) Backpressure Stratejileri (Burst için)

Burst sırasında uygulanabilecek 3 ana strateji:

### 6.1 Hard Backpressure (En güvenlisi)

- Queue dolunca poll tamamen durur
- Worker boşalınca devam eder

**Artısı:** RAM kesin korunur

**Eksisi:** Lag hızlı büyür

### 6.2 Soft Backpressure (Daha akıcı)

- Queue doldukça poll aralığı artırılır (sleep)
- Kademeli yavaşlatma yapılır

**Artısı:** Daha stabil tüketim

**Eksisi:** Uygulaması daha kompleks

### 6.3 Load Shedding (Çok özel durum)

Kafka’da doğrudan “drop” genelde istemeyiz ama bazı event türleri için (örn telemetry) drop kabul edilebilir.

---

## 7) Burst’te Batch Zorunluluğu

Eğer burst sonrası backlog’u eritmek istiyorsan batch şart olur.

Neden?

- Tek tek işlemek IO’yu boğar
- Batch ile DB’ye daha az call
- Throughput yükselir

### Burst sonrası hedef:

- Normalde 100k msg/s işliyorsun
- Burst bitince bir süre 150k–200k msg/s işleyip backlog’u eritmek isteyebilirsin

Bu “burst recovery mode” demektir.

---

## 8) Burst Recovery Mode (Toparlanma Modu)

Memory-first sistem sadece burst’te hayatta kalmaz; burst sonrası toparlanır.

Recovery modunda:

- Batch size biraz artırılır (limitli)
- Worker utilization %90+ tutulur
- Retry daha agresif DLQ’ya kaydırılabilir (sistemi tıkamasın)
- Log seviyesi düşürülebilir (I/O azalt)

Ama yine de:

- Queue bounded kalır
- In-flight bounded kalır

---

## 9) Burst’te Retry/DLQ Davranışı

Burst sırasında en tehlikeli şey:

- Hatalı mesajların retry ile sistemi tıkaması

Bu yüzden burst anında:

- Retry limitleri daha katı olmalı
- Hatalı mesajlar hızlıca DLQ’ya gidebilmeli
- Retry queue da bounded olmalı

Prensip:

> “Hatalı mesajlar sistemi yavaşlatmasın, sistem sağlıklı mesajları işlemeye devam etsin.”
> 

---

## 10) Burst’te Ölçülecek Metrikler (Alarm Mantığı)

Burst senaryosunda alarm kuralları şöyle olmalı:

### Kritik metrikler:

- Queue length: %80 üstü → backpressure devreye girmeli
- Worker utilization: %95+ uzun süre → kapasite yetersiz
- Heap inuse: sürekli artış → RAM’de buffering var (yanlış)
- Allocation rate: spike → decode/log patlaması
- GC cycles: artış → allocation basıncı
- Consumer lag: artar, bu normal; ama trend takip edilir

En önemli “doğru davranış” şudur:

> Burst geldi → lag arttı, ama heap stabil kaldı.
> 

Bu memory-first başarısıdır.

---

## 11) Burst’te En Yaygın Yanlış Refleksler

- “Queue’yu büyütelim”
- “Worker sayısını sonsuz artıralım”
- “Her mesajı hemen poll edelim”
- “Retry sonsuz olsun”
- “Logları açalım debuglayalım” (burst’te felaket)

Doğru refleks:

- “RAM’i koru”
- “Backpressure uygula”
- “Lag kabul et”
- “Recovery moduyla erit”

---

## 12) Burst Senaryosu İçin Kısa Kurallar

1. Backlog RAM’de tutulmaz
2. Queue bounded olacak
3. Queue dolunca poll yavaşlayacak/duracak
4. In-flight bounded olacak
5. Batch ile throughput artırılacak
6. Retry bounded + DLQ hazır
7. Burst bittiğinde recovery mode ile backlog eritilecek
8. “Heap stabil, lag artıyor” = doğru davranış