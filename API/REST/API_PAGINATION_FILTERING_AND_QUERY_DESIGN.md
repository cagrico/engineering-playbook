# API Pagination, Filtering & Query Design Guide

Bu doküman, production ortamında çalışan API'lerde pagination, filtering, sorting ve query tasarım kurallarını tanımlar.

Amaç:

- Liste endpoint'lerini öngörülebilir hale getirmek
- Büyük veri kümelerinde kontrollü erişim sağlamak
- Client tarafında tutarlı sorgulama davranışı oluşturmak
- Performans, okunabilirlik ve maintainability dengesini korumak

---

# 1. Temel İlke

Liste endpoint'leri sadece veri dönen endpoint'ler değildir.

Liste endpoint'leri aynı zamanda:

- veri erişim sözleşmesidir
- performans sınırıdır
- cache davranışını etkiler
- veritabanı ve search engine yükünü belirler

Yanlış query tasarımı zamanla şunlara yol açar:

- full scan eğilimi
- aşırı büyük response'lar
- tutarsız client kullanımı
- DB ve search katmanında yük patlaması
- gereksiz karmaşık endpoint yüzeyi

---

# 2. Ana Kural

Path resource'u seçer.

Query string ise o resource üzerinde:

- filtreleme
- sıralama
- sayfalama
- alan seçimi

gibi davranışları ifade eder.

Doğru:

    GET /products?page=1&size=20
    GET /products?category=shoes
    GET /products?sort=created_at_desc

Yanlış:

    GET /get-products-by-category/shoes
    GET /products/page/1/size/20
    POST /products/filter

---

# 3. Pagination Neden Zorunlu?

Liste endpoint'lerinde pagination çoğu durumda zorunludur.

Pagination olmayan liste endpoint'i:

- büyük response üretir
- memory kullanımını artırır
- latency yükseltir
- istemciyi gereksiz veriyle yükler
- DB ve network maliyetini büyütür

Production kuralı:

- collection endpoint'leri varsayılan olarak bounded olmalı
- "tüm veriyi getir" davranışı default olmamalı

---

# 4. Pagination Türleri

## 4.1 Offset Pagination

Örnek:

    GET /products?page=1&size=20

veya

    GET /products?offset=0&limit=20

Avantaj:
- basit
- anlaşılır
- admin panel ve düşük/orta ölçekli listeler için uygun

Dezavantaj:
- büyük offset'lerde pahalı olabilir
- veri akışı değişirken tutarsız sayfa sonuçları üretebilir

---

## 4.2 Cursor Pagination

Örnek:

    GET /products?cursor=eyJpZCI6InByZF8xMjMifQ&size=20

Avantaj:
- büyük veri kümelerinde daha performanslıdır
- sürekli akan veride daha stabildir

Dezavantaj:
- implementasyonu daha karmaşıktır
- admin panel gibi insan odaklı sayfalama için daha az rahattır

---

# 5. Hangi Pagination Ne Zaman Kullanılır?

## Offset pagination kullan:

- admin panel
- backoffice listeleri
- nispeten sınırlı veri setleri
- doğrudan sayfa numarası ihtiyacı

## Cursor pagination kullan:

- feed benzeri akışlar
- yüksek hacimli veri
- sonsuz scroll
- zaman serisi veya append-heavy veri

---

# 6. Ekip Standardı Belirle

Aynı sistem içinde pagination yaklaşımı rastgele karışmamalıdır.

Önerilen standart:

- Backoffice / klasik listeleme → page + size
- Feed / yüksek hacimli akış → cursor + size

---

# 7. Önerilen Offset Pagination Contract'ı

Request:

    GET /products?page=1&size=20

Response:

    HTTP/1.1 200 OK

    {
      "items": [
        {
          "id": "prd_1",
          "name": "Product A"
        }
      ],
      "pagination": {
        "page": 1,
        "size": 20,
        "total_items": 124,
        "total_pages": 7
      }
    }

Kural:
- page 1'den başlamalı
- size bounded olmalı
- total alanları tutarlı olmalı

---

# 8. Max Page Size Zorunluluğu

Client istediği kadar veri çekememelidir.

Yanlış:

    GET /products?size=100000

Doğru yaklaşım:
- default size tanımla
- max size tanımla
- limit aşılırsa clamp et veya validation error dön

Örnek politika:
- default size = 20
- max size = 100

---

# 9. Filtering Kuralları

