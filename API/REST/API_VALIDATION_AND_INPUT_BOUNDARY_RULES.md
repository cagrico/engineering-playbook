# API Validation & Input Boundary Rules

Bu doküman, production ortamında çalışan API'lerde input validation ve boundary kontrol kurallarını tanımlar.

Amaç:

- Sisteme giren veriyi kontrol altına almak
- Invalid, eksik ve zararlı veriyi boundary'de durdurmak
- Domain katmanına temiz ve güvenilir veri ulaştırmak
- Data integrity ve sistem stabilitesini korumak

---

# 1. Temel İlke

API boundary:

- sistemin dış dünyaya açılan kapısıdır

Bu yüzden:

> validation = boundary responsibility

---

# 2. En Kritik Kural

Invalid veri:

- sisteme girmemelidir

Yani:

- validation DB'de yapılmaz
- validation domain'e bırakılmaz
- validation boundary'de yapılır

---

# 3. Validation Katmanları

Validation tek katmanlı değildir.

## 3.1 Transport Validation

- JSON parse
- required field kontrolü
- type kontrolü

---

## 3.2 Application Validation

- iş kurallarına uygunluk
- cross-field validation

---

## 3.3 Domain Validation

- invariant kuralları
- state transition kontrolü

---

# 4. Boundary'de Yapılması Gereken Validation

Aşağıdakiler API katmanında yapılmalıdır:

- required field kontrolü
- type kontrolü
- length kontrolü
- format kontrolü (email, uuid vs)
- enum kontrolü
- numeric range kontrolü
- pagination limitleri
- query param validation

---

# 5. Required Field Kontrolü

Örnek:

    {
      "email": "",
      "password": ""
    }

Yanlış:
- boş string kabul etmek

Doğru:
- required ve non-empty kontrolü

---

# 6. Type Validation

Yanlış:

    "price": "abc"

Doğru:

- numeric alan numeric olmalı

---

# 7. Length ve Size Limitleri

Her input bounded olmalıdır.

Örnek:

- username max 50 karakter
- description max 1000 karakter

Neden?

- memory koruma
- DB performansı
- abuse önleme

---

# 8. Format Validation

Örnekler:

- email format
- uuid format
- date format (ISO8601)

---

# 9. Enum Validation

Örnek:

    status: active | passive | blocked

Yanlış:
- serbest string kabul etmek

---

# 10. Numeric Range

Örnek:

- price >= 0
- quantity > 0
- discount <= 100

---

# 11. Query Param Validation

Query param'lar da validate edilmelidir.

Örnek:

    GET /products?page=-1 ❌
    GET /products?size=100000 ❌

---

# 12. Allowlist Kullanımı

Özellikle:

- sort
- expand
- filter key

allowlist ile kontrol edilmelidir.

---

# 13. Validation Error Response

Standart error formatı kullanılmalıdır.

    HTTP/1.1 400 Bad Request

    {
      "code": "VALIDATION_ERROR",
      "message": "Request validation failed",
      "details": [
        {
          "field": "email",
          "message": "email is required"
        }
      ],
      "request_id": "req_abc123"
    }

---

# 14. Early Reject (Fail Fast)

Invalid request:

- hemen reject edilmelidir
- DB'ye gitmemelidir
- downstream'a ulaşmamalıdır

---

# 15. DB Constraint'e Güvenme

DB constraint:

- son savunma hattıdır

Validation değildir.

---

# 16. Sanitization vs Validation

Validation:
- doğru mu?

Sanitization:
- temizle

Örnek:

- trim whitespace
- normalize case

Ama:

- validation yerine geçmez

---

# 17. Over-Validation Hatası

Aşırı validation da yanlıştır.

Yanlış:

- gereksiz strict kurallar
- client'ı gereksiz zorlamak

Doğru:

- business ihtiyaç kadar validation

---

# 18. Nested Object Validation

Nested yapılar da validate edilmelidir.

Örnek:

    {
      "address": {
        "city": "",
        "zip": ""
      }
    }

---

# 19. Default Value Stratejisi

Bazı alanlar default alabilir.

Örnek:

- page = 1
- size = 20

Ama:

- kritik alanlar default olmamalı

---

# 20. Security Perspective

Validation aynı zamanda güvenliktir.

Korur:

- injection
- overflow
- abuse

---

# 21. Logging ile İlişki

Validation error:

- loglanmalıdır
- ama sensitive data maskelenmelidir

---

# 22. Anti-Patterns

## ❌ Validation yapmamak

## ❌ Sadece DB constraint'e güvenmek

## ❌ Her şeyi domain'e bırakmak

## ❌ Query param'ları kontrol etmemek

## ❌ Çok büyük payload kabul etmek

---

# 23. Production Checklist

- [ ] Tüm input alanları validate ediliyor mu?
- [ ] Required field kontrolü var mı?
- [ ] Type kontrolü var mı?
- [ ] Length ve size limitleri var mı?
- [ ] Enum ve range kontrolü var mı?
- [ ] Query param validation var mı?
- [ ] Allowlist kullanılıyor mu?
- [ ] Validation error standard mı?
- [ ] Fail-fast uygulanıyor mu?

---

# 24. Sonuç

Validation:

- sadece veri kontrolü değildir
- sistem stabilitesidir
- güvenliktir
- veri kalitesidir

Doğru yapılırsa:

- sistem temiz kalır
- hata oranı düşer
- debugging kolaylaşır

Yanlış yapılırsa:

- sistem kirlenir
- hata zinciri oluşur
- production incident kaçınılmaz olur