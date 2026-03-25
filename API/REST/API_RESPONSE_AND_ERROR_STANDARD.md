# API Response & Error Standard

Bu doküman, production ortamında çalışan backend servislerde HTTP response ve error response yapısının nasıl standardize edilmesi gerektiğini tanımlar.

Amaç:

- API response formatını öngörülebilir hale getirmek
- Client tarafında hata yönetimini kolaylaştırmak
- Servisler arasında tutarlı bir sözleşme oluşturmak
- Debugging, observability ve maintainability kalitesini artırmak

Bu doküman özellikle şunları hedefler:

- Tutarlı success response yapısı
- Tutarlı error response yapısı
- HTTP status code ile body semantiğinin uyumlu olması
- Validation, business ve system error'larının ayrıştırılması
- Client'ın parse etmesi kolay, backend'in üretmesi disiplinli bir yapı kurulması

---

# 1. Temel İlke

Bir API response'u:

- tahmin edilebilir olmalı
- parse edilmesi kolay olmalı
- status code ile çelişmemeli
- internal implementation detail sızdırmamalı
- log, metric ve tracing ile uyumlu olmalı

Yanlış response standardı zamanla şunlara yol açar:

- client tarafında if-else kaosu
- her endpoint için farklı parse mantığı
- anlamsız error mesajları
- support süresinin artması
- debugging maliyetinin yükselmesi

---

# 2. En Temel Kural

HTTP status code birincil sinyaldir.

Response body, status code'u tamamlar.

Bu yüzden:

- başarılı işlemde 2xx kullanılmalı
- client hatasında 4xx kullanılmalı
- server hatasında 5xx kullanılmalı
- body içinde "success: false" yazıp 200 dönülmemeli

Yanlış:

    HTTP/1.1 200 OK

    {
      "success": false,
      "message": "Validation failed"
    }

Doğru:

    HTTP/1.1 400 Bad Request

    {
      "code": "VALIDATION_ERROR",
      "message": "Request validation failed"
    }

---

# 3. Response Yapısında Ana Yaklaşım

Tüm response'ları tek tip zarf içine sokmak her zaman doğru değildir.

Özellikle production REST API'lerde en sağlıklı yaklaşım şudur:

- Success response, veriyi doğal şekilde döner
- Error response, standardize edilmiş bir hata objesi döner

Yani:

- success için gereksiz envelope kullanımından kaçınılır
- error için güçlü ve tutarlı bir contract oluşturulur

---

# 4. Success Response Standardı

## 4.1 Tek Kaynak Dönüşü

Tek bir resource dönülüyorsa body doğrudan o resource'u temsil eder.

Örnek:

    HTTP/1.1 200 OK

    {
      "id": "usr_123",
      "name": "Çağrı",
      "email": "cagri@example.com",
      "status": "active",
      "created_at": "2026-03-25T10:00:00Z"
    }

Burada ekstra olarak:

- "success": true
- "error": null
- "message": "User fetched successfully"

gibi alanlar zorunlu değildir.

Neden?

Çünkü:
- HTTP 200 zaten success bilgisini verir
- data zaten response body'nin kendisidir
- gereksiz zarf yapısı response'u şişirir

---

## 4.2 Liste Dönüşü

Liste endpoint'lerinde veri koleksiyon halinde döndürülür.

Örnek:

    HTTP/1.1 200 OK

    {
      "items": [
        {
          "id": "prd_1",
          "name": "Product A",
          "price": "99.90"
        },
        {
          "id": "prd_2",
          "name": "Product B",
          "price": "149.90"
        }
      ],
      "pagination": {
        "page": 1,
        "size": 20,
        "total_items": 124,
        "total_pages": 7
      }
    }

Liste response'larında önerilen alanlar:

- items
- pagination

Alternatif isimler mümkün olsa da ekip standardı sabitlenmelidir.

Önerilen:
- collection alanı için `items`
- pagination objesi için `pagination`

---

## 4.3 Create Response

Yeni resource oluşturulduğunda:

- status code `201 Created` olmalı
- mümkünse oluşturulan resource dönülmeli
- gerekiyorsa `Location` header eklenmeli

Örnek:

    HTTP/1.1 201 Created
    Location: /api/v1/users/usr_123

    {
      "id": "usr_123",
      "name": "Çağrı",
      "email": "cagri@example.com",
      "status": "active",
      "created_at": "2026-03-25T10:00:00Z"
    }

---

## 4.4 Delete Response

Silme işleminde iki yaygın yaklaşım vardır:

### Seçenek 1: 204 No Content

    HTTP/1.1 204 No Content

Bu en temiz yaklaşımdır.

### Seçenek 2: 200 OK + bilgi mesajı

    HTTP/1.1 200 OK

    {
      "message": "Resource deleted"
    }

