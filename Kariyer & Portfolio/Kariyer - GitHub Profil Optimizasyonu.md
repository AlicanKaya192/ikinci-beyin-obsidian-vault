---
tarih: 2026-08-17
konu: Kariyer & Portfolio
etiket: [kariyer, github, profil, portfolio, açık-kaynak]
kaynak: GitHub Docs, GitHub Blog
zorluk: başlangıç
---

## 📌 Özet

GitHub profili, DS/ML dünyasında CV'nin dijital versiyonu. İşe alım uzmanları koduna bakar, contribution graph'ına bakar, README'ye bakar. Bu not, profilini işe alım mıknatısına çeviren adımları listeler.

---

## 🧠 Detay

### Profil README (github.com/kullanıcıadı)

GitHub, `kullanıcıadı/kullanıcıadı` adında özel bir repo oluşturmana izin verir — bu repo'nun `README.md` dosyası profil sayfanda görünür.

**Etkili profil README yapısı:**

```markdown
# Merhaba, ben [Ad] 👋

🔭 Şu an: [Çalıştığın şey / öğrendiğin konu]
🌱 Öğreniyorum: [Yeni beceri]
💬 Konuş benimle: Python, ML, Data Engineering
📫 İletişim: [email veya LinkedIn]

## 🛠️ Teknolojiler
![Python](badge) ![PyTorch](badge) ![Docker](badge)

## 📊 GitHub Stats
[github-readme-stats widget]

## 🚀 Öne Çıkan Projeler
| Proje | Açıklama | Teknolojiler |
|-------|----------|--------------|
| [ChurnPredictor](link) | Müşteri kaybı tahmini | Python, XGBoost |
| [RAG-TR](link) | Türkçe RAG sistemi | LangChain, pgvector |
```

### Contribution Graph — Yeşil Kareler

```
Günde en az 1 commit → Yeşil kare
İşe alım uzmanları: "Aktif geliştirici mi?"

Stratejiler:
  ✓ Her gün küçük iyileştirme commit'i
  ✓ README güncellemeleri sayılır
  ✓ Private repo commit'leri de görünür (settings'den aç)
  ✗ Tek seferde sahte toplu commit — anlaşılır
```

### Repo Yapısı — Kaliteli Repo Anatomisi

```
proje-adi/
├── README.md          ← İLK BAKILAN YER
├── notebooks/
│   └── 01_eda.ipynb
├── src/
│   ├── __init__.py
│   ├── preprocess.py
│   └── model.py
├── tests/
│   └── test_model.py
├── requirements.txt
├── Dockerfile         ← Bonus puan
├── .github/
│   └── workflows/     ← CI/CD varsa çok iyi
└── data/
    └── README.md      ← Veri nerede bulunur
```

### Pinned Repos — 6 Slot Kullan

GitHub profilde 6 repo öne çıkarabilirsin. Seçim kriterleri:

```
Öncelik sırası:
  1. En iyi deployment'lı proje
  2. En yüksek yıldız alan repo
  3. En son ve aktif proje
  4. Domain uzmanlığı gösteren proje
  5. Open source katkı
  6. Blog/notebook serisi
```

### Star ve Fork Artırma

```bash
# Başkalarının işine yarayan şeyler yap:
✓ Cheatsheet notları (tek dosya, yüksek değer)
✓ Türkçe kaynak — neredeyse yok, dolayısıyla rekabetsiz
✓ Gerçek dünya veri seti ile proje
✓ Video tutorial ile eşleştirme (YouTube → repo)
✓ awesome-* listelerine PR gönder
```

### Topics ve Tags — Keşfedilirlik

Her repo'ya topic ekle (repo → Settings → Topics):

```
data-science, machine-learning, python, turkish, 
deep-learning, nlp, portfolio, tutorial
```

GitHub `Explore` ve `Topics` sayfaları buradan beslendir.

### README Badge'leri

```markdown
![Python](https://img.shields.io/badge/Python-3.11-blue?logo=python)
![License](https://img.shields.io/badge/License-MIT-green)
![Stars](https://img.shields.io/github/stars/user/repo)
```

### Open Source Katkı — Başlangıç

```
İlk katkı için ideal repo'lar:
  - "good first issue" etiketi ara
  - Türkçe çeviri PR'ları (belgeler her zaman eksik)
  - Hata raporu bile değer katar
  - scikit-learn, pandas, Hugging Face

Katkı süreci:
  Fork → Clone → Branch → Değişiklik → PR
```

### Profil Kontrol Listesi

```
☐ Profile picture (gerçek fotoğraf veya profesyonel avatar)
☐ Bio — 160 karakter, teknik + kişisel
☐ Location — opsiyonel ama güven verir
☐ Company / Üniversite
☐ Website → LinkedIn veya portfolio
☐ Pinned repos → 6/6 dolu
☐ README profil repo → var ve güncel
☐ Contribution graph → son 6 ayda aktif
☐ En az 3 repo → açıklama + topics dolu
```

---

## 💡 Bağlantılar
- [[Kariyer - Data Science Portfolio Proje Fikirleri]]
- [[Kariyer - CV ve LinkedIn Hazırlama]]
- [[Git - Temel Komutlar ve Workflow]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- [GitHub Profile README Generator](https://rahuldkjain.github.io/gh-profile-readme-generator/)
- [GitHub Readme Stats](https://github.com/anuraghazra/github-readme-stats)
- [Shields.io](https://shields.io/)
