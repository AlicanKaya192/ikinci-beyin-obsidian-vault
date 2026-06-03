---
tarih: 2025-01-01
konu: Eksik Veri İşleme, Imputation, MCAR MAR MNAR
etiket: [feature-engineering, eksik-veri, imputation, NaN, missing-data]
kaynak:
zorluk: ⭐⭐
---

## 📌 Özet

Eksik veriler (NaN, None, NULL) gerçek dünya verilerinin kaçınılmaz parçasıdır. Yanlış işleme önyargıya yol açar; doğru strateji eksikliğin mekanizmasına bağlıdır.

---

## 🧠 Detay

### Eksiklik Mekanizmaları

| Tür | Açıklama | Örnek | Strateji |
|---|---|---|---|
| **MCAR** (Tamamen Rastgele) | Eksiklik hiçbir değişkene bağlı değil | Anket formu kaybolmuş | Herhangi bir imputation |
| **MAR** (Rastgele) | Eksiklik gözlenen değişkenlere bağlı | Yaşlılar geliri eksik bırakmış | Conditional imputation |
| **MNAR** (Rastgele Değil) | Eksiklik verinin kendisine bağlı | Yüksek gelir eksik (utanç) | Domain bilgisi gerekli |

### Eksik Veri Tespiti

```python
import pandas as pd
import matplotlib.pyplot as plt
import missingno as msno

# Genel bakış
df.isnull().sum()
df.isnull().mean() * 100  # Yüzde olarak

# Görselleştirme
msno.matrix(df)        # Eksik değer matrisi
msno.heatmap(df)       # Sütunlar arası korelasyon
msno.bar(df)           # Doluluk bar grafiği

# Satır bazlı
df.isnull().sum(axis=1)  # Her satırda kaç eksik
```

### Basit İmputation Yöntemleri

```python
from sklearn.impute import SimpleImputer
import numpy as np

# Sayısal: Ortalama / Medyan / Sabit
imputer_mean   = SimpleImputer(strategy='mean')
imputer_median = SimpleImputer(strategy='median')
imputer_const  = SimpleImputer(strategy='constant', fill_value=0)

# Kategorik: Mod / Sabit
imputer_mode  = SimpleImputer(strategy='most_frequent')
imputer_const = SimpleImputer(strategy='constant', fill_value='Bilinmiyor')

# Pandas ile hızlı
df['col'].fillna(df['col'].mean(), inplace=True)
df['col'].fillna(method='ffill', inplace=True)  # İleriye doldur (zaman serisi)
df['col'].fillna(method='bfill', inplace=True)  # Geriye doldur
```

### İleri Düzey İmputation

#### KNN Imputer

```python
from sklearn.impute import KNNImputer

imputer = KNNImputer(n_neighbors=5)
df_imputed = pd.DataFrame(
    imputer.fit_transform(df),
    columns=df.columns
)
```
- En yakın k komşunun ortalaması ile doldurur
- Veri yapısını daha iyi korur
- Yavaş (büyük veride dikkat)

#### Iterative Imputer (MICE)

```python
from sklearn.experimental import enable_iterative_imputer
from sklearn.impute import IterativeImputer
from sklearn.ensemble import RandomForestRegressor

imputer = IterativeImputer(
    estimator=RandomForestRegressor(n_estimators=10),
    max_iter=10,
    random_state=42
)
df_imputed = imputer.fit_transform(df)
```
- Her sütunu diğer sütunlara göre tahmin eder
- En güçlü yöntem ama hesaplama maliyetli

### Eksiklik Göstergesi Özelliği

Eksiklik bilgisinin kendisi de bir özellik olabilir:

```python
# Hangi satırlarda eksik olduğunu yeni kolon olarak ekle
df['gelir_eksik'] = df['gelir'].isnull().astype(int)
df['gelir'].fillna(df['gelir'].median(), inplace=True)
```

MNAR durumlarında bu çok değerli bilgi taşıyabilir!

### Silme Yöntemleri

```python
# Satır sil (dikkatli kullan)
df.dropna(inplace=True)                        # Herhangi eksik varsa
df.dropna(subset=['kritik_kolon'], inplace=True)  # Belirli sütunda

# Sütun sil (yüksek eksiklik oranında)
threshold = 0.5
df.dropna(axis=1, thresh=int(threshold * len(df)), inplace=True)

# Kural: %50'den fazla eksik → sütunu sil
cols_to_drop = df.columns[df.isnull().mean() > 0.5]
df.drop(columns=cols_to_drop, inplace=True)
```

### Strateji Seçim Rehberi

```
Eksiklik oranı?
├── > %50 → Sütunu sil
├── %20-50 → İleri düzey imputation + gösterge özelliği
└── < %20
    ├── Sayısal?
    │   ├── Normal dağılım → Ortalama
    │   ├── Çarpık dağılım → Medyan
    │   └── Zaman serisi → ffill/bfill
    └── Kategorik?
        ├── Az kategori → Mod
        └── Çok kategori → 'Bilinmiyor' sabiti
```

### ⚠️ Data Leakage Uyarısı

```python
# YANLIŞ ❌ — tüm veri üzerinde fit
imputer.fit(df_all)

# DOĞRU ✅ — sadece train üzerinde fit, test'e transform
imputer.fit(X_train)
X_train_imputed = imputer.transform(X_train)
X_test_imputed  = imputer.transform(X_test)  # train istatistikleri kullanılır
```

---

## 💡 Bağlantılar
- [[FE - Giriş ve Genel Bakış]]
- [[FE - Aykırı Değer İşleme]]
- [[DS - Pandas Veri Temizleme]]
- [[DS - Aykırı Değer Analizi]]

## ❓ Sorular / Anlamadıklarım


## 🔗 Kaynaklar
- sklearn.impute Documentation
- missingno Library GitHub
