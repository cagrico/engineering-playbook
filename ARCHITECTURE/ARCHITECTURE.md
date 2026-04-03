# 🏗️ Mimarî Yaklaşım

Bu doküman, ekip olarak geliştireceğimiz veya refactor edeceğimiz tüm backend projelerde uygulanması hedeflenen genel mimarî yaklaşımı açıklamak için hazırlanmıştır.

Amaç yalnızca klasör yapısı belirlemek değildir. Amaç, ekip içinde ortak bir mühendislik dili oluşturmak, iş mantığını dış bağımlılıklardan ayırmak, sistemleri daha test edilebilir hale getirmek ve uzun vadede bakım maliyetini düşürmektir.

---

# 🎯 Hedef

Ekip olarak kurmak istediğimiz yapı şunları sağlamalıdır:

- İş mantığı ile framework kodu birbirine karışmamalı
- HTTP, veritabanı, queue, cache gibi teknolojiler değiştiğinde core iş mantığı bozulmamalı
- Her bounded context kendi sorumluluğu içinde kalmalı
- Test yazmak kolay olmalı
- Event-driven yapılar güvenli biçimde kurulabilmeli
- Yeni başlayan bir geliştirici kodun nereye ait olduğunu hızlıca anlayabilmeli

---

# 📌 Genel Tanım

Ekip standardı olarak benimsediğimiz yaklaşım:

> **Hexagonal Architecture (Port-Adapter) tabanlı**,  
> **Clean Architecture bağımlılık kurallarını uygulayan**,  
> ve **Domain-Driven Design (DDD) yaklaşımının tactical pattern’lerinden yararlanan hibrit bir mimarîdir.**

Bu ifade önemlidir çünkü burada tek bir mimarî şablon birebir uygulanmamaktadır. Bunun yerine, pratikte birlikte iyi çalışan üç yaklaşımın güçlü tarafları bir araya getirilmektedir.

---

# 🧠 Temel Yaklaşımlar

## 1. Hexagonal Architecture (Port-Adapter)

Hexagonal Architecture’ın temel fikri şudur:

> Sistemin merkezinde iş mantığı bulunur.  
> Dış dünya ile iletişim port ve adapter yapıları üzerinden kurulur.

### Basit anlatım

- Core, sistemin kalbidir
- Adapter’lar dış dünya ile bağlantı kurar
- Port’lar ise bu bağlantının sözleşmesidir

### Örnek dış dünya bağımlılıkları

- HTTP / REST
- MySQL / PostgreSQL / Elasticsearch
- Kafka / RabbitMQ
- Redis
- S3 / dosya sistemi
- üçüncü parti servisler

### Temel kural

Core katmanı, dış teknolojileri doğrudan bilmez.  
Dış dünya, core’a port ve adapter üzerinden bağlanır.

---

## 2. Clean Architecture

Clean Architecture’ın en önemli kuralı şudur:

> Bağımlılık yönü her zaman dıştan içe doğrudur.

Yani:

    Adapters → Core
    Core → Adapters ❌

Bu ne anlama gelir?

- Handler, usecase’i bilir
- Usecase, repository interface’ini bilir
- Repository implementasyonu MySQL’i bilir
- Ama usecase, MySQL’i bilmez
- Domain, HTTP request modelini bilmez

Bu sayede iş mantığı framework bağımlı hale gelmez.

---

## 3. Domain-Driven Design (DDD) – Tactical Yaklaşım

DDD burada “her projeye tam kapsamlı stratejik DDD uygula” anlamında kullanılmamaktadır.

Burada esas aldığımız taraf, DDD’nin günlük kod yazımına en çok katkı veren tactical pattern’leridir.

### Ekip olarak benimsediğimiz DDD öğeleri

- Bounded Context
- Entity
- Repository
- Usecase / Application Layer
- Domain validation
- Açık iş dili ve doğru isimlendirme

### Ne amaçlıyoruz?

İş mantığını doğru yerde tutmak ve teknik detaylarla karıştırmamak.

---

