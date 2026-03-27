# Git Branching & Merge Strategy Guide

Bu doküman, ekip içinde tutarlı, öngörülebilir ve production-safe bir Git workflow sağlamak amacıyla hazırlanmıştır.

Amaç:

> Korumalı branch kurallarına takılmadan, temiz commit geçmişi ile güvenli release süreçleri yönetmek.

---

# 🌳 Branch Yapısı

## Ana Branch'ler

### dev
- Geliştirme branch'i
- Tüm feature'lar buraya merge edilir

### stage
- Test / UAT ortamı
- Release öncesi doğrulama yapılır

### main
- Production (canlı) ortam
- Sadece onaylı ve test edilmiş kod gelir

---

# 🔄 Genel Akış

feature/* → dev → stage → main

NOT:
Doğrudan merge yerine promotion branch kullanılır.

---

# 🚀 PR Akışı

## 1. Feature Geliştirme

Branch:
feature/<short-description>

PR:
feature/* → dev

Merge yöntemi:
Squash and Merge

---

## 2. Dev → Stage Geçişi

Direkt dev → stage PR açılmaz

Adımlar:

git checkout dev
git pull
git checkout -b promote/dev-to-stage-YYYYMMDD

Gerekirse:

git fetch origin
git merge origin/stage

- Conflict varsa bu branch'te çözülür

PR:
promote/dev-to-stage-* → stage

Merge yöntemi:
Rebase Merge (tercih edilir)

---

## 3. Stage → Main Geçişi

Adımlar:

git checkout stage
git pull
git checkout -b promote/stage-to-main-YYYYMMDD

Gerekirse:

git fetch origin
git merge origin/main

- Conflict varsa bu branch'te çözülür

PR:
promote/stage-to-main-* → main

Merge yöntemi:
Rebase Merge (tercih edilir)

---

# ⚙️ Merge Stratejileri

## feature → dev
- Squash Merge kullanılır

Neden:
- Gereksiz commit kalabalığını önler
- Tek iş = tek commit mantığı sağlar

---

## dev → stage
- Rebase Merge tercih edilir
- Alternatif: promotion branch üzerinden merge

---

## stage → main
- Rebase Merge tercih edilir
- Alternatif: squash (ekip kararına bağlı)

---

# 🔐 Branch Protection Kuralları

## dev

- Require PR: Açık
- Required approvals: 1
- Dismiss stale approvals: Açık
- Require status checks: Açık
- Require conversation resolution: Açık
- Force push: Kapalı
- Direct push: Kapalı (önerilir)

---

## stage

- Require PR: Açık
- Required approvals: 2
- Require specific teams: Açık
- Require status checks: Açık
- Require up-to-date branch: Açık
- Require linear history: Açık
- Dismiss stale approvals: Açık
- Force push: Kapalı
- Direct push: Kapalı

---

## main

- Require PR: Açık
- Required approvals: 2 veya 3
- Require specific teams: Açık
- Require status checks: Açık
- Require up-to-date branch: Açık
- Require linear history: Açık
- Require conversation resolution: Açık
- Require latest review approval: Açık
- Force push: Kapalı
- Direct push: Kapalı

---

# 📌 Operasyonel Kurallar

## Kural 1
feature → dev her zaman squash merge

---

## Kural 2
dev → stage geçişi için promotion branch kullan

---

## Kural 3
stage → main geçişi için promotion branch kullan

---

## Kural 4
Conflict asla protected branch üzerinde çözülmez

Her zaman promotion branch'te çözülür

---

## Kural 5
main branch'e direkt dev'den PR açılmaz

---

# 🎯 Sonuç

Bu yapı sayesinde:

- Branch kurallarıyla çakışmazsın
- Conflict yönetimi kontrol altına alınır
- Commit geçmişi temiz kalır
- Release süreci netleşir
- Production riskleri azalır