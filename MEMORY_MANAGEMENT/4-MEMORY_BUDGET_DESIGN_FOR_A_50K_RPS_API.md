# 50.000 RPS API İÇİN MEMORY BUDGET TASARIMI

---

# 1) Önce Şu Soruyu Soruyoruz

Bir API saniyede:

> 50.000 request alıyorsa
> 

aynı anda kaç request işleniyor?

Bu çok kritik.

---

## 1.1 Concurrency Hesabı

Concurrency ≈ RPS × Ortalama Response Süresi

Diyelim:

- 50.000 RPS
- Ortalama latency: 40 ms (0.04 saniye)

Concurrency ≈ 50.000 × 0.04

= 2.000 aktif request

Yani sistemde aynı anda yaklaşık 2.000 request memory’de yaşıyor.

---

# 2) Request Başına Bellek Hesabı

Bir request memory’de şunları tutar:

- HTTP request objesi
- Body (örneğin JSON)
- Decode edilmiş struct
- Validation verisi
- Response struct
- Logging verisi
- DB driver buffer’ları
- Middleware context

Diyelim ortalama:

- Body: 4 KB
- Decode edilmiş struct: 2 KB
- Response: 2 KB
- Ek overhead: 2 KB

Toplam ≈ 10 KB / request

---

# 3) Toplam Concurrent Memory

2.000 concurrent request × 10 KB

= 20.000 KB

= 20 MB

Bu sadece aktif request memory’si.

---

# 4) Ekstra Memory Alanları

Buna şunları eklemeliyiz:

- Worker stack memory
- DB connection pool buffer
- Logger buffer
- Internal caches
- GC metadata
- Goroutine stack

Diyelim:

- 500 goroutine × 4 KB ≈ 2 MB
- DB pool buffer ≈ 10 MB
- Internal struct heap ≈ 10 MB
- Runtime overhead ≈ 20 MB

Toplam ek ≈ 42 MB

---

# 5) Toplam Tahmini Heap

Request memory: 20 MB

Ek overhead: 42 MB

≈ 62 MB

Ama GC free etmiyor mu?

Evet ama:

- Heap peak usage önemli
- Burst durumunda bu 2–3 katına çıkabilir

---

# 6) Burst Senaryosu

Trafik 2 katına çıktı:

100k RPS

Latency 40ms sabit

Concurrency ≈ 4.000

4.000 × 10 KB = 40 MB request memory

Yeni toplam ≈ 80–100 MB

İşte memory-first tasarım burada başlar.

---

# 7) Memory Budget Kararı

Memory-first mimari şunu yapar:

> Servis için maksimum heap hedefi belirler.
> 

Örnek:

- Hedef heap: 256 MB
- Kritik eşik: 300 MB
- Alarm: 200 MB

Yani sistem:

- 62 MB normal
- 100 MB burst
- 200 MB alarm

Bu bilinçli planlanır.

---

# 8) Allocation Budget Hesabı

Şimdi allocation rate’e bakalım.

Diyelim request başına:

- 8 heap allocation

50k RPS × 8

= 400.000 allocation/s

Bu GC cycle’ını hızlandırır.

Hedef:

- Request başına 3–4 allocation

Böylece:

50k × 4 = 200k allocation/s

%50 düşüş.

---

# 9) Concurrency Sınırı

Memory-first tasarımda:

> Max concurrency limitlidir.
> 

Eğer latency artarsa concurrency patlar.

Bu yüzden:

- Server max concurrent request limitli
- Timeout var
- Circuit breaker var

---

# 10) Worker Pool Tasarımı (API İçin)

Her request kendi goroutine’inde çalışır.

Ama iç iş mantığında:

- IO heavy işlemler için semaphore
- DB connection pool limitli
- Async job varsa bounded queue

---

# 11) Body Limit Stratejisi

Memory-first API:

- 1MB body kabul etmiyorsa
- Limit koyar (örneğin 64KB)

Bu:

- RAM’i korur
- Allocation spike’ı önler

---

# 12) Logging Budget

Hot path’te logging:

- String format
- Allocation
- IO

50k RPS’te log spam RAM ve CPU’yu patlatır.

Memory-first yaklaşım:

- Error log minimal
- Info log sampling
- Debug log kapalı

---

# 13) GC Etkisi

Allocation düşerse:

- GC frequency düşer
- Tail latency düşer
- CPU düşer

Memory-first sistem:

> GC’yi düşman değil, maliyet olarak görür.
> 

---

# 14) Öngörülebilirlik Kontrolü

Bu API için artık şunları biliyoruz:

- 50k RPS’te 62 MB
- 2x burst’te ~100 MB
- Heap limitimiz 256 MB
- Allocation rate 200k–400k/s

Bu tahmin edilebilir.

Bu:

> Predictable memory davranışı demektir.
> 

---

# 15) 50k RPS API Memory-First Checklist

- [ ]  Concurrency hesabı yapıldı mı?
- [ ]  Request başına KB ölçüldü mü?
- [ ]  Burst senaryosu hesaplandı mı?
- [ ]  Allocation budget çıkarıldı mı?
- [ ]  Body limit var mı?
- [ ]  Max concurrency limit var mı?
- [ ]  Logging minimize edildi mi?
- [ ]  Heap target belirlendi mi?

---

# 16) En Kritik Ders

Yüksek trafikli API tasarımında:

> “Kod yazdıktan sonra bakalım kaç MB”
> 
> 
> yaklaşımı yanlıştır.
> 

Doğru yaklaşım:

> “Bu servis en fazla kaç MB kullanmalı?”
> 
> 
> → Sonra mimariyi buna göre kur.
>