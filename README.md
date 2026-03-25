# Engineering Playbook

Yüksek trafikli backend sistemler geliştiren ekipler için hazırlanmış, üretim odaklı bir mühendislik rehberi.

Bu repository;

- Mimari sınırları netleştirir
- İsimlendirme disiplinini standardize eder
- Bellek davranışını tasarım aşamasında planlar
- Yük altında sistemin nasıl ayakta kalacağını anlatır
- Production ortamında kök neden analizini sistematik hale getirir

Amaç basit:

> Yük altında çökmeyen, öngörülebilir sistemler inşa etmek.

---

# 📌 Bu Repo Nedir?

Bu repo:

- Bir framework değildir
- Copy-paste proje şablonu değildir
- Akademik teori dokümanı değildir

Bu repo:

> Production ortamında gerçekten işe yarayan mühendislik kurallarının yazılı halidir.

---

# 🧭 Nasıl Okunmalı?

Eğer yeni başlıyorsan şu sırayı takip et:

## 1️⃣ Mimari Temel

- [Hexagonal / Clean Architecture Kuralları](ARCHITECTURE_GUIDELINES.md)
- [Validation Sorumlulukları ve Invariant Kuralları](VALIDATION_RESPONSIBILITY_AND_INVARIANT_RULES.md)

## 2️⃣ İsimlendirme Disiplini

- [Backend Naming Standards](NAMING_STANDARDS_BACKEND.md)

## 3️⃣ Git Disiplini

- [Commit Mesaj Standardı](GIT_COMMIT_MESSAGE.md)

## 4️⃣ API Design (Production)

Yüksek trafikli sistemlerde API tasarımı, sadece endpoint yazmak değildir.

Yanlış API tasarımı:
- Scale problemleri üretir
- Cache'i bozar
- Client-server coupling yaratır
- Maintainability düşürür

Önerilen okuma:

- [RESTful API Design Guide](API/REST/RESTFUL_API_DESIGN.md)
- [API Naming Convention Guide](API/REST/API_NAMING_CONVENTION_GUIDE.md)
- [API Response & Error Standard](API/REST/API_RESPONSE_AND_ERROR_STANDARD.md)
- [API Versioning & Compatibility Guide](API/REST/API_VERSIONING_AND_COMPATIBILITY.md)
- [API Rate Limiting & Protection Guide](API/REST/API_RATE_LIMITING_AND_PROTECTION.md)
- [API Pagination, Filtering & Query Design Guide](API/REST/API_PAGINATION_FILTERING_AND_QUERY_DESIGN.md)
- [API Authentication & Authorization Guide](API/REST/API_AUTHENTICATION_AND_AUTHORIZATION.md)
- [API Idempotency & Safe Retry Guide](API/REST/API_IDEMPOTENCY_AND_SAFE_RETRY.md)
- [API Validation & Input Boundary Rules](API/REST/API_VALIDATION_AND_INPUT_BOUNDARY_RULES.md)
- [API Timeout, Cancellation & Resilience Guide](API/REST/API_TIMEOUT_CANCELLATION_AND_RESILIENCE.md)
- [API Documentation Standard](API/REST/API_DOCUMENTATION_STANDARD.md)
- [API Observability & Audit Logging](API/REST/API_OBSERVABILITY_AND_AUDIT_LOGGING.md)
- [API Deprecation & Lifecycle Management](API/REST/API_DEPRECATION_AND_LIFECYCLE.md)
- [Bulk / Batch API Design](API/REST/API_BULK_AND_BATCH_DESIGN.md)

## 5️⃣ Bellek ve Performans (İleri Seviye)

Önerilen okuma sırası:

