# Kafka Outbox & Consumer Idempotency Guide

## 1. Problem

Producer tarafında:

- database write
- Kafka publish

iki ayrı sistemdir.

Bu nedenle:

- DB başarılı, Kafka başarısız olabilir
- Kafka başarılı, DB rollback olabilir

Bu duruma "dual write problem" denir.

---

## 2. Outbox Pattern

Çözüm:

Aynı transaction içinde:

1. domain data yazılır
2. outbox tablosuna event yazılır

---

## 3. Outbox Flow

1. business işlem yapılır
2. outbox tablosuna event insert edilir
3. transaction commit edilir
4. ayrı worker outbox’ı okur
5. Kafka’ya publish eder
6. status güncellenir

---

## 4. Avantajlar

- veri kaybı riski azalır
- producer güvenilir hale gelir
- retry kontrol edilebilir

---

## 5. Outbox Tablo Örneği

Alanlar:

id  
event_id  
event_name  
payload  
status  
created_at  
processed_at

---

## 6. Consumer Idempotency

Kafka:

aynı mesajı birden fazla gönderebilir

Bu nedenle:

consumer aynı işlemi tekrar yapmamalıdır

---

## 7. Idempotency Yöntemleri

### 7.1 Processed Events Tablosu

event_id tutulur  
aynı event tekrar işlenmez

---

### 7.2 Unique Constraint

DB seviyesinde duplicate engellenir

---

### 7.3 Business Key

order_id gibi unique alan kullanılır

---

### 7.4 Upsert

insert yerine upsert kullanılır

---

## 8. Kritik Kural

Offset commit:

sadece işlem garantili tamamlandıktan sonra yapılır

---

## 9. Anti-Patternler

- DB ve Kafka’yı ayrı transaction gibi düşünmek
- idempotency olmadan side-effect yapmak
- retry ile duplicate sorununu çözmeye çalışmak
- event_id kullanmamak

---

## 10. Kritik İlke

Event processing her zaman tekrar çalıştırılabilir (replayable) olmalıdır.