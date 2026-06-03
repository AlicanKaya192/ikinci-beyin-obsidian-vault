---
tarih: 2026-05-28
konu: Machine Learning
etiket: ["ml", "regresyon", "lineer", "gözetimli"]
kaynak: Scikit-learn Dokümantasyon
zorluk: başlangıç
---

## 📌 Özet
Lineer regresyon, bağımlı değişkeni bağımsız değişkenlerle doğrusal bir ilişki üzerinden tahmin eder. En basit ve en açıklanabilir ML modelidir.

## 🧠 Detay

### Matematiksel Temel
```
y = β₀ + β₁x₁ + β₂x₂ + ... + βₙxₙ + ε

β₀ → sabit (intercept)
β₁...βₙ → katsayılar (coefficients)
ε → hata terimi
```

### Scikit-learn ile Lineer Regresyon
```python
from sklearn.linear_model import LinearRegression
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error, r2_score
import numpy as np

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

model = LinearRegression()
model.fit(X_train, y_train)

y_pred = model.predict(X_test)

print(f"Katsayılar: {model.coef_}")
print(f"Sabit: {model.intercept_:.3f}")
print(f"R²: {r2_score(y_test, y_pred):.3f}")
print(f"RMSE: {np.sqrt(mean_squared_error(y_test, y_pred)):.3f}")
```

### Varsayımlar
```
1. Doğrusallık → X ile y arasında doğrusal ilişki
2. Bağımsızlık → Hatalar birbirinden bağımsız
3. Homoskedastik → Sabit varyans
4. Normallik → Hatalar normal dağılımlı
5. Multicollinearity yok → Özellikler birbirinden bağımsız
```

### Varsayım Kontrolü
```python
import matplotlib.pyplot as plt
import scipy.stats as stats

residuals = y_test - y_pred

# Residual plot
plt.scatter(y_pred, residuals)
plt.axhline(0, color="red")
plt.xlabel("Tahmin")
plt.ylabel("Residual")

# QQ plot
stats.probplot(residuals, plot=plt)
```

### Ridge ve Lasso (Regularization)
```python
from sklearn.linear_model import Ridge, Lasso, ElasticNet

# Ridge → L2 regularization (büyük katsayıları küçültür)
ridge = Ridge(alpha=1.0)

# Lasso → L1 regularization (özellik seçimi yapar)
lasso = Lasso(alpha=0.1)

# ElasticNet → L1 + L2
elastic = ElasticNet(alpha=0.1, l1_ratio=0.5)
```

## 💡 Bağlantılar
- [[ML - Model Değerlendirme Metrikleri]]
- [[ML - Özellik Seçimi]]
- [[ML - Hiperparametre Optimizasyonu]]
- [[İstatistik - Regresyon Analizi]]

## ❓ Sorular / Anlamadıklarım
- Ridge ve Lasso'dan hangisi ne zaman tercih edilmeli?
- R² negatif çıkabilir mi, ne anlama gelir?

## 🔗 Kaynaklar
- https://scikit-learn.org/stable/modules/linear_model.html