Öneri:
- Silme gerçekten idempotent ve ek veri gerektirmiyorsa `204 No Content`
- Client ek bilgi bekliyorsa `200 OK`

Ekip içinde tek standart seçilmelidir.

---

## 4.5 Update Response

Update sonrası en güvenli yaklaşım:

- `200 OK` + güncel resource

Örnek:

    HTTP/1.1 200 OK

    {
      "id": "usr_123",
      "name": "Çağrı Yılmaz",
      "email": "cagri@example.com",
      "status": "active",
      "updated_at": "2026-03-25T10:15:00Z"
    }

Alternatif:
- `204 No Content`

Ama çoğu client güncel state görmek istediği için `200 + updated resource` daha pratiktir.

---

# 5. Error Response Standardı

Error response'lar tüm servislerde aynı formatta olmalıdır.

Önerilen standart:

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

Ana alanlar:

- code
- message
- details
- request_id

---

## 5.1 code

Makine tarafından işlenebilir, stabil bir hata kodudur.

Örnekler:

- VALIDATION_ERROR
- UNAUTHORIZED
- FORBIDDEN
- RESOURCE_NOT_FOUND
- CONFLICT
- RATE_LIMITED
- INTERNAL_ERROR

Kural:
- code sabit olmalı
- client bu code'a göre davranabilmeli
- rastgele değişmemeli
- insan diline göre değil, contract mantığına göre tasarlanmalı

---

## 5.2 message

İnsanın okuyacağı özet hata açıklamasıdır.

Örnek:

- Request validation failed
- Authentication required
- Resource not found
- Payment could not be processed

Kural:
- message kısa ve anlaşılır olmalı
- internal stack trace içermemeli
- DB, SQL, panic, nil pointer gibi detaylar dışarı sızmamalı

Yanlış:

    {
      "code": "INTERNAL_ERROR",
      "message": "pq: duplicate key value violates unique constraint users_email_key"
    }

Doğru:

    {
      "code": "CONFLICT",
      "message": "A user with this email already exists"
    }

---

## 5.3 details

Opsiyonel ama çok faydalıdır.

Özellikle:
- validation error
- field bazlı input hataları
- bazı business rule ihlalleri

için kullanılmalıdır.

Örnek:

    {
      "code": "VALIDATION_ERROR",
      "message": "Request validation failed",
      "details": [
        {
          "field": "email",
          "message": "email must be a valid email address"
        },
        {
          "field": "password",
          "message": "password must be at least 8 characters"
        }
      ],
      "request_id": "req_abc123"
    }

Kural:
- details opsiyonel olmalı
- success response'ta bulunmamalı
- her error'da zorunlu olmamalı

---

## 5.4 request_id

Her response'ta dönmek zorunlu değil ama error response'ta çok değerlidir.

Amaç:
- kullanıcı destek süreçlerini hızlandırmak
- log korelasyonu sağlamak
- tracing ve debugging'i kolaylaştırmak

Örnek:

    "request_id": "req_abc123"

Bu ID:
- gateway'den gelebilir
- middleware tarafından üretilebilir
- log/tracing sistemleri ile korele edilebilir

---

# 6. Hata Tipleri ve Ayrımı

Error'lar karışık olmamalıdır.

En az şu ayrım net olmalıdır:

- validation errors
- authentication / authorization errors
- business rule errors
- not found errors
- conflict errors
- rate limit errors
- internal/system errors

---

## 6.1 Validation Error

İstek formatı veya alanları geçersizdir.

Status:
- 400 Bad Request

Örnek:

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

## 6.2 Unauthorized

Kimlik doğrulama yapılmamıştır veya token geçersizdir.

Status:
- 401 Unauthorized

Örnek:

    {
      "code": "UNAUTHORIZED",
      "message": "Authentication required",
      "request_id": "req_abc123"
    }

---

## 6.3 Forbidden

Kimlik doğrulama vardır ama yetki yoktur.

Status:
- 403 Forbidden

Örnek:

    {
      "code": "FORBIDDEN",
      "message": "You do not have permission to perform this action",
      "request_id": "req_abc123"
    }

---

## 6.4 Not Found

İstenen resource yoktur.

Status:
- 404 Not Found

Örnek:

    {
      "code": "RESOURCE_NOT_FOUND",
      "message": "User not found",
      "request_id": "req_abc123"
    }

---

## 6.5 Conflict

Kaynak state'i mevcut işlemle çakışıyordur.

Status:
- 409 Conflict

Örnek durumlar:
- duplicate email
- already approved order
- invalid state transition

Örnek response:

    {
      "code": "CONFLICT",
      "message": "A user with this email already exists",
      "request_id": "req_abc123"
    }

---

## 6.6 Rate Limit

İstemci limitleri aşmıştır.

Status:
- 429 Too Many Requests

