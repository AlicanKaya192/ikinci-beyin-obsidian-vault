---
tarih: 2025-01-01
konu: Git Geçmiş Yeniden Yazma, Rebase -i, Reflog, Bisect, Worktree
etiket: [git, rebase, reflog, bisect, worktree, tag, ileri-düzey]
kaynak:
zorluk: ⭐⭐⭐⭐
---

## 📌 Özet

Git'in ileri düzey araçları: Interactive rebase ile geçmiş temizleme, reflog ile "silinen" commitleri kurtarma, bisect ile hata bulma ve worktree ile çoklu dal aynı anda.

---

## 🧠 Detay

### Interactive Rebase — Geçmişi Temizle

```bash
# Son 4 commit'i düzenle
git rebase -i HEAD~4

# Editörde açılır:
# pick abc1234 feat: kullanıcı listesi
# pick def5678 fix: yazım hatası
# pick ghi9012 fix: başka yazım hatası
# pick jkl3456 feat: kullanıcı silme

# ─── Squash: İki küçük fix'i birleştir ──────
# pick abc1234 feat: kullanıcı listesi
# squash def5678 fix: yazım hatası   ← öncekiyle birleştir
# squash ghi9012 fix: başka yazım hatası
# pick jkl3456 feat: kullanıcı silme

# ─── Reword: Mesajı düzelt ──────────────────
# reword abc1234 feat: kullanıcı listesi  ← mesaj editörde açılır

# ─── Fixup: Sessiz squash ───────────────────
# pick abc1234 feat: kullanıcı listesi
# fixup def5678 fix: yazım hatası   ← mesaj atılır

# ─── Drop: Commit'i sil ─────────────────────
# drop ghi9012 test commit  ← tamamen kaldır

# ─── Reorder: Sırayı değiştir ───────────────
# pick jkl3456 feat: kullanıcı silme  ← satırları taşı
# pick abc1234 feat: kullanıcı listesi

# ─── Edit: Commit'i böl ─────────────────────
# edit abc1234 feat: büyük commit
# Rebase durur → istediğin değişikliği yap
# git reset HEAD~1
# git add -p (parça parça ekle)
# git commit -m "feat: parça 1"
# git commit -m "feat: parça 2"
# git rebase --continue
```

### git reflog — Zaman Makinesi

```bash
# HEAD'in tüm geçmişi (commit, checkout, reset her şey)
git reflog
git reflog show HEAD
git reflog --all

# Çıktı:
# abc1234 HEAD@{0}: commit: feat: yeni özellik
# def5678 HEAD@{1}: checkout: moving from main to feature
# ghi9012 HEAD@{2}: reset: moving to HEAD~3
# jkl3456 HEAD@{3}: commit: eski commit

# "Silinen" commit'i kurtar
git checkout HEAD@{3}            # O anki duruma git
git checkout -b kurtarma-dali    # Branch olarak kaydet
git cherry-pick HEAD@{3}         # Belirli commit'i al

# Yanlışlıkla reset --hard yapıldı → Kurtar!
git reset --hard HEAD@{2}        # Önceki konuma dön

# Reflog süresi (varsayılan 90 gün)
git config gc.reflogExpire 90 days
```

### git tag — Sürümleme

```bash
# Lightweight tag (sadece işaret)
git tag v1.0.0
git tag v1.0.0 abc1234         # Belirli commit'e

# Annotated tag (önerilen — imza, mesaj, tarih)
git tag -a v1.0.0 -m "İlk kararlı sürüm"
git tag -a v1.0.0 abc1234 -m "Sürüm 1.0.0"

# Tag listele
git tag
git tag -l "v1.*"              # Filtreyle
git show v1.0.0                # Tag detayı

# Tag'ı push et (ayrıca push gerekir!)
git push origin v1.0.0
git push origin --tags         # Tüm tag'ları push et

# Tag sil
git tag -d v1.0.0-beta
git push origin --delete v1.0.0-beta

# Tag'a geç
git checkout v1.0.0            # Detached HEAD modunda
git checkout -b hotfix/v1.0.1 v1.0.0  # Tag'dan branch aç

# Semantic versioning
# MAJOR.MINOR.PATCH
# 1.0.0 → İlk kararlı sürüm
# 1.1.0 → Yeni özellik eklendi (geriye uyumlu)
# 1.1.1 → Hata düzeltme
# 2.0.0 → Geriye uyumsuz değişiklik
```

