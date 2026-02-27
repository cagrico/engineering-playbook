# Engineering Playbook

Yüksek trafikli backend sistemler geliştiren ekipler için hazırlanmış, pratik ve üretim odaklı bir mühendislik rehberi.

Bu repository;

- Mimari kuralları
- İsimlendirme standartlarını
- Memory-first tasarım prensiplerini
- Yüksek yük altında sistem davranışını
- Production ortamında kök neden analizini

dokümante eder.

Amaç basit:

> Yük altında çökmeyen sistemler inşa etmek.

---

## 🎯 Bu Repo Neyi Amaçlar?

Çoğu sistem şu şekilde gelişir:

1. İş mantığı yazılır
2. Trafik artar
3. Performans problemi çıkar
4. “Optimize edelim” denir

Bu playbook farklı bir yaklaşım benimser:

> Performans ve bellek davranışı tasarımın ilk gününden planlanır.

---

## 📚 İçerik Başlıkları

### 1️⃣ Mimari Kurallar

- Hexagonal / Clean Architecture prensipleri
- Katman sorumlulukları
- Domain, Usecase, Adapter ayrımı
- Validation sorumlulukları
- Mapping kuralları
- Error yönetimi

Dosyalar:

- `ARCHITECTURE_GUIDELINES.md`
- `VALIDATION_RESPONSIBILITY_AND_INVARIANT_RULES.md`

---

### 2️⃣ İsimlendirme Standartları

Go ve backend projeleri için tutarlı isimlendirme kuralları:

- Değişken
- Struct
- Interface
- Method
- Repository
- API route
- Veritabanı (tablo, kolon, index)
- Kafka topic / event
- Test isimlendirme

Dosya:

- `NAMING_STANDARDS_BACKEND.md`

---

### 3️⃣ Git Commit Standartları

Conventional Commits tabanlı commit mesaj standardı.

Dosya:

- `GIT_COMMIT_MESSAGE.md`

---

### 4️⃣ Bellek Yönetimi ve Memory-First Mimari

Bu bölüm özellikle yüksek trafikli sistemler için hazırlanmıştır.

Kapsanan konular:

- Go’da bellek yönetimi (stack, heap, GC, allocation)
- Escape analysis
- Allocation budget mantığı
- 50k RPS API için memory budgeting
- 100k msg/s Kafka pipeline tasarımı
- Burst trafik senaryosu
- Backpressure mühendisliği
- Worker pool tasarımı
- Production memory audit rehberi

Klasör:

- `MEMORY_MANAGEMENT/`

---

## 🧠 Temel Prensipler

### 1️⃣ Predictability (Öngörülebilirlik)

Şu soruların cevabı bilinmelidir:

- 50k RPS’te heap kaç MB?
- Burst trafik geldiğinde ne olur?
- In-flight message sayısı kaç?
- Allocation rate nedir?

Eğer cevap bilinmiyorsa sistem kontrolsüzdür.

---

### 2️⃣ Bounded Everything

Sınırsız hiçbir yapı kabul edilmez:

- Sınırsız goroutine ❌
- Sınırsız queue ❌
- Sınırsız cache ❌
- Sınırsız retry ❌

Her şeyin üst limiti vardır.

---

### 3️⃣ Memory-First Yaklaşım

CPU ölçeklenebilir.

Bellek patlarsa sistem düşer.

Bu nedenle bellek, mimarinin merkezindedir.

---

## 👥 Hedef Kitle

- Go geliştiricileri
- Backend mühendisleri
- Yüksek trafikli sistemler geliştiren ekipler
- Clean Architecture uygulayan takımlar
- Production stabilitesini önemseyen teknik liderler

---

## ⚙️ Bu Repo Nedir?

- Bir framework değildir.
- Copy-paste proje şablonu değildir.
- Akademik teori kitabı değildir.

Bu repo:

> Üretim ortamında gerçekten işe yarayan mühendislik kurallarının yazılı halidir.

---

## 📌 Durum

Aktif olarak geliştirilmektedir.

Yeni başlıklar ve ileri seviye konular zamanla eklenecektir.