Örnek:

    {
      "code": "RATE_LIMITED",
      "message": "Too many requests",
      "request_id": "req_abc123"
    }

---

## 6.7 Internal Error

Beklenmeyen sistem hatasıdır.

Status:
- 500 Internal Server Error

Örnek:

    {
      "code": "INTERNAL_ERROR",
      "message": "An unexpected error occurred",
      "request_id": "req_abc123"
    }

Kural:
- internal error message generic olmalı
- asıl teknik detay log tarafında tutulmalı
- client'a stack trace dönülmemeli

---

# 7. HTTP Status Code Kullanım Standardı

Önerilen temel harita:

- 200 OK: başarılı okuma / güncelleme
- 201 Created: yeni resource oluşturuldu
- 204 No Content: içeriksiz başarılı işlem
- 400 Bad Request: validation veya malformed request
- 401 Unauthorized: authentication yok / geçersiz
- 403 Forbidden: yetki yok
- 404 Not Found: resource bulunamadı
- 409 Conflict: state conflict
- 422 Unprocessable Entity: ekip isterse business validation için kullanılabilir
- 429 Too Many Requests: rate limit
- 500 Internal Server Error: beklenmeyen hata
- 503 Service Unavailable: geçici servis problemi

Not:
400 ve 422 birlikte kullanılacaksa ekip standardı net tanımlanmalıdır.
Karışık kullanım yapılmamalıdır.

---

# 8. 200 İçinde Error Dönme Yasağı

Aşağıdaki yaklaşım yasaklanmalıdır:

    HTTP/1.1 200 OK

    {
      "success": false,
      "error": "something went wrong"
    }

Neden yanlış?

Çünkü:
- HTTP semantiğini bozar
- monitoring'i bozar
- client davranışını karıştırır
- retry kararlarını zorlaştırır
- proxy/gateway/cache davranışını anlamsızlaştırır

Kural:
- error varsa uygun 4xx/5xx dön
- 200 sadece gerçek success için kullanılmalı

---

# 9. Success Response'ta Gereksiz Alan Kullanma

Aşağıdaki yapıdan kaçınılmalıdır:

    {
      "success": true,
      "message": "Product fetched successfully",
      "data": {
        "id": "prd_1",
        "name": "Product A"
      }
    }

Bu yapı bazı ekiplerde kullanılır ama çoğu REST API için gereksizdir.

Önerilen daha sade yapı:

    {
      "id": "prd_1",
      "name": "Product A"
    }

Liste için:

    {
      "items": [
        {
          "id": "prd_1",
          "name": "Product A"
        }
      ]
    }

Not:
Eğer organizasyon genelinde envelope standardı zorunluysa ayrı konu.
Ama varsayılan üretim standardı olarak minimal ve semantik response daha sağlıklıdır.

---

# 10. Validation Error Detay Standardı

Validation error response'larında details alanı mümkün olduğunca stabil olmalıdır.

Önerilen yapı:

    {
      "code": "VALIDATION_ERROR",
      "message": "Request validation failed",
      "details": [
        {
          "field": "email",
          "message": "email is required"
        },
        {
          "field": "password",
          "message": "password must be at least 8 characters"
        }
      ],
      "request_id": "req_abc123"
    }

Öneri:
- field adı request contract ile uyumlu olmalı
- message kullanıcı tarafından anlaşılabilir olmalı
- nested field gerekiyorsa açık gösterilmeli

Örnek:

    {
      "field": "address.city",
      "message": "city is required"
    }

---

# 11. Domain Error ile System Error Ayrımı

Bu ayrım kritik önemdedir.

## Domain / Business Error

Beklenen iş kuralı ihlalidir.

Örnek:
- stock not enough
- order already cancelled
- coupon expired

Bunlar:
- kontrollü hata
- anlamlı status code
- anlamlı code/message ile dönülmeli

Örnek:

    HTTP/1.1 409 Conflict

    {
      "code": "INSUFFICIENT_STOCK",
      "message": "Insufficient stock for this product",
      "request_id": "req_abc123"
    }

## System Error

Beklenmeyen teknik problemdir.

Örnek:
- DB timeout
- network partition
- panic
- downstream unavailable

Bunlar:
- generic message ile dönülmeli
- detay loglarda tutulmalı

Örnek:

    HTTP/1.1 500 Internal Server Error

    {
      "code": "INTERNAL_ERROR",
      "message": "An unexpected error occurred",
      "request_id": "req_abc123"
    }

---

# 12. Response'ta Internal Detail Sızdırma Yasağı

Aşağıdakiler response body'de bulunmamalıdır:

- SQL sorgusu
- stack trace
- Go panic metni
- file path
- internal IP
- dependency host bilgisi
- raw exception text
- database constraint adı

Yanlış:

    {
      "code": "INTERNAL_ERROR",
      "message": "panic: runtime error: invalid memory address or nil pointer dereference"
    }

