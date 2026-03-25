# API Deprecation & Lifecycle Management

Bu doküman, production ortamında çalışan API'lerde lifecycle (yaşam döngüsü) ve deprecation (kullanımdan kaldırma) süreçlerini tanımlar.

Amaç:

- API değişikliklerini kontrollü şekilde yönetmek
- Eski endpoint'leri güvenli şekilde kaldırmak
- Client'ların kırılmasını engellemek
- Versioning stratejisini tamamlamak
- Production ortamında sürdürülebilir API yönetimi sağlamak

---

# 1. Temel İlke

Bir API:

- doğar
- kullanılır
- evrimleşir
- deprecated olur
- kaldırılır

Bu süreç:

> lifecycle management olarak adlandırılır

---

# 2. En Kritik Kural

Bir endpoint:

- aniden kaldırılmaz

Yanlış:

- endpoint'i bir gün silmek

Doğru:

- deprecate et → duyur → migrate ettir → kaldır

---

# 3. API Lifecycle Aşamaları

## 3.1 Draft

- geliştirme aşaması
- internal kullanım

---

## 3.2 Active

- production'da aktif
- desteklenir

---

## 3.3 Deprecated

- kullanılmaması önerilir
- yerine yenisi vardır

---

## 3.4 Sunset

- kaldırılma tarihi belirlenmiştir

---

## 3.5 Removed

- artık erişilemez

---

# 4. Deprecation Nedir?

Bir endpoint'in:

- kullanılmaması gerektiğini belirtmektir

Ama:

- hala çalışır

---

# 5. Ne Zaman Deprecate Edilir?

- breaking change gerekiyorsa
- daha iyi bir endpoint varsa
- eski model sürdürülemiyorsa
- performans problemi varsa

---

# 6. Deprecation Süreci

## Adımlar:

1. Yeni endpoint oluştur
2. Eski endpoint'i deprecated işaretle
3. Dokümantasyonu güncelle
4. Client'lara duyur
5. Kullanımı takip et
6. Sunset tarihi belirle
7. Endpoint'i kaldır

---

# 7. Deprecation Duyurusu

Client'a açık şekilde iletilmelidir.

Yöntemler:

- dokümantasyon
- email
- changelog
- header

---

# 8. HTTP Header ile Deprecation

Örnek:

    Deprecation: true
    Sunset: Wed, 01 Jan 2027 00:00:00 GMT

---

# 9. Versioning ile İlişki

Deprecation:

- versioning ile birlikte çalışır

Örnek:

- v1 deprecated
- v2 aktif

---

# 10. Backward Compatibility

Deprecated endpoint:

- çalışmaya devam etmelidir
- breaking change yapılmamalıdır

---

# 11. Sunset Süresi

Öneri:

- minimum 3–6 ay

Kritik sistemlerde:
- 6–12 ay

---

# 12. Migration Stratejisi

Client'lara:

- yeni endpoint
- yeni request/response yapısı

açıkça anlatılmalıdır

---

# 13. Monitoring

Deprecated endpoint:

- kullanım oranı izlenmelidir

---

## Örnek:

- günlük request sayısı
- hangi client'lar kullanıyor

---

# 14. Removal Kararı

Endpoint kaldırılmadan önce:

- kullanım %0'a yakın olmalı
- kritik client kalmamalı

---

# 15. Graceful Removal

Endpoint kaldırıldığında:

    HTTP/1.1 410 Gone

    {
      "code": "ENDPOINT_REMOVED",
      "message": "This endpoint is no longer available"
    }

---

# 16. Hard Removal Hatası

Yanlış:

- direkt 404 dönmek

Doğru:

- 410 Gone

---

# 17. Documentation Zorunluluğu

Deprecated endpoint:

- açıkça işaretlenmelidir

---

# 18. Anti-Patterns

## ❌ Aniden endpoint silmek

## ❌ Client'a haber vermemek

## ❌ Deprecated endpoint'i değiştirmek

## ❌ Sunset tarihi vermemek

---

# 19. Production Checklist

- [ ] Deprecation süreci tanımlı mı?
- [ ] Yeni endpoint hazır mı?
- [ ] Client'lar bilgilendirildi mi?
- [ ] Usage takip ediliyor mu?
- [ ] Sunset tarihi var mı?
- [ ] Removal planı var mı?
- [ ] 410 response hazır mı?

---

# 20. Sonuç

API lifecycle:

- teknik konu değildir
- ürün ve operasyon konusudur

Doğru yönetilirse:

- sistem stabil kalır
- client güveni artar

Yanlış yönetilirse:

- client kırılır
- kaos oluşur
- teknik borç büyür