---
tarih: 2025-01-01
konu: Basit Doğrusal Regresyon, En Küçük Kareler, Model Değerlendirme
etiket: [istatistik, regresyon, doğrusal, OLS, R-kare]
kaynak:
zorluk: ⭐⭐⭐
---

## 📌 Özet

Basit doğrusal regresyon, bir bağımsız değişken (X) ile bir bağımlı değişken (Y) arasındaki doğrusal ilişkiyi modelleyen yöntemdir. En küçük kareler (OLS) yöntemiyle katsayılar tahmin edilir.

---

## 🧠 Detay

### Model

$$Y = \beta_0 + \beta_1 X + \varepsilon$$

- $\beta_0$: y-eksen kesişimi (intercept)
- $\beta_1$: eğim (slope)
- $\varepsilon$: hata terimi (artık), $\varepsilon \sim N(0, \sigma^2)$

**Tahmin (fitted) değerleri:**
$$\hat{Y} = \hat{\beta}_0 + \hat{\beta}_1 X$$

**Artıklar (residuals):**
$$e_i = Y_i - \hat{Y}_i$$

### OLS Katsayı Tahmini

**Eğim:**
$$\hat{\beta}_1 = \frac{\sum(x_i - \bar{x})(y_i - \bar{y})}{\sum(x_i - \bar{x})^2} = \frac{Cov(X,Y)}{Var(X)} = r \cdot \frac{s_Y}{s_X}$$

**Kesişim:**
$$\hat{\beta}_0 = \bar{y} - \hat{\beta}_1 \bar{x}$$

> Regresyon doğrusu her zaman $(\bar{x}, \bar{y})$ noktasından geçer.

### OLS Varsayımları (LINE)

| Harf | Varsayım | Kontrol |
|---|---|---|
| **L** | **L**inearlik | Saçılım grafiği |
| **I** | Bağımsız artıklar (**I**ndependence) | Durbin-Watson |
| **N** | Normal artıklar (**N**ormality) | Q-Q plot, Shapiro-Wilk |
| **E** | Eşit varyans (**E**qual variance - Homoscedasticity) | Artık vs fitted grafiği |

### Model Değerlendirme

#### R² (Belirtme Katsayısı)

$$R^2 = 1 - \frac{SS_{Res}}{SS_{Tot}} = 1 - \frac{\sum(y_i - \hat{y}_i)^2}{\sum(y_i - \bar{y})^2}$$

- $0 \leq R^2 \leq 1$
- Modelin Y'deki değişimi ne kadar açıkladığı
- Basit regresyonda: $R^2 = r^2$

#### Varyans Analizi (ANOVA for Regression)

| Kaynak | SS | df | MS | F |
|---|---|---|---|---|
| Regresyon | $\sum(\hat{y}_i - \bar{y})^2$ | 1 | — | $MS_R/MS_E$ |
| Artık (Hata) | $\sum(y_i - \hat{y}_i)^2$ | n-2 | — | — |
| Toplam | $\sum(y_i - \bar{y})^2$ | n-1 | — | — |

#### Standart Hata

$$SE(\hat{\beta}_1) = \frac{s}{\sqrt{\sum(x_i-\bar{x})^2}}, \quad s = \sqrt{\frac{\sum e_i^2}{n-2}}$$

#### Katsayı için t-testi

$$t = \frac{\hat{\beta}_1}{SE(\hat{\beta}_1)} \sim t_{n-2}$$

$$H_0: \beta_1 = 0 \quad \text{(X ile ilişki yok)}$$

#### Güven Aralıkları

Katsayı için:
$$\hat{\beta}_1 \pm t_{\alpha/2,n-2} \cdot SE(\hat{\beta}_1)$$

**Ortalama Y tahmini** ($x_0$ değerinde):
$$\hat{y}_0 \pm t_{\alpha/2,n-2} \cdot s\sqrt{\frac{1}{n} + \frac{(x_0-\bar{x})^2}{\sum(x_i-\bar{x})^2}}$$

**Bireysel Y tahmini** (daha geniş):
$$\hat{y}_0 \pm t_{\alpha/2,n-2} \cdot s\sqrt{1 + \frac{1}{n} + \frac{(x_0-\bar{x})^2}{\sum(x_i-\bar{x})^2}}$$

### Artık Analizi

```
✅ İyi artık grafiği:     ❌ Kötü artık grafiği:
   •  •                     •  •  •
  • • •  •                 •      •
─────────────             ──────────
  •  •  •                   •  •  •
```

- **Heteroscedasticity**: Artıkların yayılımı X ile değişiyor → Log dönüşümü
- **Otokorelasyon**: Ardışık artıklar ilişkili → Zaman serisi yöntemi gerek

### Python ile Regresyon

```python
import statsmodels.api as sm
import numpy as np

# Veri hazırla
X = sm.add_constant(X)  # Sabit terim ekle

# Model
model = sm.OLS(y, X).fit()
print(model.summary())

# Tahmin
predictions = model.predict(X_new)
print(f"R²: {model.rsquared:.4f}")
print(f"Adj R²: {model.rsquared_adj:.4f}")

# Artık analizi
import matplotlib.pyplot as plt
plt.scatter(model.fittedvalues, model.resid)
plt.axhline(0, color='r')
plt.xlabel('Fitted Values')
plt.ylabel('Residuals')
```

---

## 💡 Bağlantılar
- [[STAT - Çoklu Doğrusal Regresyon]]
- [[STAT - Korelasyon Analizi]]
- [[ML - Lineer Regresyon]]

## ❓ Sorular / Anlamadıklarım


## 🔗 Kaynaklar
- OpenStax Statistics - Ch. 12
- statsmodels OLS Documentation
