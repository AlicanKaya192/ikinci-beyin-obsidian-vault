---
tarih: 2025-01-01
konu: GitHub SSH, GPG İmzalama, 2FA, Token, Deploy Keys
etiket: [github, ssh, gpg, güvenlik, 2fa, token, deploy-key]
kaynak:
zorluk: ⭐⭐⭐
---

## 📌 Özet

GitHub güvenliği: SSH anahtarları ile şifresiz bağlantı, GPG ile commit imzalama, iki faktörlü kimlik doğrulama ve Personal Access Token yönetimi.

---

## 🧠 Detay

### SSH Anahtarı Oluşturma ve Ekleme

```bash
# ─── Yeni SSH anahtarı oluştur ───────────────
ssh-keygen -t ed25519 -C "ali@example.com"
# Dosya: ~/.ssh/id_ed25519 (private) ve id_ed25519.pub (public)
# Passphrase öner: gir (güvenlik için)

# RSA (eski sistemler için)
ssh-keygen -t rsa -b 4096 -C "ali@example.com"

# ─── SSH Agent ────────────────────────────────
# Agent başlat
eval "$(ssh-agent -s)"

# Anahtarı agent'a ekle
ssh-add ~/.ssh/id_ed25519
ssh-add --apple-use-keychain ~/.ssh/id_ed25519  # Mac

# Anahtarları listele
ssh-add -l

# ─── Public key'i kopyala ─────────────────────
cat ~/.ssh/id_ed25519.pub
# veya:
pbcopy < ~/.ssh/id_ed25519.pub        # Mac
xclip -sel clip < ~/.ssh/id_ed25519.pub  # Linux
clip < ~/.ssh/id_ed25519.pub          # Windows

# ─── GitHub'a ekle ────────────────────────────
# GitHub → Settings → SSH and GPG Keys → New SSH Key
# Title: "Laptop - Ali"
# Key type: Authentication Key
# Key: (public key yapıştır)

# ─── Bağlantıyı test et ──────────────────────
ssh -T git@github.com
# "Hi ali! You've successfully authenticated..."

# Birden fazla GitHub hesabı için
# ~/.ssh/config dosyasını düzenle:
cat ~/.ssh/config
```

```
# ~/.ssh/config
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519

Host github-work
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_work

# Kullanım:
# git clone git@github-work:company/repo.git
```

```bash
# ─── HTTPS → SSH'a geç ───────────────────────
# Mevcut remote URL'i kontrol et
git remote get-url origin
# https://github.com/user/repo.git

# SSH'a çevir
git remote set-url origin git@github.com:user/repo.git
git remote get-url origin
# git@github.com:user/repo.git
```

### GPG ile Commit İmzalama

```bash
# ─── GPG Anahtar Oluştur ─────────────────────
gpg --full-generate-key
# Tür: RSA and RSA
# Boyut: 4096
# Süre: 0 (süresiz) veya 2y
# Ad ve email: GitHub ile aynı!

# Anahtar listele
gpg --list-secret-keys --keyid-format=long

# Çıktı:
# sec   4096R/3AA5C34371567BD2 2016-03-10
# uid   Ali Yılmaz <ali@example.com>

# Public key al
gpg --armor --export 3AA5C34371567BD2

# ─── GitHub'a ekle ────────────────────────────
# GitHub → Settings → SSH and GPG Keys → New GPG Key

# ─── Git'i GPG ile yapılandır ─────────────────
git config --global user.signingkey 3AA5C34371567BD2
git config --global commit.gpgsign true    # Her commit'i imzala
git config --global tag.gpgsign true       # Tag'ları da imzala

# gpg programının yolu (Mac)
git config --global gpg.program gpg

# ─── İmzalı commit ────────────────────────────
git commit -S -m "feat: güvenli commit"
git tag -s v1.0.0 -m "İmzalı sürüm"

# İmzayı kontrol et
git log --show-signature -1
git verify-commit HEAD
git verify-tag v1.0.0

# GitHub'da "Verified" rozeti görünür
```

### Personal Access Token (PAT)

