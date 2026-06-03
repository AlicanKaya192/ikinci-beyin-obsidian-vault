---
tarih: 2025-01-01
konu: Git Giriş, Versiyon Kontrol, Temel Kavramlar, Kurulum
etiket: [git, versiyon-kontrol, temel, kurulum, repository]
kaynak:
zorluk: ⭐
---

## 📌 Özet

Git, dağıtık versiyon kontrol sistemidir. Kod değişikliklerini takip eder, ekip çalışmasını kolaylaştırır ve her zaman önceki sürüme dönmeyi mümkün kılar. Her yazılım projesinin temelidir.

---

## 🧠 Detay

### Neden Git?

```
Versiyon Kontrolsüz:         Git ile:
proje_v1.zip                 git log
proje_v2.zip                 → Her değişiklik kayıtlı
proje_FINAL.zip              → Kim, ne zaman, neden değiştirdi?
proje_GERCEKSON.zip          → İstediğin versiyona dön
proje_bu_sefer_son.zip       → Ekiple paralel çalış
```

### Temel Kavramlar

| Kavram | Açıklama |
|---|---|
| **Repository (Repo)** | Projenin tüm geçmişini içeren Git veritabanı |
| **Commit** | Belirli bir andaki değişikliklerin anlık görüntüsü |
| **Branch** | Bağımsız geliştirme hattı |
| **Merge** | İki branch'i birleştirme |
| **Remote** | Uzak sunucudaki repo (GitHub, GitLab) |
| **Clone** | Uzak repoyu yerel makineye kopyalama |
| **Push** | Yerel commitleri uzak repoya gönderme |
| **Pull** | Uzak repodaki değişiklikleri indirme |
| **Working Directory** | Üzerinde çalıştığın dosyalar |
| **Staging Area (Index)** | Commit'e hazır değişiklikler |
| **HEAD** | Şu an bulunduğun commit veya branch |

### Git'in Üç Bölgesi

```
Çalışma         Staging         Git
Dizini          Alanı           Repository
(Working Dir)   (Index)         (.git/)

  [dosya]  →  git add  →  [staged]  →  git commit  →  [commit]
             ←  git restore  ←         ← git reset HEAD~ ←
```

### Kurulum

```bash
# Ubuntu / Debian
sudo apt update && sudo apt install git

# Mac
brew install git

# Windows
# git-scm.com/download/win

# Versiyon kontrol
git --version
```

### İlk Yapılandırma (Zorunlu!)

```bash
# Kimlik bilgileri — her commit'te görünür
git config --global user.name "Ali Yılmaz"
git config --global user.email "ali@example.com"

# Varsayılan branch adı
git config --global init.defaultBranch main

# Varsayılan editör
git config --global core.editor "code --wait"   # VS Code
git config --global core.editor "vim"

# Satır sonu (Windows için)
git config --global core.autocrlf true   # Windows
git config --global core.autocrlf input  # Mac/Linux

# Renkli çıktı
git config --global color.ui auto

# Alias'lar (kısayollar)
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.lg "log --oneline --graph --decorate --all"

# Yapılandırmayı gör
git config --list
git config --global --list
cat ~/.gitconfig
```

### İlk Repository Oluşturma

```bash
# Yeni proje başlat
mkdir myproject && cd myproject
git init
# → .git/ klasörü oluştu (burası Git veritabanı)

# Mevcut projeye Git ekle
cd existing-project
git init

# Uzak repoyu klonla
git clone https://github.com/kullanici/repo.git
git clone https://github.com/kullanici/repo.git benim-klasorum
git clone git@github.com:kullanici/repo.git   # SSH ile
```

### Temel Workflow

```bash
# 1. Durum kontrol
git status

# 2. Değişiklikleri staging'e ekle
git add dosya.py
git add .                    # Tüm değişiklikler
git add *.py                 # Belirli pattern
git add -p                   # Etkileşimli (parça parça)

# 3. Commit yap
git commit -m "feat: ürün arama özelliği eklendi"
git commit -am "fix: bağlantı hatası düzeltildi"  # add + commit

# 4. Geçmişi gör
git log
git log --oneline
git log --oneline --graph --all

# 5. Uzak repoya gönder
git push origin main
git push -u origin main      # upstream ayarla (ilk kez)

# 6. Uzak repodan çek
git pull origin main
git fetch origin             # İndir ama merge etme
```

### .gitignore

```bash
# .gitignore dosyası — takip edilmeyecekler

# Python
__pycache__/
*.pyc
*.pyo
.venv/
venv/
env/
.env
*.egg-info/
dist/
build/
.pytest_cache/

# Node.js
node_modules/
.npm
*.log
dist/

# IDE
.vscode/
.idea/
*.swp
*.swo
.DS_Store
Thumbs.db

# Ortam değişkenleri
.env
.env.local
.env.production
secrets.json

# Docker
docker-compose.override.yml

# Derleme çıktıları
*.o
*.exe
*.dll
*.so

# Belirli dosyayı yoksay (ama klasörü değil)
/build

# Desenle eşleşme
*.log
!important.log  # Bu bir exception (takip et)
```

```bash
# Global gitignore (tüm projeler için)
git config --global core.excludesfile ~/.gitignore_global

# gitignore oluşturucu
# gitignore.io — dil/IDE/OS seç, dosyayı indir
```

### Commit Mesajı Kuralları

```bash
# Conventional Commits formatı
<type>(<scope>): <kısa açıklama>

<isteğe bağlı uzun açıklama>

<isteğe bağlı footer (issue referansı vb.)>

# Tipler:
feat:     Yeni özellik
fix:      Hata düzeltme
docs:     Sadece dokümantasyon
style:    Kod stili (boşluk, noktalı virgül vb.)
refactor: Ne hata düzeltme ne yeni özellik
test:     Test ekleme/düzeltme
chore:    Build, bağımlılık güncelleme
ci:       CI yapılandırması
perf:     Performans iyileştirme

# Örnekler:
git commit -m "feat(auth): JWT token yenileme eklendi"
git commit -m "fix(api): null pointer hatası düzeltildi (#123)"
git commit -m "docs: README güncellendi"
git commit -m "refactor(db): bağlantı havuzu optimize edildi"
```

---

## 💡 Bağlantılar
- [[Git - Temel Komutlar ve Workflow]]
- [[Git - Branching ve Merging]]
- [[GitHub - Repository Yönetimi]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- git-scm.com/book (Pro Git — ücretsiz)
- learngitbranching.js.org (interaktif)
- ohshitgit.com (hata kurtarma)
