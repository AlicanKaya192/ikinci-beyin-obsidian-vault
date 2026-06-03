---
tarih: 2026-05-28
konu: Machine Learning
etiket: ["ml", "knn", "k-nearest-neighbors", "sınıflandırma"]
kaynak: Scikit-learn Dokümantasyon
zorluk: başlangıç
---

## 📌 Özet
KNN, yeni bir noktayı en yakın K komşusunun çoğunluk sınıfına göre sınıflandıran basit ama etkili bir algoritmadır. Eğitim gerektirmez, tahmin sırasında mesafe hesaplar.

## 🧠 Detay

### KNN Sınıflandırma
```python
from sklearn.neighbors import KNeighborsClassifier
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline

# KNN mesafeye duyarlı → ölçekleme zorunlu
pipe = Pipeline([
    ("scaler", StandardScaler()),
    ("knn", KNeighborsClassifier(
        n_neighbors=5,
        metric="euclidean",
        weights="uniform"    # veya "distance"
    ))
])

pipe.fit(X_train, y_train)
y_pred = pipe.predict(X_test)
```

### K Değeri Seçimi
```python
import matplotlib.pyplot as plt

train_skorlari = []
test_skorlari = []

for k in range(1, 31):
    pipe = Pipeline([
        ("scaler", StandardScaler()),
        ("knn", KNeighborsClassifier(n_neighbors=k))
    ])
    pipe.fit(X_train, y_train)
    train_skorlari.append(pipe.score(X_train, y_train))
    test_skorlari.append(pipe.score(X_test, y_test))

plt.plot(range(1, 31), train_skorlari, label="Train")
plt.plot(range(1, 31), test_skorlari, label="Test")
plt.xlabel("K değeri")
plt.ylabel("Doğruluk")
plt.legend()
# Test eğrisinin tepe noktası → optimal K
```

### KNN Regresyon
```python
from sklearn.neighbors import KNeighborsRegressor

knn_reg = Pipeline([
    ("scaler", StandardScaler()),
    ("knn", KNeighborsRegressor(n_neighbors=5))
])
```

### Mesafe Metrikleri
```python
# Öklid (Euclidean) → sürekli değişkenler
KNeighborsClassifier(metric="euclidean")

# Manhattan → aykırı değere dayanıklı
KNeighborsClassifier(metric="manhattan")

# Minkowski → genel form
KNeighborsClassifier(metric="minkowski", p=2)
```

### Avantaj ve Dezavantajlar
```
✅ Basit, yorumlanabilir
✅ Eğitim süresi yok
✅ Doğrusal olmayan sınırlar

❌ Büyük veri setlerinde yavaş
❌ Yüksek boyutlarda kötü (curse of dimensionality)
❌ Ölçeklemeye çok duyarlı
❌ Aykırı değerlerden etkilenir
```

## 💡 Bağlantılar
- [[ML - Makine Öğrenmesine Giriş]]
- [[ML - Model Değerlendirme Metrikleri]]
- [[DS - Veri Dönüşümleri]]

## ❓ Sorular / Anlamadıklarım
- Curse of dimensionality nedir?
- `weights="distance"` ne zaman daha iyi sonuç verir?

## 🔗 Kaynaklar
- https://scikit-learn.org/stable/modules/neighbors.html
