---
tarih: 2025-01-01
konu: Ölçeklendirme, Normalizasyon, StandardScaler, MinMaxScaler, RobustScaler
etiket: [feature-engineering, ölçeklendirme, normalizasyon, scaling, StandardScaler]
kaynak:
zorluk: ⭐⭐
---

## 📌 Özet

Farklı ölçeklerdeki sayısal değişkenler (yaş: 0-100, gelir: 0-1.000.000) modeli bozar. Ölçeklendirme tüm özellikleri karşılaştırılabilir aralığa getirir.

---

## 🧠 Detay

### Neden Ölçeklendirme Gerekir?

| Durum | Sorun |
|---|---|
| Gradient tabanlı modeller | Büyük değerli özellik gradient'i domine eder |
| KNN, K-Means | Uzaklık hesabı yanlı olur |
| SVM | Kernel fonksiyonu bozulur |
| PCA | Yüksek varyanslı değişken baskın çıkar |
| Ridge / Lasso | Ceza terimi farklı ölçeklerde anlamsız |

**Ağaç modelleri (RF, XGBoost, LightGBM) ölçeklendirmeye gerek duymaz.**

---

### 1. Standard Scaler (Z-Score Normalizasyonu)

$$x' = \frac{x - \mu}{\sigma}$$

- Ortalama = 0, Std = 1
- Aykırı değerlere duyarlı

```python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled  = scaler.transform(X_test)  # fit() ÇAĞIRMA!

# Geri al
X_original = scaler.inverse_transform(X_train_scaled)
print(f"Ortalama: {scaler.mean_}")
print(f"Std: {scaler.scale_}")
```

✅ **Kullanım**: Doğrusal regresyon, lojistik regresyon, SVM, PCA, sinir ağları.

---

### 2. MinMax Scaler

$$x' = \frac{x - x_{min}}{x_{max} - x_{min}}$$

- Aralık: [0, 1] (veya belirtilen aralık)
- Aykırı değerlere **çok** duyarlı

```python
from sklearn.preprocessing import MinMaxScaler

scaler = MinMaxScaler(feature_range=(0, 1))
X_train_scaled = scaler.fit_transform(X_train)

# Özel aralık [-1, 1]
scaler = MinMaxScaler(feature_range=(-1, 1))
```

✅ **Kullanım**: Görüntü pikselleri (0-255→0-1), sinir ağları, [0,1] beklenen modeller.

---

### 3. Robust Scaler

$$x' = \frac{x - Q2}{IQR}$$

- Medyan ve IQR kullanır
- Aykırı değerlere **dayanıklı**

```python
from sklearn.preprocessing import RobustScaler

scaler = RobustScaler(quantile_range=(25.0, 75.0))
X_scaled = scaler.fit_transform(X_train)
```

✅ **Kullanım**: Aykırı değer çok olan veriler.

---

### 4. MaxAbs Scaler

$$x' = \frac{x}{|x_{max}|}$$

- Aralık: [-1, 1]
- Sıfır ortalamayı bozmuyor → sparse veriler için uygun

```python
from sklearn.preprocessing import MaxAbsScaler

scaler = MaxAbsScaler()
X_scaled = scaler.fit_transform(X_train)
```

✅ **Kullanım**: Seyrek (sparse) matrisler, TF-IDF verisi.

---

### 5. Normalizer (L1 / L2 Norm)

Satır bazında normalizasyon (sütun değil!).

$$x' = \frac{x}{||x||_p}$$

```python
from sklearn.preprocessing import Normalizer

normalizer = Normalizer(norm='l2')  # 'l1', 'l2', 'max'
X_normalized = normalizer.fit_transform(X)
```

✅ **Kullanım**: Metin verisi (TF-IDF), kosinüs benzerliği.

---

### Karşılaştırma Tablosu

| Scaler | Aralık | Aykırıya Duyarlı | Kullanım |
|---|---|---|---|
| StandardScaler | [-∞, +∞] | Evet | Genel amaç |
| MinMaxScaler | [0, 1] | Çok | Görüntü, NN |
| RobustScaler | Esnek | Hayır | Aykırı değer var |
| MaxAbsScaler | [-1, 1] | Orta | Sparse veri |
| Normalizer | Birim vektör | — | Metin, cos sim |

---

### Görselleştirme

```python
import matplotlib.pyplot as plt
import numpy as np
from sklearn.preprocessing import StandardScaler, MinMaxScaler, RobustScaler

X = np.random.exponential(scale=2, size=(1000, 1))

fig, axes = plt.subplots(1, 4, figsize=(16, 4))
titles = ['Orijinal', 'Standard', 'MinMax', 'Robust']
data = [
    X,
    StandardScaler().fit_transform(X),
    MinMaxScaler().fit_transform(X),
    RobustScaler().fit_transform(X)
]
for ax, d, t in zip(axes, data, titles):
    ax.hist(d, bins=50)
    ax.set_title(t)
plt.tight_layout()
plt.show()
```

---

### Pipeline'da Ölçeklendirme

```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression

pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('model', LogisticRegression())
])

pipeline.fit(X_train, y_train)
score = pipeline.score(X_test, y_test)
```

---

### ⚠️ Sık Yapılan Hatalar

```python
# ❌ YANLIŞ — tüm veri üzerinde fit → data leakage
scaler.fit(X_all)

# ❌ YANLIŞ — test'e de fit
X_test_scaled = scaler.fit_transform(X_test)

# ✅ DOĞRU
scaler.fit(X_train)
X_train_scaled = scaler.transform(X_train)
X_test_scaled  = scaler.transform(X_test)
```

---

## 💡 Bağlantılar
- [[FE - Giriş ve Genel Bakış]]
- [[FE - Sayısal Dönüşümler]]
- [[FE - Aykırı Değer İşleme]]
- [[ML - Veri Ön İşleme Pipeline]]

## ❓ Sorular / Anlamadıklarım


## 🔗 Kaynaklar
- sklearn.preprocessing Documentation
- Compare the effect of different scalers on data (sklearn example)
