# PRODUCTION MEMORY AUDIT REHBERİ

# (Adım adım kök neden analizi)

Bu bölüm tamamen pratik.
Bir production servis yavaşladı, RAM artıyor, GC spike var… ne yapacağız?

---

# 1) Senaryo: Problem Nasıl Başlar?

Tipik alarm:

- 🚨 Heap sürekli artıyor
- 🚨 GC cycle sayısı artmış
- 🚨 P99 latency yükselmiş
- 🚨 CPU artmış
- 🚨 Pod/container restart olmuş (OOM)

Panik yapmadan ilerliyoruz.

---

# 2) İlk Kural: Varsayım Yapma

En büyük hata:

> “Kesin GC yüzünden”
> 
> 
> “Kesin Kafka yüzünden”
> 
> “Kesin memory leak var”
> 

Hayır.

Ölçmeden yorum yapılmaz.

---

# 3) İlk Adım: Metriklere Bak

Önce monitoring paneli:

Bakılacak minimum metrikler:

- heap_inuse
- heap_alloc
- gc_cycles
- gc_pause
- goroutine_count
- queue_length
- worker_utilization
- consumer_lag (Kafka ise)
- RPS veya msg/s

---

## 3.1 Heap Artış Türleri

### A) Heap dalgalı ama trend sabit

Normal davranış.

### B) Heap sürekli artıyor (trend yukarı)

Retention / leak ihtimali.

### C) Heap spike → düşüyor → spike

Allocation rate çok yüksek olabilir.

---

# 4) İkinci Adım: pprof Aç

Production’da genelde internal porttan açık olur:

```
/debug/pprof/heap
/debug/pprof/allocs
/debug/pprof/goroutine
/debug/pprof/profile
```

---

# 5) Heap Profili Analizi

Komut:

```
go tool pprof http://host/debug/pprof/heap
```

Bakılacak komutlar:

```
top
top -cum
list <func>
web
```

---

## 5.1 “top” Ne Gösterir?

En çok memory tutan fonksiyonlar.

Sorulacak sorular:

- Bu fonksiyon neden bu kadar heap tutuyor?
- Bu normal mi?
- Bu hot path mi?
- Burada retention olabilir mi?

---

## 5.2 Retention Nasıl Anlaşılır?

Şu durum varsa şüphelen:

- Büyük slice tutuluyor
- Global map büyüyor
- Cache temizlenmiyor
- Retry queue boşalmıyor

---

# 6) Allocation Profili (allocs)

Komut:

```
go tool pprof http://host/debug/pprof/allocs
```

Bu şunu gösterir:

> Kim sürekli allocation yapıyor?
> 

Örnek bulgu:

- J"SON decode çok allocation yapıyor
- fmt.Sprintf spam var
- map[string]interface{} hot path’te

Bu durumda:

> Leak yoktur, allocation rate yüksektir.
> 

---

# 7) Goroutine Profili

Komut:

```
go tool pprof http://host/debug/pprof/goroutine
```

Bakılacak:

- Goroutine sayısı stabil mi?
- Artıyor mu?
- Block olmuş goroutine var mı?

Eğer sayı sürekli artıyorsa:

> Goroutine leak olabilir.
> 

Tipik leak sebepleri:

- Channel kapanmıyor
- Context cancel edilmemiş
- time.After yanlış kullanımı
- Retry sonsuz

---

# 8) GC Analizi

GC ile ilgili bakılacaklar:

- GC cycle frequency
- GC pause süresi
- Allocation rate

Senaryo 1:

- Allocation rate çok yüksek
- GC çok sık çalışıyor
- Heap stabil

→ Zero allocation çalışması gerekir.

Senaryo 2:

- Heap sürekli büyüyor
- GC çalışıyor ama düşmüyor

→ Retention var.

---

# 9) Kök Neden Analizi Örneği

Senaryo:

- 100k msg/s Kafka consumer
- Heap 200MB → 400MB → 800MB
- OOM oldu

Adım adım analiz:

1️⃣ Queue length bak → büyüyor

2️⃣ Worker utilization %100

3️⃣ Consumer lag artıyor

4️⃣ Heap profile → büyük slice queue içinde tutuluyor

Sonuç:

> Backpressure yok, RAM backlog tutuyor.
> 

Çözüm:

- Queue bounded
- Poll control
- Retry bounded

---

# 10) Başka Bir Senaryo

Problem:

- CPU %90
- P99 latency 3 kat artmış
- Heap stabil

Allocation profile:

- JSON decode %60 allocation
- fmt %20

Sorun:

> Allocation rate yüksek → GC frequency artmış → CPU spike
> 

Çözüm:

- Decode optimize
- fmt azalt
- Batch artır

---

# 11) Memory Leak vs Allocation Storm

Bunu ayırmayı öğrenmek çok önemli.

| Durum | Leak | Allocation Storm |
| --- | --- | --- |
| Heap trend | Sürekli artar | Dalgalı |
| GC | Çalışır ama düşmez | Çok sık çalışır |
| allocs profile | Normal olabilir | Çok yüksektir |
| Çözüm | Referans temizle | Allocation azalt |

---

# 12) Production Memory Audit Checklist

Adım sırası:

1. Heap trend incele
2. Allocation rate bak
3. GC frequency bak
4. Goroutine count bak
5. Queue length bak
6. Worker utilization bak
7. pprof heap al
8. pprof allocs al
9. Goroutine dump al
10. Kök nedeni kategorize et

---

# 13) En Büyük Ders

Production memory audit’te amaç:

> Semptomu değil, sistemi anlamak.
> 

Çoğu ekip GC’yi suçlar.

Ama genelde:

- Unbounded queue
- Allocation hot path
- Retry birikmesi
- Logging patlaması
- Büyük slice retention

gibi mimari hatalar vardır.

---

# 14) Memory-First Düşüncenin Gücü

Memory-first tasarlanmış sistemde:

- Burst geldi → heap stabil
- Queue doldu → poll yavaşladı
- Lag arttı ama RAM artmadı
- Recovery mode backlog eritti

Audit sırasında şunu görürsün:

> Heap predictable, allocation kontrollü, sistem sağlıklı.
> 

---

# 15) Bu Doküman Serisinin Seviyesi

Artık şunları kapsadık:

- Go bellek temeli
- High-load API memory tasarımı
- Kafka 100k msg/s pipeline
- Burst senaryosu
- Production memory audit

Bu artık senior-level memory engineering perspektifidir.