Filtering query string ile yapılmalıdır.

Örnek:

    GET /products?category=shoes
    GET /products?brand=nike
    GET /orders?status=pending

Her filtre:
- açık olmalı
- tutarlı olmalı
- endpoint'ler arasında benzer davranmalı

---

# 10. Multi-Filter Kullanımı

Birden fazla filtre birlikte kullanılabilmelidir.

Örnek:

    GET /products?category=shoes&brand=nike&status=active

Bu ifade genelde AND semantiği taşır.

Kural:
- default mantık açıkça dokümante edilmeli
- aynı filtre davranışı tüm benzer endpoint'lerde aynı olmalı

---

# 11. Çoklu Değerli Filtreler

Bir filter key birden fazla değer alabilir.

Önerilen yaklaşım:

    GET /products?brand=nike,adidas,puma

Alternatif:

    GET /products?brand=nike&brand=adidas&brand=puma

Ekip içinde tek standart seçilmelidir.

Öneri:
- basit HTTP client uyumu için tekrar eden param yaklaşımı iyi olabilir
- okunabilirlik için comma-separated yaklaşım da uygundur

Ama karışık kullanım yasaklanmalıdır.

---

# 12. Range Filtering

Range filtreleri açık ve öngörülebilir olmalıdır.

Örnek:

    GET /products?min_price=100&max_price=1000
    GET /orders?created_from=2026-03-01T00:00:00Z&created_to=2026-03-31T23:59:59Z

Öneri:
- min/max veya from/to çiftleri kullan
- belirsiz isimlerden kaçın

Yanlış:

    ?price=100-1000
    ?date=last-month

Bu tip yapılar dokümantasyon ve parse karmaşası üretir.

---

# 13. Sorting Standardı

Sorting query string ile yapılmalıdır.

Önerilen:

    GET /products?sort=created_at_desc
    GET /products?sort=price_asc

Alternatif:
- minus prefix yaklaşımı

  GET /products?sort=-created_at

Ekip içinde tek standart seçilmelidir.

Öneri:
- daha açık olduğu için field_direction formatı daha anlaşılırdır

---

# 14. Çoklu Sorting

Bazı endpoint'lerde birden fazla sorting desteklenebilir.

Örnek:

    GET /products?sort=category_asc,price_desc

Bu ihtiyaç gerçekten varsa eklenmelidir.
Default olarak her endpoint'i gereksiz karmaşık hale getirme.

---

# 15. Default Sort Zorunluluğu

Her liste endpoint'inin bir default sort davranışı olmalıdır.

Neden?
- deterministik sonuç
- pagination stabilitesi
- cache davranışı
- tekrar edilebilir response

Örnek:
- created_at_desc
- id_desc
- score_desc

Yanlış yaklaşım:
- veritabanının doğal sırasına güvenmek

---

# 16. Search Query Param Tasarımı

Basit arama davranışı için genelde tek bir param yeterlidir.

Örnek:

    GET /products?query=protein

Kural:
- query param adı stabil olmalı
- bazen q de kullanılabilir ama ekip standardı sabit olmalı

Öneri:
- açıklık için `query`

---

# 17. Field Selection

Client bazı alanları seçebilmelidir.

Örnek:

    GET /products?fields=id,name,price

Bu özellik:
- ağır response'ları azaltabilir
- özellikle internal API'lerde faydalı olabilir

Ama dikkat:
- her endpoint'te zorunlu değildir
- fazla esneklik backend complexity üretir

---

# 18. Expand Parametresi

Bazı ilişkili alanlar varsayılan olarak dönülmez, istenirse expand ile açılır.

Örnek:

    GET /orders/ord_123?expand=items,payment

Bu yaklaşım:
- response boyutunu kontrol eder
- N+1 benzeri gereksiz yükleri daha görünür yapar
- client'a kontrollü esneklik sunar

---

# 19. Query Param İsimlendirme Standardı

Tutarlı isimler kullanılmalıdır.

Önerilen standart:

- page
- size
- sort
- query
- fields
- expand
- min_price
- max_price
- created_from
- created_to
- status
- category

Yanlış:
- pageNumber
- perPageItemCount
- filterText
- orderingExpression

---

# 20. Query Param Validation Zorunluluğu

Query param'lar serbest giriş alanı değildir.

Validate edilmelidir:

- page >= 1
- size > 0
- size <= max_size
- sort allowlist içinde mi
- expand allowlist içinde mi
- tarih formatı geçerli mi

