---
tarih: 2025-01-01
konu: Git Branching, Merge, Rebase, Conflict Çözümleme
etiket: [git, branch, merge, rebase, conflict, dal, birleştirme]
kaynak:
zorluk: ⭐⭐⭐
---

## 📌 Özet

Branch (dal), ana geliştirme hattından bağımsız çalışmanızı sağlar. Merge ve rebase, dalları birleştirmenin iki farklı yoludur. Conflict çözümü en kritik beceridir.

---

## 🧠 Detay

### Branch Yönetimi

```bash
# Branch listesi
git branch                    # Yerel dallar
git branch -r                 # Uzak dallar
git branch -a                 # Hepsi
git branch -v                 # Her dalın son commit'i
git branch --merged           # Merge edilmiş dallar
git branch --no-merged        # Merge edilmemiş dallar

# Branch oluştur
git branch feature/login      # Oluştur (geçme)
git checkout -b feature/login  # Oluştur ve geç
git switch -c feature/login    # Git 2.23+ (önerilen)

# Branch'e geç
git checkout feature/login
git switch feature/login       # Önerilen

# Branch sil
git branch -d feature/login    # Merge edildiyse sil
git branch -D feature/login    # Zorla sil (merge edilmeden)

# Uzak branch sil
git push origin --delete feature/login
git push origin :feature/login  # Eski yöntem

# Branch yeniden adlandır
git branch -m eski-ad yeni-ad
git branch -M main              # Zorla yeniden adlandır

# Uzaktan yerel branch oluştur
git checkout --track origin/feature/login
git switch -c feature/login origin/feature/login
```

### git merge

```bash
# Mevcut branch'e başka bir dal'ı merge et
git checkout main
git merge feature/login

# Fast-Forward Merge (dal ayrıldıktan sonra main değişmedi)
git merge feature/login          # Otomatik FF
git merge --ff-only feature/login  # Sadece FF, aksi halde hata

# 3-Way Merge (her iki taraf da ilerledi)
git merge feature/login          # Merge commit oluşturur

# No Fast-Forward — Her zaman merge commit oluştur
git merge --no-ff feature/login
git merge --no-ff -m "Merge: login özelliği eklendi" feature/login

# Merge iptal
git merge --abort

# Squash merge — Branch geçmişini tek commit yap
git merge --squash feature/login
git commit -m "feat: login özelliği (squashed)"
```

### Merge Türleri Görsel

```
Fast-Forward Merge:              No-Fast-Forward (--no-ff):
                                  (Her zaman merge commit)
main:  A──B                       main:  A──B──────M
              ↘                                   /
feature:       C──D               feature:  C──D─/

→ main: A──B──C──D                → main: A──B──M
  (Commit grafiği düz kalır)        (Merge commit görünür)
```

### git rebase

```bash
# Feature branch'ini main'in üstüne taşı
git checkout feature/login
git rebase main

# Rebase sürecinde:
# Conflict varsa → düzelt → git add → git rebase --continue
# İptal etmek için → git rebase --abort
# Adımı atlamak → git rebase --skip

# Interactive rebase — geçmişi düzenle
git rebase -i HEAD~3         # Son 3 commit'i düzenle
git rebase -i main           # main'den bu yana tüm commit'ler

# Interactive seçenekler:
# pick   → Commit'i koru
# reword → Mesajı değiştir
# edit   → Commit'i düzenle
# squash → Önceki commit ile birleştir (mesaj seçimi)
# fixup  → Önceki commit ile birleştir (mesajı at)
# drop   → Commit'i sil
# exec   → Shell komutu çalıştır

# Rebase ile remote güncelleme
git fetch origin
git rebase origin/main       # Pull --rebase
git pull --rebase            # Fetch + rebase (önerilen)
```

### Merge vs Rebase Karşılaştırması

