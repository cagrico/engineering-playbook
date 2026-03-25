# API Async Patterns (Event-Driven Architecture Guide)

Bu doküman, production ortamında çalışan API'lerde async (asenkron) işlem tasarımı ve event-driven mimari kurallarını tanımlar.

Amaç:

- Uzun süren işlemleri non-blocking hale getirmek
- Sistem throughput'unu artırmak
- Microservice mimaride loose coupling sağlamak
- Event-driven sistemlerde tutarlılığı korumak
- High traffic altında stabil davranış elde etmek

---

# 1. Temel İlke

Her işlem:

- synchronous olmak zorunda değildir

> Uzun süren işlemler async olmalıdır

---

# 2. Sync vs Async

## Synchronous

- request → işlem → response

## Asynchronous

- request → kabul → işlem arka planda → sonuç sonradan

---

# 3. Ne Zaman Async Kullanılır?

- uzun süren işlemler
- external servis bağımlılığı
- yüksek hacimli işlemler
- batch operasyonlar
- event propagation

---

# 4. Async API Pattern

## 4.1 Request

    POST /orders

---

## 4.2 Response

    HTTP/1.1 202 Accepted

    {
      "job_id": "job_123",
      "status": "pending"
    }

---

# 5. Job Tracking

Client:

    GET /jobs/job_123

---

## 5.1 Job Response

    {
      "id": "job_123",
      "status": "completed",
      "result": {
        "order_id": "ord_123"
      }
    }

---

# 6. Event-Driven Architecture

Async sistemlerde:

- event publish edilir
- consumer'lar işler

---

# 7. Event Flow

    API → Kafka → Consumer → DB

---

# 8. Event Design

Event:

- immutable olmalıdır
- self-contained olmalıdır

---

## 8.1 Örnek Event

    {
      "event_id": "evt_123",
      "type": "order_created",
      "data": {
        "order_id": "ord_123"
      },
      "created_at": "2026-03-25T10:00:00Z"
    }

---

# 9. Event Idempotency

Consumer:

- duplicate event alabilir

Bu yüzden:

- idempotent olmalıdır

---

# 10. At-Least-Once Delivery

Kafka:

- duplicate delivery yapabilir

Sistem:

- bunu tolere etmelidir

---

# 11. Exactly-Once Yanılgısı

Gerçek:

- exactly-once zordur
- pahalıdır

Öneri:

> at-least-once + idempotency

---

# 12. Event Ordering

Bazı durumlarda:

- ordering önemlidir

Örnek:

- order create → order update

Çözüm:

- partition key kullan

---

# 13. Event Schema

Event:

- versionlanmalıdır

---

## 13.1 Örnek

    {
      "version": 1,
      "type": "order_created"
    }

---

# 14. Event Evolution

Yeni field eklemek:

- breaking değildir

Field silmek:

- breaking'dir

---

# 15. Retry Strategy

Consumer:

- retry yapmalıdır

Ama:

- sonsuz retry yapılmamalıdır

---

# 16. DLQ (Dead Letter Queue)

Başarısız event:

- DLQ'ya gönderilmelidir

---

# 17. Backpressure

Consumer:

- sınırsız hızda işlememelidir

---

# 18. Async vs Sync Kararı

## Sync

- hızlı işlem
- immediate response

## Async

- uzun işlem
- eventual consistency

---

# 19. Eventual Consistency

Async sistemlerde:

- anında tutarlılık yoktur

---

# 20. Saga Pattern

Distributed işlem:

- birden fazla servis içerir

---

## 20.1 Orchestration

- merkezi kontrol

## 20.2 Choreography

- event ile koordinasyon

---

# 21. Monitoring

Takip et:

- lag
- processing time
- failure rate

---

# 22. Anti-Patterns

## ❌ Her şeyi async yapmak

## ❌ Event schema'yı değiştirmek

## ❌ Idempotency olmaması

## ❌ DLQ kullanmamak

---

# 23. Production Checklist

- [ ] Async gerektiren işlemler ayrılmış mı?
- [ ] Job tracking var mı?
- [ ] Event idempotent mi?
- [ ] Retry stratejisi var mı?
- [ ] DLQ var mı?
- [ ] Event versioning var mı?
- [ ] Ordering kontrol edilmiş mi?

---

# 24. Sonuç

Async API:

- performans konusudur
- scalability konusudur

Doğru uygulanırsa:

- sistem hızlı olur
- yük dengelenir

Yanlış uygulanırsa:

- data inconsistency
- debug zorluğu
- production chaos