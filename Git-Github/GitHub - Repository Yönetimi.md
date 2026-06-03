---
tarih: 2025-01-01
konu: GitHub Repository, Fork, Issues, Releases, GitHub CLI
etiket: [github, repository, fork, issues, releases, gh-cli, remote]
kaynak:
zorluk: ⭐⭐
---

## 📌 Özet

GitHub, Git repository'lerini barındıran ve ekip işbirliğini sağlayan platformdur. Fork, Issues, Releases ve GitHub CLI ile verimli proje yönetimi.

---

## 🧠 Detay

### Remote Yönetimi

```bash
# Remote listele
git remote -v

# Remote ekle
git remote add origin https://github.com/user/repo.git
git remote add upstream https://github.com/original/repo.git

# Remote URL değiştir (HTTPS → SSH)
git remote set-url origin git@github.com:user/repo.git

# Remote sil
git remote remove upstream
git remote rename origin backup

# Remote bilgisi
git remote show origin

# Uzak değişiklikleri indir (merge etmeden)
git fetch origin
git fetch --all                    # Tüm remote'lar
git fetch --prune                  # Silinmiş uzak branch'leri temizle

# Pull = fetch + merge
git pull origin main
git pull --rebase origin main      # Merge yerine rebase (önerilen)
git pull --ff-only                 # Sadece fast-forward

# Push
git push origin main
git push -u origin feature/login   # Upstream ayarla
git push --all origin              # Tüm branch'ler
git push --tags                    # Tag'lar
git push origin --delete branch    # Uzak branch sil
```

### Fork Workflow (Open Source)

```bash
# 1. GitHub'da fork (UI'dan — sağ üst "Fork" butonu)

# 2. Fork'u klonla
git clone git@github.com:SENİN_KULLANICI/repo.git
cd repo

# 3. Upstream (orijinal) repo'yu ekle
git remote add upstream git@github.com:ORJİNAL/repo.git
git remote -v
# origin    git@github.com:SENİN/repo.git (fetch/push)
# upstream  git@github.com:ORJİNAL/repo.git (fetch)

# 4. Upstream'den güncelle
git fetch upstream
git checkout main
git merge upstream/main
# veya:
git rebase upstream/main

# 5. Feature branch oluştur
git checkout -b feature/yeni-ozellik

# 6. Geliştir + push
git push origin feature/yeni-ozellik

# 7. GitHub'da Pull Request aç (UI'dan)

# 8. PR merge edildikten sonra temizlik
git checkout main
git pull upstream main
git push origin main
git branch -d feature/yeni-ozellik
git push origin --delete feature/yeni-ozellik
```

### GitHub CLI (gh)

```bash
# Kurulum
brew install gh                    # Mac
sudo apt install gh                # Ubuntu

# Giriş
gh auth login
gh auth status

# ─── Repository İşlemleri ────────────────────
# Repo oluştur
gh repo create myapp --public --source=. --remote=origin
gh repo create myapp --private
gh repo create org/myapp --public

# Repo klonla
gh repo clone user/repo
gh repo clone user/repo -- --depth=1

# Repo fork et
gh repo fork original/repo
gh repo fork original/repo --clone

# Repo listele
gh repo list
gh repo list --public --limit 20

# Repo görüntüle
gh repo view
gh repo view user/repo --web   # Tarayıcıda aç

# ─── Issue İşlemleri ─────────────────────────
# Issue oluştur
gh issue create --title "Bug: Login hatası" --body "Detay..."
gh issue create --title "feat: Arama özelliği" --label "enhancement" --assignee "@me"

# Issue listele
gh issue list
gh issue list --state open --label bug
gh issue list --assignee "@me"

# Issue görüntüle ve yorum yap
gh issue view 42
gh issue comment 42 --body "Bakıyorum..."
gh issue close 42
gh issue reopen 42

# ─── Pull Request ────────────────────────────
# PR oluştur
gh pr create --title "feat: login özelliği" --body "Açıklama" --base main
gh pr create --draft --reviewer colleague1,colleague2

# PR listele
gh pr list
gh pr list --state merged

# PR görüntüle ve merge et
gh pr view 15
gh pr merge 15 --merge         # Regular merge
gh pr merge 15 --squash        # Squash merge
gh pr merge 15 --rebase        # Rebase merge
gh pr checkout 15              # PR'ı yerel olarak incele

# ─── Workflow (Actions) ──────────────────────
gh workflow list
gh workflow run deploy.yml
gh run list
gh run view 123456

# ─── Release ─────────────────────────────────
gh release create v1.0.0 --title "v1.0.0" --notes "İlk sürüm"
gh release create v1.1.0 --generate-notes    # Otomatik release notes
gh release list
gh release download v1.0.0

# ─── Genel ───────────────────────────────────
gh browse                          # Repo'yu tarayıcıda aç
gh browse --no-browser             # URL göster
gh gist create dosya.py            # Gist oluştur
gh gist list
gh api repos/user/repo/topics      # GitHub API'ye doğrudan erişim
```

