# Validasyon Sorumlulukları ve Kuralları

## 1) Validation ikiye ayrılır

### A) **Input validation** (request/doğrulama)

**Nerede?** Usecase (ports/in) veya handler boundary

**Nedir?**

- `page >= 1` mi?
- `sort` whitelist’te mi?
- `minPrice <= maxPrice` mı?
- zorunlu alan gelmiş mi?
- format doğru mu? (UUID, email formatı vs.)

**Neden domain’de değil?**

Bunlar *iş kuralı* değil, *istek/taşıma katmanı* kurallarıdır.

> Kural: “Request şekli” validation’ı domain’e girmez.
> 

---

### B) **Business rule validation** (invariant / iş kuralı)

**Nerede?** Domain (entity/value object)

**Nedir?**

- Fiyat negatif olamaz
- İndirimli fiyat list fiyattan büyük olamaz
- Ürün yayınlanmadan stoğu değiştirilemez
- Durum geçişleri (Draft → Published) gibi kurallar

**Neden domain’de?**

Çünkü bu kurallar **hangi adapter’dan gelirse gelsin** (HTTP, Kafka, cron job) aynı kalmalı ve tek yerde korunmalı.

> Kural: “Her koşulda doğru kalması gereken şey” domain invariant’tır.
> 

## 2) Domain’de validation’ı nasıl yapacağım?

### En doğru yöntem: “constructor + method guard”

### Value Object örneği (Email gibi)

- Format kontrolü burada olur çünkü bu iş kavramıdır.

### Entity örneği (Product gibi)

- State değiştirirken guard koyarsın.

**Pattern:**

- `NewX(...)` → invariant kontrol eder
- `X.ChangeY(...)` → invariant kontrol eder, sonra state değiştirir

---

## 3) Kısa pseudo-örnek (genel)

### Value Object

- `NewMoney(amount)` → `amount >= 0` değilse error

### Entity

- `ChangePrice(newPrice)` → `newPrice >= 0` değilse error

Bu “validation” domain’de.

Ama:

- “HTTP’de price alanı gelmiş mi?” → usecase/handler
- “minPrice/maxPrice query param mantıklı mı?” → usecase

---

## 4) Listeleme / arama tarafında domain validation olur mu?

Genelde **olmaz**. Çünkü listeleme:

- state değiştirmez
- invariant uygulamaz
- daha çok “query param sanity check” ister → usecase

Yani mesela “listeleme/arama” usecase’lerinde domain entity hiç kullanılmayabilir; bu normal.

---

## 5) Net kurallar (ekip için)

1. **Request validation** = usecase/handler
2. **Business rule validation (invariant)** = domain
3. Domain validation sadece **state değişiminde** çalışır
4. Query (read-side) genelde domain’e girmez
5. “Her koşulda doğru kalmalı” dediğin her kural domain’dedir

Aşağıda tek bir bounded context üzerinden (örnek: **Catalog / Product**) **validation’ın nerede nasıl yapılacağı** kodla gösterilmiştir. Buradaki en önemli ayrım:

- **Input validation (request/doğrulama)** → *Usecase/Handler*
- **Business rule validation (invariant / iş kuralı)** → *Domain (Entity/Value Object)*

---

## 0) Örnek klasör (hexagonal/clean)

```go
core/catalog/domain/...
core/catalog/ports/in/...
core/catalog/ports/out/...
core/catalog/usecase/...

adapters/http/catalog/...
adapters/mysql/catalog/...
adapters/elasticsearch/catalog/...
```

---

# 1) Domain: Error’lar (iş kuralı ihlali)

`core/catalog/domain/errors.go`

```go
packagedomain

import"errors"

// Domain-level (business) errors
var (
ErrInvalidSKU=errors.New("invalid sku")
ErrInvalidName=errors.New("invalid name")
ErrInvalidMoney=errors.New("invalid money")
ErrInvalidPrice=errors.New("invalid price")
ErrInvalidListPrice=errors.New("invalid list price")
ErrInvalidStateChange=errors.New("invalid state change")
)
```