```bash
# ─── Token Oluştur ────────────────────────────
# GitHub → Settings → Developer settings → Personal access tokens

# Fine-grained token (önerilen — spesifik repo ve izinler):
# - Repository access: Belirli repolar
# - Permissions: Contents (read/write), Pull requests (read/write), vb.

# Classic token (geniş izinler):
# - Scopes: repo, workflow, write:packages

# ─── Token Kullanımı ──────────────────────────
# HTTPS clone ile
git clone https://TOKEN@github.com/user/repo.git

# Git credential store
git config --global credential.helper store
# ~/.git-credentials dosyasına yazar (dikkatli — şifresiz)

# Git credential cache (geçici, 15 dakika)
git config --global credential.helper 'cache --timeout=3600'

# macOS Keychain
git config --global credential.helper osxkeychain

# Windows Credential Manager
git config --global credential.helper manager

# CI/CD için environment variable
export GITHUB_TOKEN=ghp_xxx
git clone https://x-access-token:$GITHUB_TOKEN@github.com/user/repo.git

# gh CLI ile token kullanımı
echo $GITHUB_TOKEN | gh auth login --with-token
```

### Deploy Keys

```bash
# Belirli bir repo için read-only SSH anahtarı (sunucu deployment için)

# ─── Deploy key oluştur ───────────────────────
ssh-keygen -t ed25519 -C "deploy@myserver" -f ~/.ssh/deploy_key
# Passphrase: boş bırak (otomatik deployment için)

# ─── GitHub'a ekle ────────────────────────────
cat ~/.ssh/deploy_key.pub
# GitHub → Repo → Settings → Deploy keys → Add
# Allow write access: Production için genellikle HAYIR (read-only)

# ─── Sunucuda kullan ──────────────────────────
# ~/.ssh/config
Host github.com-myapp
  HostName github.com
  User git
  IdentityFile ~/.ssh/deploy_key

# Kullanım:
git clone git@github.com-myapp:user/repo.git
```

### GitHub Apps ve OAuth

```bash
# GitHub Apps — Repository bazlı token
# Settings → Developer settings → GitHub Apps → New GitHub App
# Daha güvenli: Kısa ömürlü token (1 saat), spesifik izinler

# Installation token al (Actions'ta)
- uses: actions/create-github-app-token@v1
  id: app-token
  with:
    app-id: ${{ secrets.APP_ID }}
    private-key: ${{ secrets.APP_PRIVATE_KEY }}

# Token kullan
- run: gh pr create ...
  env:
    GITHUB_TOKEN: ${{ steps.app-token.outputs.token }}
```

### 2FA ve Hesap Güvenliği

```bash
# GitHub 2FA aktifleştirme:
# Settings → Password and authentication → Enable two-factor authentication

# 2FA seçenekleri:
# ✅ Authenticator app (TOTP) — GitHub Mobile, Authy, Google Authenticator
# ✅ Security keys (YubiKey, FIDO2) — En güvenli
# ✅ GitHub Mobile
# ❌ SMS — Güvensiz (SIM swap saldırısı)

# Recovery codes → Güvenli yerde sakla!

# Passkeys (WebAuthn — en yeni)
# Settings → Password and authentication → Add a passkey
```

### Token Sızması Durumunda

```bash
# 1. Hemen GitHub'da tokeni iptal et
# Settings → Developer settings → Personal access tokens → Revoke

# 2. Commit geçmişinden temizle
git filter-repo --replace-text <(echo 'ghp_ESKİTOKEN==>REDACTED')
git push --force --all

# 3. GitHub'ın Secret Scanning özelliğini aktifleştir
# Settings → Code security → Secret scanning

# 4. Pre-commit hooks ile önle
pip install pre-commit detect-secrets
# .pre-commit-config.yaml:
# - repo: https://github.com/Yelp/detect-secrets
#   hooks:
#   - id: detect-secrets
```

---

## 💡 Bağlantılar
- [[GitHub - Repository Yönetimi]]
- [[GitHub - Actions ve CI-CD]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- docs.github.com/en/authentication
- docs.github.com/en/authentication/connecting-to-github-with-ssh
