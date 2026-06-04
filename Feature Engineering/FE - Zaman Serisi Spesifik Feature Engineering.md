---
tarih: 2026-05-28
konu: Feature Engineering
etiket: [feature-engineering, zaman-serisi, tsfresh, lag, pencere, döngüsel]
kaynak: tsfresh Dokümantasyon
zorluk: ⭐⭐⭐
---

## 📌 Özet
Zaman serisi verilerinden özellik üretmek; lag özellikler, pencere istatistikleri, frekans domain özellikleri ve tsfresh gibi otomatik araçlarla yapılır. Gelecek bilgisi sızmaması (temporal leakage) kritiktir.

---

## 🧠 Detay

### Temel Lag ve Pencere Özellikleri

```python
import pandas as pd
import numpy as np

# Seri sıralı olmalı!
df = df.sort_values('tarih').reset_index(drop=True)

def lag_ozellikler(df, hedef, lag_listesi):
    """Gecikmeli değerler"""
    for lag in lag_listesi:
        df[f'{hedef}_lag{lag}'] = df[hedef].shift(lag)
    return df

def pencere_ozellikler(df, hedef, pencereler):
    """Hareketli istatistikler (shift(1) → leakage önler)"""
    seri = df[hedef].shift(1)  # ⚠️ shift zorunlu!

    for p in pencereler:
        df[f'{hedef}_ma{p}']      = seri.rolling(p).mean()
        df[f'{hedef}_std{p}']     = seri.rolling(p).std()
        df[f'{hedef}_min{p}']     = seri.rolling(p).min()
        df[f'{hedef}_max{p}']     = seri.rolling(p).max()
        df[f'{hedef}_median{p}']  = seri.rolling(p).median()
        df[f'{hedef}_skew{p}']    = seri.rolling(p).skew()
        df[f'{hedef}_range{p}']   = (seri.rolling(p).max() -
                                      seri.rolling(p).min())
    return df

df = lag_ozellikler(df, 'satis', lag_listesi=[1, 2, 3, 7, 14, 21, 28])
df = pencere_ozellikler(df, 'satis', pencereler=[3, 7, 14, 28])
```

### Üstel Hareketli Ortalama (EMA)

```python
def ema_ozellikler(df, hedef, spanlar):
    """Üstel ağırlıklı — son değerlere daha fazla ağırlık"""
    for span in spanlar:
        df[f'{hedef}_ema{span}'] = (
            df[hedef].shift(1).ewm(span=span, adjust=False).mean()
        )
    return df

df = ema_ozellikler(df, 'satis', spanlar=[3, 7, 14, 28])
```

### Değişim ve Momentum Özellikleri

```python
def degisim_ozellikler(df, hedef, periyotlar):
    """Yüzde değişim, fark ve momentum"""
    for p in periyotlar:
        df[f'{hedef}_pct{p}']  = df[hedef].pct_change(p)
        df[f'{hedef}_diff{p}'] = df[hedef].diff(p)

    # Momentum: kısa MA - uzun MA
    df[f'{hedef}_momentum'] = (
        df[hedef].rolling(7).mean() - df[hedef].rolling(28).mean()
    )

    # Trend yönü
    df[f'{hedef}_trend_yukari'] = (df[hedef] > df[hedef].shift(1)).astype(int)

    return df

df = degisim_ozellikler(df, 'satis', periyotlar=[1, 7, 14, 28])
```

### Mevsimsel Özellikler

