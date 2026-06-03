---
tarih: 2025-01-01
konu: Tarih ve Zaman Özellikleri, Döngüsel Encoding, Gecikme Özellikleri
etiket: [feature-engineering, tarih, zaman, datetime, lag, döngüsel-encoding]
kaynak:
zorluk: ⭐⭐⭐
---

## 📌 Özet

Tarih ve zaman sütunları ham haliyle modele verilemez. Yıl, ay, gün, haftanın günü gibi bileşenlere ayrılmalı; döngüsel özellikler için trigonometrik encoding uygulanmalıdır.

---

## 🧠 Detay

### Temel Datetime İşlemleri

```python
import pandas as pd

df['tarih'] = pd.to_datetime(df['tarih'])

# Tüm bileşenler
df['yil']           = df['tarih'].dt.year
df['ay']            = df['tarih'].dt.month          # 1-12
df['gun']           = df['tarih'].dt.day            # 1-31
df['saat']          = df['tarih'].dt.hour           # 0-23
df['dakika']        = df['tarih'].dt.minute         # 0-59
df['saniye']        = df['tarih'].dt.second
df['haftanin_gunu'] = df['tarih'].dt.dayofweek      # 0=Pzt, 6=Paz
df['hafta_no']      = df['tarih'].dt.isocalendar().week
df['yilin_gunu']    = df['tarih'].dt.dayofyear      # 1-366
df['ceyrek']        = df['tarih'].dt.quarter        # 1-4
df['ayin_gunu']     = df['tarih'].dt.day
df['yil_haftasi']   = df['tarih'].dt.weekofyear

# Ay sonu / başı
df['ay_basi_mi']   = (df['gun'] == 1).astype(int)
df['ay_sonu_mu']   = df['tarih'].dt.is_month_end.astype(int)
df['ceyrek_sonu']  = df['tarih'].dt.is_quarter_end.astype(int)
```

---

### İkili (Binary) Zaman Özellikleri

```python
# Hafta sonu
df['hafta_sonu']  = (df['haftanin_gunu'] >= 5).astype(int)
df['is_gunu']     = (df['haftanin_gunu'] < 5).astype(int)

# Günün saatleri
df['sabah']       = df['saat'].between(6, 11).astype(int)
df['oglen']       = df['saat'].between(12, 17).astype(int)
df['aksam']       = df['saat'].between(18, 22).astype(int)
df['gece']        = (~df['saat'].between(6, 22)).astype(int)

# Mevsim
def mevsim(ay):
    if ay in [12, 1, 2]:  return 'Kış'
    elif ay in [3, 4, 5]: return 'İlkbahar'
    elif ay in [6, 7, 8]: return 'Yaz'
    else:                  return 'Sonbahar'

df['mevsim'] = df['ay'].apply(mevsim)

# Tatil günleri (Türkiye)
import holidays
tr_holidays = holidays.Turkey()
df['tatil_mi'] = df['tarih'].dt.date.isin(tr_holidays).astype(int)
```

---

### Döngüsel (Cyclic) Encoding ⭐

**Sorun**: Ay=12 ve Ay=1 arasındaki fark gerçekte küçük ama sayısal olarak 11!
**Çözüm**: Sinüs ve kosinüs encoding ile dairesel yapıyı koru.

```python
import numpy as np

# Ay (1-12)
df['ay_sin'] = np.sin(2 * np.pi * df['ay'] / 12)
df['ay_cos'] = np.cos(2 * np.pi * df['ay'] / 12)

# Haftanın günü (0-6)
df['gun_sin'] = np.sin(2 * np.pi * df['haftanin_gunu'] / 7)
df['gun_cos'] = np.cos(2 * np.pi * df['haftanin_gunu'] / 7)

# Saat (0-23)
df['saat_sin'] = np.sin(2 * np.pi * df['saat'] / 24)
df['saat_cos'] = np.cos(2 * np.pi * df['saat'] / 24)

# Yılın günü (1-365)
df['yilgun_sin'] = np.sin(2 * np.pi * df['yilin_gunu'] / 365)
df['yilgun_cos'] = np.cos(2 * np.pi * df['yilin_gunu'] / 365)
```

