# API Idempotency & Safe Retry Guide

Bu doküman, production ortamında çalışan API'lerde idempotency ve retry-safe sistem tasarımını tanımlar.

Amaç:

- Aynı isteğin tekrar gönderilmesi durumunda sistemin bozulmamasını sağlamak
- Network hataları ve retry senaryolarını güvenli hale getirmek
- Duplicate işlem riskini ortadan kaldırmak
- Distributed sistemlerde veri tutarlılığını korumak

---

# 1. Temel İlke

Bir API:

- retry edildiğinde aynı sonucu üretmelidir
- duplicate işlem oluşturmamalıdır

Bu özellik:

> idempotency olarak adlandırılır

---

# 2. Idempotency Nedir?

Aynı request birden fazla kez gönderildiğinde:

- sistem aynı sonucu üretir
- state değişmez veya tekrar edilmez

---

## Örnek

### Idempotent

    DELETE /orders/123

1. çağrı → order silinir
2. çağrı → zaten silinmiş, sonuç değişmez

---

### Non-Idempotent

    POST /orders

1. çağrı → order oluşturulur
2. çağrı → ikinci order oluşur ❌

---

# 3. HTTP Method ve Idempotency

| Method | Idempotent |
|--------|------------|
| GET    | Evet |
| PUT    | Evet |
| DELETE | Evet |
| POST   | Hayır |

Ama:

> POST endpoint'ler de idempotent hale getirilebilir

---

# 4. En Kritik Problem: Retry

Client retry edebilir:

- timeout
- network drop
- 5xx error
- gateway retry

Eğer API idempotent değilse:

- duplicate işlem oluşur

---

# 5. Idempotency Key Yaklaşımı

En yaygın çözüm:

Client her request için unique key gönderir.

Header:

    Idempotency-Key: abc123

---

# 6. Server Davranışı

Server:

1. key'i kontrol eder
2. daha önce işlenmiş mi bakar

---

## 6.1 İlk İstek

- işlem yapılır
- sonuç saklanır
- key store edilir

---

## 6.2 Tekrar Gelen İstek

- işlem yapılmaz
- önceki response döndürülür

---

# 7. Idempotency Nerede Kullanılmalı?

Özellikle:

- ödeme (payment)
- sipariş oluşturma
- para transferi
- kupon kullanımı
- external API çağrıları

---

# 8. Storage Stratejisi

Idempotency key store edilmelidir.

Seçenekler:

- Redis (önerilen)
- DB (daha ağır)
- in-memory (distributed sistemde yetersiz)

---

## 8.1 Redis Örneği

Key:

    idempotency:{key}

Value:

- response
- status
- timestamp

TTL:

- genelde 24 saat

---

# 9. Response Replay

Aynı key ile gelen request:

- aynı response'u almalıdır

Bu:

- client için deterministic davranış sağlar
- retry'yi güvenli hale getirir

---

# 10. Race Condition Problemi

Aynı key ile iki request aynı anda gelirse:

- double işlem riski oluşur

Çözüm:

- distributed lock
- atomic insert (SETNX)

---

# 11. Idempotency Key Scope

Key:

- endpoint bazlı olmalı
- global reuse edilmemeli

Örnek:

    POST /orders → key A
    POST /payments → key A ❌

---

# 12. Validation ile İlişki

Aynı key farklı payload ile gelirse:

- hata dönülmelidir

Örnek:

    {
      "code": "IDEMPOTENCY_CONFLICT",
      "message": "Request payload mismatch"
    }

---

# 13. Retry Stratejisi (Client)

Client:

- exponential backoff kullanmalı
- sonsuz retry yapmamalı

Örnek:

- 1s → 2s → 4s → 8s

---

# 14. Safe Retry Kuralları

Retry sadece şu durumlarda yapılmalı:

- network timeout
- connection error
- 5xx error

Retry yapılmamalı:

- 4xx error
- validation error

---

# 15. Payment Senaryosu

Problem:

- ödeme request gönderildi
- response gelmedi
- client tekrar gönderdi

Idempotency yoksa:

- double charge ❌

Idempotency varsa:

- aynı işlem tekrar edilmez ✅

---

# 16. Kafka ve Idempotency

Event-driven sistemlerde:

- consumer duplicate event alabilir

Çözüm:

- event id sakla
- processed table kullan
- idempotent consumer yaz

---

# 17. DB Constraint ile Birlikte Kullanım

Ek koruma:

- unique constraint

Örnek:

    unique(user_id, external_id)

Ama tek başına yeterli değildir.

---

# 18. Timeout ve Retry İlişkisi

Timeout düşük olursa:

- retry artar

Timeout yüksek olursa:

- latency artar

Bu yüzden:

- dengeli timeout
- idempotent API

birlikte kullanılmalıdır

---

# 19. Anti-Patterns

## ❌ Idempotency key kullanmamak

## ❌ Key'i ignore etmek

## ❌ Payload değişimine rağmen aynı key'i kabul etmek

## ❌ Response'u saklamamak

## ❌ Race condition'ı düşünmemek

---

# 20. Production Checklist

- [ ] Kritik endpoint'lerde idempotency var mı?
- [ ] Idempotency key header kullanılıyor mu?
- [ ] Redis veya store var mı?
- [ ] Response replay destekleniyor mu?
- [ ] Race condition çözülmüş mü?
- [ ] Payload mismatch kontrolü var mı?
- [ ] Retry kuralları belirlenmiş mi?
- [ ] Payment gibi kritik işlemler korunuyor mu?

---

# 21. Sonuç

Idempotency:

- distributed sistemlerin temelidir
- veri tutarlılığının garantisidir
- retry güvenliğinin anahtarıdır

Doğru uygulanırsa:

- duplicate işlem olmaz
- sistem güvenilir olur
- retry güvenli hale gelir

Yanlış uygulanırsa:

- double charge
- veri bozulması
- production incident kaçınılmaz olur