```
git merge feature:              git rebase feature:
                                (feature dalını main'e yaz)
main:  A──B──C──M              main:  A──B──C──D'──E'
              /                                ↑
feature: D──E                  Orijinal: D──E (feature)

✅ Geçmişi olduğu gibi korur     ✅ Temiz, düz geçmiş
✅ Güvenli (push edilmiş için)   ✅ Pull request öncesi
✅ Tam zaman damgası             ❌ Geçmişi yeniden yazar
❌ Karmaşık graph (çok dalda)    ❌ Push edilmişse tehlikeli!
```

**Altın kural**: **Push edilmiş commitleri rebase yapma!**

### Conflict Çözümleme

```bash
# Conflict durumunda git status
git status
# Her iki tarafça değiştirilmiş: src/login.py

# Conflict işaretçileri
cat src/login.py
```

```python
<<<<<<< HEAD (mevcut branch — main)
def login(username, password):
    return authenticate(username, password)
=======
def login(username, password, remember=False):
    user = authenticate(username, password)
    if remember:
        create_session(user)
    return user
>>>>>>> feature/login (gelen branch)
```

```bash
# 1. Manuel düzelt (conflict işaretçilerini kaldır)
# 2. İstenen son hali yaz
# 3. Stage et
git add src/login.py

# 4. Merge tamamla
git commit                    # Merge commit mesajı açılır
# veya:
git rebase --continue         # Rebase ise

# Conflict çözüm araçları
git mergetool                 # Yapılandırılmış araç aç
git mergetool --tool=vimdiff
git mergetool --tool=vscode

# VS Code'u merge aracı olarak ayarla
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'

# Tek tarafı seç (conflict çözmek yerine)
git checkout --ours dosya.py    # Bizim versiyonu seç
git checkout --theirs dosya.py  # Karşı tarafı seç
git add dosya.py
```

### Branch Stratejileri

#### Git Flow

```
main ──────────────────────────────── v1.0 ─── v1.1
     \                               /   \
develop ─────────────────────────────     hotfix
        \           \               \
     feature/A   feature/B       release/1.0
```

```bash
# Git Flow araç setiyle
git flow init
git flow feature start login
git flow feature finish login
git flow release start 1.0
git flow release finish 1.0
git flow hotfix start kritik-hata
git flow hotfix finish kritik-hata
```

#### GitHub Flow (Basit — Önerilen)

```
main ───────────────────────────────────────►
      \         /      \              /
    feature   PR+Merge  feature    PR+Merge
```

```bash
# 1. main'den branch aç
git checkout -b feature/yeni-ozellik

# 2. Geliştir + commit et
git commit -m "feat: ..."

# 3. Push et ve PR aç
git push -u origin feature/yeni-ozellik

# 4. Code review + merge

# 5. Branch sil
git branch -d feature/yeni-ozellik
```

#### Trunk-Based Development

```bash
# Çok kısa ömürlü branch'ler (1-2 gün max)
# veya doğrudan main'e commit (küçük ekip)
git checkout -b fix/bug-123
# Geliştir
git push
# PR + Hızlı merge
```

### Cherry-Pick — Belirli Commit'i Taşı

```bash
# Başka bir branch'ten belirli commit'i al
git cherry-pick abc1234
git cherry-pick abc1234..def5678   # Aralık
git cherry-pick --no-commit abc1234  # Stage et, commit etme
git cherry-pick -x abc1234          # Kaynak commit bilgisini ekle

# Örnek: Hotfix'i hem main hem develop'a uygula
git checkout main
git cherry-pick hotfix-commit-hash
git checkout develop
git cherry-pick hotfix-commit-hash
```

---

## 💡 Bağlantılar
- [[Git - Temel Komutlar ve Workflow]]
- [[Git - Geçmişi Yeniden Yazma]]
- [[GitHub - Pull Request ve Code Review]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- git-scm.com/book/tr/v2/Git-Dallanması
- learngitbranching.js.org
