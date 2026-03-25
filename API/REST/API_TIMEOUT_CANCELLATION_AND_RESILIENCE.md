# API Timeout, Cancellation & Resilience Guide

Bu doküman, production ortamında çalışan API'lerde timeout, cancellation ve resilience (dayanıklılık) kurallarını tanımlar.

Amaç:

- Uzun süren request'leri kontrol altına almak
- Sistem kaynaklarını korumak
- Downstream servis hatalarına karşı dayanıklı olmak
- Tail latency'yi düşürmek
- Production ortamında sistem stabilitesini sağlamak

---

# 1. Temel İlke

Her request:

- sınırsız süre çalışmamalıdır
- kontrol edilebilir olmalıdır
- iptal edilebilir olmalıdır

> Unbounded request = production risk

---

# 2. Timeout Nedir?

Bir request'in maksimum çalışma süresidir.

Örnek:

- 200ms
- 500ms
- 2s

Timeout olmayan sistem:

- stuck request üretir
- goroutine leak oluşturur
- resource tüketimini kontrol edemez

---

# 3. Timeout Türleri

## 3.1 Request Timeout (Inbound)

Client → API

---

## 3.2 Downstream Timeout (Outbound)

API → başka servis

---

## 3.3 Global Timeout

Request'in toplam süresi

---

# 4. En Kritik Kural

Her external call:

- timeout ile yapılmalıdır

Yanlış:

    http.Get(url)

Doğru:

    context.WithTimeout(...)

---

# 5. Go Context Kullanımı

Go’da timeout ve cancellation:

- context ile yönetilir

---

## 5.1 Örnek

    ctx, cancel := context.WithTimeout(ctx, 500*time.Millisecond)
    defer cancel()

---

## 5.2 Context Propagation

Context:

- handler → service → repository → external call

zincirinde taşınmalıdır.

---

# 6. Cancellation Nedir?

Client request'i iptal ettiğinde:

- backend işlemi de iptal edilmelidir

---

## 6.1 Neden Önemli?

- gereksiz CPU kullanımı engellenir
- DB query'ler durdurulur
- resource leak önlenir

---

# 7. Downstream Timeout Stratejisi

Her dependency için timeout tanımlanmalıdır.

Örnek:

- Redis → 50ms
- DB → 200ms
- External API → 300ms

---

# 8. Timeout Budget

Toplam süre bölünmelidir.

Örnek:

Toplam: 500ms

- DB: 200ms
- Redis: 50ms
- External: 150ms
- buffer: 100ms

---

# 9. Retry ile İlişki

Timeout + retry birlikte düşünülmelidir.

Yanlış:

- kısa timeout + agresif retry

Doğru:

- kontrollü retry + exponential backoff

---

# 10. Circuit Breaker

Downstream sürekli hata veriyorsa:

- request gönderilmemelidir

Bu mekanizma:

- circuit breaker

---

## 10.1 Durumlar

- closed → normal
- open → request engellenir
- half-open → test edilir

---

# 11. Bulkhead Pattern

Farklı resource'lar izole edilmelidir.

Örnek:

- DB pool
- HTTP client pool
- worker pool

---

# 12. Graceful Degradation

Servis tamamen fail olmamalıdır.

Örnek:

- recommendation servisi down → boş dön
- analytics down → ignore et

---

# 13. Fail Fast

Yavaş fail etmek yerine:

- hızlı fail et

---

# 14. Tail Latency Problemi

%99 latency genelde:

- birkaç yavaş request yüzünden artar

Çözüm:

- timeout
- cancellation
- bounded resource

---

# 15. Head-of-Line Blocking

Uzun süren request:

- diğer request'leri bloklayabilir

Çözüm:

- concurrency limit
- timeout

---

# 16. Resource Bounding

Her şey bounded olmalıdır:

- goroutine
- queue
- worker
- connection

---

# 17. HTTP Client Ayarları

Örnek:

- connection timeout
- TLS timeout
- idle timeout

---

# 18. DB Query Timeout

DB query'ler:

- sınırsız çalışmamalı

---

# 19. Observability

Takip edilmesi gerekenler:

- timeout rate
- latency distribution (p50, p95, p99)
- cancellation rate
- downstream latency

---

# 20. Anti-Patterns

## ❌ Timeout kullanmamak

## ❌ Context taşımamak

## ❌ Cancellation ignore etmek

## ❌ Sınırsız retry

## ❌ Circuit breaker olmaması

---

# 21. Production Checklist

- [ ] Tüm request'lerde timeout var mı?
- [ ] Downstream call'lar timeout'lu mu?
- [ ] Context propagation var mı?
- [ ] Cancellation destekleniyor mu?
- [ ] Circuit breaker var mı?
- [ ] Retry kontrollü mü?
- [ ] Resource bounded mı?
- [ ] Latency ölçülüyor mu?

---

# 22. Sonuç

Timeout ve resilience:

- performans konusu değildir
- hayatta kalma konusudur

Doğru uygulanırsa:

- sistem stabil kalır
- latency kontrol edilir
- downstream hatalar izole edilir

Yanlış uygulanırsa:

- sistem çöker
- latency patlar
- resource leak oluşur