# NAMING STANDARDS (Go + Backend)

---

## 0) Temel prensipler

- **Okunabilirlik > kısalık**, ama gereksiz uzunluk da yok.
- **Tutarlılık** en kritik şey: aynı konsept her yerde aynı isimle anılmalı.
- İsimler **niyeti** anlatmalı: “ne?” + gerekiyorsa “neden?”.
- Bir isim **tek sorumluluğu** çağrıştırmalı (God-name yok).
- Kısaltma kullanacaksan: **herkesin bildiği** kısaltma olsun (ID, URL, HTTP, JSON, SQL).
- “tmp, data, info, obj, stuff” gibi **anlamsız** isimler yasak.

---

## 1) Case tipleri ve ne zaman hangisi

- **PascalCase**: exported (paket dışına açık) type/func/const/var
- **camelCase**: unexported (paket içi) type/func/var
- **snake_case**: DB tablo/kolon, bazı config/env alanları
- **kebab-case**: URL path segmentleri, bazı CLI komutları / doküman başlıkları
- **SCREAMING_SNAKE_CASE**: environment variable

> Go’da “CamelCase” denince genelde `PascalCase` ve `camelCase` birlikte kast edilir.
> 

---

## 2) Go kodu isimlendirme kuralları

### 2.1 Paket (package) isimleri

- **tamamen küçük harf**, **tek kelime** tercih: `auth`, `cache`, `logger`, `product`
- `_` ve  yok
- çoğul genelde yok (`users` yerine `user`), ama koleksiyon paketi değilse.
- domain/feature odaklı: `shipping`, `pricing`, `inventory`

✅ `package product`

❌ `package ProductService`, `package product_service`

---

### 2.2 Dosya ve klasör isimleri

- **kebab-case** veya **snake_case** değil: Go ekosisteminde en yaygın: `snake_case`
- Go dosyası: `order_service.go`, `http_handler.go`, `repo_mysql.go`
- Test: `_test.go`
- Platform/ortam: `_linux.go`, `_darwin.go`

> Ekip standardı olarak **snake_case** öneririm; arama/grep kolay.
> 

---

### 2.3 Değişkenler (variables)

- kısa scope → kısa isim, geniş scope → daha açıklayıcı.
- birim/ölçek belirgin olmalı:
    - `timeoutMs`, `ttlSeconds`, `priceCents`, `createdAt`
- koleksiyonlar çoğul:
    - `users []User`, `userIDs []int64`

Örnek:

- `u` sadece 2-3 satırlık loop içinde kabul.
- `user` daha iyi.
- `userDTO` (tip zaten DTO ise) gereksiz, ama aynı isim çatışıyorsa kullanılabilir.

✅ `user`, `users`, `userID`, `createdAt`

❌ `usr`, `dt`, `data`, `list`

---

### 2.4 Sabitler (constants)

- Exported ise `PascalCase`, değilse `camelCase`.
- “Magic number/string” yok: anlamlı const kullan.
- Sabit grupları için `const (...)`

✅

- `const MaxRetries = 3`
- `const defaultTimeout = 3 * time.Second`

---

### 2.5 Fonksiyon ve method isimleri

- Fiil + nesne: `CreateUser`, `GetByID`, `ListProducts`
- Boolean dönen fonksiyonlar: `IsValid`, `HasAccess`, `CanPublish`
- Side-effect açık olsun: `Save`, `Delete`, `Publish`, `Enqueue`
- Constructor benzeri: `NewXxx(...)`
    - `NewRepository`, `NewService`, `NewHandler`

✅ `ValidateEmail`, `ParseToken`, `BuildQuery`

❌ `DoThing`, `HandleStuff`, `ProcessData` (çok genel)

---

### 2.6 Struct / Type isimleri

- Type isimleri **isim (noun)** olmalı: `User`, `Order`, `Price`
- Interface isimleri:
    - Eğer tek davranış: `Reader`, `Writer`, `Closer` gibi `er`
    - Domain odaklı: `UserRepository`, `OrderService`
- “Manager”, “Helper”, “Util” **anti-pattern** (genelde çöp kutusu olur)

✅ `UserRepository`, `TokenSigner`

❌ `UserManager`, `CommonHelper`

---

### 2.7 Interface isimlendirme standardı (hexagonal/clean uyumlu)

- Port’lar:
    - **Outbound port**: `UserRepository`, `EventPublisher`, `Clock`
    - **Inbound port / usecase**: `CreateUserUseCase`, `LoginUseCase`
