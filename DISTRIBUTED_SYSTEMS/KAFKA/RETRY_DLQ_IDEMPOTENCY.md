# Kafka Retry, DLQ & Idempotency Guide

## 1. Amaç

Consumer tarafında:

- hata yönetimi
- retry stratejisi
- veri kaybını önleme
- duplicate işlemeyi engelleme

---

## 2. Temel Gerçekler

- Kafka at-least-once delivery sağlar
- aynı mesaj birden fazla gelebilir
- consumer duplicate işleyebilir

---

## 3. Retry Stratejisi

### Retry Yapılması Gereken Durumlar

- geçici DB hatası
- network timeout
- downstream servis geçici unavailable

---

### Retry Yapılmaması Gereken Durumlar

- invalid payload
- schema hatası
- domain validation failure

---

## 4. Retry Yaklaşımları

- bounded retry
- exponential backoff
- retry topic
- delayed retry

---

## 5. DLQ (Dead Letter Queue)

DLQ:

- işlenemeyen mesajların tutulduğu yerdir
- veri kaybını önler
- debugging sağlar

---

## 6. Kritik İlke

DLQ, hata gizleme mekanizması değildir.

---

## 7. Idempotency

Kafka duplicate mesaj gönderebilir.

Bu nedenle:

- consumer aynı event’i tekrar işleyebilir
- işlem sonucu aynı kalmalıdır

---

## 8. Idempotency Yöntemleri

- processed_events tablosu
- unique constraint
- business key kontrolü
- upsert kullanımı

---

## 9. Anti-Patternler

- sonsuz retry
- memory içinde retry queue
- DLQ’yu ana akış gibi kullanmak
- idempotency olmadan side-effect yapmak

---

## 10. Kritik İlke

Consumer logic her zaman idempotent olmalıdır.