# RESTful API Design Guide

Bu doküman, üretim ortamında çalışan sistemler için gerçekten RESTful API nasıl tasarlanır sorusuna net ve uygulanabilir bir rehber sunar.

Amaç:

> HTTP kullanan API ile gerçek RESTful API arasındaki farkı netleştirmek ve ekip içinde standart oluşturmak.

---

# 1. REST Nedir?

REST (Representational State Transfer), Roy Fielding tarafından tanımlanmış bir mimari stildir.

REST bir protokol değildir.

REST:
- Kurallar setidir
- Constraint (kısıtlar) içerir
- Bu kurallara uyan sistemlere RESTful denir

---

# 2. RESTful API Olmak İçin Constraint'ler

## 2.1 Client - Server Separation

Client ve server birbirinden bağımsızdır.

- UI ile backend ayrıdır
- Frontend ve backend ayrı geliştirilebilir
- Sistem daha kolay scale edilir

---

## 2.2 Stateless (Durumsuzluk)

Her request kendi içinde yeterli olmalıdır.

Server:
- Önceki request'i hatırlamaz
- Session tutmaz

Doğru kullanım:

    GET /orders
    Authorization: Bearer YOUR_ACCESS_TOKEN

Yanlış:
- Server-side session
- Hidden state

Stateless yapı yüksek trafikli sistemlerin temelidir.

---

## 2.3 Cacheable

Response cache'lenebilir olmalıdır.

    Cache-Control: max-age=60
    ETag: "abc123"

Kazanç:
- Daha az load
- Daha hızlı response

---

## 2.4 Uniform Interface (EN KRİTİK KURAL)

REST'in en önemli constraint'idir.

---

### 2.4.1 Resource-Based URI

Endpoint'ler isim olmalı, fiil değil.

Doğru:

    GET /users
    GET /users/123

Yanlış:

    GET /getUsers
    POST /createUser

---

### 2.4.2 HTTP Method Doğru Kullanımı

| Method | Amaç |
|--------|------|
| GET | Okuma |
| POST | Create |
| PUT | Full update |
| PATCH | Partial update |
| DELETE | Silme |

Doğru:

    POST   /orders
    GET    /orders/1
    PATCH  /orders/1
    DELETE /orders/1

---

### 2.4.3 Self-Descriptive Messages

Her request kendini açıklamalıdır.

    Content-Type: application/json
    Accept: application/json
    Authorization: Bearer YOUR_ACCESS_TOKEN

---

### 2.4.4 HATEOAS (Advanced)

Response içinde next action'lar olabilir:

    {
      "id": 1,
      "status": "pending",
      "links": [
        { "rel": "self", "href": "/orders/1" },
        { "rel": "cancel", "href": "/orders/1/cancel" }
      ]
    }

Not:
Çoğu production sistem bunu uygulamaz.

---

## 2.5 Layered Architecture

Client hangi katmanla konuştuğunu bilmemelidir.

Katmanlar:
- CDN
- API Gateway
- Service
- Database

---

## 2.6 Code on Demand (Opsiyonel)

Server client'a code gönderebilir.

Pratikte nadir kullanılır.

---

# 3. REST HTTP Değildir

REST bir mimaridir.

HTTP sadece transport'tur.

---

# 4. Yaygın Hatalar (Anti-Patterns)

## RPC API yazmak

    POST /createOrder
    POST /cancelOrder

Bu REST değildir.

---

## HTTP Method Ignore Etmek

    POST /orders/get

---

## Stateful Auth

- Session store kullanmak
- Server tarafında kullanıcı durumu tutmak

---

## Action-Based Endpoint

    POST /users/123/activate

REST değildir ama pratikte sık görülür.

---

# 5. Production-Level REST API Tasarımı

## 5.1 Versioning

    /api/v1/products
    /api/v2/products

---

## 5.2 Filtering ve Pagination

    GET /products?category=shoes&page=1&size=20

---

## 5.3 Sorting

    GET /products?sort=price_desc

---

## 5.4 Field Selection

    GET /products?fields=id,name,price

---

# 6. Doğru vs Yanlış API

## Yanlış

    POST /getProducts
    POST /createProduct
    POST /deleteProduct

---

## Doğru

    GET    /products
    POST   /products
    GET    /products/{id}
    PATCH  /products/{id}
    DELETE /products/{id}

---

# 7. Gerçek Dünya

Çoğu API tam RESTful değildir.

Seviyeler:

| Level | Açıklama |
|------|----------|
| 0 | RPC |
| 1 | Resource |
| 2 | HTTP verbs |
| 3 | HATEOAS |

Çoğu sistem Level 2 seviyesindedir.

---

# 8. Sonuç

RESTful API demek:

- Stateless
- Resource-based
- HTTP semantics doğru
- Cache destekli
- Layered

bir sistem demektir.

---

# 9. Production Checklist

- [ ] URI'ler resource-based mi?
- [ ] HTTP methodlar doğru mu?
- [ ] Stateless mi?
- [ ] Status code doğru mu?
- [ ] Pagination var mı?
- [ ] Filtering var mı?
- [ ] Versioning var mı?

---

# Final

REST bir standart değil, disiplin meselesidir.

Doğru uygulanırsa:
- Sistem anlaşılır olur
- Kolay scale edilir
- Maintain edilir

Yanlış uygulanırsa:
- Sistem karmaşıklaşır