---
tarih: 2026-05-28
konu: Time Series
etiket: ["time-series", "prophet", "facebook", "tahmin", "mevsimsel"]
kaynak: Prophet Dokümantasyon
zorluk: orta
---

## 📌 Özet
Prophet, Meta (Facebook) tarafından geliştirilmiş, tatil ve çoklu mevsimselliği destekleyen güçlü bir tahmin kütüphanesidir. Parametreler sezgisel, kullanımı kolaydır.

## 🧠 Detay

### Kurulum ve Veri Formatı
```python
pip install prophet

from prophet import Prophet
import pandas as pd

# Prophet zorunlu sütun formatı: ds (tarih) ve y (değer)
df_prophet = df.reset_index().rename(columns={"tarih": "ds", "satis": "y"})
print(df_prophet.head())
#          ds      y
# 0 2020-01-01  1234.5
```

### Temel Model
```python
model = Prophet(
    yearly_seasonality=True,
    weekly_seasonality=True,
    daily_seasonality=False,
    seasonality_mode="multiplicative",  # veya "additive"
    changepoint_prior_scale=0.05,       # trend değişim hassasiyeti
    seasonality_prior_scale=10.0
)

model.fit(df_prophet)

# Gelecek veri çerçevesi
gelecek = model.make_future_dataframe(periods=365, freq="D")
tahmin = model.predict(gelecek)

print(tahmin[["ds", "yhat", "yhat_lower", "yhat_upper"]].tail())
```

### Görselleştirme
```python
# Tahmin grafiği
fig1 = model.plot(tahmin)
fig1.suptitle("Prophet Tahmini")

# Bileşenler
fig2 = model.plot_components(tahmin)
# Trend, haftalık mevsimsellik, yıllık mevsimsellik
```

### Tatil Günleri
```python
from prophet.make_holidays import make_holidays_df

# Türkiye tatilleri
tr_tatiller = pd.DataFrame({
    "holiday": "turkiye_tatil",
    "ds": pd.to_datetime([
        "2024-01-01",  # Yılbaşı
        "2024-04-23",  # Ulusal Egemenlik
        "2024-05-01",  # İşçi Bayramı
        "2024-05-19",  # Atatürk'ü Anma
        "2024-08-30",  # Zafer Bayramı
        "2024-10-29",  # Cumhuriyet Bayramı
    ]),
    "lower_window": -1,
    "upper_window": 1
})

model = Prophet(holidays=tr_tatiller)
model.fit(df_prophet)
```

### Özel Mevsimsellik Ekleme
```python
model = Prophet(yearly_seasonality=False)  # varsayılanı kapat

# Aylık mevsimsellik
model.add_seasonality(name="monthly", period=30.5, fourier_order=5)

# Çeyreklik
model.add_seasonality(name="quarterly", period=91.25, fourier_order=3)

model.fit(df_prophet)
```

### Dış Değişken (Regressor)
```python
# Fiyat, reklam harcaması gibi
df_prophet["fiyat"] = fiyat_listesi

model = Prophet()
model.add_regressor("fiyat")
model.fit(df_prophet)

gelecek["fiyat"] = gelecek_fiyat
tahmin = model.predict(gelecek)
```

### Cross Validation
```python
from prophet.diagnostics import cross_validation, performance_metrics

df_cv = cross_validation(
    model,
    initial="365 days",
    period="90 days",
    horizon="90 days"
)

df_p = performance_metrics(df_cv)
print(df_p[["horizon", "mape", "rmse"]].head())
```

## 💡 Bağlantılar
- [[TS - SARIMA Modeli]]
- [[TS - Model Değerlendirme Metrikleri]]
- [[TS - Mevsimsellik Analizi]]

## ❓ Sorular / Anlamadıklarım
- changepoint_prior_scale nasıl ayarlanır?
- ARIMA ile Prophet karşılaştırması ne zaman Prophet daha iyi?

## 🔗 Kaynaklar
- https://facebook.github.io/prophet/docs/quick_start.html
