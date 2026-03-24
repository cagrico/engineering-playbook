# Kafka Topic Configuration Guideline

## 1. Amaç

Kafka topic ayarları aşağıdaki başlıkları doğrudan etkiler:

- ölçeklenebilirlik (scalability)
- dayanıklılık (durability)
- performans (throughput)
- disk kullanımı (retention)

Her topic, kullanım amacına göre ayrı değerlendirilmelidir.

---

## 2. Topic Türleri

### 2.1 Business Critical Events

Örnekler:
- `order-created`
- `payment-completed`
- `stock-reserved`

Özellik:
- veri kaybı kabul edilemez
- replay ihtiyacı olabilir
- yüksek dayanıklılık gerekir

### 2.2 Analytics / Logs

Örnekler:
- `page-view`
- `click-event`
- `search-log`

Özellik:
- veri kaybı belirli ölçüde tolere edilebilir
- yüksek hacim olabilir
- retention genelde daha kısa tutulur

### 2.3 State Topics (Compacted)

Örnekler:
- `product-current-state`
- `user-preferences-current`

Özellik:
- geçmişten çok son state önemlidir
- genelde key bazlı çalışır
- compact policy uygundur

### 2.4 Temporary / Ephemeral Topics

Örnekler:
- `notification-temp`
- `transient-job-events`

Özellik:
- kısa ömürlü veri taşır
- uzun süre saklama gerekmez

---

## 3. Temel Ayarlar

### 3.1 Number of Partitions

Ne işe yarar:
- parallel processing kapasitesini belirler
- consumer group içindeki maksimum paralel tüketimi belirler

Temel kural:

    partition sayısı >= hedeflenen paralel consumer sayısı

Öneri:
- küçük sistem: `3`
- orta ölçek: `6`
- yüksek trafik: `12+`

Notlar:
- partition sayısı arttıkça throughput artabilir
- fazla partition operasyonel karmaşıklık ve rebalance maliyeti oluşturabilir
- ordering, partition bazında korunur

---

### 3.2 Replication Factor

Ne işe yarar:
- verinin kaç kopya tutulacağını belirler

Öneri:
- dev: `1`
- stage: `1` veya `2`
- production: `3`

Notlar:
- replication factor, broker sayısından büyük olamaz
- business critical topic’lerde genel tercih `3` olmalıdır

---

### 3.3 Min In Sync Replicas (minISR)

Ne işe yarar:
- yazının başarılı sayılması için minimum kaç replica’nın senkron durumda olması gerektiğini belirler

Production için önerilen kombinasyon:

    replication.factor = 3
    min.insync.replicas = 2
    producer acks = all

Notlar:
- `min.insync.replicas` tek başına yeterli değildir
- producer tarafında `acks=all` kullanılmalıdır
- bu kombinasyon durability ile availability arasında dengeli bir yaklaşımdır

---

### 3.4 Retention (retention.ms)

Ne işe yarar:
- verinin ne kadar süre saklanacağını belirler

Öneri:
- critical events: `7-30 gün`
- analytics: `1-7 gün`
- temporary: `saatler - 1 gün`
- audit: `uzun süre`

Notlar:
- retention ihtiyacı replay, debugging ve yeni consumer senaryolarına göre belirlenmelidir
- gereksiz uzun retention, disk maliyetini artırır

---

### 3.5 Disk Limit (retention.bytes)

Ne işe yarar:
- topic verisinin boyut bazlı sınırlandırılmasını sağlar

Not:
- bu değer çoğu durumda partition başına uygulanır

Toplam yaklaşık disk hesabı:

    total ~= retention.bytes x partition sayısı x replication factor

Kullanım:
- opsiyoneldir
- runaway producer veya beklenmeyen veri büyümesine karşı ek güvenlik sağlar

---

## 4. Custom Parameters

### 4.1 cleanup.policy

Olası değerler:
- `delete`
- `compact`

Kullanım:
- event stream için: `delete`
- state topic için: `compact`

Açıklama:
- `delete`: retention süresi veya boyut limiti dolunca eski segmentler silinir
- `compact`: aynı key için son değerler korunur

---

### 4.2 max.message.bytes

Ne işe yarar:
- topic’e yazılabilecek maksimum mesaj boyutunu belirler

Öneri:
- büyük payload Kafka’ya yazılmamalıdır
- büyük veri object storage’da tutulmalı, Kafka’ya referans yazılmalıdır

---

### 4.3 compression.type

Ne işe yarar:
- disk ve network kullanımını optimize eder

Öneri:
- yüksek hacimli topic’lerde değerlendirilmelidir

---

## 5. Standart Profiller

### 5.1 Critical Event Topic

Önerilen ayarlar:

    Partitions: 6
    Replication Factor: 3
    Min ISR: 2
    Retention: 7 days
    Cleanup Policy: delete

Uygun kullanım:
- sipariş
- ödeme
- stok
- kritik iş olayları

---

### 5.2 Analytics Topic

Önerilen ayarlar:

    Partitions: 12+
    Replication Factor: 2-3
    Min ISR: 1-2
    Retention: 1-7 days
    Cleanup Policy: delete

Uygun kullanım:
- metrics
- clickstream
- log benzeri yüksek hacimli veriler

---

### 5.3 Compacted State Topic

Önerilen ayarlar:

    Partitions: ihtiyaca göre
    Replication Factor: 3
    Min ISR: 2
    Cleanup Policy: compact

Uygun kullanım:
- current state dağıtımı
- son durumun tutulduğu key bazlı topic’ler

---

### 5.4 Temporary Topic

Önerilen ayarlar:

    Partitions: ihtiyaca göre
    Replication Factor: 1
    Min ISR: 1
    Retention: kısa süre
    Cleanup Policy: delete

Uygun kullanım:
- geçici veri
- kısa ömürlü pipeline mesajları

---

## 6. Karar Checklist

Yeni topic açmadan önce şu sorular cevaplanmalıdır:

- veri kritik mi?
- replay ihtiyacı var mı?
- kaç consumer paralel çalışacak?
- veri ne kadar süre değerli?
- disk maliyeti ne kadar önemli?
- event mi taşınıyor, state mi taşınıyor?

---

## 7. Default Production Baseline

Özel bir ihtiyaç yoksa başlangıç için aşağıdaki baseline kullanılabilir:

    Partitions: 6
    Replication Factor: 3
    Min ISR: 2
    Retention: 7 days
    Cleanup Policy: delete

Producer tarafı:

    acks = all

Ek öneri:
- mümkünse idempotent producer kullanılmalıdır

---

## 8. Önemli Notlar

- partition sayısı sonradan artırılabilir, ancak key dağılımı ve tüketim davranışı etkilenebilir
- replication factor, broker sayısını aşamaz
- minISR her zaman producer `acks` ayarı ile birlikte düşünülmelidir
- retention süresi ve disk kullanımı birlikte planlanmalıdır
- Kafka, büyük binary payload taşımak için uygun bir yer değildir

---

## 9. Sonuç

Kafka topic konfigürasyonu standart ama kör bir şekilde verilmemelidir.

Karar verirken şu denge kurulmalıdır:

- dayanıklılık
- performans
- operasyonel maliyet
- replay ihtiyacı
- veri ömrü

Genel production başlangıç profili çoğu business event topic için yeterlidir. Özel senaryolarda topic bazlı teknik gerekçe ile ayrı konfigürasyon tanımlanmalıdır.