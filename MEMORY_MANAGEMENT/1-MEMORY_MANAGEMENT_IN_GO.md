# GO’DA BELLEK YÖNETİMİ

---

# 1) Bellek (Memory) Nedir?

Bir program çalışırken verileri bir yerde tutmak zorundadır.

Örnek:

- Kullanıcının adı
- Gelen HTTP isteğinin body’si
- Kafka mesajı
- Bir liste
- Bir sayı
- Bir struct

Bu verilerin hepsi **RAM’de** tutulur.

RAM = bilgisayarın geçici çalışma belleği.

Program çalışırken:

- Veri oluşturur
- Kullanır
- Siler

İşte bu sürecin tamamına:

> Bellek Yönetimi (Memory Management)
> 

denir.

---

# 2) Go’da Bellek Nasıl Yönetilir?

Go’da bellek yönetimi otomatik yapılır.

Yani:

- C/C++ gibi `malloc` / `free` yapmazsın.
- Go kendi “çöp toplayıcısı” ile kullanılmayan belleği temizler.

Bu çöp toplayıcıya:

> Garbage Collector (GC)
> 

denir.

Ama otomatik olması, bizim hiçbir şey bilmememiz gerektiği anlamına gelmez.

Yanlış kod yazarsak:

- Bellek şişer
- CPU artar
- Latency yükselir
- Sistem yavaşlar
- Hatta crash olur

---

# 3) Go’da Bellek Nerede Durur?

Go’da veri iki ana yerde tutulur:

1. Stack (Yığın Bellek)
2. Heap (Öbek Bellek)

Bu ikisini anlamadan performans anlaşılmaz.

---

# 4) Stack Nedir?

Stack, fonksiyon çağrıları için kullanılan hızlı bellek alanıdır.

Basit düşün:

- Bir fonksiyon çalıştı.
- İçinde bazı değişkenler oluşturdu.
- Fonksiyon bitti.
- O değişkenler otomatik silindi.

İşte bu stack’tir.

Örnek:

```
funcadd(a,bint)int {
c:=a+b
returnc
}
```

Buradaki:

- `a`
- `b`
- `c`

genelde stack’te tutulur.

---

## Stack Özellikleri

- Çok hızlıdır
- Allocation maliyeti neredeyse sıfırdır
- Fonksiyon bitince otomatik temizlenir
- GC tarafından izlenmez

Bu yüzden:

> Stack iyidir. Stack’te kalabilen veri performanslıdır.
> 

---

# 5) Heap Nedir?

Heap, dinamik ve uzun ömürlü verilerin tutulduğu bellek alanıdır.

Ne zaman heap kullanılır?

- Bir veri fonksiyon dışına çıkıyorsa
- Bir pointer ile taşınıyorsa
- Büyük veri yapısı varsa
- Map, slice, channel kullanıyorsan

Heap’in özelliği:

- Allocation daha pahalıdır
- GC tarafından izlenir
- Yanlış kullanımda performansı düşürür

---

# 6) Stack mi Heap mi? Kararı Kim Veriyor?

Buna Go compiler karar verir.

Bu analiz sürecine:

> Escape Analysis
> 

denir.

---

## Escape Ne Demek?

“Escape”, bir değişkenin fonksiyon dışına referansla çıkması demektir.

Örnek:

```
funcfoo()*int {
x:=10
return&x
}
```

Burada:

- `x` normalde stack’te olmalıydı.
- Ama fonksiyon bittikten sonra da kullanılacak.
- O yüzden heap’e taşınır.

---

## Escape’i Nasıl Görürüz?

Derlerken:

```
go build-gcflags="-m"
```

çıktıda:

```
x escapes to heap
```

yazar.

---

# 7) Garbage Collector (GC) Nedir?

Garbage Collector:

> Heap’te artık kullanılmayan verileri bulan ve silen sistemdir.
> 

---

## GC Neden Var?

C gibi dillerde:

- Belleği sen ayırırsın
- Sen free edersin

Unutursan → memory leak

Go’da:

- Runtime takip eder
- Kullanılmayanı temizler

Ama unutma:

GC var diye yanlış yazarsan sistem yavaşlar.

---

# 8) GC Nasıl Çalışır? (Basit Anlatım)

GC şu soruyu sorar:

> Bu veri hâlâ bir yerden erişilebilir mi?
> 

Eğer erişilemiyorsa:

- Silinir
- Bellek geri kazanılır

---

## Tri-Color Mantık (Basitleştirilmiş)

Objeler 3 kategoriye ayrılır:

- Beyaz: Henüz incelenmedi
- Gri: İnceleniyor
- Siyah: İncelendi, kullanılabilir

