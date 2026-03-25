# API Naming Convention Guide

Bu doküman, production ortamında çalışan backend sistemlerde API endpoint isimlendirmesi için net kurallar tanımlar.

Amaç:

- Endpoint isimlendirmesini standardize etmek
- Resource ve action ayrımını netleştirmek
- HTTP semantics ile uyumlu bir API yüzeyi oluşturmak
- Uzun vadede maintainable bir route yapısı kurmak

---

# 1. Temel İlke

API route isimleri:

- Tahmin edilebilir olmalı
- Kaynak odaklı olmalı
- HTTP method ile birlikte anlam kazanmalı
- Ekip içinde tutarlı olmalı

Doğru API tasarımı sadece çalışan endpoint üretmez.

Doğru API tasarımı:
- okunabilirliği artırır
- client entegrasyonunu kolaylaştırır
- dokümantasyonu sadeleştirir
- büyüyen sistemlerde kaosu önler

---

# 2. Ana Kural: Route'larda Fiil Değil Resource Kullan

HTTP method zaten eylemi taşır.

Bu nedenle route içinde yeniden fiil yazılmaz.

Doğru:

    GET    /users
    GET    /users/{id}
    POST   /users
    PATCH  /users/{id}
    DELETE /users/{id}

Yanlış:

    GET    /getUsers
    POST   /createUser
    POST   /updateUser
    POST   /deleteUser

Kural:

- URL resource'u ifade eder
- Method işlemi ifade eder

---

# 3. Resource İsimleri Çoğul Olmalı

Collection endpoint'lerde çoğul isim kullanılır.

Doğru:

    /users
    /orders
    /products
    /categories

Yanlış:

    /user
    /order
    /product
    /category

Neden?

Çünkü endpoint çoğu zaman bir resource type'ı temsil eder.
`/users`, user koleksiyonunu anlatır.
`/users/{id}`, bu koleksiyon içindeki tek bir kaynağı anlatır.

---

# 4. Tekil Kaynak Altında ID Kullan

Tek bir kaynağa erişim için path parameter kullanılır.

Doğru:

    GET /users/{id}
    GET /orders/{id}
    PATCH /products/{id}

Yanlış:

    GET /user-detail/{id}
    POST /get-order-by-id
    POST /product/update/{id}

---

# 5. Route Segment İsimleri Kısa ve Net Olmalı

Path segment'leri sade tutulmalıdır.

Doğru:

    /users
    /shop-orders
    /inventory-items

Yanlış:

    /all-registered-platform-users
    /shop-order-management-items
    /inventory-product-stock-record-list

Kural:
- Gereksiz kelime kullanma
- Açıklamayı path'e değil dokümantasyona taşı
- Route adı bir cümle olmamalı

---

# 6. Kebab-Case Kullan

Multi-word path segment'lerde kebab-case kullan.

Doğru:

    /admin-users
    /shop-orders
    /product-reviews

Yanlış:

    /adminUsers
    /shop_orders
    /ProductReviews

Kural:
- Path segment'lerinde kebab-case
- Query param'larda da mümkünse aynı disiplin
- JSON body naming ile route naming birbirine karıştırılmamalı

---

# 7. Collection ve Item Ayrımını Net Koru

## Collection

    GET  /products
    POST /products

## Single Item

    GET    /products/{id}
    PATCH  /products/{id}
    DELETE /products/{id}

Bu ayrım bozulmamalıdır.

Yanlış örnek:

    POST /products/create
    POST /products/update
    POST /products/delete

---

# 8. Nested Resource Kullanımı

Bir resource başka bir resource'un doğal alt kümesiyse nested route kullanılabilir.

Doğru:

    GET /users/{id}/addresses
    GET /orders/{id}/items
    GET /products/{id}/reviews

Ancak nesting derinliği kontrolsüz büyümemelidir.

Kaçınılması gereken:

    /companies/{companyId}/shops/{shopId}/orders/{orderId}/items/{itemId}/notes

Bu tip yapılar:
- okunabilirliği düşürür
- maintainability'yi bozar
- client tarafında gereksiz complexity üretir

