---
tarih: 2026-05-28
konu: Machine Learning
etiket: ["ml", "sınıflandırma", "lojistik-regresyon", "gözetimli"]
kaynak: Scikit-learn Dokümantasyon
zorluk: başlangıç
---

## 📌 Özet
Lojistik regresyon, ikili sınıflandırma için kullanılan temel algoritmadır. Sigmoid fonksiyonu ile olasılık üretir. Açıklanabilirliği yüksektir.

## 🧠 Detay

### Matematiksel Temel
```
P(y=1) = 1 / (1 + e^(-z))
z = β₀ + β₁x₁ + ... + βₙxₙ

Sigmoid → [0,1] arasında olasılık döndürür
Eşik (threshold) → genellikle 0.5
```

### Scikit-learn ile Uygulama
```python
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split
from sklearn.metrics import classification_report

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

model = LogisticRegression(max_iter=1000, random_state=42)
model.fit(X_train, y_train)

y_pred = model.predict(X_test)
y_prob = model.predict_proba(X_test)[:, 1]

print(classification_report(y_test, y_pred))
```

### Katsayı Yorumlama
```python
import pandas as pd

katsayilar = pd.DataFrame({
    "Özellik": X.columns,
    "Katsayı": model.coef_[0],
    "Odds Ratio": np.exp(model.coef_[0])
}).sort_values("Katsayı", ascending=False)

print(katsayilar)
# Pozitif katsayı → pozitif sınıf olasılığını artırır
```

### Eşik Ayarlama
```python
# Varsayılan eşik 0.5
y_pred_default = model.predict(X_test)

# Özel eşik
esik = 0.3
y_pred_custom = (y_prob >= esik).astype(int)

# Recall/Precision dengesi için eşik seç
from sklearn.metrics import precision_recall_curve
precision, recall, thresholds = precision_recall_curve(y_test, y_prob)
```

### Çok Sınıflı (Multiclass)
```python
model = LogisticRegression(
    multi_class="multinomial",
    solver="lbfgs",
    max_iter=1000
)
```

### Regularization
```python
# C → 1/lambda, küçük C = güçlü regularization
model_l1 = LogisticRegression(penalty="l1", C=0.1, solver="liblinear")
model_l2 = LogisticRegression(penalty="l2", C=1.0)
```

## 💡 Bağlantılar
- [[ML - Model Değerlendirme Metrikleri]]
- [[ML - Lineer Regresyon]]
- [[ML - Karar Ağaçları]]

## ❓ Sorular / Anlamadıklarım
- Lineer regresyon ile lojistik regresyon arasındaki temel fark?
- Eşiği nasıl optimal seçerim?

## 🔗 Kaynaklar
- https://scikit-learn.org/stable/modules/linear_model.html#logistic-regression
