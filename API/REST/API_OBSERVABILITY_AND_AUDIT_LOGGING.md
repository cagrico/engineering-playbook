# API Observability & Audit Logging

Bu doküman, production ortamında çalışan API'lerde observability (gözlemlenebilirlik) ve audit logging standartlarını tanımlar.

Amaç:

- Sistem davranışını ölçülebilir hale getirmek
- Production hatalarını hızlı tespit etmek
- Root cause analysis süresini azaltmak
- Kullanıcı aksiyonlarını izlenebilir hale getirmek
- SLA ve performans takibini mümkün kılmak

---

# 1. Temel İlke

Bir sistem:

- ölçülemiyorsa yönetilemez
- izlenemiyorsa debug edilemez

> Observability = production güvenilirliği

---

# 2. Observability Nedir?

Bir sistemin iç durumunu:

- log
- metric
- trace

ile anlayabilmektir

---

# 3. 3 Ana Bileşen

## 3.1 Logs

Olay bazlı bilgi:

- ne oldu?

---

## 3.2 Metrics

Sayısal veri:

- ne kadar oldu?

---

## 3.3 Tracing

Request akışı:

- nerede oldu?

---

# 4. Logging Standardı

Log'lar:

- structured olmalıdır
- JSON formatında olmalıdır

---

## 4.1 Örnek Log

    {
      "level": "info",
      "message": "order created",
      "request_id": "req_123",
      "user_id": "usr_123",
      "order_id": "ord_123",
      "duration_ms": 45
    }

---

# 5. Request ID Zorunluluğu

Her request:

- unique bir ID taşımalıdır

---

## 5.1 Amaç

- log korelasyonu
- tracing
- debug kolaylığı

---

# 6. Log Seviyeleri

- DEBUG
- INFO
- WARN
- ERROR

---

## 6.1 Kural

- production'da DEBUG kapalı olmalı
- ERROR log'lar alert üretmeli

---

# 7. Sensitive Data Yasağı

Log'larda bulunmamalıdır:

- password
- token
- credit card
- kişisel veri

---

# 8. Metrics Standardı

Takip edilmesi gereken temel metrikler:

- request count
- error rate
- latency (p50, p95, p99)
- throughput

---

# 9. Latency Ölçümü

Her endpoint için:

- duration ölçülmelidir

---

# 10. Error Rate

Ölç:

- 4xx rate
- 5xx rate

---

# 11. Tracing

Distributed sistemlerde:

- request zinciri izlenmelidir

---

## 11.1 Trace ID

Her request:

- trace_id taşır

---

# 12. Downstream Observability

Her external call:

- ayrı loglanmalıdır
- latency ölçülmelidir

---

# 13. Audit Logging Nedir?

Kullanıcı aksiyonlarını kayıt altına almaktır.

---

# 14. Audit Log Örnekleri

- kullanıcı login oldu
- sipariş oluşturdu
- ödeme yaptı
- veri sildi

---

# 15. Audit Log Yapısı

    {
      "event": "order_created",
      "user_id": "usr_123",
      "resource_id": "ord_123",
      "timestamp": "2026-03-25T10:00:00Z"
    }

---

# 16. Audit vs Application Log

Audit log:

- iş aksiyonu

Application log:

- teknik olay

---

# 17. Logging Granularity

Çok az log:

- debug zor

Çok fazla log:

- gürültü

---

# 18. Alerting

Şunlar alert üretmelidir:

- error spike
- latency spike
- timeout artışı

---

# 19. Dashboard

İzlenmesi gerekenler:

- RPS
- latency
- error rate
- saturation

---

# 20. Sampling

High traffic sistemlerde:

- tüm log'lar tutulmaz
- sampling uygulanabilir

---

# 21. Anti-Patterns

## ❌ Log atmamak

## ❌ Text log kullanmak

## ❌ request_id olmaması

## ❌ sensitive data loglamak

---

# 22. Production Checklist

- [ ] Structured logging var mı?
- [ ] request_id var mı?
- [ ] latency ölçülüyor mu?
- [ ] error rate izleniyor mu?
- [ ] tracing var mı?
- [ ] audit log tutuluyor mu?
- [ ] sensitive data maskeleniyor mu?
- [ ] alerting var mı?

---

# 23. Sonuç

Observability:

- lüks değildir
- zorunluluktur

Doğru uygulanırsa:

- sorunlar hızlı çözülür
- sistem kontrol altında kalır

Yanlış uygulanırsa:

- production karanlık olur
- hata çözmek saatler sürer