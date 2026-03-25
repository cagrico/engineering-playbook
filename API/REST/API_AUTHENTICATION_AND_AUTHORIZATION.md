# API Authentication & Authorization Guide

Bu doküman, production ortamında çalışan API'lerde authentication (kimlik doğrulama) ve authorization (yetkilendirme) kurallarını tanımlar.

Amaç:

- API erişimini güvenli hale getirmek
- Kimlik doğrulama ve yetkilendirme sorumluluklarını net ayırmak
- Token tabanlı sistemlerde standart oluşturmak
- Yetki modelini öngörülebilir hale getirmek
- Production ortamında güvenlik açıklarını minimize etmek

---

# 1. Temel Kavramlar

## 1.1 Authentication (Kimlik Doğrulama)

Kullanıcı kimdir?

## 1.2 Authorization (Yetkilendirme)

Bu kullanıcı ne yapabilir?

---

# 2. En Kritik Kural

Authentication ve authorization birbirine karıştırılmamalıdır.

Yanlış:

- "login olduysa her şeyi yapabilir"

Doğru:

- login → kimlik doğrulama
- her endpoint → ayrı yetki kontrolü

---

# 3. Önerilen Authentication Modeli

Production sistemlerde:

- stateless authentication
- token tabanlı yaklaşım

---

## 3.1 Bearer Token Standardı

Request:

    Authorization: Bearer YOUR_ACCESS_TOKEN

Kural:
- tüm protected endpoint'lerde zorunlu
- header dışında taşınmamalı

---

## 3.2 Token Türleri

## Access Token

- kısa ömürlü (5–30 dk)
- her request'te kullanılır

## Refresh Token

- uzun ömürlü
- access token yenilemek için kullanılır

---

# 4. Stateless Authentication

Server:

- session tutmaz
- kullanıcı state'i saklamaz

Tüm bilgi:

- token içinde veya
- merkezi store’da (ör. Redis)

---

# 5. Token İçeriği (JWT Örneği)

Token payload:

    {
      "sub": "usr_123",
      "role": "user",
      "exp": 1710000000
    }

Önerilen alanlar:

- sub (user id)
- role
- exp
- iat

---

# 6. Token Güvenliği

## 6.1 Zorunlu Kurallar

- HTTPS zorunlu
- token loglanmamalı
- token URL'de taşınmamalı
- localStorage yerine secure storage tercih edilmeli

---

## 6.2 Token Süresi

- kısa ömürlü access token
- refresh mekanizması zorunlu

---

# 7. Authorization Modelleri

## 7.1 Role-Based Access Control (RBAC)

Kullanıcı rolüne göre yetki:

    admin
    user
    moderator

---

## 7.2 Permission-Based Model

Daha granular yaklaşım:

    product.read
    product.write
    order.cancel

---

## 7.3 Resource-Based Authorization

Kullanıcı sadece kendi kaynağına erişebilir:

    GET /users/{id}

Kural:

- token.user_id == path.id

---

# 8. Endpoint Bazlı Yetkilendirme

Her endpoint:

- açıkça yetki kontrolü yapmalıdır

Örnek:

    GET /admin/users → admin only
    GET /orders → authenticated user
    DELETE /products/{id} → admin or owner

---

# 9. Unauthorized vs Forbidden

## 401 Unauthorized

- authentication yok
- token yok / geçersiz

## 403 Forbidden

- auth var ama yetki yok

---

## Örnek

401:

    {
      "code": "UNAUTHORIZED",
      "message": "Authentication required"
    }

403:

    {
      "code": "FORBIDDEN",
      "message": "You do not have permission"
    }

---

# 10. Public vs Protected Endpoint Ayrımı

## Public

- login
- register
- product listing

## Protected

- order create
- profile update
- payment

---

# 11. Middleware Kullanımı

Auth kontrolü merkezi olmalıdır.

Akış:

- request gelir
- token parse edilir
- user context oluşturulur
- handler'a aktarılır

---

# 12. Context Taşıma

Auth sonrası:

- user_id
- role
- permissions

context içinde taşınmalıdır.

---

# 13. Token Revocation

JWT stateless olduğu için:

- revoke mekanizması gerekir

Çözümler:

- blacklist (Redis)
- short-lived token
- versioning (token version)

---

# 14. Multi-Service Authentication

Microservice mimaride:

- merkezi auth service
- diğer servisler token validate eder

---

# 15. Service-to-Service Authentication

Internal servisler için:

- API key
- mTLS
- signed token

---

# 16. Rate Limit ile Entegrasyon

Auth varsa:

- rate limit user bazlı

Auth yoksa:

- IP bazlı

---

# 17. Audit Logging

Auth ile ilgili olaylar loglanmalıdır:

- login
- logout
- failed login
- permission denied

---

# 18. Common Anti-Patterns

## ❌ Session-based auth (stateless API'de)

## ❌ Token'ı query param ile taşımak

    GET /orders?token=xxx

## ❌ Herkese açık endpoint bırakmak

## ❌ Authorization kontrolünü unutmak

---

# 19. Production Checklist

- [ ] Tüm protected endpoint'lerde auth zorunlu mu?
- [ ] Token header'dan mı geliyor?
- [ ] Token süresi kontrollü mü?
- [ ] Authorization kontrolü var mı?
- [ ] 401 ve 403 ayrımı doğru mu?
- [ ] Token loglanmıyor mu?
- [ ] HTTPS zorunlu mu?
- [ ] Rate limit auth ile entegre mi?
- [ ] Audit log var mı?

---

# 20. Sonuç

Authentication ve authorization:

- sadece security konusu değildir
- sistem stabilitesi ve güvenilirliğidir

Doğru uygulanırsa:

- sistem güvenli olur
- erişim kontrolü net olur
- abuse azalır

Yanlış uygulanırsa:

- veri sızıntısı olur
- yetki ihlali olur
- production incident kaçınılmaz olur