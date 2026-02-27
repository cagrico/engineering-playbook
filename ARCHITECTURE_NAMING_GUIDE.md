# 🚀 Go & Hexagonal/Clean Architecture: İsimlendirme Standartları

Bu rehber, projemizdeki kod karmaşasını önlemek, katmanların sorumluluklarını netleştirmek ve "neyin nerede yapıldığını" bir bakışta anlamak için oluşturulmuştur.

---

## 1. Temel Go Prensipleri (Kısa & Öz)

* **Paket Adını Tekrarlamayın:** `product.ProductService` yerine `product.Service` kullanın. Paket ismi zaten bağlamı verir.
* **Getter/Setter Takıntısından Kaçının:** `GetPrice()` yerine `Price()`, `SetPrice()` yerine `UpdatePrice()` veya sadece `SetPrice()` tercih edin.
* **Interface İsimlendirmesi:** Tek metodlu ise `-er` eki (`Reader`, `Writer`). Katman bazlı ise sorumluluk adı (`Repository`, `Usecase`).
* **Kısaltmalar:** `ID`, `HTTP`, `URL`, `JSON` gibi kısaltmalar her zaman büyük yazılır. (`productID` ❌ -> `ProductID` ✅)

---

## 2. Katman Bazlı Metod İsimlendirme Stratejisi

Aynı iş mantığı, katman değiştikçe **niyetine (intent)** göre isim değiştirmelidir.

| Eylem | **Handler** (Giriş) | **Usecase** (İş Mantığı) | **Repository** (Kalıcılık) |
| --- | --- | --- | --- |
| **Veri Listeleme** | `GetOrders` | `ListOrders` | `FindAll` / `Filter` |
| **Tekil Veri** | `GetOrder` | `GetOrderDetails` | `FindByID` |
| **Yeni Kayıt** | `CreateUser` | `RegisterUser` / `SignUp` | `Store` / `Save` |
| **Güncelleme** | `UpdateOrder` | `ProcessPayment` | `Update` / `UpdateStatus` |
| **Silme** | `DeletePost` | `ArchivePost` / `Remove` | `Delete` / `SoftDelete` |

---

## 3. Katman Detayları

### 🌐 Handler (Adapters - Driving)

Dış dünya ile (HTTP, gRPC, CLI) konuşan katmandır.

* **Kural:** Protokol metodlarını veya tetikleyici eylemi yansıtır.
* **Örnek:** `CreateProductHandler`, `HandleOrderCreated` (Kafka için), `UploadImage`.

### 🧠 Usecase / Service (Domain Layer)

Uygulamanın kalbidir. **Teknik terim (SQL, JSON, Request) içermez.**

* **Kural:** İş biriminin (Business) dilini kullanır. "Ne yapılıyor?" sorusuna iş mantığıyla cevap verir.
* **Örnek:** `EnrollStudent`, `ApplyDiscount`, `VerifyEmail`.
* *Not:* Eğer sadece ham veri dönüyorsa `List...` veya `Search...` fiilleri uygundur.

### 🗄️ Repository (Adapters - Driven)

Veri tabanı veya Cache ile konuşan katmandır. Bir koleksiyon gibi davranır.

* **Kural:** Veriye erişim biçimini belirtir. Teknik (Postgres, Redis) detay içermez.
* **Örnek:** `FindByID`, `FindAllByStatus`, `Store`, `UpdateBalance`, `Remove`.

### 🔌 External Clients (Outbound Adapters)

Üçüncü parti servislerle (Stripe, AWS, Mailgun) konuşan katmandır.

* **Kural:** "Kiminle" değil, "Ne iş" yapıldığına odaklanır.
* **Doğru:** `paymentProvider.Charge()`, `mailService.Send()`
* **Yanlış:** `stripeClient.PayWithStripe()`, `awsS3.UploadToS3()`

---

## 4. Değişken ve Nesne İsimlendirme

* **Slices/Lists:** `productList` yerine `products`.
* **Maps:** `userMap` yerine `usersByID` veya `usersByEmail`.
* **Booleans:** Soru sormalıdır: `isActive`, `hasPermission`, `canDelete`.
* **Errors:** Paket seviyesinde tanımlanmalı ve `Err` ile başlamalıdır: `ErrNotFound`, `ErrPermissionDenied`.

---

## 5. Örnek Bir Akış (Order Cancellation)

İsimlendirmenin katmanlar arasında nasıl evrildiğine bakın:

1. **Handler:** `CancelOrder` (Kullanıcı butona bastı)
2. **Usecase:** `ValidateAndCancelOrder` (İptal kuralları kontrol ediliyor)
3. **Repository:** `UpdateStatus` (Veri tabanında durum 'cancelled' yapılıyor)
4. **Integration/Event:** `PublishOrderCancelled` (Diğer servislere haber veriliyor)

---

## 🔍 Code Review Kontrol Listesi

1. **Gereksizlik:** `order.OrderRepository` yazıyor mu? (Sadece `order.Repository` olmalı).
2. **Sızıntı:** Usecase içinde `UpdateSQL` gibi teknik bir kelime var mı?
3. **Belirsizlik:** `ProcessData` gibi ne yaptığı belli olmayan jenerik isimler var mı?
4. **Tutarlılık:** Bir yerde `Delete` bir yerde `Remove` mu denmiş? (Birini seçin ve sadık kalın).

---

## 6. Test İsimlendirme Standartları

Testler, kodun ne yapması gerektiğini anlatan bir dokümantasyon görevi görmelidir.

### 🧪 6.1. Birim (Unit) Testler

Birim testleri genellikle Usecase veya Domain katmanında yoğunlaşır.

* **Kural:** `Test[MetodAdi]_[Senaryo]_[BeklenenSonuc]`
* **Örnekler:**
* `TestRegisterUser_ValidInput_Success`
* `TestRegisterUser_DuplicateEmail_ReturnsError`
* `TestApplyDiscount_ExpiredCoupon_NoChange`



### 🏗️ 6.2. Entegrasyon (Integration) Testleri

Repository veya External Adapter gibi dış dünya ile temas eden yerlerde kullanılır.

* **Kural:** `Test[Adapter]_[Aksiyon]`
* **Örnekler:**
* `TestPostgresRepo_FindByID`
* `TestRedisCache_SetAndGet`
* `TestStripeAdapter_Charge_InvalidCard`



### 📋 6.3. Table-Driven Tests (Go Standard)

Go'da testleri bir dizi senaryo (cases) içinde koştururken, her senaryoya bir isim vermek zorunludur.

* **Kural:** Senaryo isimleri küçük harfle başlamalı ve "should" (yapmalı) ifadesini hissettirmelidir.

```go
tests := []struct {
    name    string // Örn: "should return error when stock is empty"
    input   int
    wantErr bool
}{ ... }

```

---

## 7. Mock ve Interface Test İsimlendirmeleri

Hexagonal mimaride bağımlılıkları taklit etmek (mocking) yaygındır.

* **Mock Yapıları:** `MockProductRepository`, `SpyNotificationSender`.
* **Mock Dosyaları:** Genellikle `mock_repository.go` veya `repository_mock.go` şeklinde adlandırılır.