# 🧩 Neden Hibrit Bir Yaklaşım Kullanıyoruz?

Çünkü gerçek projelerde bu kavramlar birbirinin alternatifi değil, çoğu zaman tamamlayıcısıdır.

- **Hexagonal**, sistemin dış dünya ile nasıl bağlanacağını iyi anlatır
- **Clean**, bağımlılık yönünü ve katman disiplinini iyi korur
- **DDD**, iş alanını doğru modellemeye yardımcı olur

Bu yüzden ekip standardı olarak tek bir etikete sıkışmak yerine, bunların birlikte çalıştığı bir yaklaşım benimsenir.

---

# 🏛️ Ana Tasarım Prensipleri

## 1. İş mantığı merkezde olmalı

Framework kodu, SQL sorgusu, HTTP request modeli veya üçüncü parti kütüphane detayları iş mantığının içine girmemelidir.

## 2. Dış bağımlılıklar içeri sızmamalı

Örneğin:

- `multipart.FileHeader`
- `fiber.Ctx`
- `gorm.Model`
- `sql.Tx`
- `kafka.Message`

gibi tipler doğrudan domain veya usecase katmanına taşınmamalıdır.

## 3. Her bounded context kendi sınırını korumalı

Bir bounded context başka bir context’in iç detaylarına değil, yalnızca gerekirse kontrollü sözleşmelerine temas etmelidir.

## 4. Aynı model her yerde kullanılmamalı

Tek bir struct’ı request, response, domain entity, database row ve event payload olarak kullanmak yasaktır.

## 5. İş mantığı ve sunum mantığı ayrılmalı

Örneğin bir `state` veya `displayText` alanı çoğu zaman domain değil, presentation concern’dür.

---

# 🧱 Katmanlar ve Sorumluluklar

Aşağıdaki yapı ekip standardı olarak düşünülmelidir. Projenin ihtiyacına göre küçük farklar olabilir, ama katman sorumlulukları korunmalıdır.

---

## 1. Domain Katmanı

Domain katmanı iş kurallarının merkezidir.

### Burada neler olur?

- Entity’ler
- Value object’ler
- Domain error’ları
- Domain davranışları
- Invariant kontrolleri

### Burada neler olmaz?

- HTTP request/response
- JSON tag zorunluluğu
- DB tag
- SQL sorguları
- framework bağımlılığı
- queue/client kodu

### Örnek

Bir `Brand` entity’si şunları içerebilir:

- isim boş olamaz
- status belirli değerlerden biri olmalı
- normalize edilmesi gereken alanlar normalize edilir

Bu kurallar domain’e aittir.

---

## 2. Usecase / Application Katmanı

Usecase katmanı, iş akışlarını yönetir.

### Burada neler olur?

- Bir işin baştan sona akışı
- Transaction orchestration
- Repository çağrıları
- Domain object oluşturma veya güncelleme
- Event üretme
- Output hazırlama

### Burada neler olmaz?

- SQL
- HTTP response üretme
- framework spesifik kod
- dosya sistemi veya broker client detayları

### Örnek

`CreateBrand` usecase’i şunları yapabilir:

- context kontrolü
- duplicate kontrolü
- brand oluşturma
- brand’i kaydetme
- outbox event yazma
- çıktı üretme

---

## 3. Ports

Port’lar interface’lerdir. Core katmanın dış dünya ile nasıl konuşacağını tanımlar.

### Inbound Port

Sisteme giriş sözleşmesidir.

Örnek:

- `CreateBrandUseCase`
- `DeleteBrandUseCase`

### Outbound Port

Sistemin dışarıdan ihtiyaç duyduğu şeylerin sözleşmesidir.

Örnek:

- `BrandRepository`
- `OutboxRepository`
- `TxManager`
- `Clock`
- `IDGenerator`

### Amaç

Usecase, doğrudan MySQL veya Kafka kullanmak yerine interface’lerle çalışır.

---

## 4. Adapters

Adapter’lar dış dünya ile çalışan gerçek implementasyonlardır.

