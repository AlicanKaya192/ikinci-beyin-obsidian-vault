---
tarih: 2026-05-28
konu: Data Science
etiket: ["data-science", "dönüşüm", "normalizasyon", "encoding", "feature-engineering"]
kaynak: Scikit-learn Dokümantasyon
zorluk: orta
---

## 📌 Özet
Veri dönüşümleri, ham veriyi makine öğrenmesi algoritmalarına uygun hale getirmek için yapılan ölçekleme, kodlama ve dönüştürme işlemleridir.

## 🧠 Detay

### Sayısal Ölçekleme

```python
from sklearn.preprocessing import StandardScaler, MinMaxScaler, RobustScaler
import pandas as pd

# StandardScaler → Ortalama=0, Std=1
scaler = StandardScaler()
df["gelir_scaled"] = scaler.fit_transform(df[["gelir"]])

# MinMaxScaler → [0, 1] aralığı
mms = MinMaxScaler()
df["gelir_norm"] = mms.fit_transform(df[["gelir"]])

# RobustScaler → aykırı değerlere dayanıklı
rs = RobustScaler()
df["gelir_robust"] = rs.fit_transform(df[["gelir"]])
```

### Hangi Scaler Ne Zaman?
| Scaler | Ne Zaman |
|--------|----------|
| StandardScaler | Normal dağılıma yakın veri |
| MinMaxScaler | Belirli aralık gerektiğinde |
| RobustScaler | Aykırı değer çok olduğunda |

### Kategorik Kodlama

```python
# Label Encoding → sıralı kategoriler için
from sklearn.preprocessing import LabelEncoder
le = LabelEncoder()
df["sehir_enc"] = le.fit_transform(df["sehir"])

# One-Hot Encoding → sırasız kategoriler için
df_encoded = pd.get_dummies(df, columns=["sehir"], drop_first=True)

# Ordinal Encoding → sıralı kategoriler
from sklearn.preprocessing import OrdinalEncoder
oe = OrdinalEncoder(categories=[["düşük","orta","yüksek"]])
df["seviye_enc"] = oe.fit_transform(df[["seviye"]])
```

### Log Dönüşümü
```python
import numpy as np

# Sağa çarpık dağılımları normalleştir
df["gelir_log"] = np.log1p(df["gelir"])  # log(1+x)
df["gelir_sqrt"] = np.sqrt(df["gelir"])
```

### Özellik Üretimi (Feature Engineering)
```python
# Tarihten özellik
df["yil"] = df["tarih"].dt.year
df["ay"] = df["tarih"].dt.month
df["gun_adi"] = df["tarih"].dt.day_name()

# Etkileşim özellikleri
df["bmi"] = df["kilo"] / (df["boy"] / 100) ** 2
df["yas_gelir"] = df["yas"] * df["gelir"]
```

## 💡 Bağlantılar
- [[DS - Pandas Veri Temizleme]]
- [[DS - Aykırı Değer Analizi]]
- [[ML - Veri Ön İşleme Pipeline]]

## ❓ Sorular / Anlamadıklarım
- fit_transform ile transform arasındaki fark neden önemli?
- One-hot encoding'de neden drop_first=True kullanılır?

## 🔗 Kaynaklar
- https://scikit-learn.org/stable/modules/preprocessing.html