---

### Tarih Farkı Özellikleri

```python
bugun = pd.Timestamp.now()

# Müşteri ilişki süresi
df['musteri_gunu']   = (bugun - df['kayit_tarihi']).dt.days
df['musteri_yili']   = df['musteri_gunu'] / 365
df['son_satin_alma'] = (bugun - df['son_siparis']).dt.days

# İki olay arası fark
df['siparis_teslimat_suresi'] = (df['teslimat_tarihi'] - df['siparis_tarihi']).dt.days

# Referans tarihten uzaklık
covid_start = pd.Timestamp('2020-03-11')
df['covid_gunu'] = (df['tarih'] - covid_start).dt.days.clip(lower=0)
```

---

### Gecikme (Lag) Özellikleri — Zaman Serisi

```python
# Zaman serisi verisi sıralı olmalı!
df = df.sort_values('tarih')

# Gecikmeli değerler
df['satis_lag1'] = df['satis'].shift(1)   # 1 gün önce
df['satis_lag7'] = df['satis'].shift(7)   # 7 gün önce
df['satis_lag30'] = df['satis'].shift(30) # 30 gün önce

# Hareketli ortalama
df['satis_ma7']  = df['satis'].rolling(7).mean()
df['satis_ma30'] = df['satis'].rolling(30).mean()
df['satis_ma7_std'] = df['satis'].rolling(7).std()

# Üstel hareketli ortalama
df['satis_ema7'] = df['satis'].ewm(span=7, adjust=False).mean()

# Hareketli min/max
df['satis_max7'] = df['satis'].rolling(7).max()
df['satis_min7'] = df['satis'].rolling(7).min()

# Çeyreklik değişim
df['satis_pct1'] = df['satis'].pct_change(1)
df['satis_pct7'] = df['satis'].pct_change(7)
```

---

### Grup Bazlı Zaman Özellikleri

```python
# Her ürün için kümülatif satış
df['urun_kumulatif_satis'] = df.groupby('urun_id')['satis'].cumsum()

# Her müşteri için önceki sipariş sayısı
df['musteri_siparis_no'] = df.groupby('musteri_id').cumcount() + 1

# Gruba göre gecikme
df['musteri_lag1_satis'] = df.groupby('musteri_id')['satis'].shift(1)

# Son N günün ortalaması (grup bazlı)
df['musteri_7gun_ort'] = (
    df.groupby('musteri_id')['satis']
    .transform(lambda x: x.rolling(7, min_periods=1).mean())
)
```

---

### Zaman Serisi için Train-Test Ayrımı

```python
# ❌ YANLIŞ — Gelecek bilgisi sızar
from sklearn.model_selection import train_test_split
X_train, X_test = train_test_split(df, test_size=0.2)

# ✅ DOĞRU — Kronolojik bölme
cutoff = df['tarih'].quantile(0.8)
train = df[df['tarih'] < cutoff]
test  = df[df['tarih'] >= cutoff]

# TimeSeriesSplit ile CV
from sklearn.model_selection import TimeSeriesSplit
tscv = TimeSeriesSplit(n_splits=5)
for train_idx, test_idx in tscv.split(X):
    X_tr, X_te = X.iloc[train_idx], X.iloc[test_idx]
```

---

## 💡 Bağlantılar
- [[FE - Özellik Türetme]]
- [[DS - Zaman Serisi Analizi]]
- [[STAT - Zaman Serisi Analizi]]

## ❓ Sorular / Anlamadıklarım


## 🔗 Kaynaklar
- pandas.DatetimeTZDtype Documentation
- Kaggle: Feature Engineering - Date/Time Features
