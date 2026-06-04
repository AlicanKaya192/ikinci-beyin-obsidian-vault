---
tarih: 2026-05-28
konu: Machine Learning
etiket: ["ml", "regresyon", "lineer", "gözetimli"]
kaynak: Scikit-learn Dokümantasyon
zorluk: başlangıç
---

## 📌 Özet
Lineer Regresyon, bir hedef değişken (bağımlı değişken) ile bir veya daha fazla tahmin edici değişken (bağımsız değişken) arasındaki doğrusal ilişkiyi modelleyen, hem istatistikte hem de makine öğrenmesinde temel taş kabul edilen bir algoritmadır. Temel amacı, gözlemlenen veri noktaları ile modelin tahmin ettiği doğru arasındaki hata kareler toplamını (Residual Sum of Squares) minimize eden en uygun doğruyu (veya hiper-düzlemi) bulmaktır. Katsayıların doğrudan yorumlanabilir olması (örneğin; X'teki 1 birimlik artışın Y'de ne kadar değişikliğe yol açtığı), bu modeli finans, sağlık ve ekonomi gibi alanlarda vazgeçilmez kılar. Ancak başarısı; doğrusallık, eş varyanslılık ve normal dağılım gibi varsayımların karşılanmasına bağlıdır.

## 🧠 Detay

### Lineer Regresyon Mekanizması
```mermaid
graph LR
    X["Girdiler (X1, X2, ... Xn)"] --> W["Ağırlıklar (β1, β2, ... βn)"]
    W --> S["Toplam (Σ W*X + β0)"]
    S --> Y["Tahmin Edilen Değer (ŷ)"]
    Y --> E["Hata Hesaplama (y - ŷ)"]
    E --> O["Katsayı Güncelleme (OLS)"]
```

### Matematiksel Temel
Lineer regresyonun arkasındaki temel denklem şudur:
```
y = β₀ + β₁x₁ + β₂x₂ + ... + βₙxₙ + ε

β₀ → Sabit terim (intercept), tüm X'ler sıfırken Y'nin değeri
β₁...βₙ → Katsayılar (coefficients), bağımsız değişkenlerin etkisi
ε → Hata terimi (residual), modelin açıklayamadığı kısım
```
...
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
