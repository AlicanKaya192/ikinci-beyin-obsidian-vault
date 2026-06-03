---
tarih: 2026-05-28
konu: Data Science
etiket: ["data-science", "korelasyon", "istatistik", "ilişki"]
kaynak: 
zorluk: orta
---

## 📌 Özet
Korelasyon, iki değişken arasındaki doğrusal ilişkinin yönünü ve gücünü ölçer. -1 ile +1 arasında değer alır. Nedensellik değil, ilişki gösterir.

## 🧠 Detay

### Pearson Korelasyonu
```python
import numpy as np
import pandas as pd
from scipy import stats
import seaborn as sns

# İki değişken arası
r, p = stats.pearsonr(df["yas"], df["gelir"])
print(f"r={r:.3f}, p={p:.4f}")

# Tüm sayısal sütunlar
df.corr(method="pearson")
```

### Korelasyon Yorumlama
| r değeri | Yorum |
|----------|-------|
| 0.9 - 1.0 | Çok güçlü pozitif |
| 0.7 - 0.9 | Güçlü pozitif |
| 0.5 - 0.7 | Orta pozitif |
| 0.3 - 0.5 | Zayıf pozitif |
| 0.0 - 0.3 | Çok zayıf |
| Negatif | Ters yönde |

### Spearman Korelasyonu
```python
# Sıralama bazlı, doğrusal olmayan ilişkiler için
r, p = stats.spearmanr(df["yas"], df["gelir"])
df.corr(method="spearman")
```

### Isı Haritası
```python
import matplotlib.pyplot as plt

plt.figure(figsize=(10, 8))
sns.heatmap(
    df.corr(),
    annot=True,
    fmt=".2f",
    cmap="coolwarm",
    center=0,
    square=True
)
plt.title("Korelasyon Matrisi")
plt.show()
```

### Hedef Değişkenle Korelasyon
```python
# ML öncesi özellik seçimi için
korelasyonlar = df.corr()["hedef"].sort_values(ascending=False)
print(korelasyonlar)

# Görselleştir
korelasyonlar.drop("hedef").plot(kind="bar")
plt.title("Hedef Değişkenle Korelasyon")
plt.show()
```

### Dikkat Edilecekler
```python
# Korelasyon ≠ Nedensellik!
# Yüksek korelasyon → ilişki var, ama neden değil

# Çok yüksek korelasyon → multicollinearity
# VIF ile kontrol
from statsmodels.stats.outliers_influence import variance_inflation_factor
```

## 💡 Bağlantılar
- [[DS - EDA - Keşifsel Veri Analizi]]
- [[DS - Betimsel İstatistik]]
- [[ML - Özellik Seçimi]]

## ❓ Sorular / Anlamadıklarım
- Pearson ve Spearman korelasyonu ne zaman hangisini kullanmalıyım?
- Multicollinearity modeli nasıl etkiler?

## 🔗 Kaynaklar
- https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.pearsonr.html