> Not: Bunlar **input format** hatası değil, **iş kuralı** hatası. (Örn: fiyat negatif olamaz.)
> 

---

# 2) Domain: Value Object (Money) — validation burada

`core/catalog/domain/money.go`

```go
packagedomain

// Money: örnek olarak float yerine minor unit (kuruş) ile tutuyoruz.
// Bu hem financial hesaplarda güvenli hem de invariant için ideal.
typeMoneystruct {
	amountint64// minor units (e.g., kuruş)
	currencystring
}

funcNewMoney(amountint64,currencystring) (Money,error) {
ifcurrency=="" {
returnMoney{},ErrInvalidMoney
	}
ifamount<0 {
returnMoney{},ErrInvalidMoney
	}
returnMoney{amount:amount,currency:currency},nil
}

func (mMoney) Amount()int64     {returnm.amount }
func (mMoney) Currency()string {returnm.currency }
```

**Buradaki validation domain’dedir** çünkü:

- “para negatif olamaz” **her yerde** geçerli bir iş kuralıdır.

---

# 3) Domain: Value Object (SKU) — format/invariant burada

`core/catalog/domain/sku.go`

```go
packagedomain

import"unicode"

typeSKUstring

funcNewSKU(vstring) (SKU,error) {
// örnek kural: 6-32 char, alfanümerik + '-' + '_'
iflen(v)<6||len(v)>32 {
return"",ErrInvalidSKU
	}
for_,r:=rangev {
ifunicode.IsLetter(r)||unicode.IsDigit(r)||r=='-'||r=='_' {
continue
		}
return"",ErrInvalidSKU
	}
returnSKU(v),nil
}

func (sSKU) String()string {returnstring(s) }
```

Bu da domain’de çünkü SKU iş kavramı; HTTP’den mi geldi, Kafka’dan mı geldi fark etmez.

---

# 4) Domain: Entity (Product) — state change + business validation burada

`core/catalog/domain/product.go`

```go
packagedomain

typeProductStatusstring

const (
StatusDraftProductStatus="draft"
StatusPublishedProductStatus="published"
StatusArchivedProductStatus="archived"
)

typeProductstruct {
	idint64
	skuSKU
	namestring
	statusProductStatus
	priceMoney
	listPriceMoney
}

typeNewProductParamsstruct {
	IDint64
	SKUstring
	Namestring
	Priceint64// minor units
	ListPriceint64// minor units
	Currencystring
}

funcNewProduct(pNewProductParams) (*Product,error) {
ifp.ID<=0 {
returnnil,ErrInvalidStateChange// örnek: id kuralı domain dışı da olabilir, burada gösterim
	}

sku,err:=NewSKU(p.SKU)
iferr!=nil {
returnnil,err
	}

iflen(p.Name)<2||len(p.Name)>200 {
returnnil,ErrInvalidName
	}

price,err:=NewMoney(p.Price,p.Currency)
iferr!=nil {
returnnil,ErrInvalidPrice
	}

listPrice,err:=NewMoney(p.ListPrice,p.Currency)
iferr!=nil {
returnnil,ErrInvalidListPrice
	}

// Business rule: listPrice >= price
iflistPrice.Amount()<price.Amount() {
returnnil,ErrInvalidListPrice
	}

return&Product{
id:p.ID,
sku:sku,
name:p.Name,
status:StatusDraft,
price:price,
listPrice:listPrice,
	},nil
}

// --- Read-only getters (domain dışına kontrollü aç)
func (p*Product) ID()int64              {returnp.id }
func (p*Product) SKU()string            {returnp.sku.String() }
func (p*Product) Name()string           {returnp.name }
func (p*Product) Status()ProductStatus  {returnp.status }
func (p*Product) Price()Money           {returnp.price }
func (p*Product) ListPrice()Money       {returnp.listPrice }

// --- State change methods: validation burada
func (p*Product) Rename(newNamestring)error {
iflen(newName)<2||len(newName)>200 {
returnErrInvalidName
	}
ifp.status==StatusArchived {
returnErrInvalidStateChange
	}
p.name=newName
returnnil
}

func (p*Product) ChangePrice(newPriceMoney)error {
// Currency mismatch check (iş kuralı)
ifnewPrice.Currency()!=p.price.Currency() {
returnErrInvalidPrice
	}
// listPrice >= price kuralını koru
ifp.listPrice.Amount()<newPrice.Amount() {
returnErrInvalidPrice
	}
ifp.status==StatusArchived {
returnErrInvalidStateChange
	}
p.price=newPrice
returnnil
}

func (p*Product) Publish()error {
ifp.status!=StatusDraft {
returnErrInvalidStateChange
	}
// örnek invariant: publish için fiyat > 0
ifp.price.Amount()<=0 {
returnErrInvalidStateChange
	}
p.status=StatusPublished
returnnil
}

func (p*Product) Archive()error {
// örnek: published veya draft fark etmez; archived’a geçer
ifp.status==StatusArchived {
returnErrInvalidStateChange
	}
p.status=StatusArchived
returnnil
}
```

