# Git Commit Mesajları

Bu doküman ekibimizin git commit mesajlarında ortak bir dil kullanması için hazırlanmıştır. Amaç, daha düzenli, tutarlı ve anlaşılır bir commit geçmişi oluşturmaktır. Bu kurallar **Conventional Commits** adı verilen, genel kabul görmüş bir standarda dayanıyor. 

## Genel Kurallar:

- Commit mesajları **İngilizce** yazılmalıdır.
- Mesajlar aşağıdaki formata uymalıdır:
    - **tip(opsiyonel kapsam): açıklama**
- **tip** bölümü commit'in amacını belirtir.
- **kapsam** (opsiyonel) commit'in hangi modül, dosya veya bölümle ilgili olduğunu belirtir.
- **açıklama** commit'in kısa ve öz özetidir.
- İlk harf de dahil olmak üzere tamamı küçük olmalı, sonuna nokta konulmamalıdır.

### Tipler

Aşağıdaki tipler kullanılmalıdır:

- **feat**: Yeni bir özellik eklerken.
    - Örnek: `feat(auth): add login functionality`
- **fix**: Hataları düzeltirken.
    - Örnek: `fix(api): correct user authentication issue`
- **chore**: Kodu veya projeyi geliştirmeyen işlerle ilgili commit'ler (ör. derleme görevleri, bağımlılık güncellemeleri).
    - Örnek: `chore(deps): update project dependencies`
- **docs**: Yalnızca dökümantasyon değişiklikleri.
    - Örnek: `docs(README): update installation instructions`
- **style**: Kodun işleyişini değiştirmeyen, sadece stil düzenlemeleri (ör. boşluk, virgül ekleme gibi).
    - Örnek: `style(lint): apply linting rules`
- **refactor**: Bir özelliğin davranışını değiştirmeyen kod düzenlemeleri.
    - Örnek: `refactor(cart): improve performance of price calculation`
- **test**: Test ekleme veya güncelleme.
    - Örnek: `test(checkout): add unit tests for checkout`
- **perf**: Performans iyileştirmeleri.
    - Örnek: `perf(search): optimize search query`
- **build**: Derleme sistemi veya dış bağımlılıklarla ilgili değişiklikler.
    - Örnek: `build(ci): update CI configuration for new branch`
- **ci**: Sürekli entegrasyon (CI) ile ilgili değişiklikler.
    - Örnek: `ci(workflow): add test step to GitHub actions`
- **revert**: Önceki bir commit'i geri almak.
    - Örnek: `revert: revert feat(auth): add login functionality`

### Kapsam

Kapsam, hangi modülün veya bölümün değiştiğini belirtir. Opsiyonel olarak kullanılabilir, fakat proje büyüdükçe faydalıdır. Örnek kapsamlar:

- **auth**: Kimlik doğrulama modülü
- **api**: API ile ilgili değişiklikler
- **ui**: Kullanıcı arayüzü
- **db**: Veritabanı işlemleri

### Açıklama

- Açıklama kısa ve öz olmalı, maksimum 50 karakter uzunluğunda olmalıdır.
- Açıklamanın amacı, değişikliğin ne olduğunu anlamayı sağlamaktır. "Neden?" ve "Nasıl?" soruları, gerekirse commit mesajının gövdesinde açıklanabilir.

### Gövde (Opsiyonel)

Eğer commit mesajı uzun açıklama gerektiriyorsa, başlıktan sonra bir satır boşluk bırakılarak gövde yazılabilir.

- **Neden bu değişiklik yapıldı?**
- **Nasıl yapıldı?**

### Örnek Commit Mesajları:

```
feat(auth): add login functionality

fix(cart): correct price calculation issue

docs(README): update installation instructions

style(lint): apply linting rules to checkout component

perf(search): optimize search query performance
```

Bu kurallar belgesi, ekibin commit mesajlarını tutarlı hale getirerek, daha iyi bir proje yönetimi ve kod takibi sağlamak amacıyla hazırlanmıştır.