Pratik kural:
- Mümkünse 1 veya 2 seviye nesting
- Daha derini gerekiyorsa yeniden tasarla

---

# 9. Query Param ile Filtrele, Path ile Resource Seç

Path:
- Hangi resource'a erişildiğini belirtir

Query:
- O resource üzerinde filtreleme, sıralama, sayfalama yapar

Doğru:

    GET /products?category=shoes
    GET /products?brand=nike
    GET /orders?status=pending&page=1&size=20

Yanlış:

    GET /products/category/shoes
    GET /get-products-by-brand/nike
    GET /orders/pending/list

Not:
Bazı path tabanlı filtre yapıları teknik olarak çalışır.
Ama standardizasyon için filtre query string tarafında tutulmalıdır.

---

# 10. Search için Ayrı Endpoint Kararı

Genel filtreleme query param ile yapılır.

Örnek:

    GET /products?query=running-shoes

Ancak gerçekten özel bir arama davranışı varsa ayrı endpoint düşünülebilir:

    GET /search/products?query=running-shoes

Kural:
- Basit liste filtreleme ise collection endpoint içinde kal
- Ayrı scoring, typo tolerance, suggestion, faceting gibi özel arama davranışları varsa dedicated search endpoint kullanılabilir

---

# 11. Action Endpoint'ler Ne Zaman Kabul Edilebilir?

Saf REST yaklaşımında fiil odaklı route tercih edilmez.
Ancak production sistemlerde bazı business action'lar için kontrollü şekilde kullanılabilir.

Örnek:

    POST /orders/{id}/cancel
    POST /payments/{id}/refund
    POST /users/{id}/verify-email

Bu endpoint'ler istisnadır.
CRUD route'ların yerine geçmemelidir.

Kural:
- Önce resource modelle
- Gerçekten domain action gerekiyorsa action endpoint kullan
- Her şeyi action endpoint'e çevirme

Yanlış yaklaşım:

    POST /orders/create
    POST /orders/update
    POST /orders/delete
    POST /orders/get

Doğru yaklaşım:

    POST   /orders
    GET    /orders/{id}
    PATCH  /orders/{id}
    DELETE /orders/{id}
    POST   /orders/{id}/cancel

---

# 12. Admin ve Public API Ayrımı

Farklı erişim modeli ve farklı kullanım amacı olan API'ler path üzerinden ayrılabilir.

Örnek:

    /api/v1/admin/users
    /api/v1/admin/orders
    /api/v1/public/products
    /api/v1/public/categories

Alternatif olarak domain bazlı ayrım da yapılabilir:

    /api/v1/backoffice/users
    /api/v1/storefront/products

Burada kritik nokta:
- Ayrım anlamlı olmalı
- Security boundary ile uyumlu olmalı
- Rastgele prefix kullanılmamalı

---

# 13. Versioning İsimlendirmesi

Version path içinde açık ve stabil olmalıdır.

Doğru:

    /api/v1/users
    /api/v1/orders
    /api/v2/orders

Yanlış:

    /api/users/v1
    /v1/api/users
    /users/v1/list

Önerilen standart:

    /api/v1/{resource}

---

# 14. Route İçinde Teknoloji veya Uygulama Detayı Taşıma

Route'lar implementation detail içermemelidir.

Yanlış:

    /mysql-users
    /elastic-products
    /redis-cache/keys
    /fiber-orders

Doğru:

    /users
    /products
    /orders

Client, backend'in hangi teknolojiyle çalıştığını bilmek zorunda değildir.

---

# 15. Route'da UI Kelimeleri Kullanma

Route'lar ekran odaklı değil, domain odaklı olmalıdır.

Yanlış:

    /homepage-products
    /sidebar-categories
    /checkout-page-data
    /product-detail-widget

Doğru:

    /products
    /categories
    /checkouts
    /product-details

UI ihtiyacı backend resource modelini bozmaz.
Gerekirse BFF veya aggregation endpoint ayrı tasarlanır.

---