En sonunda beyaz kalanlar silinir.

---

# 9) Allocation Nedir?

Allocation = Bellekten yer ayırmak.

Örnek:

```
x:=new(User)
```

Bu heap allocation’dır.

Stack allocation ucuzdur.

Heap allocation pahalıdır.

Neden pahalı?

- Runtime bookkeeping yapar
- GC izler
- CPU tüketir

---

# 10) Allocation Rate Neden Önemli?

Diyelim saniyede 10.000 request var.

Her request’te 5 heap allocation yapıyorsan:

= 50.000 allocation/s

Bu:

- GC frequency artırır
- CPU yükseltir
- Tail latency patlatır

Bu yüzden:

> Hot path’te allocation sayısı çok kritiktir.
> 

---

# 11) Hot Path Nedir?

Hot path:

> Programın en sık çalışan kısmı.
> 

Örnek:

- HTTP handler
- Kafka consumer loop
- Event processing pipeline

Buradaki küçük hatalar bile büyük maliyet üretir.

---

# 12) Zero Allocation Nedir?

Zero allocation:

> Hot path’te mümkün olduğunca heap allocation yapmamak.
> 

Tam sıfır demek değil.

Ama gereksiz allocation’ları kesmek demek.

---

## Zero Allocation Teknikleri

1. Slice kapasitesini önceden vermek
2. strings.Builder kullanmak
3. fmt yerine strconv
4. sync.Pool ile reuse
5. Unbounded goroutine açmamak
6. Batch processing

---

# 13) Slice Nedir ve Neden Tehlikelidir?

Slice aslında:

- pointer
- length
- capacity

tutan küçük bir yapıdır.

Ama pointer büyük bir array’i gösterebilir.

---

## Büyük Slice Retention

```
big:= make([]byte,10_000_000)
small:=big[:10]
```

small küçük ama 10MB belleği tutar.

Çünkü aynı backing array’i gösterir.

Bu çok yaygın memory problemidir.

---

# 14) Goroutine ve Bellek

Her goroutine:

- Başlangıçta ~2KB stack

1 milyon goroutine açarsan:

2GB’a yaklaşabilirsin.

Bu yüzden:

> Unbounded goroutine spawn yasaktır.
> 

Worker pool kullanılır.

---

# 15) Worker Pool Nedir?

Worker pool:

- Sabit sayıda worker
- Bir job queue
- Worker’lar sırayla işleri tüketir

Faydası:

- Goroutine sayısı sabit
- Memory predictable
- Backpressure kontrolü mümkün

---

# 16) Backpressure Nedir?

Backpressure:

> Sistem bir noktada yetişemediğinde biriken yük.
> 

Kafka’da örnek:

- 20.000 msg/s geliyor
- Sen 10.000 msg/s işliyorsun

Queue büyür → RAM artar → GC artar → OOM olabilir

Çözüm:

- Bounded queue
- Worker limit
- Batch processing

---

# 17) Batch Processing Nedir?

Tek tek işlemek yerine:

- 100 mesajı birlikte işlemek

Faydaları:

- DB call sayısı azalır
- Allocation amortize olur
- Throughput artar

---

# 18) High Load API’de Bellek Sorunları

API akışı:

1. Body okunur
2. JSON decode
3. Validation
4. İş mantığı
5. JSON encode

En pahalı kısım genelde:

- JSON decode/encode
- fmt
- log

---

## API İçin Kurallar

- fmt hot path’te minimum
- strings.Builder kullan
- DTO küçük tut
- Slice kapasitesi ver
- Unbounded map yok
- pprof ile ölç

---

# 19) pprof Nedir?

pprof:

> Programın CPU ve memory davranışını ölçen araçtır.
> 

Heap analizi:

```
/debug/pprof/heap
```

Allocation analizi:

```
/debug/pprof/allocs
```

Goroutine analizi:

```
/debug/pprof/goroutine
```

---

# 20) GC Tuning

GOGC default 100’dür.

Yükseltirsen:

- Daha az GC
- Daha fazla RAM

Düşürürsen:

- Daha sık GC
- Daha az RAM

Ama tuning ölçmeden yapılmaz.

---

# 21) En Yaygın Memory Hataları

1. Global sınırsız map
2. Goroutine leak
3. Channel kapanmaması
4. Slice retention
5. time.Tick leak
6. Logging spam
7. Interface boxing
8. Gereksiz pointer

---

# 22) Kısa Kural Özeti

Stack iyidir.

Heap pahalıdır.

Allocation az olsun.

Hot path’te fmt az olsun.

Worker sayısı sabit olsun.

Queue sınırlı olsun.

Slice retention’a dikkat.

Profiling yapmadan optimize etme.

---