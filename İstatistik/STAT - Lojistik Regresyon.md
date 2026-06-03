---
tarih: 2025-01-01
konu: Lojistik Regresyon, İkili Sınıflandırma, Odds Ratio
etiket: [istatistik, lojistik-regresyon, sınıflandırma, odds-ratio, logit]
kaynak:
zorluk: ⭐⭐⭐
---

## 📌 Özet

Lojistik regresyon, ikili (0/1) bağımlı değişkenin olasılığını modelleyen yöntemdir. Çıktı olasılık olduğundan sigmoid fonksiyon kullanılır, doğrusal regresyon uygulanamaz.

---

## 🧠 Detay

### Neden Doğrusal Regresyon Çalışmaz?

- Olasılık $[0,1]$ aralığında, doğrusal model bu sınırı aşabilir
- Hata terimi normal dağılımı takip etmez
- Varyans sabit değildir

### Logit Dönüşümü

**Odds (Oran)**:
$$\text{Odds} = \frac{p}{1-p}$$

**Log-odds (Logit)**:
$$\text{logit}(p) = \ln\!\left(\frac{p}{1-p}\right)$$

**Lojistik (Sigmoid) Fonksiyon**:
$$p = \frac{e^{\eta}}{1+e^{\eta}} = \frac{1}{1+e^{-\eta}}, \quad \eta = \beta_0 + \beta_1 X$$

### Model

$$\ln\!\left(\frac{p}{1-p}\right) = \beta_0 + \beta_1 X_1 + \cdots + \beta_k X_k$$

$$p(Y=1|X) = \frac{1}{1 + e^{-(\beta_0 + \beta_1 X_1 + \cdots)}}$$

### Katsayı Yorumlama

**$\beta_j$**: $X_j$ bir birim artınca log-odds ne kadar değişir.

**Odds Ratio (OR)**:
$$OR = e^{\beta_j}$$

| OR | Yorum |
|---|---|
| OR = 1 | $X_j$'nin etkisi yok |
| OR > 1 | $X_j$ artınca olay olasılığı artar |
| OR < 1 | $X_j$ artınca olay olasılığı azalır |

**Örnek**: $\beta_{sigara} = 0.8$ → $OR = e^{0.8} = 2.23$ → Sigara içenlerin hastalık odds'u 2.23 kat fazla.

### Katsayı Tahmini: Maximum Likelihood

OLS değil, MLE kullanılır.

**Log-likelihood:**
$$\ell(\boldsymbol{\beta}) = \sum_{i=1}^n \left[y_i \ln(\hat{p}_i) + (1-y_i)\ln(1-\hat{p}_i)\right]$$

**Log-loss (Binary Cross-Entropy):**
$$\mathcal{L} = -\frac{1}{n}\ell(\boldsymbol{\beta})$$

### Model Değerlendirme

#### Deviance ve Likelihood Ratio Testi

$$G^2 = -2\ln\!\left(\frac{L_0}{L_M}\right) = -2(\ell_0 - \ell_M) \sim \chi^2_{df}$$

Null modele göre iyileşmeyi test eder.

#### Pseudo R²

Gerçek R² gibi yorumlanamaz, karşılaştırma amaçlı.

- **McFadden's**: $R^2_{McF} = 1 - \frac{\ell_M}{\ell_0}$
- **Nagelkerke**: Yorumlanması daha kolay

#### Hosmer-Lemeshow Testi

Model kalibrasyonu: Tahmin edilen olasılıklar gerçek oranları ne kadar yansıtıyor?

$$H_0: \text{Model veriye uyum sağlıyor}$$

### Sınıflandırma Performansı

**Eşik (Threshold) = 0.5** (varsayılan):
- $\hat{p} \geq 0.5$ → $\hat{Y} = 1$

| | Gerçek 1 | Gerçek 0 |
|---|---|---|
| **Tahmin 1** | TP | FP |
| **Tahmin 0** | FN | TN |

$$Accuracy = \frac{TP+TN}{n}, \quad Precision = \frac{TP}{TP+FP}$$

$$Recall = \frac{TP}{TP+FN}, \quad F1 = \frac{2 \cdot Precision \cdot Recall}{Precision+Recall}$$

**ROC-AUC**: Tüm eşikler için sensitivite vs 1-spesifite. AUC = 1 mükemmel, 0.5 = rastgele.

### Çok Sınıflı Uzantılar

- **Multinomial Lojistik**: >2 sınıf, nominal
- **Ordinal Lojistik**: Sıralı kategoriler

### Python

```python
from sklearn.linear_model import LogisticRegression
import statsmodels.api as sm
import numpy as np

# sklearn (ML odaklı)
model = LogisticRegression(max_iter=1000)
model.fit(X_train, y_train)
probs = model.predict_proba(X_test)[:, 1]

# statsmodels (istatistiksel çıkarım, p-değerleri)
model = sm.Logit(y, sm.add_constant(X)).fit()
print(model.summary())

# Odds Ratios
print(np.exp(model.params))

# ROC Curve
from sklearn.metrics import roc_auc_score, roc_curve
auc = roc_auc_score(y_true, probs)
fpr, tpr, thresholds = roc_curve(y_true, probs)
```

---

## 💡 Bağlantılar
- [[STAT - Çoklu Doğrusal Regresyon]]
- [[ML - Lojistik Regresyon]]
- [[ML - Model Değerlendirme Metrikleri]]
- [[STAT - Hipotez Testleri - Ki-kare ve F Testi]]

## ❓ Sorular / Anlamadıklarım


## 🔗 Kaynaklar
- Introduction to Statistical Learning (ISLR) - Ch. 4
- statsmodels.Logit Documentation