Burada “validation” dediğimiz şey:

- `Publish()` ön koşulları
- `ChangePrice()` invariants
- `Rename()` state kuralları
    
    Bunların hepsi **domain validation**.
    

---

# 5) Ports/In: Usecase kontratı + input/output DTO (request validation burada değil, “shape” burada)

`core/catalog/ports/in/update_price.go`

```go
packagein

import"context"

typeUpdateProductPriceCommandstruct {
	ProductIDint64
	Priceint64// minor units
	Currencystring
}

typeUpdateProductPriceResultstruct {
	ProductIDint64
}

typeUpdateProductPriceinterface {
	Execute(ctxcontext.Context,cmdUpdateProductPriceCommand) (UpdateProductPriceResult,error)
}
```

---

# 6) Ports/Out: Repository kontratı (rol bazlı)

`core/catalog/ports/out/product_repository.go`

```go
packageout

import (
"context"
"yourapp/core/catalog/domain"
)

typeProductRepositoryinterface {
	FindByID(ctxcontext.Context,idint64) (*domain.Product,error)
	Save(ctxcontext.Context,p*domain.Product)error
}
```

> Burada domain entity ile çalışmak **write-side** için ideal.
> 

---

# 7) Usecase: Input validation + domain validation çağırma

`core/catalog/usecase/update_product_price.go`

```go
packageusecase

import (
"context"
"errors"

"yourapp/core/catalog/domain"
"yourapp/core/catalog/ports/in"
"yourapp/core/catalog/ports/out"
)

var (
// Usecase-level validation errors (request/sanity)
ErrInvalidCommand=errors.New("invalid command")
ErrNotFound=errors.New("not found")
)

typeupdateProductPricestruct {
	repoout.ProductRepository
}

funcNewUpdateProductPrice(repoout.ProductRepository)in.UpdateProductPrice {
return&updateProductPrice{repo:repo}
}

func (uc*updateProductPrice) Execute(ctxcontext.Context,cmdin.UpdateProductPriceCommand) (in.UpdateProductPriceResult,error) {
// A) INPUT VALIDATION (request/sanity) -> usecase
ifcmd.ProductID<=0 {
returnin.UpdateProductPriceResult{},ErrInvalidCommand
	}
ifcmd.Currency=="" {
returnin.UpdateProductPriceResult{},ErrInvalidCommand
	}
ifcmd.Price<0 {
// bu “business rule” gibi görünse de request sanity olarak burada da yakalanabilir
// yine de asıl garanti domain Money’de
returnin.UpdateProductPriceResult{},ErrInvalidCommand
	}

p,err:=uc.repo.FindByID(ctx,cmd.ProductID)
iferr!=nil {
returnin.UpdateProductPriceResult{},err
	}
ifp==nil {
returnin.UpdateProductPriceResult{},ErrNotFound
	}

// B) DOMAIN VALIDATION -> entity/value object method’ları
newPrice,err:=domain.NewMoney(cmd.Price,cmd.Currency)
iferr!=nil {
returnin.UpdateProductPriceResult{},err// domain error
	}

iferr:=p.ChangePrice(newPrice);err!=nil {
returnin.UpdateProductPriceResult{},err// domain error
	}

iferr:=uc.repo.Save(ctx,p);err!=nil {
returnin.UpdateProductPriceResult{},err
	}

returnin.UpdateProductPriceResult{ProductID:p.ID()},nil
}
```

