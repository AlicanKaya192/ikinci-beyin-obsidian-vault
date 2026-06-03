---
tarih: 2026-05-28
konu: Time Series
etiket: ["time-series", "lstm", "derin-öğrenme", "neural-network", "tensorflow"]
kaynak: TensorFlow / Keras Dokümantasyon
zorluk: ileri
---

## 📌 Özet
LSTM (Long Short-Term Memory), uzun vadeli bağımlılıkları öğrenebilen özel bir RNN türüdür. Karmaşık örüntüleri ve çok değişkenli zaman serilerini modellemede güçlüdür.

## 🧠 Detay

### Veri Hazırlama
```python
import numpy as np
import pandas as pd
from sklearn.preprocessing import MinMaxScaler

# Ölçekleme (LSTM için zorunlu)
scaler = MinMaxScaler()
veri_scaled = scaler.fit_transform(df["satis"].values.reshape(-1, 1))

# Pencereli veri seti oluştur
def pencere_olustur(veri, pencere=12):
    X, y = [], []
    for i in range(len(veri) - pencere):
        X.append(veri[i:i+pencere, 0])
        y.append(veri[i+pencere, 0])
    return np.array(X), np.array(y)

pencere = 12
X, y = pencere_olustur(veri_scaled, pencere)

# LSTM input şekli: (samples, timesteps, features)
X = X.reshape(X.shape[0], X.shape[1], 1)

# Train/test
train_boyut = int(len(X) * 0.8)
X_train, X_test = X[:train_boyut], X[train_boyut:]
y_train, y_test = y[:train_boyut], y[train_boyut:]
```

### LSTM Modeli
```python
import tensorflow as tf
from tensorflow import keras

model = keras.Sequential([
    keras.layers.LSTM(64, return_sequences=True,
        input_shape=(pencere, 1)),
    keras.layers.Dropout(0.2),
    keras.layers.LSTM(32, return_sequences=False),
    keras.layers.Dropout(0.2),
    keras.layers.Dense(16, activation="relu"),
    keras.layers.Dense(1)
])

model.compile(
    optimizer=keras.optimizers.Adam(learning_rate=0.001),
    loss="mse",
    metrics=["mae"]
)

model.summary()

# Eğitim
history = model.fit(
    X_train, y_train,
    epochs=100,
    batch_size=32,
    validation_split=0.2,
    callbacks=[
        keras.callbacks.EarlyStopping(patience=15, restore_best_weights=True),
        keras.callbacks.ReduceLROnPlateau(patience=7, factor=0.5)
    ],
    verbose=1
)
```

### Tahmin ve Geri Ölçekleme
```python
import matplotlib.pyplot as plt

y_pred_scaled = model.predict(X_test)
y_pred = scaler.inverse_transform(y_pred_scaled)
y_gercek = scaler.inverse_transform(y_test.reshape(-1, 1))

mape = np.mean(np.abs((y_gercek - y_pred) / y_gercek)) * 100
print(f"MAPE: {mape:.2f}%")

plt.figure(figsize=(12, 5))
plt.plot(y_gercek, label="Gerçek")
plt.plot(y_pred, label="Tahmin", linestyle="--")
plt.legend()
plt.title("LSTM Tahmini")
```

### Çok Değişkenli LSTM
```python
# Birden fazla özellik
ozellikler = ["satis", "fiyat", "reklam"]
veri_multi = df[ozellikler].values
scaler_multi = MinMaxScaler()
veri_scaled_multi = scaler_multi.fit_transform(veri_multi)

def multi_pencere(veri, pencere=12):
    X, y = [], []
    for i in range(len(veri) - pencere):
        X.append(veri[i:i+pencere, :])   # tüm özellikler
        y.append(veri[i+pencere, 0])      # sadece satis
    return np.array(X), np.array(y)

X_multi, y_multi = multi_pencere(veri_scaled_multi)
# X şekli: (samples, timesteps, features=3)
```

### Eğitim Geçmişi
```python
fig, axes = plt.subplots(1, 2, figsize=(12, 4))
axes[0].plot(history.history["loss"], label="Eğitim")
axes[0].plot(history.history["val_loss"], label="Validasyon")
axes[0].set_title("Loss")
axes[0].legend()

axes[1].plot(history.history["mae"], label="Eğitim MAE")
axes[1].plot(history.history["val_mae"], label="Validasyon MAE")
axes[1].set_title("MAE")
axes[1].legend()
plt.tight_layout()
```

## 💡 Bağlantılar
- [[TS - ML ile Zaman Serisi Tahmini]]
- [[TS - Model Değerlendirme Metrikleri]]
- [[ML - Sinir Ağları Temel]]

## ❓ Sorular / Anlamadıklarım
- Pencere boyutu nasıl seçilir?
- GRU ile LSTM karşılaştırması ne zaman GRU daha iyi?

## 🔗 Kaynaklar
- https://www.tensorflow.org/tutorials/structured_data/time_series
