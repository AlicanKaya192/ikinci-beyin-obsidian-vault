---
tarih: 2025-01-01
konu: Sayısal Dönüşümler, Log, Box-Cox, Power Transform, Bağlama
etiket: [feature-engineering, dönüşüm, log-transform, box-cox, binning, power-transform]
kaynak:
zorluk: ⭐⭐
---

## 📌 Özet

Çarpık dağılımlı sayısal değişkenleri dönüştürmek normalliği artırır, modeli iyileştirir. Log, Box-Cox ve Yeo-Johnson en yaygın dönüşümlerdir.

---

## 🧠 Detay

### Neden Dönüşüm?

- Doğrusal modeller normal dağılım varsayar
- Çarpık veri → büyük değerler modeli domine eder
- Dönüşüm → özellikler arası ilişkiyi doğrusallaştırabilir
- Aykırı değerlerin etkisini azaltır

### Çarpıklık Kontrolü

```python
import scipy.stats as stats
import matplotlib.pyplot as plt

# Çarpıklık değeri
skewness = df['col'].skew()
print(f"Çarpıklık: {skewness:.3f}")
# |skew| > 0.5 → dönüşüm düşün
# |skew| > 1.0 → dönüşüm gerekli

# Görsel kontrol
fig, axes = plt.subplots(1, 2, figsize=(10, 4))
df['col'].hist(ax=axes[0])
stats.probplot(df['col'], plot=axes[1])  # Q-Q plot
```

---

### 1. Log Dönüşümü

$$x' = \ln(x) \quad \text{veya} \quad x' = \log_{10}(x)$$

```python
import numpy as np

# Pozitif değerler için
df['col_log'] = np.log(df['col'])

# Sıfır içeren veriler için log(x+1)
df['col_log1p'] = np.log1p(df['col'])  # log(x+1)

# Negatif değerler için
df['col_log'] = np.log(df['col'] - df['col'].min() + 1)
```

✅ **Kullanım**: Sağa çarpık veri (gelir, fiyat, nüfus).

---

### 2. Karekök ve Küp Kök

```python
# Karekök (negatif olamaz)
df['col_sqrt'] = np.sqrt(df['col'])

# Küp kök (negatif olabilir)
df['col_cbrt'] = np.cbrt(df['col'])

# Genel güç
df['col_pow'] = df['col'] ** 0.25
```

Log'dan daha az güçlü ama daha güvenli.

---

### 3. Box-Cox Dönüşümü

$$x' = \begin{cases} \frac{x^\lambda - 1}{\lambda} & \lambda \neq 0 \\ \ln(x) & \lambda = 0 \end{cases}$$

```python
from scipy import stats
from sklearn.preprocessing import PowerTransformer

# scipy (sadece pozitif değerler)
col_boxcox, lambda_val = stats.boxcox(df['col'])
print(f"Optimal lambda: {lambda_val:.4f}")

# sklearn
pt = PowerTransformer(method='box-cox')  # Sadece pozitif
df_transformed = pt.fit_transform(df[['col']])
```

Lambda değerleri:
- λ = 1 → Dönüşüm yok
- λ = 0 → Log
- λ = 0.5 → Karekök
- λ = -1 → Ters (1/x)

---

### 4. Yeo-Johnson Dönüşümü

Box-Cox'un negatif değerler için genellemesi.

```python
from sklearn.preprocessing import PowerTransformer

pt = PowerTransformer(method='yeo-johnson')  # Negatif değer de olabilir
pt.fit(X_train[['col']])
X_train_transformed = pt.transform(X_train[['col']])
X_test_transformed  = pt.transform(X_test[['col']])
```

✅ **Genel tavsiye**: Yeo-Johnson — hem pozitif hem negatif veri için çalışır.

---

### 5. Quantile Transformer

Değerleri belirli bir dağılıma (normal veya uniform) dönüştürür.

```python
from sklearn.preprocessing import QuantileTransformer

qt = QuantileTransformer(output_distribution='normal', random_state=42)
X_transformed = qt.fit_transform(X_train)

# Uniform dağılıma
qt_uniform = QuantileTransformer(output_distribution='uniform')
```

⚠️ **Dikkat**: Eğitim seti dışı değerlerde ekstrapolasyon sorunlu.

---

### 6. Bağlama (Binning / Discretization)

Sayısal değişkeni kategoriye dönüştürür.

```python
# Eşit genişlikte (Equal Width)
df['yas_grup'] = pd.cut(df['yas'],
                         bins=[0, 18, 35, 50, 65, 100],
                         labels=['Çocuk', 'Genç', 'Orta', 'Olgun', 'Yaşlı'])

# Eşit frekanslı (Equal Frequency / Quantile)
df['gelir_grup'] = pd.qcut(df['gelir'], q=4,
                             labels=['Düşük', 'Orta-Alt', 'Orta-Üst', 'Yüksek'])

# Sayısal bağlama
df['yas_bin'] = pd.cut(df['yas'], bins=5, labels=False)  # 0,1,2,3,4

# sklearn
from sklearn.preprocessing import KBinsDiscretizer
kbd = KBinsDiscretizer(n_bins=5, encode='ordinal', strategy='quantile')
df['yas_bin'] = kbd.fit_transform(df[['yas']])
```

**Ne zaman bağlama?**
- Doğrusal olmayan ilişki var
- Aykırı değer sorunu
- Domain bilgisine dayalı gruplar (yaş grupları)
- Tree modeli dışında doğrusal model kullanıyorsun

---

### 7. Ters Dönüşüm (Reciprocal)

$$x' = \frac{1}{x}$$

```python
df['col_inv'] = 1 / df['col']
```

Hız → süre dönüşümlerinde kullanışlı.

---

### dönüşüm Sonrası Kontrol

```python
def check_skew(df, cols):
    results = []
    for col in cols:
        before = df[col].skew()
        log_skew = np.log1p(df[col]).skew()
        results.append({'col': col, 'before': before, 'log': log_skew})
    return pd.DataFrame(results).sort_values('before', ascending=False)

check_skew(df, numeric_cols)
```

---

### Hangi Dönüşüm Ne Zaman?

| Durum | Öneri |
|---|---|
| Sağa çarpık, pozitif | Log veya Box-Cox |
| Sağa çarpık, sıfır var | log1p |
| Sağa çarpık, negatif var | Yeo-Johnson |
| İki yönlü dağılım | Yeo-Johnson veya QuantileTransformer |
| Nonlineer ilişki | Bağlama |
| Ağaç modeli | Genellikle gerekmez |

---

## 💡 Bağlantılar
- [[FE - Ölçeklendirme ve Normalizasyon]]
- [[FE - Aykırı Değer İşleme]]
- [[STAT - Normal Dağılım]]
- [[STAT - Betimsel İstatistik]]

## ❓ Sorular / Anlamadıklarım


## 🔗 Kaynaklar
- sklearn.preprocessing.PowerTransformer Documentation
- Feature Engineering for Machine Learning - Ch. 3
