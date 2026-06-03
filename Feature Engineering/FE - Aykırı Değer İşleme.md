---
tarih: 2025-01-01
konu: Aykırı Değer Tespiti ve İşleme, IQR, Z-Skoru, Winsorizing
etiket: [feature-engineering, aykırı-değer, outlier, IQR, winsorizing, z-skoru]
kaynak:
zorluk: ⭐⭐
---

## 📌 Özet

Aykırı değerler modeli çarpıtabilir veya gerçek sinyaller taşıyabilir. Tespit edilmeli, kaynağı anlaşılmalı, sonra uygun yöntemle işlenmelidir.

---

## 🧠 Detay

### Aykırı Değer Türleri

| Tür | Açıklama | Örnek |
|---|---|---|
| **Tek değişkenli** | Tek sütunda aşırı değer | Yaş = 999 |
| **Çok değişkenli** | Birlikte bakınca aykırı | Boy=150cm, Kilo=150kg |
| **Bağlamsal** | Bağlama göre aykırı | Yaz ayında -10°C |
| **Veri hatası** | Ölçüm/giriş hatası | Silmek gerekir |
| **Gerçek aykırı** | Nadir ama gerçek | Dolandırıcılık işlemi |

### Tespit Yöntemleri

#### 1. IQR Yöntemi (Tukey)

```python
Q1 = df['col'].quantile(0.25)
Q3 = df['col'].quantile(0.75)
IQR = Q3 - Q1

alt_sinir = Q1 - 1.5 * IQR
ust_sinir = Q3 + 1.5 * IQR

outliers = df[(df['col'] < alt_sinir) | (df['col'] > ust_sinir)]
print(f"Aykırı değer sayısı: {len(outliers)}")
```

Aşırı aykırılar için: `3 * IQR` kullan.

#### 2. Z-Skoru Yöntemi

```python
from scipy import stats
import numpy as np

z_scores = np.abs(stats.zscore(df['col']))
outliers = df[z_scores > 3]  # 3 standart sapma
```

Normallik varsayar → çarpık veriye uygun değil.

#### 3. Modified Z-Score (Medyan tabanlı)

```python
median = df['col'].median()
mad = np.median(np.abs(df['col'] - median))  # Median Absolute Deviation
modified_z = 0.6745 * (df['col'] - median) / mad

outliers = df[np.abs(modified_z) > 3.5]
```

Aykırı değerlere karşı daha dayanıklı.

#### 4. Isolation Forest

```python
from sklearn.ensemble import IsolationForest

clf = IsolationForest(contamination=0.05, random_state=42)
df['outlier'] = clf.fit_predict(df[['col1', 'col2']])
# -1: aykırı, 1: normal
```

Çok değişkenli aykırı değer tespiti için.

#### 5. Local Outlier Factor (LOF)

```python
from sklearn.neighbors import LocalOutlierFactor

lof = LocalOutlierFactor(n_neighbors=20, contamination=0.05)
df['outlier'] = lof.fit_predict(df[numeric_cols])
```

Yoğunluk tabanlı, kümesel aykırı değerler için.

### Görselleştirme

```python
import matplotlib.pyplot as plt
import seaborn as sns

# Tek değişken
fig, axes = plt.subplots(1, 2, figsize=(12, 4))
df['col'].hist(ax=axes[0], bins=50)
df.boxplot(column='col', ax=axes[1])

# Çok değişken
sns.scatterplot(data=df, x='col1', y='col2', hue='outlier')
```

### İşleme Yöntemleri

#### 1. Silme

```python
df_clean = df[(df['col'] >= alt_sinir) & (df['col'] <= ust_sinir)]
```
Veri kaybı yaratır — sadece veri hatası ise kullan.

#### 2. Winsorizing (Kırpma / Capping)

```python
# Alt ve üst yüzdeliğe kırp
lower = df['col'].quantile(0.01)
upper = df['col'].quantile(0.99)
df['col'] = df['col'].clip(lower=lower, upper=upper)

# scipy ile
from scipy.stats import mstats
df['col'] = mstats.winsorize(df['col'], limits=[0.01, 0.01])
```

En yaygın yöntem — veri kaybı olmaz.

#### 3. Dönüştürme

```python
import numpy as np

df['col_log']  = np.log1p(df['col'])    # log(x+1) → negatif olmayan için
df['col_sqrt'] = np.sqrt(df['col'])     # karekök
df['col_cbrt'] = np.cbrt(df['col'])     # küp kök (negatif de olabilir)
df['col_box'], _ = stats.boxcox(df['col'] + 1)  # Box-Cox
```

Dağılımı normalleştirir, aykırıların etkisini azaltır.

#### 4. Imputation (Aykırıyı Eksik Gibi Gör)

```python
df.loc[df['col'] > ust_sinir, 'col'] = np.nan
df.loc[df['col'] < alt_sinir, 'col'] = np.nan
# Ardından imputation uygula
df['col'].fillna(df['col'].median(), inplace=True)
```

#### 5. Ayrı Model

Aykırı değerler için ayrı bir model (örn. dolandırıcılık tespiti).

### Model Bazlı Strateji

| Model | Aykırı Değere Duyarlı? | Strateji |
|---|---|---|
| Doğrusal/Lojistik Regresyon | ✅ Çok duyarlı | Winsorize veya dönüştür |
| Karar Ağaçları / RF | ❌ Dayanıklı | Genellikle işlem gerekmez |
| KNN | ✅ Duyarlı | Ölçeklendirme + winsorize |
| SVM | ✅ Duyarlı | Dönüştür |
| Gradient Boosting | ❌ Nispeten dayanıklı | İsteğe bağlı |
| Sinir Ağları | ✅ Duyarlı | Normalize + winsorize |

### Karar Akışı

```
Aykırı Değer Tespit Et
        ↓
Veri hatası mı?  → Evet → Sil / Düzelt
        ↓ Hayır
Gerçek aykırı mı? → Evet → Koru veya ayrı model
        ↓ Hayır
Model duyarlı mı? → Evet → Winsorize / Dönüştür
        ↓ Hayır
                           Olduğu gibi bırak
```

---

## 💡 Bağlantılar
- [[FE - Eksik Veri İşleme]]
- [[FE - Sayısal Dönüşümler]]
- [[FE - Ölçeklendirme ve Normalizasyon]]
- [[DS - Aykırı Değer Analizi]]

## ❓ Sorular / Anlamadıklarım


## 🔗 Kaynaklar
- sklearn.ensemble.IsolationForest Documentation
- Feature Engineering for Machine Learning - Ch. 4
