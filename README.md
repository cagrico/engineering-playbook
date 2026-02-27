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

## 4️⃣ Bellek ve Performans (İleri Seviye)

Önerilen okuma sırası:

1. [Go'da Bellek Yönetimi](MEMORY_MANAGEMENT/1-MEMORY_MANAGEMENT_IN_GO.md)
2. [Memory-First Architecture](MEMORY_MANAGEMENT/2-MEMORY-FIRST_ARCHITECTURE.md)
3. [Yüksek Trafikli Sistemlerde Bellek Yönetimi](MEMORY_MANAGEMENT/3-MEMORY_MANAGEMENT_IN_HIGH-TRAFFIC_SYSTEMS.md)
4. [50K RPS API için Memory Budget Tasarımı](MEMORY_MANAGEMENT/4-MEMORY_BUDGET_DESIGN_FOR_A_50K_RPS_API.md)
5. [100K msg/s Kafka Pipeline için Memory-First Tasarım](MEMORY_MANAGEMENT/5-MEMORY-FIRST_DESIGN_FOR_100K_MSG_S_KAFKA_PIPELINE.md)
6. [Burst Traffic Senaryosu](MEMORY_MANAGEMENT/6-MEMORY_MANAGEMENT_IN_THE_BURST_TRAFFIC_SCENARIO.md)
7. [Production Memory Audit Rehberi](MEMORY_MANAGEMENT/7-PRODUCTION_MEMORY_AUDIT_GUIDE.md)

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

Lisans bilgisi public hale getirildiğinde eklenecektir.