### git bisect — Hatayı Bul

```bash
# Hangi commit hatayı yarattı? Binary search ile bul

git bisect start
git bisect bad                 # Şu an bozuk
git bisect good v1.0.0         # Bu tag'da iyiydi

# Git otomatik olarak ortadaki commit'e atar
# Test et, sonra:
git bisect good                # Bu commit iyi
git bisect bad                 # Bu commit bozuk

# Git daraltmaya devam eder...
# Sonunda: "abc1234 is the first bad commit"

# Bitir
git bisect reset               # Başlangıç durumuna dön

# Otomatik bisect (test komutu ile)
git bisect start HEAD v1.0.0
git bisect run python -m pytest tests/test_login.py
# → Otomatik olarak bozuk commit'i bulur
```

### git worktree — Birden Fazla Branch Aynı Anda

```bash
# Aynı anda birden fazla branch'i farklı dizinlerde aç
git worktree add ../hotfix-branch hotfix/v1.0.1
git worktree add ../review-branch feature/login

# Worktree listesi
git worktree list

# Bağlantısız worktree
git worktree add --detach ../temp-dir abc1234

# Worktree sil
git worktree remove ../hotfix-branch
git worktree prune              # Silinmiş dizinleri temizle

# Kullanım senaryosu:
# Ana projede feature geliştiriyorsun
# Acil hotfix geliyor
# → worktree ile farklı klasörde hotfix yap, ana çalışma bozulmaz
```

### git submodule — Alt Modüller

```bash
# Başka repo'yu alt modül olarak ekle
git submodule add https://github.com/org/library.git lib/library
git submodule add git@github.com:org/lib.git external/lib

# Submodule'lü repo klonla
git clone --recursive https://github.com/org/repo.git
# veya klonlandıktan sonra:
git submodule init
git submodule update

# Tüm submodule'leri güncelle
git submodule update --remote
git submodule update --init --recursive

# Submodule'de komut çalıştır
git submodule foreach 'git pull origin main'

# Submodule kaldır
git submodule deinit lib/library
git rm lib/library
rm -rf .git/modules/lib/library
```

### git archive — Snapshot Arşivi

```bash
# Belirli bir versiyonu ZIP olarak al
git archive --format=zip --output=v1.0.0.zip v1.0.0
git archive HEAD --format=tar.gz | gzip > snapshot.tar.gz
git archive --prefix=myapp-v1.0/ v1.0.0 | gzip > release.tar.gz
```

### git shortlog — Özet İstatistik

```bash
# Commit sayısına göre yazarlar
git shortlog -sn
git shortlog -sn --no-merges   # Merge commit'ler hariç

# Belirli dönem
git shortlog -sn --since="1 month ago"

# Contributors dosyası
git shortlog -sn > CONTRIBUTORS
```

### Tehlikeli Komutlar — Dikkat!

```bash
# ⚠️ Push edilmiş commitleri değiştirme
git push --force origin main          # Tehlikeli
git push --force-with-lease origin main  # Daha güvenli (başkası push ettiyse hata verir)

# ⚠️ Tüm geçmişten dosya sil (şifre açıldıysa)
# BFG Repo Cleaner (daha hızlı)
java -jar bfg.jar --delete-files gizli-dosya.txt
java -jar bfg.jar --replace-text passwords.txt

# git filter-branch (yavaş, eski)
git filter-branch --force --index-filter \
  'git rm --cached --ignore-unmatch gizli.txt' \
  --prune-empty --tag-name-filter cat -- --all

# git filter-repo (modern, hızlı)
pip install git-filter-repo
git filter-repo --path gizli.txt --invert-paths
git push --force --all
```

---

## 💡 Bağlantılar
- [[Git - Branching ve Merging]]
- [[GitHub - Actions ve CI-CD]]
- [[Git - Temel Komutlar ve Workflow]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- git-scm.com/docs/git-reflog
- git-scm.com/docs/git-bisect
- rtyley.github.io/bfg-repo-cleaner
