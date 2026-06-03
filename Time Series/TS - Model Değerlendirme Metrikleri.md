---
tarih: 2026-05-28
konu: Time Series
etiket: ["time-series", "metrik", "mape", "rmse", "mae", "değerlendirme"]
kaynak: 
zorluk: orta
---

## 📌 Özet
Zaman serisi tahmin modellerini doğru metrikleri kullanarak değerlendirmek kritiktir. MAE, RMSE, MAPE ve MASE en yaygın kullanılan metriklerdir.

## 🧠 Detay

### Temel Metrikler
```python
import numpy as np
import pandas as pd

def ts_metrikler(gercek, tahmin, ad="Model"):
    gercek = np.array(gercek)
    tahmin = np.array(tahmin)
    hata = gercek - tahmin

    mae   = np.mean(np.abs(hata))
    mse   = np.mean(hata**2)
    rmse  = np.sqrt(mse)
    mape  = np.mean(np.abs(hata / gercek)) * 100
    smape = np.mean(2 * np.abs(hata) / (np.abs(gercek) + np.abs(tahmin))) * 100
    r2    = 1 - np.sum(hata**2) / np.sum((gercek - gercek.mean())**2)

    print(f"\n{'='*35}")
    print(f"Model: {ad}")
    print(f"{'='*35}")
    print(f"MAE:   {mae:.4f}")
    print(f"RMSE:  {rmse:.4f}")
    print(f"MAPE:  {mape:.2f}%")
    print(f"sMAPE: {smape:.2f}%")
    print(f"R²:    {r2:.4f}")

    return {"MAE": mae, "RMSE": rmse, "MAPE": mape, "sMAPE": smape, "R2": r2}
```

### Metrik Açıklamaları
| Metrik | Formül | Yorumlama | Dezavantaj |
|--------|--------|-----------|------------|
| MAE | mean(|y-ŷ|) | Ortalama mutlak hata | Ölçeğe bağlı |
| RMSE | √mean((y-ŷ)²) | Büyük hatalara duyarlı | Ölçeğe bağlı |
| MAPE | mean(|y-ŷ|/y)*100 | % hata | y≈0 sorun |
| sMAPE | 2*mean(|y-ŷ|/(|y|+|ŷ|))*100 | Simetrik % hata | Daha güvenilir |
| MASE | MAE / naif_MAE | Naif modele göre | Basit ama faydalı |

### MASE (Naif Model Karşılaştırması)
```python
def mase(gercek, tahmin, mevsim=1):
    gercek = np.array(gercek)
    tahmin = np.array(tahmin)
    n = len(gercek)

    # Naif model hatası (s adım önceki değer)
    naif_hata = np.mean(np.abs(np.diff(gercek, n=mevsim)))
    model_hata = np.mean(np.abs(gercek - tahmin))

    return model_hata / naif_hata
    # < 1 → modelin naif modelden iyi olduğu
```

### Walk-Forward Cross Validation Metrikleri
```python
from sklearn.model_selection import TimeSeriesSplit

def cv_metrikler(model, X, y, n_splits=5):
    tss = TimeSeriesSplit(n_splits=n_splits)
    mape_listesi = []

    for train_idx, test_idx in tss.split(X):
        X_tr, X_te = X.iloc[train_idx], X.iloc[test_idx]
        y_tr, y_te = y.iloc[train_idx], y.iloc[test_idx]

        model.fit(X_tr, y_tr)
        y_pred = model.predict(X_te)

        mape = np.mean(np.abs((y_te - y_pred) / y_te)) * 100
        mape_listesi.append(mape)

    return {
        "MAPE_ort": np.mean(mape_listesi),
        "MAPE_std": np.std(mape_listesi),
        "MAPE_min": np.min(mape_listesi),
        "MAPE_max": np.max(mape_listesi)
    }
```

### Model Karşılaştırma Tablosu
```python
import matplotlib.pyplot as plt

modeller = {
    "ARIMA": arima_pred,
    "SARIMA": sarima_pred,
    "HoltWinters": hw_pred,
    "Prophet": prophet_pred,
    "XGBoost": xgb_pred
}

karsilastirma = []
for ad, pred in modeller.items():
    m = ts_metrikler(y_test, pred, ad)
    m["Model"] = ad
    karsilastirma.append(m)

df_kar = pd.DataFrame(karsilastirma).set_index("Model")
print(df_kar.sort_values("MAPE"))

# MAPE karşılaştırma grafiği
df_kar["MAPE"].plot(kind="bar", color="steelblue")
plt.title("Model MAPE Karşılaştırması (%)")
plt.ylabel("MAPE (%)")
plt.xticks(rotation=45)
plt.tight_layout()
```

## 💡 Bağlantılar
- [[TS - ARIMA Modeli]]
- [[TS - Prophet ile Tahmin]]
- [[TS - ML ile Zaman Serisi Tahmini]]

## ❓ Sorular / Anlamadıklarım
- MAPE ve sMAPE hangi durumda tercih edilmeli?
- Güven aralığı kalitesi nasıl ölçülür?

## 🔗 Kaynaklar
- https://otexts.com/fpp3/accuracy.html