Yanlış:

    {
      "code": "INTERNAL_ERROR",
      "message": "dial tcp 10.0.12.7:5432: connect: connection refused"
    }

Doğru:

    {
      "code": "INTERNAL_ERROR",
      "message": "An unexpected error occurred",
      "request_id": "req_abc123"
    }

---

# 13. API Response ile Log Yapısını Karıştırma

Response body ile log body aynı şey değildir.

Response:
- client içindir
- güvenli olmalıdır
- sade olmalıdır

Log:
- operasyon içindir
- teknik detay içerebilir
- root cause analysis için zengin olmalıdır

Bu yüzden:
- response sade
- log detaylı
- request_id ile korele

olmalıdır.

---

# 14. Boş Liste Durumu

Liste endpoint'lerinde sonuç yoksa:

- 200 OK dönülmeli
- boş array verilmelidir

Doğru:

    HTTP/1.1 200 OK

    {
      "items": [],
      "pagination": {
        "page": 1,
        "size": 20,
        "total_items": 0,
        "total_pages": 0
      }
    }

Yanlış:
- 404 dönmek
- null dönmek
- response shape'i değiştirmek

Kural:
- collection endpoint boşsa yine collection döner

---

# 15. Error Code İsimlendirme Standardı

Error code'lar:

- UPPER_SNAKE_CASE olmalı
- stabil olmalı
- ekip genelinde tekrar kullanılabilir olmalı

Örnekler:

- VALIDATION_ERROR
- UNAUTHORIZED
- FORBIDDEN
- RESOURCE_NOT_FOUND
- CONFLICT
- RATE_LIMITED
- INTERNAL_ERROR
- INSUFFICIENT_STOCK
- COUPON_EXPIRED
- ORDER_ALREADY_CANCELLED

Kural:
- çok genel code ile çok spesifik durumu karıştırma
- bir kısmı generic, bir kısmı domain-specific olabilir
- ama naming disiplini aynı kalmalı

---

# 16. Önerilen Standart Response Örnekleri

## Tek resource

    HTTP/1.1 200 OK

    {
      "id": "prd_123",
      "name": "Protein Powder",
      "price": "799.90",
      "currency": "TRY"
    }

## Liste

    HTTP/1.1 200 OK

    {
      "items": [
        {
          "id": "prd_123",
          "name": "Protein Powder",
          "price": "799.90",
          "currency": "TRY"
        }
      ],
      "pagination": {
        "page": 1,
        "size": 20,
        "total_items": 1,
        "total_pages": 1
      }
    }

## Validation error

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

## Not found

    HTTP/1.1 404 Not Found

    {
      "code": "RESOURCE_NOT_FOUND",
      "message": "Product not found",
      "request_id": "req_abc123"
    }

## Internal error

    HTTP/1.1 500 Internal Server Error

    {
      "code": "INTERNAL_ERROR",
      "message": "An unexpected error occurred",
      "request_id": "req_abc123"
    }

---

# 17. Ekip Standardı İçin Zorunlu Kurallar

Aşağıdaki kurallar ekip standardı olarak sabitlenmelidir:

- HTTP status code birincil sinyaldir
- 200 içinde error dönülmez
- Success response sade tutulur
- Error response standardize edilir
- Error response'ta en az code ve message bulunur
- Validation error'larda details kullanılabilir
- Internal detail client'a sızdırılmaz
- request_id ile log korelasyonu sağlanır
- Liste endpoint'lerinde boş sonuç boş array ile döner
- Tüm servisler aynı error contract'ı uygular

---

# 18. Production Checklist

Yeni endpoint geliştirirken kontrol et:

- [ ] Status code body ile uyumlu mu?
- [ ] Success response gereksiz envelope içeriyor mu?
- [ ] Error response standard formatta mı?
- [ ] Error code stabil ve makine okunabilir mi?
- [ ] Validation detail yapısı tutarlı mı?
- [ ] Internal error detayları dışarı sızıyor mu?
- [ ] request_id korelasyonu var mı?
- [ ] Empty list davranışı tutarlı mı?
- [ ] Domain error ile system error ayrılmış mı?

---

# 19. Sonuç

API response formatı sadece çıktı şekli değildir.

Bu yapı:
- client contract'ıdır
- operasyonel disiplindir
- debugging ergonomisidir
- uzun vadeli bakım kalitesidir

İyi tasarlanmış response standardı:
- client kodunu sadeleştirir
- servis davranışını öngörülebilir yapar
- hata yönetimini disipline eder
- production desteğini hızlandırır

Kötü tasarlanmış response standardı ise:
- her endpoint'te farklı davranış üretir
- client tarafında karmaşa oluşturur
- monitoring ve alerting kalitesini düşürür
- root cause analysis süresini uzatır