### Repository Ayarları

```bash
# Branch protection (gh ile)
gh api repos/USER/REPO/branches/main/protection \
  --method PUT \
  --field required_pull_request_reviews='{"required_approving_review_count":1}' \
  --field required_status_checks='{"strict":true,"contexts":["ci/tests"]}' \
  --field enforce_admins=true

# Genel API kullanımı
gh api user                        # Kendi profil bilgileri
gh api repos/USER/REPO             # Repo bilgisi
gh api orgs/ORG/members            # Org üyeleri

# Secret ekle
gh secret set API_KEY
gh secret set DATABASE_URL --body "postgresql://..."
gh secret list
```

### Issues ve Labels

```bash
# Label oluştur
gh label create "priority: high" --color "d73a4a" --description "Acil"
gh label create "good first issue" --color "7057ff"

# Milestone oluştur
gh api repos/USER/REPO/milestones \
  --method POST \
  --field title="v1.0.0" \
  --field due_on="2024-12-31T00:00:00Z"

# Issue template (.github/ISSUE_TEMPLATE/bug_report.yml)
```

```yaml
# .github/ISSUE_TEMPLATE/bug_report.yml
name: Bug Raporu
description: Bir hata bildirin
title: "[BUG] "
labels: ["bug"]
body:
  - type: markdown
    attributes:
      value: "Hata bildirdiğiniz için teşekkürler!"
  - type: textarea
    id: description
    attributes:
      label: Hata Açıklaması
      placeholder: Ne oldu?
    validations:
      required: true
  - type: textarea
    id: steps
    attributes:
      label: Yeniden Oluşturma Adımları
      value: |
        1. '...' sayfasına git
        2. '...' butonuna tıkla
        3. Hatayı gör
  - type: dropdown
    id: version
    attributes:
      label: Versiyon
      options:
        - 1.0.0
        - 1.1.0
        - 2.0.0
```

### Releases

```bash
# Tag oluştur ve release yayınla
git tag -a v1.0.0 -m "İlk kararlı sürüm"
git push origin v1.0.0

# GitHub CLI ile release
gh release create v1.0.0 \
  --title "Sürüm 1.0.0" \
  --notes "## Yenilikler
  - Kullanıcı girişi eklendi
  - Dashboard yenilendi
  
  ## Hata Düzeltmeleri
  - #42 Login hatası düzeltildi" \
  dist/app-linux.tar.gz \   # Binary dosyalar
  dist/app-windows.zip

# Otomatik release notes (PR'lardan)
gh release create v1.1.0 --generate-notes --latest
```

### GitHub API

```bash
# gh ile API çağrısı
gh api /repos/USER/REPO/commits --jq '.[].commit.message'
gh api /repos/USER/REPO/pulls --method POST \
  --field title="yeni PR" \
  --field head="feature/login" \
  --field base="main"

# Repo istatistikleri
gh api repos/USER/REPO/stats/contributors
gh api repos/USER/REPO/stats/commit_activity

# Pagination
gh api repos/USER/REPO/issues --paginate | jq '.[].title'
```

---

## 💡 Bağlantılar
- [[GitHub - Pull Request ve Code Review]]
- [[GitHub - Actions ve CI-CD]]
- [[Git - Branching ve Merging]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- cli.github.com/manual
- docs.github.com
