# Kafka Producer & Consumer Strategy

## 1. Amaç

Kafka producer ve consumer davranışını:

- güvenilir
- deterministik
- ölçeklenebilir

hale getirmek.

---

## 2. Producer Stratejisi

### 2.1 acks Ayarı

Production için:

acks = all

Bu:
- leader + follower onayı gerektirir
- veri kaybı riskini azaltır

---

### 2.2 Idempotent Producer

Önerilir:

- duplicate write riskini azaltır
- retry sırasında veri tutarlılığı sağlar

---

### 2.3 Key Seçimi

Key, partition belirler.

Kural:

Aynı aggregate için ordering gerekiyorsa aynı key kullanılmalıdır.

Örnek:

order_id  
user_id  
payment_id

---

### 2.4 Partitioning

- Kafka ordering sadece partition içinde garantilidir
- farklı partition’larda ordering yoktur

---

### 2.5 Batch ve Throughput

Producer:

- batch gönderir
- network ve disk verimliliği sağlar

Ama:

- latency artabilir
- trade-off düşünülmelidir

---

## 3. Consumer Stratejisi

### 3.1 Consumer Group

- her partition aynı anda tek consumer tarafından okunur
- paralellik partition sayısı ile sınırlıdır

---

### 3.2 Delivery Semantics

Kafka:

at-least-once delivery sağlar

Bu demektir:

- duplicate mesaj olabilir
- idempotency zorunludur

---

### 3.3 Offset Commit

Kritik kural:

Mesaj işlenmeden offset commit edilmez.

---

### 3.4 Doğru Akış

1. mesaj al
2. işle
3. external side effect yap
4. başarı doğrula
5. offset commit et

---

### 3.5 Yanlış Akış

1. mesaj al
2. offset commit et
3. işle

Bu durumda veri kaybı olur.

---

### 3.6 Parallel Processing

- partition başına paralellik kontrol edilmelidir
- unordered processing yapılmamalıdır (ordering önemliyse)

---

## 4. Anti-Patternler

- auto-commit’e kör güvenmek
- message işlenmeden commit yapmak
- sınırsız goroutine açmak
- key kullanmadan partition’a güvenmek
- ordering’i göz ardı etmek

---

## 5. Kritik İlke

Kafka’dan hızlı okumak başarı değildir.

Sistemin güvenle işleyebileceği hızda tüketmek başarıdır.