- Adapter’lar:
    - `MySQLUserRepository`, `KafkaEventPublisher`, `RedisSessionStore`

✅ `type UserRepository interface { ... }`

✅ `type MySQLUserRepository struct { ... }`

---

### 2.8 Error isimlendirme

- Sentinel error: `var ErrNotFound = errors.New("not found")`
- Error mesajı:
    - küçük harfle başlar
    - nokta ile bitmez
    - “failed to …” zinciriyle wrap edilir

✅ `fmt.Errorf("create user: %w", err)`

❌ `"User not found."`

---

### 2.9 Kısaltmalar (ID/URL/HTTP)

Go standardına yakın git:

- `userID`, `URL`, `HTTPServer`, `JSONEncoder`
- “Id” değil “ID”
- “Uuid” değil “UUID” (ama sahada çoğu ekip `UUID` kullanıyor; standart: `UUID`)

---

## 3) API/HTTP isimlendirme

### 3.1 URL path

- **kebab-case**, çoğul resource:
    - `/v1/users`, `/v1/order-items`
- ID param: `/v1/users/{userId}` veya `{id}` (ekip standardı seç, tutarlı ol)

### 3.2 Query param

- **snake_case** veya **kebab-case**; backend’de en yaygın: `snake_case`
    - `?page=1&page_size=50&sort_by=created_at`

---

## 4) Veritabanı isimlendirme (DB / Table / Column)

### 4.1 Veritabanı adı

- `snake_case`, environment suffix:
    - `shop_prod`, `shop_stage`, `shop_dev`

### 4.2 Tablo adı

- `snake_case`:
    - Tekil (`user`, `order`, `order_item`)
- Join table:
    - `user_role`, `product_category`

### 4.3 Kolon adı

- `snake_case`
- zaman: `created_at`, `updated_at`, `deleted_at`
- boolean: `is_active`, `has_access`, `can_refund`
- foreign key: `{entity}_id` → `user_id`, `order_id`

### 4.4 Index / constraint

- index: `idx_<table>_<cols>` → `idx_orders_user_id_created_at`
- unique: `uq_<table>_<cols>`
- fk: `fk_<from>_<to>`

---

## 5) Messaging / Kafka / Event isimlendirme

### 5.1 Topic

- `kebab-case` veya `snake_case`; tercihimiz: **kebab-case**
- domain + aggregate + event-type:
    - `product.price-updated`
    - `order.created`
    - `inventory.stock-decreased`

### 5.2 Event schema field

- JSON field’lar: `snake_case`
    - `event_id`, `aggregate_id`, `aggregate_version`, `occurred_at`

### 5.3 Consumer group

- `<service>.<purpose>`:
    - `search-service.product-sync`
    - `basket-service.order-events`

---

## 6) Config / Env / Flags

- ENV: `SCREAMING_SNAKE_CASE`
    - `DB_HOST`, `DB_PORT`, `KAFKA_BROKERS`, `REDIS_URL`
- YAML/JSON config: `snake_case`

---

## 7) Log / Metric / Trace isimleri

- Log field: `snake_case`
    - `request_id`, `user_id`, `duration_ms`
- Metric: `snake_case` ya da Prometheus standardına yakın:
    - `http_requests_total`, `http_request_duration_seconds`
- Trace span: fiil + nesne:
    - `db.query`, `kafka.publish`, `http.client`

---

## 8) Kısa “Do / Don’t” listesi

### Do

- `userID`, `createdAt`, `MaxRetries`
- `NewUserService`, `MySQLUserRepository`
- `users`, `order_items`, `created_at`
- `/v1/order-items`

### Don’t

- `User_manager`, `doStuff`, `data2`
- `Id`, `Uuid`
- `tblUsers`, `colCreatedAt` (DB’de prefix’ler gereksiz)
- karma case kullanmak (aynı projede hem `userId` hem `user_id`)

---

## 9) Ekip için “karar kilitleri”

1. DB table: tekil mi
    
    Bu bir standart değil tercihtir. Bizim kriterimizi seçen ekipler;
    
    - DDD odaklı ekipler
    - Domain model’e yakın tasarım
    - Strong schema governance olan ekipler
    - Enterprise legacy sistemler
    
    Çoğul seçen ekipler;
    
    - REST ile doğal uyum (endpoint)
    - Koleksiyon mantığına uygunluk (NoSQL)
    - ORM varsayılanları (ORM Kullanmıyoruz)
2. URL ID param: `{id}`
3. Query param case: `snake_case`
4. Topic format: `domain.aggregate.event`