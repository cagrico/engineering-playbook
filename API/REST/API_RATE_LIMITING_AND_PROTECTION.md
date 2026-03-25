# API Rate Limiting & Protection Guide

Bu doküman, production ortamında çalışan API'lerin rate limiting, abuse prevention ve genel koruma stratejilerini tanımlar.

Amaç:

- API'yi kötüye kullanıma karşı korumak
- Sistem kaynaklarını kontrollü kullanmak
- Burst traffic ve saldırılara karşı stabil kalmak
- Adil kullanım (fair usage) sağlamak
- Production sistemlerde predictability oluşturmak

---

# 1. Temel İlke

Bir API:

- sadece doğru çalışmak zorunda değildir
- aynı zamanda korunmak zorundadır

Korunmayan API:

- memory patlatır
- CPU spike üretir
- downstream servisleri çökertir
- cache’i anlamsızlaştırır
- Kafka, DB, Redis gibi sistemleri overload eder

---

# 2. Rate Limiting Nedir?

Belirli bir zaman aralığında:

- kaç request yapılabileceğini sınırlamaktır

Örnek:

- 1 saniyede 100 request
- 1 dakikada 1000 request

---

# 3. Neden Rate Limiting Zorunlu?

Rate limit olmayan sistem:

- brute force saldırıya açık
- DDoS’a açık
- buglı client tarafından çökertilebilir
- internal servisleri domino etkisiyle düşürür

---

# 4. Rate Limiting Türleri

## 4.1 Global Rate Limit

Tüm sistem için limit:

    10.000 req/s

---

## 4.2 Per IP Rate Limit

Her IP için limit:

    100 req/s per IP

---

## 4.3 Per User Rate Limit

Authenticated kullanıcı bazlı limit:

    200 req/s per user

---

## 4.4 Per API Key Rate Limit

API consumer bazlı limit:

    500 req/s per API key

---

## 4.5 Per Endpoint Rate Limit

Endpoint bazlı limit:

    /login → 10 req/min
    /products → 200 req/s

---

# 5. En Kritik Ayrım: Hard vs Soft Limit

## 5.1 Hard Limit

Limit aşılırsa request reddedilir:

    HTTP/1.1 429 Too Many Requests

---

## 5.2 Soft Limit

Limit aşılırsa:

- degrade edilir
- yavaşlatılır
- queue’ya alınır

Production sistemlerde genelde:

- kritik endpoint → hard limit
- non-critical → soft limit

---

# 6. Rate Limiting Algoritmaları

## 6.1 Fixed Window

- basit
- hızlı
- ama burst toleransı kötü

---

## 6.2 Sliding Window

- daha doğru
- burst yönetimi daha iyi
- production için uygun

---

## 6.3 Token Bucket (ÖNERİLEN)

- burst toleransı vardır
- steady rate sağlar
- production’da en çok tercih edilen

---

## 6.4 Leaky Bucket

- sabit çıkış hızı sağlar
- smoothing yapar

---

# 7. Önerilen Production Setup

Senin sistem tipine göre:

- API Gateway (Traefik / Kong / Nginx)
- + Application level rate limit

---

## 7.1 Gateway Level

- global protection
- IP bazlı limit
- DDoS protection

---

## 7.2 Application Level

- user bazlı limit
- business rule bazlı limit
- endpoint bazlı limit

---

# 8. Rate Limit Response Standardı

Limit aşıldığında:

    HTTP/1.1 429 Too Many Requests

    {
      "code": "RATE_LIMITED",
      "message": "Too many requests",
      "request_id": "req_abc123"
    }

---

## 8.1 Header Kullanımı

Önerilen:

    X-RateLimit-Limit: 100
    X-RateLimit-Remaining: 0
    X-RateLimit-Reset: 1710000000

Bu header’lar:

- client davranışını iyileştirir
- retry stratejisini kolaylaştırır

---

# 9. Burst Traffic Yönetimi

Burst traffic kaçınılmazdır.

Örnek:

- kampanya
- flash sale
- push notification

---

## 9.1 Burst Yönetim Stratejileri

- Token bucket kullan
- Worker pool ile sınırla
- Queue ile bufferla
- Backpressure uygula

---

# 10. Rate Limit Key Seçimi

Rate limit neye göre yapılacak?

Seçenekler:

- IP
- User ID
- API Key
- Session ID

---

## 10.1 Kritik Kural

Auth varsa:

    user bazlı limit

Auth yoksa:

    IP bazlı limit

---

# 11. Distributed Rate Limiting

Single instance rate limit yeterli değildir.

Horizontal scale varsa:

- Redis / distributed counter kullan

---

## 11.1 Redis ile Rate Limiting

- atomic increment
- TTL ile window yönetimi
- high performance

---

# 12. Rate Limiting ve Cache İlişkisi

Cache varsa:

- cache hit → rate limit sayılmayabilir

Ama bu karar:

- business logic’e bağlıdır

---

# 13. Whitelist & Blacklist

## 13.1 Whitelist

- internal servisler
- trusted client'lar

---

## 13.2 Blacklist

- abuse yapan IP
- saldırı kaynakları

---

# 14. Login & Auth Endpoint Koruması

En kritik endpoint’ler:

- login
- register
- password reset

Öneri:

    /login → 5 req/min per IP
    /register → 3 req/min

---

# 15. Rate Limit ve Retry İlişkisi

Client:

- 429 alınca retry etmemeli
- exponential backoff kullanmalı

---

# 16. Circuit Breaker ile Birlikte Kullanım

Rate limit:

- inbound koruma

Circuit breaker:

- outbound koruma

Birlikte kullanılmalıdır.

---

# 17. Monitoring ve Alerting

Takip edilmesi gereken metrikler:

- rate limit hit sayısı
- 429 oranı
- IP bazlı dağılım
- user bazlı dağılım

---

# 18. Abuse Detection

Rate limit tek başına yeterli değildir.

Ek olarak:

- anomaly detection
- unusual traffic pattern
- geo-based filtering

---

# 19. Production Checklist

- [ ] Gateway level rate limit var mı?
- [ ] Application level rate limit var mı?
- [ ] Token bucket veya sliding window kullanılıyor mu?
- [ ] Distributed rate limit var mı?
- [ ] Login endpoint korunuyor mu?
- [ ] 429 response standardı doğru mu?
- [ ] Rate limit header'ları var mı?
- [ ] Burst traffic senaryosu düşünülmüş mü?
- [ ] Monitoring var mı?

---

# 20. Sonuç

Rate limiting:

- sadece koruma mekanizması değildir
- sistem stabilitesinin temelidir

Doğru uygulanırsa:

- sistem ayakta kalır
- kaynaklar korunur
- abuse engellenir

Yanlış uygulanırsa:

- sistem çöker
- kullanıcı deneyimi bozulur
- production incident kaçınılmaz olur