---
tarih: 2026-05-28
konu: Time Series
etiket: ["time-series", "durağanlık", "adf", "kpss", "birim-kök"]
kaynak: Statsmodels Dokümantasyon
zorluk: orta
---

## 📌 Özet
Durağanlık, zaman serisi modellerinin temel varsayımıdır. Ortalama, varyans ve otokorelasyon zamanla sabit olmalıdır. ARIMA gibi modeller durağan veri gerektirir.

## 🧠 Detay

### Durağanlık Nedir?
```
Zayıf Durağanlık (Weak Stationarity):
  1. E[y(t)] = μ (sabit ortalama)
  2. Var[y(t)] = σ² (sabit varyans)
  3. Cov[y(t), y(t-k)] sadece k'ya bağlı

Güçlü Durağanlık: Tüm dağılım zamanla değişmez
Pratikte zayıf durağanlık yeterlidir
```

### Görsel Kontrol
```python
import pandas as pd
import matplotlib.pyplot as plt

def durağanlık_grafik(seri, pencere=12):
    fig, axes = plt.subplots(3, 1, figsize=(12, 8))

    # Ham seri
    axes[0].plot(seri)
    axes[0].set_title("Ham Seri")

    # Hareketli ortalama
    axes[1].plot(seri.rolling(pencere).mean(), label="HO", color="red")
    axes[1].plot(seri.rolling(pencere).std(), label="Std", color="blue")
    axes[1].legend()
    axes[1].set_title("Hareketli Ortalama ve Std")

    # Fark alınmış seri
    axes[2].plot(seri.diff())
    axes[2].set_title("1. Fark")

    plt.tight_layout()

durağanlık_grafik(df["satis"])
```

### ADF Testi (Augmented Dickey-Fuller)
```python
from statsmodels.tsa.stattools import adfuller

def adf_test(seri, ad="Seri"):
    sonuc = adfuller(seri.dropna(), autolag="AIC")
    print(f"\n{'='*40}")
    print(f"ADF Testi: {ad}")
    print(f"{'='*40}")
    print(f"ADF İstatistiği : {sonuc[0]:.4f}")
    print(f"p-değeri        : {sonuc[1]:.4f}")
    print(f"Kritik Değerler:")
    for k, v in sonuc[4].items():
        print(f"  {k}: {v:.4f}")

    if sonuc[1] < 0.05:
        print("✅ Durağan (H0 reddedildi)")
    else:
        print("❌ Durağan DEĞİL (H0 reddedilemedi)")

adf_test(df["satis"])
```

### KPSS Testi
```python
from statsmodels.tsa.stattools import kpss

def kpss_test(seri):
    stat, p, lags, kritik = kpss(seri.dropna(), regression="c")
    print(f"KPSS İstatistiği: {stat:.4f}")
    print(f"p-değeri: {p:.4f}")
    # p < 0.05 → Durağan DEĞİL
    # ADF ile birlikte kullan
```

### Durağan Hale Getirme

```python
# 1. Fark alma (trend için)
df["fark1"] = df["satis"].diff(1)
adf_test(df["fark1"].dropna())

# 2. Mevsimsel fark (S=12 için)
df["mevsim_fark"] = df["satis"].diff(12)

# 3. Hem trend hem mevsimsel
df["cift_fark"] = df["satis"].diff(1).diff(12)

# 4. Log dönüşümü (artan varyans için)
import numpy as np
df["log_satis"] = np.log(df["satis"])
df["log_fark"] = df["log_satis"].diff(1)

# 5. Box-Cox dönüşümü
from scipy.stats import boxcox
df["boxcox"], lam = boxcox(df["satis"])
print(f"Lambda: {lam:.4f}")
```

### Kaç Fark Almak Gerekir?
```python
from pmdarima.arima import ndiffs, nsdiffs

d = ndiffs(df["satis"], test="adf")   # trend için
D = nsdiffs(df["satis"], m=12)        # mevsimsel için
print(f"d={d}, D={D}")
```

## 💡 Bağlantılar
- [[TS - Zaman Serisi Temel Kavramlar]]
- [[TS - ACF ve PACF Analizi]]
- [[TS - ARIMA Modeli]]

## ❓ Sorular / Anlamadıklarım
- ADF ve KPSS testleri çelişirse ne yapmalıyım?
- Kaç fark almak yeterli, fazla fark almak zararlı mı?

## 🔗 Kaynaklar
- https://www.statsmodels.org/stable/generated/statsmodels.tsa.stattools.adfuller.html