```python
def mevsimsel_ozellikler(df, tarih_col='tarih'):
    """Döngüsel zaman bileşenleri"""
    df[tarih_col] = pd.to_datetime(df[tarih_col])

    # Ham bileşenler
    df['ay']          = df[tarih_col].dt.month
    df['gun']         = df[tarih_col].dt.dayofweek   # 0=Pzt
    df['haftano']     = df[tarih_col].dt.isocalendar().week.astype(int)
    df['yilin_gunu']  = df[tarih_col].dt.dayofyear
    df['ceyrek']      = df[tarih_col].dt.quarter

    # Döngüsel encoding (ay 12 → ay 1 yakın!)
    df['ay_sin'] = np.sin(2 * np.pi * df['ay'] / 12)
    df['ay_cos'] = np.cos(2 * np.pi * df['ay'] / 12)
    df['gun_sin'] = np.sin(2 * np.pi * df['gun'] / 7)
    df['gun_cos'] = np.cos(2 * np.pi * df['gun'] / 7)
    df['yilgun_sin'] = np.sin(2 * np.pi * df['yilin_gunu'] / 365)
    df['yilgun_cos'] = np.cos(2 * np.pi * df['yilin_gunu'] / 365)

    # İkili özellikler
    df['hafta_sonu']   = (df['gun'] >= 5).astype(int)
    df['ay_sonu']      = df[tarih_col].dt.is_month_end.astype(int)
    df['ceyrek_sonu']  = df[tarih_col].dt.is_quarter_end.astype(int)

    return df
```

### Grup Bazlı Zaman Özellikleri

```python
def grup_zaman_ozellikler(df, grup_col, hedef, pencereler=[7, 28]):
    """Her grup (ürün, mağaza) için ayrı pencere istatistikleri"""

    df = df.sort_values([grup_col, 'tarih'])

    for p in pencereler:
        # Her grup için bağımsız rolling
        df[f'{hedef}_grup_ma{p}'] = (
            df.groupby(grup_col)[hedef]
            .transform(lambda x: x.shift(1).rolling(p, min_periods=1).mean())
        )
        df[f'{hedef}_grup_std{p}'] = (
            df.groupby(grup_col)[hedef]
            .transform(lambda x: x.shift(1).rolling(p, min_periods=1).std())
        )

    # Kümülatif istatistikler
    df[f'{hedef}_kumulatif'] = (
        df.groupby(grup_col)[hedef].cumsum().shift(1)
    )
    df[f'{hedef}_grup_rank'] = (
        df.groupby([grup_col, 'ay'])[hedef].rank(method='dense')
    )

    return df
```

### tsfresh ile Otomatik Özellik Üretimi ⭐

```python
from tsfresh import extract_features, select_features
from tsfresh.utilities.dataframe_functions import impute
from tsfresh.feature_extraction import ComprehensiveFCParameters, MinimalFCParameters

# Veri formatı: id, time, value
df_ts = pd.DataFrame({
    'id': np.repeat(range(100), 50),    # 100 zaman serisi
    'time': list(range(50)) * 100,
    'value': np.random.randn(5000)
})

# Minimal (hızlı, ~10 özellik)
features_minimal = extract_features(
    df_ts,
    column_id='id',
    column_sort='time',
    column_value='value',
    default_fc_parameters=MinimalFCParameters()
)

# Comprehensive (yavaş, 700+ özellik)
features_full = extract_features(
    df_ts,
    column_id='id',
    column_sort='time',
    column_value='value',
    default_fc_parameters=ComprehensiveFCParameters(),
    n_jobs=4    # Paralel işlem
)

# Eksik değer doldur
impute(features_full)
print(f"Üretilen özellik: {features_full.shape[1]}")

# Hedefle ilgili olanları seç
features_filtered = select_features(features_full, y)
print(f"Seçilen özellik: {features_filtered.shape[1]}")
```

### tsfresh Özel Parametreler

```python
from tsfresh.feature_extraction import EfficientFCParameters

# Belirli özellik aileleri
ozel_parametreler = {
    "mean": None,
    "standard_deviation": None,
    "maximum": None,
    "minimum": None,
    "skewness": None,
    "kurtosis": None,
    "autocorrelation": [{"lag": l} for l in [1, 2, 3, 7, 14]],
    "fourier_entropy": [{"bins": 10}],
    "number_peaks": [{"n": 3}, {"n": 5}],
    "longest_strike_above_mean": None,
    "count_above_mean": None
}

features_custom = extract_features(
    df_ts,
    column_id='id',
    column_sort='time',
    column_value='value',
    default_fc_parameters=ozel_parametreler
)
```