# 16. Route Naming için Domain Dili Kullan

İsimlendirme teknik değil, iş diliyle uyumlu olmalıdır.

Doğru:

    /orders
    /shipments
    /payments
    /refunds

Yanlış:

    /order-process-records
    /shipment-transport-units
    /money-actions

Kural:
- Domain'de hangi isim kullanılıyorsa route'da da onu kullan
- Takım içi dil ile API dili farklılaşmamalı

---

# 17. Tutarlı Çiftler Kur

API yüzeyi tutarlı olmalıdır.

Örnek:

    /products
    /products/{id}
    /products/{id}/reviews

    /orders
    /orders/{id}
    /orders/{id}/items

Tutarsız örnek:

    /products
    /product-detail/{id}
    /orders
    /get-order-items/{id}

Bir endpoint'e bakan geliştirici, diğerlerini tahmin edebilmelidir.

---

# 18. Query Parameter İsimlendirme Standardı

Query param isimleri kısa, açık ve stabil olmalı.

Önerilen örnekler:

    ?page=1
    ?size=20
    ?sort=created_at_desc
    ?status=active
    ?category=shoes
    ?fields=id,name,price

Kaçınılması gerekenler:

    ?pageNumber=1&perPageItemCount=20
    ?orderByFieldAndDirection=created_at_desc

Kural:
- Kısa ama anlamsız değil
- Her endpoint'te aynı amaca aynı isim ver
- Bir yerde `size`, başka yerde `limit` kullanma

---

# 19. Route Tasarımında Kaçınılması Gereken Kötü Örnekler

## Kötü

    POST /doLogin
    POST /createOrder
    POST /updateOrderStatus
    POST /deleteAddress
    GET  /getAllProducts
    POST /user/changePassword

## Daha Doğru

    POST   /sessions
    POST   /orders
    PATCH  /orders/{id}
    DELETE /addresses/{id}
    GET    /products
    POST   /users/{id}/password-reset

Not:
Authentication ve password reset gibi bazı alanlar saf CRUD'dan sapabilir.
Burada amaç, sapmayı kontrollü ve tutarlı yapmaktır.

---

# 20. Ekip Standardı İçin Zorunlu Kurallar

Aşağıdaki kurallar ekip standardı olarak sabitlenmelidir:

- Route'larda resource isimleri çoğul yazılır
- Path segment'lerinde kebab-case kullanılır
- CRUD işlemleri HTTP method ile ifade edilir
- Query param filtreleme, sıralama ve pagination için kullanılır
- Route içine gereksiz fiil yazılmaz
- Deep nesting yapılmaz
- Implementation detail route'a taşınmaz
- UI dili değil domain dili kullanılır
- Version prefix standardı korunur
- Aynı problemi çözen endpoint'lerde aynı isimlendirme disiplini uygulanır

---

# 21. Production Checklist

Yeni endpoint eklemeden önce kontrol et:

- [ ] Route resource-based mi?
- [ ] İsim çoğul mu?
- [ ] HTTP method doğru mu?
- [ ] Fiil route içine gereksiz taşınmış mı?
- [ ] Query param'lar tutarlı mı?
- [ ] Nested route derinliği makul mü?
- [ ] Domain dili kullanılmış mı?
- [ ] Versioning standardı korunmuş mu?
- [ ] UI veya implementation detail route'a sızmış mı?

---

# 22. Sonuç

İyi API isimlendirmesi kozmetik bir konu değildir.

İyi isimlendirme:
- mimari kaliteyi yükseltir
- öğrenme maliyetini azaltır
- dokümantasyonu sadeleştirir
- uzun vadeli bakım maliyetini düşürür

Kötü isimlendirme ise zamanla şunlara dönüşür:
- route kaosu
- tahmin edilemeyen API yüzeyi
- duplicate endpoint'ler
- artan entegrasyon maliyeti

Bu yüzden endpoint ismi yazmak, sadece path seçmek değildir.

Bu:
- domain modelleme kararıdır
- API ergonomisi kararıdır
- uzun vadeli bakım kararıdır