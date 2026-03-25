# API Versioning & Compatibility Guide

Bu doküman, production ortamında çalışan API'lerde versioning stratejisi ve backward compatibility kurallarını tanımlar.

Amaç:

- API değişikliklerini kontrollü şekilde yönetmek
- Breaking change'leri minimize etmek
- Client'ların stabil çalışmasını sağlamak
- API evrimini öngörülebilir hale getirmek

---

# 1. Temel İlke

Bir API:

- bir kez publish edildikten sonra değiştirilemez
- sadece genişletilebilir

Bu yüzden:

> API design = contract design

Yanlış yapılan versioning:
- client kırar
- rollout sürecini zorlaştırır
- production incident üretir

---

# 2. Breaking vs Non-Breaking Change

## 2.1 Breaking Change

Client'ın mevcut kodunu bozan değişikliktir.

Örnekler:

- response field silmek
- field tipini değiştirmek
- required field eklemek
- endpoint path değiştirmek
- HTTP method değiştirmek
- response shape değiştirmek

Örnek:

    {
      "name": "Product A"
    }

↓

    {
      "title": "Product A"
    }

Bu breaking change'dir.

---

## 2.2 Non-Breaking Change

Client'ı bozmaz.

Örnekler:

- yeni field eklemek
- optional field eklemek
- yeni endpoint eklemek

Örnek:

    {
      "name": "Product A",
      "price": "100"
    }

↓

    {
      "name": "Product A",
      "price": "100",
      "currency": "TRY"
    }

Bu non-breaking'dir.

---

# 3. Versioning Zorunlu mu?

Evet.

Production sistemlerde:

- versioning olmadan API publish etmek risklidir
- değişiklik yönetimi yapılamaz
- rollback zorlaşır

---

# 4. Versioning Stratejileri

## 4.1 Path Versioning (ÖNERİLEN)

En yaygın ve en stabil yöntem.

    /api/v1/users
    /api/v2/users

Avantaj:
- açık ve anlaşılır
- gateway ve routing kolay
- debug kolay

---

## 4.2 Header Versioning

    Accept: application/vnd.company.v1+json

Avantaj:
- URL temiz kalır

Dezavantaj:
- debug zor
- client implementasyonu zor
- gateway uyumu zor

---

## 4.3 Query Versioning (ÖNERİLMEZ)

    /api/users?version=1

Bu yaklaşım production'da tercih edilmez.

---

# 5. Önerilen Standart

Her API şu pattern'i kullanmalı:

    /api/v1/{resource}

Örnek:

    /api/v1/users
    /api/v1/orders
    /api/v1/products

---

# 6. Version Ne Zaman Artırılır?

Sadece breaking change olduğunda.

Non-breaking change'ler için:

- yeni field ekle
- version artırma

---

## 6.1 Version Artırılması Gereken Durumlar

- response field silindi
- field tipi değişti
- endpoint kaldırıldı
- request contract değişti
- semantic değişiklik oldu

---

## 6.2 Version Artırılmaması Gereken Durumlar

- yeni field eklendi
- yeni endpoint eklendi
- optional param eklendi
- performans iyileştirildi

---

# 7. Backward Compatibility Kuralları

Bir API version publish edildikten sonra:

- response field silinmez
- mevcut field değiştirilmez
- meaning değişmez

---

## 7.1 Field Removal Yasağı

Yanlış:

    {
      "name": "Product"
    }

↓

    {
      "title": "Product"
    }

Doğru:

    {
      "name": "Product",
      "title": "Product"
    }

Sonra:

- eski field deprecated edilir
- yeni version ile kaldırılır

---

## 7.2 Field Tipi Değiştirme Yasağı

Yanlış:

    "price": "100.00"

↓

    "price": 100

Bu breaking change'dir.

---

# 8. Deprecation Süreci

Bir field veya endpoint kaldırılacaksa:

1. Deprecated olarak işaretle
2. Dokümantasyonda belirt
3. Client'lara duyur
4. Yeni version çıkar
5. Eski version'u kapat

---

## 8.1 Deprecation Örneği

    {
      "name": "Product",
      "old_field": "deprecated"
    }

Dokümantasyon:

- old_field deprecated
- v2'de kaldırılacak

---

# 9. Parallel Version Çalıştırma

Production sistemlerde:

- birden fazla version aynı anda çalışmalıdır

Örnek:

    /api/v1/users
    /api/v2/users

Neden?

- client migration zaman alır
- mobil app'ler hemen update edilmez
- backward compatibility gerekir

---

# 10. Version Migration Stratejisi

Yeni version çıkarken:

1. v2 deploy edilir
2. v1 çalışmaya devam eder
3. client'lar migrate edilir
4. usage izlenir
5. v1 kapatılır

---

# 11. Version'lar Arası Davranış Farkı

Version'lar:

- farklı contract'a sahip olabilir
- ama aynı domain'i temsil eder

Örnek:

v1:

    {
      "price": "100.00"
    }

v2:

    {
      "price": 100,
      "currency": "TRY"
    }

---

# 12. Version'ı Route İçine Göm

Yanlış:

    /users/v1
    /v1/api/users

Doğru:

    /api/v1/users

---

# 13. Version'ı Saklama Yasağı

Version:

- header'da gizlenmemeli
- config'te saklanmamalı
- implicit olmamalı

Version her zaman açık olmalıdır.

---

# 14. Client Perspektifi

Client için API:

- predictable olmalı
- sürpriz değişiklik içermemeli

Bu yüzden:

- breaking change → yeni version
- eski version → stabil kalır

---

# 15. Gateway ve Routing

API Gateway:

- version'a göre routing yapabilmelidir
- log ve metric version bazlı tutulmalıdır

Örnek:

- v1 latency
- v2 error rate

---

# 16. Version Explosion Risk

Her küçük değişiklikte version artırmak yanlıştır.

Bu yüzden:

- sadece breaking change için version artır
- minor değişiklikleri aynı version'da tut

---

# 17. Contract Testing (ÖNERİ)

Version yönetimi için:

- contract test yazılmalıdır
- response schema doğrulanmalıdır

Bu sayede:
- breaking change erken yakalanır

---

# 18. Production Checklist

Yeni değişiklik yapmadan önce:

- [ ] Bu değişiklik breaking mi?
- [ ] Yeni version gerekiyor mu?
- [ ] Eski client'lar etkilenir mi?
- [ ] Deprecation planı var mı?
- [ ] Gateway routing hazır mı?
- [ ] Monitoring version bazlı mı?

---

# 19. Sonuç

API versioning:

- sadece path değişimi değildir
- contract yönetimidir
- production risk yönetimidir

Doğru versioning:
- sistem stabil kalır
- client güveni artar
- rollout kolaylaşır

Yanlış versioning:
- client kırılır
- deploy riski artar
- teknik borç büyür