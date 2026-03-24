# Kafka Topic Naming Convention

## 1. Amaç

Kafka topic isimlendirmesini standart hale getirmek.

Amaç:
- okunabilirlik
- tutarlılık
- domain ayrımı
- operasyonel kolaylık

---

## 2. Temel Prensip

Topic adı şu bilgileri taşımalıdır:

- domain
- entity / aggregate
- event / stream amacı

---

## 3. Önerilen Format

<domain>.<entity>.<event>

---

## 4. Örnekler

### Doğru Kullanım

order.created  
order.cancelled  
payment.completed  
payment.failed  
product.stock.updated  
notification.email.requested

---

### State / Snapshot Topic

product.current  
inventory.snapshot  
user.preferences.current

---

### Retry ve DLQ

payment.completed.retry  
payment.completed.dlq  
order.created.retry  
order.created.dlq

---

## 5. Naming Kuralları

### 5.1 Lowercase Kullan

order.created ✔  
Order.Created ❌

---

### 5.2 Nokta (.) Kullan

order.created ✔  
order_created ❌  
order-created ❌

---

### 5.3 Past Tense Kullan

order.created ✔  
payment.completed ✔

create-order ❌  
payment-process ❌

---

### 5.4 Kısa ve Net Ol

product.stock.updated ✔  
product_stock_update_event_final ❌

---

### 5.5 Service Adı Kullanma

order.created ✔  
order-service-topic ❌

---

## 6. Versioning Stratejisi

Varsayılan yaklaşım:

- topic adında versiyon taşıma
- önce schema backward compatibility sağla

Sadece breaking change varsa:

order.created  
order.created.v2

---

## 7. Anti-Patternler

topic1  
test-topic  
order_topic_new  
product_update_final_v3  
payment_event_topic  
stockChanged

---

## 8. Kritik İlke

Topic ismi, sistemde akan verinin semantiğini açık şekilde ifade etmelidir.