Yanlış:
- tüm query string'i doğrudan DB query'ye taşımak
- sıralama alanını raw SQL/ES order ifadesine çevirmek

---

# 21. Allowlist Kuralı

Sorting, filtering ve expand alanlarında allowlist yaklaşımı kullanılmalıdır.

Örnek:
- sort alanları: created_at, price, name
- expand alanları: items, payment, customer

Neden?
- güvenlik
- kontrol
- gereksiz karmaşıklığı önleme
- query injection riskini azaltma

---

# 22. Boş Sonuç Davranışı

Filtre sonucu veri bulunmazsa:

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

Kural:
- collection endpoint boşsa boş collection döner
- 404 dönülmez

---

# 23. Ağır Query'leri Default Açma

Her endpoint'te her filtreyi açmak doğru değildir.

Kaçınılmalı:
- onlarca query param
- pahalı relation expand'leri
- aşırı serbest search expression yapıları

Kural:
- ihtiyaç kadar esneklik
- ölçülebilir complexity
- bounded query surface

---

# 24. Search Engine ve DB Arasındaki Fark

Query design, alttaki storage'a göre değişebilir.

## DB tabanlı listeleme
- offset daha yaygın
- relation maliyeti kritik

## Search engine tabanlı listeleme
- faceting
- score sorting
- cursor veya search_after benzeri yaklaşımlar
- deep pagination limitleri

Bu yüzden:
- dış API contract'ı stabil tutulmalı
- internal implementation değişse de client contract mümkün olduğunca korunmalı

---

# 25. Aggregation ve Listing Ayrımı

Liste endpoint'i ile aggregation endpoint'i her zaman aynı şey değildir.

Örnek:
- products listesi
- products facet/aggregation verisi

Bazı sistemlerde ayrı endpoint daha doğrudur:

    GET /products
    GET /products/aggregations

veya

    GET /products/facets

Kural:
- liste ve analiz yükünü bilinçsizce tek endpoint'e yığma
- response contract'ı amaç odaklı tasarla

---

# 26. Pagination Metadata Standardı

Pagination objesi ekip genelinde sabit olmalıdır.

Önerilen yapı:

    "pagination": {
      "page": 1,
      "size": 20,
      "total_items": 124,
      "total_pages": 7
    }

Cursor pagination için:

    "pagination": {
      "size": 20,
      "next_cursor": "abc123",
      "has_next": true
    }

Bu iki yapı birbirine karıştırılmamalıdır.

---

# 27. Query Complexity Kontrolü

Aşağıdaki durumlar kontrol altına alınmalıdır:

- çok fazla filter key
- çok fazla expand
- çok büyük size
- pahalı sort kombinasyonları
- wildcard benzeri pahalı aramalar

Gerekirse:
- reject et
- degrade et
- sınırlı support ver

---

# 28. Documentation Zorunluluğu

Her liste endpoint'i için şu bilgiler açık olmalıdır:

- desteklenen filtreler
- desteklenen sort alanları
- pagination tipi
- default sort
- max size
- expand alanları
- boş sonuç davranışı

Dokümante edilmeyen query API zamanla kaosa dönüşür.

---

# 29. Production Checklist

Yeni collection endpoint eklemeden önce kontrol et:

- [ ] Pagination zorunlu mu ve bounded mı?
- [ ] Default size ve max size tanımlı mı?
- [ ] Default sort var mı?
- [ ] Sorting allowlist ile kontrol ediliyor mu?
- [ ] Filtering tutarlı mı?
- [ ] Query param isimleri standarda uygun mu?
- [ ] Empty result davranışı sabit mi?
- [ ] Ağır query'ler kontrollü mü?
- [ ] Gerekliyse cursor pagination düşünülmüş mü?
- [ ] Dokümantasyon tamam mı?

---

# 30. Sonuç

Pagination, filtering ve query design:

- sadece listeleme konusu değildir
- performans mühendisliğidir
- API ergonomisidir
- veri erişim disiplinidir

İyi tasarlanmış query surface:
- client kullanımını sadeleştirir
- backend maliyetini kontrol eder
- veri erişimini öngörülebilir yapar
- production riskini azaltır

Kötü tasarlanmış query surface ise:
- pahalı sorgular üretir
- scaling sorunları yaratır
- endpoint karmaşası doğurur
- bakım maliyetini büyütür