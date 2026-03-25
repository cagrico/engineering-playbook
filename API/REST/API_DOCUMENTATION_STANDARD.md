# API Documentation Standard

Bu doküman, production ortamında çalışan API'lerin nasıl dokümante edilmesi gerektiğini tanımlar.

Amaç:

- API kullanımını açık ve anlaşılır hale getirmek
- Client entegrasyon süresini azaltmak
- Ekip içi iletişimi standartlaştırmak
- API contract'ını net şekilde ifade etmek
- Dokümantasyon ile gerçek sistem arasında uyum sağlamak

---

# 1. Temel İlke

Dokümantasyon:

- opsiyonel değildir
- API'nin kendisinin bir parçasıdır

> Dokümantasyon yoksa API yoktur

---

# 2. En Kritik Kural

Dokümantasyon:

- her zaman güncel olmalıdır

Outdated documentation:

- yanlış implementasyon üretir
- client bug'larına sebep olur
- güven kaybı yaratır

---

# 3. Ne Dokümante Edilmelidir?

Her endpoint için:

- HTTP method
- path
- description
- request parametreleri
- request body
- response örnekleri
- error response'lar
- authentication gereksinimi
- rate limit bilgisi (opsiyonel)
- idempotency durumu (kritik endpoint'lerde)

---

# 4. Endpoint Dokümantasyon Şablonu

Her endpoint aşağıdaki formatta olmalıdır:

## Endpoint

    POST /api/v1/orders

## Description

Yeni bir sipariş oluşturur.

## Authentication

Bearer token required.

## Request Body

    {
      "product_id": "prd_123",
      "quantity": 2
    }

## Response (201)

    {
      "id": "ord_123",
      "status": "created"
    }

## Error Responses

400 - Validation error  
401 - Unauthorized  
409 - Conflict

---

# 5. Request Param Dokümantasyonu

Her parametre için:

- adı
- tipi
- required mı?
- açıklaması

Örnek:

| Field       | Type   | Required | Description            |
|------------|--------|----------|------------------------|
| product_id | string | yes      | Ürün ID               |
| quantity   | int    | yes      | Sipariş adedi         |

---

# 6. Query Param Dokümantasyonu

Örnek:

    GET /products?page=1&size=20&sort=price_desc

| Param | Type | Description |
|------|------|------------|
| page | int  | Sayfa numarası |
| size | int  | Sayfa boyutu |
| sort | string | Sıralama |

---

# 7. Response Örnekleri Zorunlu

Her endpoint:

- gerçek response örneği içermelidir

Yanlış:

- sadece schema vermek

Doğru:

- gerçek JSON örneği vermek

---

# 8. Error Response Dokümantasyonu

Tüm error'lar:

- code
- message
- status code

ile dokümante edilmelidir

---

# 9. OpenAPI / Swagger Kullanımı

Production sistemlerde:

- OpenAPI spec zorunlu olmalıdır

---

## 9.1 Ne Sağlar?

- otomatik dokümantasyon
- client SDK üretimi
- contract validation
- test kolaylığı

---

# 10. Human-Readable + Machine-Readable

Dokümantasyon:

- insanlar için okunabilir olmalı
- makineler için parse edilebilir olmalı

---

# 11. Versioning ile Uyum

Her version:

- ayrı dokümante edilmelidir

Örnek:

- v1 docs
- v2 docs

---

# 12. Example-First Yaklaşım

Önce örnek göster:

- sonra açıklama yap

Bu:

- öğrenmeyi hızlandırır
- entegrasyonu kolaylaştırır

---

# 13. Authentication Dokümantasyonu

Açıkça belirtilmelidir:

- hangi endpoint auth ister
- hangi header kullanılır

Örnek:

    Authorization: Bearer YOUR_ACCESS_TOKEN

---

# 14. Rate Limit Dokümantasyonu

Özellikle public API'lerde:

- limitler belirtilmelidir

---

# 15. Idempotency Dokümantasyonu

Kritik endpoint'lerde:

- idempotency key kullanımı belirtilmelidir

---

# 16. Deprecation Bilgisi

Deprecated endpoint'ler:

- açıkça işaretlenmelidir

---

# 17. Try It / Test Edilebilirlik

İyi dokümantasyon:

- test edilebilir olmalıdır

Örnek:

- Swagger UI
- Postman collection

---

# 18. Anti-Patterns

## ❌ Dokümantasyon yazmamak

## ❌ Güncellememek

## ❌ Sadece schema vermek

## ❌ Örnek vermemek

## ❌ Gerçek sistemle uyumsuz dokümantasyon

---

# 19. Production Checklist

- [ ] Tüm endpoint'ler dokümante mi?
- [ ] Request ve response örnekleri var mı?
- [ ] Error'lar dokümante mi?
- [ ] Auth bilgisi açık mı?
- [ ] Versioning uyumlu mu?
- [ ] OpenAPI mevcut mu?
- [ ] Dokümantasyon güncel mi?

---

# 20. Sonuç

Dokümantasyon:

- developer experience'dır
- entegrasyon hızıdır
- sistem güvenilirliğidir

İyi dokümantasyon:

- onboarding süresini düşürür
- hataları azaltır
- ekip verimini artırır

Kötü dokümantasyon:

- kaos üretir
- yanlış implementasyon doğurur
- support yükünü artırır