### Frekans Domain Özellikleri (FFT)

```python
from scipy import fft, signal

def fft_ozellikler(seri, n_harmonik=10):
    """Fourier dönüşümü ile frekans özellikleri"""
    n = len(seri)
    fft_values = np.abs(fft.fft(seri))[:n//2]
    freqs = fft.fftfreq(n)[:n//2]

    # En güçlü harmonikler
    top_idx = np.argsort(fft_values)[-n_harmonik:]

    ozellikler = {}
    for i, idx in enumerate(top_idx):
        ozellikler[f'fft_freq_{i}'] = freqs[idx]
        ozellikler[f'fft_amp_{i}'] = fft_values[idx]

    # Toplam güç
    ozellikler['fft_toplam_guc'] = np.sum(fft_values**2)
    ozellikler['fft_dominant_freq'] = freqs[np.argmax(fft_values)]

    return ozellikler

# Zaman serisi başı için uygula
df_fft = pd.DataFrame([
    fft_ozellikler(grp['satis'].values)
    for _, grp in df.groupby('urun_id')
])
```

### Temporal Leakage Önleme ⚠️

```python
# ❌ YANLIŞ — gelecek bilgisi sızıyor
df['ma7'] = df['satis'].rolling(7).mean()  # son 7 gün (bugün dahil!)

# ✅ DOĞRU — shift(1) ile geçmişe bak
df['ma7'] = df['satis'].shift(1).rolling(7).mean()

# ❌ YANLIŞ — TimeSeriesSplit yerine random split
X_train, X_test = train_test_split(df, test_size=0.2)

# ✅ DOĞRU — kronolojik bölme
cutoff = df['tarih'].quantile(0.8)
train = df[df['tarih'] < cutoff]
test  = df[df['tarih'] >= cutoff]

# ✅ CV için TimeSeriesSplit
from sklearn.model_selection import TimeSeriesSplit
tscv = TimeSeriesSplit(n_splits=5, gap=7)  # gap: sızıntı önler
```

### Tam Pipeline Örneği

```python
def ts_feature_pipeline(df, hedef='satis', tarih='tarih', grup=None):
    """Eksiksiz zaman serisi FE pipeline'ı"""
    df = df.copy().sort_values(tarih)

    # 1. Lag özellikler
    df = lag_ozellikler(df, hedef, [1, 2, 3, 7, 14, 21, 28])

    # 2. Pencere istatistikleri
    df = pencere_ozellikler(df, hedef, [3, 7, 14, 28])

    # 3. EMA
    df = ema_ozellikler(df, hedef, [3, 7, 14])

    # 4. Değişim
    df = degisim_ozellikler(df, hedef, [1, 7, 28])

    # 5. Mevsimsel
    df = mevsimsel_ozellikler(df, tarih)

    # 6. Grup bazlı (varsa)
    if grup:
        df = grup_zaman_ozellikler(df, grup, hedef, [7, 28])

    # 7. Eksik değerleri düşür (lag nedeniyle oluşan)
    df = df.dropna()

    return df

df_features = ts_feature_pipeline(df, hedef='satis', tarih='tarih', grup='urun_id')
print(f"Üretilen özellik sayısı: {df_features.shape[1]}")
```

---

## 💡 Bağlantılar
- [[FE - Tarih ve Zaman Özellikleri]]
- [[FE - Otomatik Feature Engineering]]
- [[FE - Data Leakage ve Pipeline Doğruluğu]]
- [[TS - Zaman Serisi Temel Kavramlar]]
- [[TS - ML ile Zaman Serisi Tahmini]]

## ❓ Sorular / Anlamadıklarım
- tsfresh Comprehensive vs Efficient ne zaman hangisi?
- Çok değişkenli zaman serisi için özellikler nasıl üretilir?

## 🔗 Kaynaklar
- https://tsfresh.readthedocs.io/
- https://pandas.pydata.org/docs/user_guide/timeseries.html