### HTTP Adapter

- request parse eder
- validation yapar
- request → input dönüşümü yapar
- usecase çağırır
- output → response dönüşümü yapar

### Database Adapter

- SQL çalıştırır
- row map eder
- repository interface’ini implemente eder

### Messaging Adapter

- Kafka/RabbitMQ publish eder
- outbox worker tarafında kullanılır

---

# 📦 Model Türleri ve Ayrım Kuralları

Ekip içinde en çok karışan konu budur. Bu yüzden açık kural koymak gerekir.

---

## 1. Request Model

HTTP veya başka bir transport katmanından gelen veridir.

### Özellikler

- JSON/form/query tag içerebilir
- dış dünyaya aittir
- adapter katmanında yaşar

### Örnek

`CreateBrandRequest`

---

## 2. Input DTO

Usecase’e verilen application modelidir.

### Özellikler

- transport bağımlılığı yoktur
- JSON tag şart değildir
- core tarafında yaşar

### Örnek

`CreateBrandInput`

---

## 3. Domain Entity

İşin gerçek modelidir.

### Özellikler

- business rule içerir
- sade olmalıdır
- dış dünya bağımlılığı taşımaz

### Örnek

`Brand`

---

## 4. Output DTO

Usecase’in ürettiği application çıktısıdır.

### Özellikler

- response ile aynı olmak zorunda değildir
- presentation logic içermez
- core tarafında yaşar

### Örnek

`CreateBrandOutput`

---

## 5. Response Model

HTTP veya başka bir transport katmanına dönen veridir.

### Özellikler

- JSON tag içerebilir
- presentation alanları eklenebilir
- adapter katmanında yaşar

### Örnek

`CreateBrandResponse`

---

## Önemli kural

Aşağıdaki modeller ayrı düşünülmelidir:

- Request
- Input
- Entity
- Output
- Response

İlk aşamada alanları birebir aynı görünse bile ayrı tutulmalıdır.

Sebep:
- bağımlılık yönünü korumak
- future-proof olmak
- transport detayı ile domain’i karıştırmamak

---

# 🔁 Mapper Nedir, Neden Gerekir?

Mapper, bir modeli başka bir modele dönüştüren kod parçasıdır.

### Örnek dönüşümler

- Request → Input
- Output → Response
- Domain → Event payload
- DB row → Domain

### Neden gerekir?

Çünkü her katmanın kullandığı model farklı amaç taşır.

### Kural

Mapper yazmak gereksiz tekrar gibi görünse de uzun vadede bağımlılık sızıntısını önler.

---

# ✅ Validation Nerede Yapılmalı?

Validation tek katmanda yapılmaz. Katmanlara göre ayrılır.

---

## 1. Adapter Seviyesi Validation

Burada:

- request parse edilebilir mi
- zorunlu alan var mı
- basic format doğru mu

Bu seviye transport validation’dır.

---

## 2. Usecase Seviyesi Validation

Burada:

- duplicate var mı
- iş akışı uygun mu
- gerekli repository kontrolleri sağlandı mı

Bu seviye application validation’dır.

---

## 3. Domain Seviyesi Validation

Burada:

- entity invalid state’e giriyor mu
- invariant’lar korunuyor mu

Bu seviye en kritik validation’dır.

---

# 🔐 Transaction Yönetimi

Transaction, birden fazla işlemi tek bir bütün gibi ele almak için kullanılır.

### Kural

Bir usecase içinde birbirine bağlı iki veya daha fazla write işlemi varsa, transaction ciddi şekilde değerlendirilmelidir.

### Örnek

Brand oluştururken:

1. `brand` tablosuna insert
2. `outbox_events` tablosuna insert

Bu iki işlem birbirine bağlıdır.  
Biri başarılı, diğeri başarısız olursa veri tutarsızlığı oluşur.

Bu yüzden ikisi aynı transaction içinde yapılmalıdır.

---

# 📬 Outbox Pattern Nedir?

Outbox pattern, event-driven sistemlerde veri tutarlılığını korumak için kullanılır.

