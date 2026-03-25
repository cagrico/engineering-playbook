# API File Upload / Download Design Guide

Bu doküman, production ortamında çalışan API'lerde file upload ve download tasarım kurallarını tanımlar.

Amaç:

- Büyük dosya işlemlerini güvenli ve performanslı hale getirmek
- Memory kullanımını kontrol altında tutmak
- Upload ve download işlemlerini ölçeklenebilir yapmak
- CDN ve object storage entegrasyonunu standartlaştırmak

---

# 1. Temel İlke

Backend API:

- dosya taşımaz
- dosya proxy'lemez

> API sadece metadata yönetir, dosya storage'a gider

---

# 2. En Kritik Kural

Dosya upload:

- API üzerinden yapılmamalıdır

Yanlış:

    POST /upload
    (file backend'e gelir)

Bu:

- memory tüketir
- CPU yükü oluşturur
- scaling'i zorlaştırır

---

# 3. Doğru Yaklaşım: Direct Upload

Akış:

1. Client → API → presigned URL alır
2. Client → storage (S3) upload eder

---

## 3.1 Örnek

    POST /uploads

Response:

    {
      "upload_url": "https://s3...",
      "file_id": "file_123"
    }

---

# 4. Storage Kullanımı

Önerilen:

- S3
- MinIO
- GCS

---

# 5. Upload Flow

    Client → API → presigned URL
    Client → S3 → upload
    Client → API → confirm

---

# 6. Download Tasarımı

Yanlış:

    GET /files/{id}
    (API dosyayı stream eder)

Doğru:

- presigned URL döndür

---

## 6.1 Örnek

    GET /files/{id}

Response:

    {
      "download_url": "https://s3..."
    }

---

# 7. Streaming Ne Zaman Kullanılır?

Sadece:

- küçük dosyalar
- internal servisler

---

# 8. File Metadata

DB'de tutulur:

- file_id
- filename
- size
- content_type
- storage_path

---

# 9. Content-Type Kontrolü

Upload sırasında:

- mime type validate edilmeli

---

# 10. File Size Limit

Her upload:

- bounded olmalıdır

Örnek:

- max 5MB (image)
- max 100MB (video)

---

# 11. Security

## 11.1 Dosya Türü

- whitelist kullanılmalı

---

## 11.2 Malware Riski

- scan (opsiyonel)

---

## 11.3 Private vs Public

- private file → signed URL
- public file → CDN

---

# 12. Expiring URL

Presigned URL:

- kısa süreli olmalı

Örnek:

- 5 dk
- 15 dk

---

# 13. CDN Kullanımı

Önerilen:

- Cloudflare
- CloudFront

---

# 14. Image Optimization

- resize
- webp dönüşümü

---

# 15. Multi-Part Upload

Büyük dosyalar için:

- chunk upload

---

# 16. Retry Stratejisi

Upload:

- retry edilebilir olmalı

---

# 17. Idempotency

Upload:

- duplicate olmamalı

---

# 18. File Naming

Öneri:

- UUID kullan

---

# 19. Folder Structure

Örnek:

    /users/{id}/images/{file_id}

---

# 20. Delete İşlemi

    DELETE /files/{id}

---

# 21. Orphan File Problemi

Upload olmuş ama:

- DB kaydı yok

Çözüm:

- cleanup job

---

# 22. Monitoring

Takip et:

- upload rate
- failure rate
- file size dağılımı

---

# 23. Anti-Patterns

## ❌ Dosyayı API'den geçirmek

## ❌ Sınırsız file size

## ❌ Validation yapmamak

## ❌ Public/private ayrımı olmaması

---

# 24. Production Checklist

- [ ] Direct upload var mı?
- [ ] Presigned URL kullanılıyor mu?
- [ ] File size limit var mı?
- [ ] Content-type validate ediliyor mu?
- [ ] CDN kullanılıyor mu?
- [ ] Metadata DB'de mi?
- [ ] Cleanup mekanizması var mı?

---

# 25. Sonuç

File handling:

- performans konusudur
- maliyet konusudur
- güvenlik konusudur

Doğru yapılırsa:

- sistem hızlı olur
- maliyet düşer

Yanlış yapılırsa:

- sistem çöker
- resource tüketimi patlar