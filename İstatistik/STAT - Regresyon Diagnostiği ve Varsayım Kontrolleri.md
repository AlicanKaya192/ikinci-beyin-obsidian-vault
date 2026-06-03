---
tarih: 2025-01-01
konu: Regresyon Diagnostiği, Aykırı Değer, Kaldıraç, Etkili Gözlem
etiket: [istatistik, regresyon, diagnostik, cook-distance, leverage, heteroscedasticity]
kaynak:
zorluk: ⭐⭐⭐
---

## 📌 Özet

Regresyon modelini kurduktan sonra varsayımların sağlandığı kontrol edilmelidir. Aykırı değerler, etkili gözlemler ve varsayım ihlalleri modeli bozabilir.

---

## 🧠 Detay

### Temel Varsayım Kontrolleri (LINE)

| Varsayım | Grafik | İstatistiksel Test |
|---|---|---|
| **L**inearlik | Artık vs Fitted | RESET testi |
| **I**bağımsızlık | Artık vs sıra | Durbin-Watson |
| **N**ormallik | Q-Q plot | Shapiro-Wilk, Jarque-Bera |
| **E**şit varyans | Scale-Location | Breusch-Pagan, White |

### Artık Türleri

**Ham Artık:**
$$e_i = y_i - \hat{y}_i$$

**Standartlaştırılmış Artık:**
$$r_i = \frac{e_i}{s\sqrt{1-h_{ii}}}$$

**Öğrenci Artıkları (Studentized):**
$$t_i = \frac{e_i}{s_{(i)}\sqrt{1-h_{ii}}}$$

$s_{(i)}$: i. gözlem çıkarıldığında hesaplanan MSE.

**Kural:** $|r_i| > 3$ → Şüpheli aykırı değer.

### Kaldıraç (Leverage)

Gözlemin $\hat{y}$ üzerindeki potansiyel etkisi:

$$h_{ii} = \mathbf{x}_i^T(\mathbf{X}^T\mathbf{X})^{-1}\mathbf{x}_i$$

Hat matrisinin köşegen elemanları.

- $0 \leq h_{ii} \leq 1$
- $\sum h_{ii} = p + 1$
- **Kural:** $h_{ii} > 2(p+1)/n$ → Yüksek kaldıraç

**Fark:** Yüksek kaldıraç mutlaka sorun değildir; artığına bakılmalı.

### Etkili Gözlemler (Influential Points)

#### Cook's Distance

Gözlemin tüm katsayılara toplam etkisi:

$$D_i = \frac{(\hat{\boldsymbol{\beta}} - \hat{\boldsymbol{\beta}}_{(i)})^T(\mathbf{X}^T\mathbf{X})(\hat{\boldsymbol{\beta}} - \hat{\boldsymbol{\beta}}_{(i)})}{(p+1)s^2}$$

**Hesap kolaylığı:**
$$D_i = \frac{r_i^2}{p+1} \cdot \frac{h_{ii}}{1-h_{ii}}$$

**Kural:** $D_i > 1$ veya $D_i > 4/n$ → Etkili gözlem

#### DFFITS

Gözlemin kendi fitted değerine etkisi:

$$DFFITS_i = t_i\sqrt{\frac{h_{ii}}{1-h_{ii}}}$$

**Kural:** $|DFFITS_i| > 2\sqrt{(p+1)/n}$

#### DFBETAS

Her katsayıya bireysel etki:

$$DFBETAS_{j,i} = \frac{\hat{\beta}_j - \hat{\beta}_{j,(i)}}{s_{(i)}\sqrt{[(\mathbf{X}^T\mathbf{X})^{-1}]_{jj}}}$$

**Kural:** $|DFBETAS| > 2/\sqrt{n}$

---

### Heteroscedasticity (Eşit Olmayan Varyans)

**Belirtiler:** Artık vs Fitted grafiğinde huni şekli.

**Testler:**
- **Breusch-Pagan**: $H_0$: Homoscedasticity
- **White Testi**: Daha genel (doğrusal olmayan da yakalar)
- **Goldfeld-Quandt**: Veri ikiye bölünür, varyanslar karşılaştırılır

**Çözümler:**
- Log dönüşümü: $\ln(Y)$
- Ağırlıklı en küçük kareler (WLS)
- Robust standart hatalar (HC3)

### Otokorelasyon (Zaman Serisi Verisinde)

**Durbin-Watson Testi:**
$$DW = \frac{\sum_{t=2}^n (e_t - e_{t-1})^2}{\sum_{t=1}^n e_t^2} \approx 2(1-r)$$

| DW Değeri | Yorum |
|---|---|
| ≈ 2 | Otokorelasyon yok |
| 0-2 | Pozitif otokorelasyon |
| 2-4 | Negatif otokorelasyon |

**Breusch-Godfrey Testi**: Daha yüksek dereceli otokorelasyon için.

### Aykırı Değer Tespiti (Regresyonda)

Hem X hem Y yönündeki aykırılığa bakmak gerekir:

```
         Y
         |
    *    |
         |           ← Yüksek kaldıraç, düşük artık (iyi hizalanmış)
         |      *
─────────────────── X
         |
    *    |        ← Yüksek kaldıraç, yüksek artık (sorunlu!)
```

### Çözüm Stratejileri

| Sorun | Çözüm |
|---|---|
| Normallik ihlali | Log/karekök dönüşümü |
| Heteroscedasticity | WLS, robust SE, log dönüşümü |
| Otokorelasyon | GLS, zaman serisi modeli |
| Çoklu doğrusallık | Ridge, Lasso, değişken çıkarma |
| Aykırı değer | İnceleme, robust regresyon |

### Python Diagnostik

```python
import statsmodels.api as sm
from statsmodels.stats.diagnostic import (
    het_breuschpagan, acorr_ljungbox
)
from statsmodels.stats.stattools import durbin_watson
import matplotlib.pyplot as plt

model = sm.OLS(y, X).fit()
influence = model.get_influence()

# Standartlaştırılmış artıklar
std_resid = influence.resid_studentized_internal

# Cook's Distance
cook_d = influence.cooks_distance[0]

# Kaldıraç
leverage = influence.hat_matrix_diag

# Breusch-Pagan testi
bp_test = het_breuschpagan(model.resid, model.model.exog)
print(f'BP İstatistiği: {bp_test[0]:.4f}, p: {bp_test[1]:.4f}')

# Durbin-Watson
dw = durbin_watson(model.resid)
print(f'Durbin-Watson: {dw:.4f}')

# Diagnostik grafikleri (4'lü)
fig, axes = plt.subplots(2, 2, figsize=(12, 10))
# 1. Artık vs Fitted
axes[0,0].scatter(model.fittedvalues, model.resid)
# 2. Q-Q Plot
sm.qqplot(model.resid, line='s', ax=axes[0,1])
# 3. Scale-Location
axes[1,0].scatter(model.fittedvalues, np.sqrt(np.abs(std_resid)))
# 4. Cook's Distance
axes[1,1].stem(cook_d)
```

---

## 💡 Bağlantılar
- [[STAT - Regresyon Analizi - Basit Doğrusal]]
- [[STAT - Çoklu Doğrusal Regresyon]]
- [[STAT - Normal Dağılım]]

## ❓ Sorular / Anlamadıklarım


## 🔗 Kaynaklar
- Applied Regression Analysis (Draper & Smith)
- statsmodels.OLS Diagnostics Documentation
