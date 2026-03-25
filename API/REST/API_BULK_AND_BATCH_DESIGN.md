# API Bulk / Batch Design Guide

Bu doküman, production ortamında çalışan API'lerde bulk (toplu) ve batch (parçalı) işlem tasarım kurallarını tanımlar.

Amaç:

- Network maliyetini azaltmak
- Yüksek hacimli veri işlemlerini optimize etmek
- Throughput'u artırmak
- Sistem kaynaklarını kontrollü kullanmak
- Large-scale operasyonları güvenli hale getirmek

---

# 1. Temel İlke

Tek tek işlem yapmak:

- pahalıdır
- yavaştır
- network yükü üretir

Bu yüzden:

> high-volume işlemler batch veya bulk yapılmalıdır

---

# 2. Bulk vs Batch Farkı

## Bulk

- tek request
- birden fazla işlem

Örnek:

    POST /products/bulk

---

## Batch

- işlemler parçalara bölünür
- sırayla veya paralel çalışır

---

# 3. Ne Zaman Kullanılır?

- toplu ürün ekleme
- fiyat güncelleme
- stok güncelleme
- veri import
- mass update operasyonları

---

# 4. En Kritik Kural

Bulk endpoint:

- bounded olmalıdır

Yanlış:

    1 request = 1 milyon kayıt ❌

Doğru:

- max limit belirle (ör: 100, 500, 1000)

---

# 5. Request Tasarımı

Örnek:

    POST /products/bulk

    {
      "items": [
        {
          "name": "Product A",
          "price": 100
        },
        {
          "name": "Product B",
          "price": 200
        }
      ]
    }

---

# 6. Response Tasarımı

## 6.1 Tam Başarı

    {
      "processed": 2,
      "success": 2,
      "failed": 0
    }

---

## 6.2 Kısmi Başarı (ÖNERİLEN)

    {
      "processed": 2,
      "success": 1,
      "failed": 1,
      "errors": [
        {
          "index": 1,
          "code": "VALIDATION_ERROR",
          "message": "price must be positive"
        }
      ]
    }

---

# 7. Partial Success Kuralı

Bulk işlemler:

- tamamen fail olmak zorunda değildir

Kural:

- partial success desteklenmeli

---

# 8. Atomic vs Non-Atomic

## Atomic (Tüm ya da hiç)

- küçük batch'ler için uygun

## Non-Atomic (ÖNERİLEN)

- her item bağımsız işlenir
- scalable

---

# 9. Validation Stratejisi

Her item:

- ayrı validate edilmelidir

---

# 10. Idempotency ile Birlikte Kullanım

Bulk işlemler:

- idempotent olmalıdır

---

## 10.1 Örnek

Her item:

- unique key taşıyabilir

---

# 11. Rate Limiting ile İlişki

Bulk endpoint:

- ayrı rate limit'e sahip olmalıdır

---

# 12. Timeout Riski

Bulk işlemler:

- uzun sürebilir

Çözüm:

- async processing
- job queue

---

# 13. Async Batch İşleme

Büyük batch'ler için:

    POST /products/import

Response:

    {
      "job_id": "job_123"
    }

---

## 13.1 Job Status

    GET /jobs/job_123

---

# 14. Parallel Processing

Batch işlemler:

- paralel işlenebilir

Ama:

- bounded concurrency olmalı

---

# 15. Ordering Problemi

Bazı işlemler:

- sıraya bağlı olabilir

Bu durumda:

- ordering korunmalı

---

# 16. Error Handling

Her item:

- kendi error'unu taşımalı

---

# 17. Payload Boyutu

Max payload:

- sınırlandırılmalıdır

Örnek:

- 1MB
- 5MB

---

# 18. Streaming Alternatifi

Çok büyük veri için:

- streaming
- file upload

---

# 19. DB ve Transaction

Bulk işlemler:

- büyük transaction olmamalı

---

# 20. Kafka ile Entegrasyon

Bulk işlemler:

- event olarak publish edilebilir

---

# 21. Monitoring

Takip et:

- batch size
- success rate
- failure rate
- duration

---

# 22. Anti-Patterns

## ❌ Sınırsız bulk

## ❌ Tek transaction

## ❌ Partial success desteklememek

## ❌ Error detay vermemek

---

# 23. Production Checklist

- [ ] Max batch size var mı?
- [ ] Partial success destekleniyor mu?
- [ ] Timeout düşünülmüş mü?
- [ ] Async opsiyon var mı?
- [ ] Rate limit var mı?
- [ ] Validation item bazlı mı?
- [ ] Payload limit var mı?

---

# 24. Sonuç

Bulk ve batch tasarımı:

- performans konusudur
- ölçeklenebilirlik konusudur

Doğru uygulanırsa:

- throughput artar
- sistem stabil kalır

Yanlış uygulanırsa:

- sistem overload olur
- latency patlar