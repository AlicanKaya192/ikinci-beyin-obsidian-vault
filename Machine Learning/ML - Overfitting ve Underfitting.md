---
tarih: 2026-05-28
konu: Machine Learning
etiket: ["ml", "overfitting", "underfitting", "regularization", "bias-variance"]
kaynak: 
zorluk: orta
---

## 📌 Özet
Overfitting eğitim verisini ezberleme, underfitting ise yeterince öğrenememe sorunudur. Bias-Variance dengesi bu iki uç arasında optimal noktayı bulmayı gerektirir.

## 🧠 Detay

### Bias-Variance Trade-off
```
Yüksek Bias (Underfitting):
  - Eğitim hatası yüksek
  - Test hatası yüksek
  - Model çok basit

Yüksek Variance (Overfitting):
  - Eğitim hatası düşük
  - Test hatası yüksek
  - Model çok karmaşık

Hedef: Düşük bias + Düşük variance
```

### Tespit
```python
from sklearn.model_selection import learning_curve
import matplotlib.pyplot as plt
import numpy as np

train_boyut, train_skor, val_skor = learning_curve(
    model, X, y,
    cv=5,
    train_sizes=np.linspace(0.1, 1.0, 10),
    scoring="accuracy"
)

plt.plot(train_boyut, train_skor.mean(axis=1), label="Eğitim")
plt.plot(train_boyut, val_skor.mean(axis=1), label="Validasyon")
plt.xlabel("Eğitim Seti Boyutu")
plt.ylabel("Doğruluk")
plt.legend()
plt.title("Öğrenme Eğrisi")

# Overfitting → iki eğri arası büyük açık
# Underfitting → her iki eğri de düşük
```

### Overfitting Çözümleri
```python
# 1. Regularization
from sklearn.linear_model import Ridge, Lasso
ridge = Ridge(alpha=10.0)

# 2. Daha fazla veri
# 3. Daha basit model
from sklearn.tree import DecisionTreeClassifier
dt = DecisionTreeClassifier(max_depth=5)  # sınırlı derinlik

# 4. Dropout (derin öğrenmede)
# 5. Early stopping
# 6. Özellik azaltma
```

### Underfitting Çözümleri
```python
# 1. Daha karmaşık model
rf = RandomForestClassifier(n_estimators=300, max_depth=None)

# 2. Daha fazla özellik / feature engineering
# 3. Regularization'ı azalt
ridge = Ridge(alpha=0.001)

# 4. Daha uzun eğitim
```

### Validation Curve
```python
from sklearn.model_selection import validation_curve

train_skor, val_skor = validation_curve(
    DecisionTreeClassifier(), X, y,
    param_name="max_depth",
    param_range=range(1, 20),
    cv=5
)

plt.plot(range(1, 20), train_skor.mean(axis=1), label="Eğitim")
plt.plot(range(1, 20), val_skor.mean(axis=1), label="Validasyon")
plt.xlabel("max_depth")
plt.legend()
```

## 💡 Bağlantılar
- [[ML - Eğitim Test Ayrımı ve Cross Validation]]
- [[ML - Hiperparametre Optimizasyonu]]
- [[ML - Lineer Regresyon]]

## ❓ Sorular / Anlamadıklarım
- Eğitim ve test skoru arasında ne kadar fark overfitting sayılır?
- Veri artırma (data augmentation) overfitting'i nasıl azaltır?

## 🔗 Kaynaklar
- https://scikit-learn.org/stable/modules/learning_curve.html