### Problem

Aşağıdaki yaklaşım risklidir:

    DB insert → Kafka publish

Eğer DB başarılı ama Kafka başarısız olursa sistemler arası tutarsızlık oluşur.

---

## Çözüm

Transaction içinde:

1. business data yazılır
2. event outbox tablosuna yazılır

Sonra ayrı bir worker:

- outbox tablosunu okur
- event’i Kafka’ya publish eder
- durumu günceller

### Avantajlar

- event kaybı azalır
- retry yapılabilir
- at-least-once delivery kontrol altına alınır
- veri ve event üretimi arasında güvenli köprü kurulur

---

# 🧭 Bounded Context Yaklaşımı

Her bounded context mümkün olduğunca kendi içinde bütünleşik olmalıdır.

### Örnek context’ler

- brand
- category
- order
- product
- catalog

### Amaç

Bir context’in iş mantığı, başka bir context’in iç modeline doğrudan yaslanmamalıdır.

---

# 📁 Önerilen Genel Yapı

Proje ihtiyaçlarına göre isimler farklı olabilir, ama prensip şu olmalıdır:

    internal/
      shared/
        event/
        ports/
        adapters/

      brand/
        core/
          domain/
          ports/
            in/
            out/
          usecase/
          eventmapper/
        adapters/
          http/
          mysql/

      category/
        core/
        adapters/

      order/
        core/
        adapters/

Buradaki mantık:

- context’e özel şeyler context içinde
- gerçekten ortak olanlar shared içinde

---

# 🚫 Kaçınılması Gereken Hatalar

## 1. Tek model her yerde kullanmak

Aynı struct’ı request, response, entity, DB row ve event payload yapmak.

## 2. Domain içine framework tipi sokmak

Örnek:
- `fiber.Ctx`
- `multipart.FileHeader`
- `sql.Tx`

## 3. Usecase içinde doğrudan Kafka publish yapmak

Bu, outbox pattern’in mantığını bozar.

## 4. Presentation logic’i core’a koymak

Örnek:
- `state = "active"` gibi alanları usecase veya domain’e gömmek

## 5. Shared klasörünü çöp kutusu gibi kullanmak

Sadece gerçekten cross-context reusable olan şeyler shared’e konmalıdır.

---

# 🧪 Test Edilebilirlik Neden Artar?

Bu mimari sayesinde:

- usecase testlerinde DB’ye ihtiyaç kalmaz
- port’lar mock’lanabilir
- handler testleri usecase’ten bağımsız yapılabilir
- repository testleri ayrı yazılabilir

Bu da ekip hızını artırır.

---

# 📌 Bu Mimarîyi Uygularken Altın Kurallar

1. İş mantığını merkeze koy
2. Dış bağımlılığı adapter’a it
3. Interface’i ihtiyaca göre tanımla
4. Her modeli kendi bağlamında tut
5. Transaction sınırını bilinçli çiz
6. Event üretimini outbox ile güvenli hale getir
7. Shared’i yalnızca gerçekten ortak olan şeyler için kullan
8. Bounded context sınırlarını koru
9. Kodun “çalışması” kadar “taşınabilir ve sürdürülebilir olması”na da odaklan
10. Mimariyi dogma gibi değil, disiplinli bir rehber gibi kullan

---

# 🧾 Sonuç

Ekip standardı olarak benimsediğimiz bu yaklaşım:

- **Hexagonal Architecture** ile dış dünya entegrasyonlarını kontrol eder
- **Clean Architecture** ile bağımlılık yönünü korur
- **DDD tactical pattern’leri** ile iş mantığını daha doğru modeller

Bu nedenle bu yaklaşımı şu şekilde özetleyebiliriz:

> **Hexagonal tabanlı, Clean prensipleriyle uygulanan, DDD etkilenimli hibrit backend mimarisi**

Bu mimari, ekip olarak geliştireceğimiz yeni projelerde ve refactor edilecek mevcut projelerde temel yönlendirici standart olarak kullanılmalıdır.