**Buradaki mantık:**

- Usecase: “komut doğru mu?” (shape/sanity)
- Domain: “iş kuralı korunuyor mu?” (invariant)

---

# 8) HTTP Handler: parse + status map (domain/usecase error map)

`adapters/http/catalog/handler.go`

```go
packagehttpcatalog

import (
"errors"
"strconv"

"github.com/gofiber/fiber/v3"
"yourapp/core/catalog/domain"
"yourapp/core/catalog/ports/in"
"yourapp/core/catalog/usecase"
)

typeProductHandlerstruct {
	updatePriceUCin.UpdateProductPrice
}

funcNewProductHandler(updatePriceUCin.UpdateProductPrice)*ProductHandler {
return&ProductHandler{updatePriceUC:updatePriceUC}
}

func (h*ProductHandler) UpdatePrice(cfiber.Ctx)error {
idStr:=c.Params("id")
id,err:=strconv.ParseInt(idStr,10,64)
iferr!=nil {
returnc.Status(fiber.StatusBadRequest).JSON(fiber.Map{"error":"invalid id"})
	}

varbodystruct {
		Priceint64`json:"price"`
		Currencystring`json:"currency"`
	}
iferr:=c.Bind().Body(&body);err!=nil {
returnc.Status(fiber.StatusBadRequest).JSON(fiber.Map{"error":"invalid body"})
	}

res,err:=h.updatePriceUC.Execute(c.Context(),in.UpdateProductPriceCommand{
ProductID:id,
Price:body.Price,
Currency:body.Currency,
	})
iferr!=nil {
// usecase-level errors
iferrors.Is(err,usecase.ErrInvalidCommand) {
returnc.Status(fiber.StatusBadRequest).JSON(fiber.Map{"error":"invalid command"})
		}
iferrors.Is(err,usecase.ErrNotFound) {
returnc.Status(fiber.StatusNotFound).JSON(fiber.Map{"error":"not found"})
		}

// domain errors -> 422 Unprocessable Entity sık kullanılır
switch {
caseerrors.Is(err,domain.ErrInvalidPrice),
errors.Is(err,domain.ErrInvalidMoney),
errors.Is(err,domain.ErrInvalidStateChange),
errors.Is(err,domain.ErrInvalidListPrice),
errors.Is(err,domain.ErrInvalidName),
errors.Is(err,domain.ErrInvalidSKU):
returnc.Status(fiber.StatusUnprocessableEntity).JSON(fiber.Map{"error":err.Error()})
		}

returnc.Status(fiber.StatusInternalServerError).JSON(fiber.Map{"error":"internal"})
	}

returnc.JSON(res)
}
```

Handler’ın işi:

- parse (id/body)
- usecase çağır
- error mapping (400/404/422/500)

---

# 9) “Domain sadece create/update/delete’de mi kullanılır?” net cevap

**Domain’i en doğru şekilde:**

- **State değişimi** olan her yerde kullanırsın (create/update/delete/publish/price change/rename…).
- **Query/read-side** tarafında çoğu zaman domain entity kullanmazsın. (Projection/DTO ile gidersin.)

“Domain method’larında validation” dediğimiz şey de zaten **state change method guard**’larıdır.

---

## Tek cümlelik kural

- **Usecase**: “Gelen istek mantıklı mı?”
- **Domain**: “İş kuralları her zaman doğru mu kalıyor?”