# API Webhook Design Guide

Bu doküman, production ortamında çalışan sistemlerde webhook tasarım kurallarını tanımlar.

Amaç:

- Sistemler arası event iletişimini güvenli ve güvenilir hale getirmek
- Event kaybını önlemek
- Retry ve idempotency ile tutarlılığı sağlamak
- External entegrasyonları standartlaştırmak

---

# 1. Temel İlke

Webhook:

- sistemin dış dünyaya event göndermesidir

> Webhook = push-based event delivery

---

# 2. Webhook Nedir?

Bir sistem:

- bir event oluştuğunda
- başka bir sisteme HTTP request gönderir

---

## Örnek

- payment başarılı → webhook gönder
- order oluşturuldu → webhook gönder

---

# 3. Pull vs Push

## Pull

Client:

- API'yi sürekli sorgular

## Push (Webhook)

Server:

- event olduğunda client'a gönderir

---

# 4. Webhook Endpoint Tasarımı

Client tarafında endpoint:

    POST /webhooks/payments

---

# 5. Event Payload Tasarımı

Webhook payload:

- self-contained olmalıdır

---

## 5.1 Örnek

    {
      "event_id": "evt_123",
      "type": "payment_succeeded",
      "data": {
        "payment_id": "pay_123",
        "amount": 100
      },
      "created_at": "2026-03-25T10:00:00Z"
    }

---

# 6. En Kritik Kural: Idempotency

Webhook consumer:

- duplicate event alabilir

Bu yüzden:

> webhook handler idempotent olmalıdır

---

# 7. Event ID Kullanımı

Her event:

- unique id taşımalıdır

---

## 7.1 Amaç

- duplicate detection
- replay kontrolü

---

# 8. Retry Mekanizması

Webhook gönderimi:

- başarısız olabilir

Bu yüzden:

- retry yapılmalıdır

---

## 8.1 Retry Stratejisi

- exponential backoff
- max retry limit

---

# 9. Delivery Garantisi

Genelde:

- at-least-once delivery

Bu yüzden:

- duplicate event mümkündür

---

# 10. Timeout

Webhook call:

- timeout ile yapılmalıdır

---

# 11. HTTP Status Davranışı

Consumer:

- 2xx → başarı
- 4xx/5xx → retry

---

# 12. Security

Webhook:

- doğrulanmalıdır

---

## 12.1 Signature

Örnek header:

    X-Signature: abc123

---

## 12.2 Verification

- payload hash
- shared secret

---

# 13. Replay Attack Koruması

Event:

- tekrar gönderilebilir

Koruma:

- timestamp
- signature

---

# 14. Endpoint Güvenliği

- sadece POST kabul et
- IP allowlist (opsiyonel)
- rate limit uygula

---

# 15. Ordering Problemi

Webhook'lar:

- sırasız gelebilir

Çözüm:

- event version
- state kontrolü

---

# 16. Event Versioning

Payload:

- version içermelidir

---

# 17. Event Evolution

Yeni field eklemek:

- safe

Field silmek:

- breaking

---

# 18. Dead Letter Queue

Başarısız webhook:

- DLQ'ya alınmalıdır

---

# 19. Monitoring

Takip et:

- success rate
- retry count
- failure rate

---

# 20. Logging

Webhook:

- loglanmalıdır

---

# 21. Webhook Registration

Client:

- endpoint register eder

---

## Örnek

    POST /webhooks

---

# 22. Subscription Model

Client:

- hangi event'leri almak istediğini belirtir

---

# 23. Anti-Patterns

## ❌ Idempotency olmaması

## ❌ Signature kullanmamak

## ❌ Retry yapmamak

## ❌ Event ID olmaması

---

# 24. Production Checklist

- [ ] Event ID var mı?
- [ ] Idempotency var mı?
- [ ] Retry mekanizması var mı?
- [ ] Signature doğrulaması var mı?
- [ ] Timeout var mı?
- [ ] Monitoring var mı?
- [ ] DLQ var mı?

---

# 25. Sonuç

Webhook:

- entegrasyon noktasıdır
- güvenlik ve tutarlılık konusudur

Doğru uygulanırsa:

- sistemler sorunsuz konuşur

Yanlış uygulanırsa:

- veri kaybı
- duplicate işlem
- güvenlik açığı