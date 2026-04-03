# Architecture Guidelines
## Hexagonal / Clean Architecture – General Rules

Bu doküman, projede katmanların sorumluluklarını ve model kullanım kurallarını netleştirmek için hazırlanmıştır.

Amaç:
- Katmanların birbirine karışmasını engellemek
- Domain modelini korumak
- Uzun vadede sürdürülebilir bir yapı oluşturmak

---

# 1. Domain Layer

## Amaç
İş modelini ve iş kurallarını tanımlar.

## İçine Konur
- Entity
- Value Object
- Domain method’ları (state change, validation)
- Domain-level error’lar

## İçine Konmaz
- HTTP
- JSON tag
- Pagination
- Query string
- Database bilgisi
- Elasticsearch bilgisi
- Framework bağımlılığı

## Kural
> Domain hiçbir dış teknoloji bilmez.

---

# 2. Ports/In (Usecase Contract)

## Amaç
Core’un dış dünyadan beklediği input ve dış dünyaya verdiği output kontratıdır.

## İçine Konur
- Usecase interface
- Input DTO (Query / Command)
- Output DTO
- Usecase seviyesinde error’lar

## İçine Konmaz
- Repository implementasyonu
- HTTP parse
- SQL/ES query
- Framework bağımlılığı

## Kural
> Ports/In = Core’a gelen istek ve Core’un döneceği cevap şekli.

---

# 3. Ports/Out (External Dependency Contract)

## Amaç
Core’un dış sistemlerden (DB, Cache, Search, Messaging vb.) ne istediğini tanımlar.

## İçine Konur
- Repository interface
- External sistemlere gidecek filter/model struct’ları
- External sistemlerden dönecek projection struct’ları

## İçine Konmaz
- JSON tag
- HTTP response modeli
- SQL/ES DSL
- Framework bağımlılığı

## Kural
> Ports/Out = Core’un dış sistemden beklediği şey.

---

# 4. Usecase Layer

## Amaç
İş akışını yürütür.

## Sorumlulukları
- Input doğrulama
- Default değer atama
- İş kurallarını çalıştırma
- Ports/Out çağrısı
- Mapping (projection → output DTO)
- Pagination / hesaplama

## Yapmaz
- HTTP parse
- JSON encode
- SQL yazma
- ES DSL yazma

## Kural
> Usecase = Orkestra şefi.

---

# 5. Adapter Layer

## HTTP Adapter
- Request parse eder
- Usecase çağırır
- HTTP status mapping yapar
- JSON response döner

## Database Adapter
- SQL üretir
- DB response parse eder
- Ports/Out projection üretir

## Search Adapter
- DSL üretir
- Response parse eder
- Ports/Out projection üretir

## Kural
> Adapter = Çevirmen.

---

# 6. Model Kullanım Kuralları

## Domain Model
- İş modeli taşır
- Write-side (create/update/delete) işlemlerde aktif kullanılır
- Davranış içerir

---

## Input DTO (Query / Command)
- Ports/In’da tanımlanır
- HTTP’den gelen veri
- İş kuralı değildir

---

## Output DTO
- Ports/In’da tanımlanır
- HTTP’ye dönecek modeldir
- Domain olmak zorunda değildir

---

## Projection Model
- Ports/Out’da tanımlanır
- External sistemden gelen veri şeklidir
- Domain olmak zorunda değildir

---

# 7. JSON Tag Kuralı

- Domain’de olmaz
- Ports/Out’da olmaz
- Sadece transport boundary’de (genellikle HTTP response veya adapter-specific struct)

---

# 8. Error Yönetimi

- Domain error → Domain katmanında
- Usecase validation error → Ports/In
- HTTP status mapping → HTTP adapter

---

# 9. Mapping Kuralları

Mapping yapılabilecek yerler:

- Adapter (external response → ports/out projection)
- Usecase (projection → output DTO)

Mapping asla Domain katmanında yapılmaz.

---

# 10. Bağımlılık Yönü

Bağımlılık akışı şu yönde olmalıdır:

Domain  
↑  
Usecase  
↑  
Ports  
↑  
Adapters

Ters bağımlılık yasaktır.

---

# Özet

- Domain = İş modeli
- Usecase = İş akışı
- Ports = Kontrat
- Adapter = Çeviri katmanı
- DTO ≠ Domain  