1. [Go'da Bellek Yönetimi](MEMORY_MANAGEMENT/1-MEMORY_MANAGEMENT_IN_GO.md)
2. [Memory-First Architecture](MEMORY_MANAGEMENT/2-MEMORY-FIRST_ARCHITECTURE.md)
3. [Yüksek Trafikli Sistemlerde Bellek Yönetimi](MEMORY_MANAGEMENT/3-MEMORY_MANAGEMENT_IN_HIGH-TRAFFIC_SYSTEMS.md)
4. [50K RPS API için Memory Budget Tasarımı](MEMORY_MANAGEMENT/4-MEMORY_BUDGET_DESIGN_FOR_A_50K_RPS_API.md)
5. [100K msg/s Kafka Pipeline için Memory-First Tasarım](MEMORY_MANAGEMENT/5-MEMORY-FIRST_DESIGN_FOR_100K_MSG_S_KAFKA_PIPELINE.md)
6. [Burst Traffic Senaryosu](MEMORY_MANAGEMENT/6-MEMORY_MANAGEMENT_IN_THE_BURST_TRAFFIC_SCENARIO.md)
7. [Production Memory Audit Rehberi](MEMORY_MANAGEMENT/7-PRODUCTION_MEMORY_AUDIT_GUIDE.md)

## 6️⃣ Distributed Systems (İleri Seviye)

## Nereden Başlamalı?

Eğer Kafka’ya yeni başlıyorsan:
1. Topic Configuration
2. Topic Naming Convention
3. Event Design Guide

Eğer production problemi çözüyorsan:
1. Retry, DLQ & Idempotency
2. Outbox & Consumer Idempotency
3. Backpressure & Memory Guide


Kafka tabanlı event-driven sistemlerde production-grade kurallar:

### Kafka Playbook

- [Kafka Overview](DISTRIBUTED_SYSTEMS/KAFKA/README.md)

#### Core Topics

- [Topic Configuration](DISTRIBUTED_SYSTEMS/KAFKA/TOPIC_CONFIGURATION.md)
- [Topic Naming Convention](DISTRIBUTED_SYSTEMS/KAFKA/TOPIC_NAMING_CONVENTION.md)
- [Event Design Guide](DISTRIBUTED_SYSTEMS/KAFKA/EVENT_DESIGN_GUIDE.md)

#### Runtime Behavior

- [Producer & Consumer Strategy](DISTRIBUTED_SYSTEMS/KAFKA/PRODUCER_CONSUMER_STRATEGY.md)
- [Retry, DLQ & Idempotency](DISTRIBUTED_SYSTEMS/KAFKA/RETRY_DLQ_IDEMPOTENCY.md)
- [Outbox & Consumer Idempotency](DISTRIBUTED_SYSTEMS/KAFKA/OUTBOX_AND_CONSUMER_IDEMPOTENCY.md)

#### Performance & Stability

- [Backpressure & Memory Guide](DISTRIBUTED_SYSTEMS/KAFKA/BACKPRESSURE_AND_MEMORY_GUIDE.md)

---

# 🧠 Temel Felsefe

## 1️⃣ Predictability (Öngörülebilirlik)

Şu soruların cevabı bilinmelidir:

- 50k RPS’te heap kaç MB?
- Allocation rate nedir?
- In-flight request sayısı kaç?
- Burst trafik geldiğinde sistem ne yapar?

Eğer bu soruların cevabı bilinmiyorsa sistem kontrolsüzdür.

---

## 2️⃣ Bounded Everything

Sınırsız hiçbir yapı kabul edilmez:

- Sınırsız goroutine ❌
- Sınırsız queue ❌
- Sınırsız cache ❌
- Sınırsız retry ❌
- Sınırsız batch ❌

Her şeyin üst sınırı vardır.

---

## 3️⃣ Memory-First Yaklaşım

Çoğu sistem CPU odaklı tasarlanır.

Gerçekte:

- CPU ölçeklenebilir
- RAM patlarsa sistem düşer
- GC spike üretirse P99 latency patlar
- OOM olursa container ölür

Bu nedenle:

> Bellek, mimarinin merkezindedir.

---

# 🏗 Kapsanan Konular

### Mimari

- Hexagonal / Clean Architecture
- Katman sorumlulukları
- Domain izolasyonu
- Usecase orkestrasyonu
- Adapter sorumlulukları
- Mapping kuralları
- Error yönetimi

---

### İsimlendirme

- Değişken ve struct isimleri
- Interface stratejisi
- Repository isimlendirme
- API route standardı
- Veritabanı tablo/kolon/index isimleri
- Kafka topic ve event isimleri
- Test isimlendirme disiplini

---

### Bellek Yönetimi

- Stack vs Heap
- Escape analysis
- Garbage Collector davranışı
- Allocation rate ve etkisi
- Zero allocation yaklaşımı
- Worker pool tasarımı
- Backpressure mühendisliği
- Batch stratejileri
- Burst traffic senaryosu
- Memory budgeting
- Production memory audit

---

# 📊 Hangi Seviyeye Hitap Ediyor?

Bu repo:

- Junior geliştiriciler için temel açıklamalar içerir
- Mid-level geliştiriciler için sistematik yapı sunar
- Senior geliştiriciler için tasarım kararlarını sayısal düşünmeye zorlar
- Teknik liderler için referans çerçeve oluşturur

---

# ⚙️ Hangi Problemleri Çözer?

- Yük altında artan heap
- GC spike kaynaklı latency
- Unbounded goroutine sorunu
- Kafka consumer memory patlaması
- Burst traffic çökmesi
- Mimari katmanların birbirine karışması
- Naming kaosu
- Tutarsız commit geçmişi

---

# 🚀 Hedef

Bu playbook’un hedefi:

- Rastgele büyüyen sistemler değil
- Ölçülebilir, tahmin edilebilir, kontrollü sistemler inşa etmektir.

---

# 📌 Durum

Aktif olarak geliştirilmektedir.

Yeni başlıklar eklenecek:

- Distributed consistency
- Advanced Kafka patterns
- Cache stratejileri
- Outbox ve idempotency tasarım rehberi
- Observability ve metrics engineering
- Tail latency engineering

---

# 🛡 Lisans

MIT