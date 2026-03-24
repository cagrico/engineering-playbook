# Kafka Backpressure & Memory Guide

## 1. Amaç

Kafka consumer sistemlerinde:

- memory patlamasını önlemek
- sistemin çökmesini engellemek
- kontrollü throughput sağlamak

---

## 2. Temel Problem

Kafka hızlıdır.

Ama downstream sistem:

- database
- API
- external servis

yavaş olabilir.

Sonuç:

consumer buffer büyür  
memory şişer  
GC spike oluşur  
sistem çöker

---

## 3. Temel İlke

Consumer throughput:

broker’dan okunabilen hız değil  
işlenebilen hızdır

---

## 4. Backpressure Nedir?

Backpressure:

sistem kapasitesini aşan yükü sınırlama mekanizmasıdır

---

## 5. Stratejiler

### 5.1 Bounded Worker Pool

- sabit worker sayısı
- sınırsız goroutine yok

---

### 5.2 Bounded Queue

- queue boyutu limitli olmalı
- sınırsız channel kabul edilmez

---

### 5.3 Concurrency Limit

- aynı anda işlenen mesaj sayısı sınırlanır

---

### 5.4 Pause / Resume

- consumer gerektiğinde pause edilir
- sistem rahatladığında resume edilir

---

## 6. Memory Model

Her mesaj:

- payload
- decode
- object allocation

ile memory tüketir

---

## 7. Örnek Hesap

Mesaj boyutu: 8 KB  
In-flight mesaj: 20,000

Sadece payload:

160 MB

Gerçek kullanım:

çok daha yüksek

---

## 8. Riskler

- burst traffic
- slow downstream
- retry storm
- DLQ accumulation

---

## 9. Anti-Patternler

- her mesaj için goroutine açmak
- sınırsız buffered channel
- memory içinde retry queue tutmak
- batch boyutunu kontrolsüz büyütmek

---

## 10. Doğru Yaklaşım

- bounded concurrency
- bounded queue
- controlled retry
- monitoring

---

## 11. Kritik İlke

Sistem:

yük altında yavaşlayabilir  
ama kontrolsüz büyüyemez

---

## 12. Sonuç

Kafka consumer:

en hızlı çalışan değil  
en stabil çalışan olmalıdır