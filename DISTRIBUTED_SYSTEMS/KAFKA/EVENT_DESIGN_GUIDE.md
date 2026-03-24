# Kafka Event Design Guide

## 1. Amaç

Event payload’larının:

- anlaşılır
- sürdürülebilir
- backward compatible
- domain odaklı

olmasını sağlamak.

---

## 2. Event Nedir?

Event:

- bir state değişiminin kaydıdır
- geçmişte olmuş bir şeyi temsil eder

Event değildir:

- command
- request
- future intent

---

## 3. Event İsimlendirme

Past tense kullanılmalıdır:

order.created  
payment.completed  
stock.decreased

---

## 4. Event Payload Yapısı

Önerilen standart alanlar:

event_id  
event_name  
event_version  
occurred_at  
producer  
aggregate_type  
aggregate_id  
payload

---

## 5. Alan Açıklamaları

### event_id
- unique olmalıdır
- idempotency için kullanılır

### event_name
- topic ile uyumlu olmalıdır

### event_version
- schema değişikliklerini yönetir

### occurred_at
- event’in gerçekleştiği zamandır
- processing time değildir

### producer
- event’i üreten servis

### aggregate_type
- entity tipi

### aggregate_id
- event’in ait olduğu entity
- ordering için kritiktir

### payload
- asıl veriyi taşır

---

## 6. Tasarım Prensipleri

### 6.1 Payload Minimal Olmalı

Sadece gerekli data gönderilmeli.

---

### 6.2 Internal Model Publish Edilmez

Event payload, producer’ın internal entity modeli değildir.

---

### 6.3 Immutable Olmalı

Event publish edildikten sonra değiştirilemez.

---

### 6.4 Backward Compatible Olmalı

- yeni field eklenebilir
- mevcut field değiştirilmemeli
- field silmek risklidir

---

## 7. Schema Evolution

Güvenli:

- optional field eklemek

Riskli:

- field silmek
- type değiştirmek
- zorunlu alan eklemek

---

## 8. Anti-Patternler

- tüm entity’yi payload’a koymak
- gereksiz büyük JSON göndermek
- API response publish etmek
- PII veri gömmek
- belirsiz alan isimleri kullanmak

---

## 9. Kritik İlke

Event payload, domainler arası veri sözleşmesidir.