---
tarih: 2026-08-17
konu: Kariyer & Portfolio
etiket: [kariyer, kaggle, yarışma, data-science, portfolio]
kaynak: Kaggle Grandmaster blogları, fast.ai
zorluk: orta
---

## 📌 Özet

Kaggle sadece yarışma platformu değil — öğrenme ekosistemi, portfolio belgesi ve kariyer kapısı. Bu not, Kaggle'ı kariyer stratejisinin bir parçası olarak nasıl kullanacağını açıklar: Hangi yarışmaya katılmalı, nasıl ilerlemeli ve medal nasıl kazanılır.

---

## 🧠 Detay

### Kaggle Tier Sistemi

```
Novice      → Sadece kayıt oldu
Contributor → Notebook paylaştı, discussion yaptı
Expert      → 1 bronze medal (yarışma) veya 5 notebook/dataset medal
Master      → 1 gold + 2 silver (yarışma) veya birleşik eşik
Grandmaster → 5 gold + 1 solo gold
```

**Kariyer için hedef:** Expert seviyesi yeterli. Master+ araştırma pozisyonlarında fark yaratır.

### Kaggle Bölümleri ve Medal Sistemi

| Bölüm | Ne Yapılır | Medal Değeri |
|-------|-----------|--------------|
| **Competitions** | Yarışma — tahmin, sınıf. | En değerli |
| **Datasets** | Veri seti paylaş | Orta |
| **Notebooks** | Eğitici notebook | Orta |
| **Discussions** | Yorum, çözüm paylaş | En kolay |

### Yarışma Stratejisi — Başlangıçtan Finale

#### 1. Doğru Yarışmayı Seç

```python
yeni_başlayanlar = [
    "Getting Started competitions (sıfır tarih yok)",
    "Playground Series (aylık, eğitim odaklı)",
    "Tabular data yarışmaları (başlangıç)"
]

orta_seviye = [
    "Featured competitions (para ödüllü)",
    "Kaggle Days yarışmaları"
]

ileri = [
    "Solo yarışmalar (GM için gerekli)",
    "Araştırma odaklı (CV, NLP)"
]
```

#### 2. Yarışmaya Katılım Süreci

```
HAFTA 1: Anlama
  → Problem türünü belirle
  → Baseline model kur (basit, hızlı)
  → Discussion tab'ı oku — veri ipuçları burada

HAFTA 2-N: İyileştirme
  → Feature engineering (en büyük etki)
  → Model ensembling
  → Cross-validation stratejisi
  → Public LB score ≠ Private LB score (dikkat!)

SON HAFTA: Ensemble
  → Farklı model ailelerini karıştır
  → Stacking/Blending
  → Shake-up riskini azalt: CV güven
```

#### 3. LB (Leaderboard) Shake-up Nedir?

```
Public LB: Testin %30'u ile hesaplanır → submission sırasında görürsün
Private LB: Testin %70'i → yarışma bittikten sonra açılır

Shake-up: Public'te iyi olan özel LB'de düşebilir
  Neden? Overfit to public test

Çözüm: Kendi CV (cross-validation) skorunu güven
  CV skoru > Public LB skoru ise overfitting var demektir
```

### Notebook Medalyası Kazanmak

Notebooks bölümünde "gold" almak görece kolay:

```
✓ EDA notebook — güzel görselleştirme, iş sorusu
✓ "Beginner's Guide to X" formatı
✓ Mermaid diyagram + açıklama
✓ Türkçe mi yazacaksın? — Çok az rekabet!

Upvote sayısı:
  Bronze: ~20 upvote
  Silver: ~50 upvote
  Gold:   ~100+ upvote
```

### Kazananların Paylaştığı Teknikler

```python
# 1. Pseudo-labeling
model.fit(train_X, train_y)
pseudo_labels = model.predict(test_X)
# Yüksek güvenilirli tahminleri eğitime ekle

# 2. Target Encoding (CV içinde!)
from category_encoders import TargetEncoder
enc = TargetEncoder(cols=['category'])
X_train[enc_cols] = enc.fit_transform(X_train, y_train)

# 3. Stratified K-Fold
from sklearn.model_selection import StratifiedKFold
skf = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)

# 4. Optuna ile Hyperparameter Optimization
import optuna
def objective(trial):
    params = {
        'n_estimators': trial.suggest_int('n_estimators', 100, 1000),
        'learning_rate': trial.suggest_float('lr', 0.01, 0.3, log=True),
        'max_depth': trial.suggest_int('depth', 3, 10)
    }
    ...
```

### Kaggle → Kariyer Bağlantısı

```
CV'de nasıl göster:
"Kaggle Expert — Tabular Competitions (Top 5%, 2 Silver medal)"
"Kaggle — Müşteri Churn Prediction (Rank 47/2840)"

LinkedIn'de:
"Achievements" bölümüne ekle

Portfolio'da:
GitHub'a notebook koy + Kaggle linkini ekle
```

### Kaggle Öğrenme Yolu (Ücretsiz Kurslar)

```
Kaggle Learn (tamamen ücretsiz):
  1. Python
  2. Intro to Machine Learning
  3. Intermediate Machine Learning
  4. Feature Engineering
  5. Intro to Deep Learning
  6. Computer Vision
  7. NLP
  8. Time Series
  9. Intro to SQL / Advanced SQL

Toplam: ~30 saat, sertifika var
```

---

## 💡 Bağlantılar
- [[Kariyer - Data Science Portfolio Proje Fikirleri]]
- [[ML - Gradient Boosting (XGBoost & LightGBM)]]
- [[FE - Feature Engineering Temelleri]]
- [[Kariyer - GitHub Profil Optimizasyonu]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- [Kaggle Learn](https://www.kaggle.com/learn)
- [Abhishek Thakur - Approaching Any ML Problem](https://github.com/abhishekkrthakur/approachingalmost)
- [Kaggle Grandmaster blogları](https://www.kaggle.com/rankings)
