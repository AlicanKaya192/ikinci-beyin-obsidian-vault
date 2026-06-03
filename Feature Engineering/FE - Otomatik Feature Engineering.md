---
tarih: 2025-01-01
konu: Otomatik Feature Engineering, Featuretools, AutoFeat, tsfresh
etiket: [feature-engineering, otomatik, featuretools, AutoML, tsfresh, autofe]
kaynak:
zorluk: ⭐⭐⭐
---

## 📌 Özet

Otomatik Feature Engineering araçları, manuel efor gerektirmeden çok sayıda özellik türetir. Özellikle ilişkisel veriler (birden fazla tablo) ve zaman serileri için çok güçlüdür.

---

## 🧠 Detay

### 1. Featuretools (İlişkisel Veri)

```python
import featuretools as ft

# Entity Set oluştur
es = ft.EntitySet(id='eticaret')

# Tabloları ekle
es.add_dataframe(
    dataframe_name='musteriler',
    dataframe=df_musteriler,
    index='musteri_id'
)
es.add_dataframe(
    dataframe_name='siparisler',
    dataframe=df_siparisler,
    index='siparis_id',
    time_index='siparis_tarihi'
)

# İlişkileri tanımla
es.add_relationship('musteriler', 'musteri_id',
                    'siparisler', 'musteri_id')

# Deep Feature Synthesis (DFS)
feature_matrix, feature_defs = ft.dfs(
    entityset=es,
    target_dataframe_name='musteriler',
    max_depth=2,              # Derin türetme derinliği
    agg_primitives=[          # Aggregate: COUNT, SUM, MEAN...
        'count', 'sum', 'mean', 'max', 'min', 'std',
        'n_unique', 'last', 'first'
    ],
    trans_primitives=[        # Transform: YIL, AY, LOG...
        'year', 'month', 'day', 'hour',
        'weekday', 'is_weekend'
    ]
)

print(f"Üretilen özellik sayısı: {feature_matrix.shape[1]}")
print(feature_matrix.head())
```

---

### 2. tsfresh (Zaman Serisi)

```python
from tsfresh import extract_features, select_features
from tsfresh.utilities.dataframe_functions import impute

# Her zaman serisi için yüzlerce istatistik üretir
features = extract_features(
    df_timeseries,
    column_id='id',
    column_sort='zaman',
    column_value='deger'
)

# Eksikleri doldur
impute(features)

# Hedefle ilgili olanları seç
features_filtered = select_features(features, y)

print(f"Toplam: {features.shape[1]}, Seçilen: {features_filtered.shape[1]}")
```

**Üretilen özellikler**: mean, std, min, max, median, skewness, kurtosis, FFT katsayıları, entropy, autocorrelation, peak sayısı, vb. (700+)

---

### 3. AutoFeat

```python
from autofeat import AutoFeatClassifier, AutoFeatRegressor

# Sınıflandırma
afc = AutoFeatClassifier(
    feateng_steps=2,    # Türetme adım sayısı
    max_gb=1,           # Bellek sınırı (GB)
    verbose=1
)
X_train_new = afc.fit_transform(X_train, y_train)
X_test_new  = afc.transform(X_test)

# Regresyon
afr = AutoFeatRegressor(feateng_steps=2)
X_train_new = afr.fit_transform(X_train, y_train)
```

---

### 4. Polinom Özellik Üretimi (sklearn)

```python
from sklearn.preprocessing import PolynomialFeatures

# 2. derece polinom + etkileşimler
poly = PolynomialFeatures(
    degree=2,
    interaction_only=False,  # x², x*y, y² dahil
    include_bias=False
)

X_poly = poly.fit_transform(X[['yas', 'gelir', 'egitim_yil']])
feature_names = poly.get_feature_names_out(['yas', 'gelir', 'egitim_yil'])
# → yas, gelir, egitim_yil, yas^2, yas*gelir, yas*egitim_yil, ...

# Sadece etkileşim
poly_interact = PolynomialFeatures(degree=2, interaction_only=True)
```

---

### 5. Kategori Bazlı Agregasyon (Pandas)

```python
def auto_group_features(df, cat_cols, num_cols, target=None):
    """
    Kategorik sütunlara göre sayısal sütunların istatistiklerini türet
    """
    new_features = []

    for cat in cat_cols:
        for num in num_cols:
            grp = df.groupby(cat)[num].agg(['mean', 'std', 'median', 'max', 'min'])
            grp.columns = [f'{cat}_{num}_{s}' for s in grp.columns]
            df = df.merge(grp, on=cat, how='left')

            # Kişinin grup ortalamasından sapması
            df[f'{cat}_{num}_sapma'] = df[num] - df[f'{cat}_{num}_mean']
            df[f'{cat}_{num}_oran']  = df[num] / (df[f'{cat}_{num}_mean'] + 1e-9)

    return df

df = auto_group_features(df, ['sehir', 'meslek'], ['gelir', 'yas'])
```

---

### 6. Özellik Önemi ile Otomatik Eleme

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.feature_selection import SelectFromModel

def auto_feature_select(X_train, y_train, X_test, threshold='mean'):
    rf = RandomForestClassifier(n_estimators=100, random_state=42, n_jobs=-1)
    rf.fit(X_train, y_train)

    sel = SelectFromModel(rf, threshold=threshold, prefit=True)
    X_train_sel = sel.transform(X_train)
    X_test_sel  = sel.transform(X_test)

    selected_features = X_train.columns[sel.get_support()]
    print(f"Seçilen özellik: {len(selected_features)}/{X_train.shape[1]}")
    return X_train_sel, X_test_sel, selected_features
```

---

### AutoML Çerçevelerinde Feature Engineering

| Araç | FE Desteği | Notlar |
|---|---|---|
| **H2O AutoML** | ✅ Otomatik | Java tabanlı, hızlı |
| **Auto-sklearn** | ✅ Otomatik | sklearn uyumlu |
| **TPOT** | ✅ Genetik | Pipeline optimizasyonu |
| **Featuretools** | ✅ DFS | İlişkisel veri için |
| **tsfresh** | ✅ 700+ özellik | Zaman serisi |

```python
# Auto-sklearn örneği
import autosklearn.classification

automl = autosklearn.classification.AutoSklearnClassifier(
    time_left_for_this_task=120,
    per_run_time_limit=30
)
automl.fit(X_train, y_train)
print(automl.leaderboard())
```

---

### En İyi Uygulama

1. **Elle türet önce** → domain bilgisi en güçlü özellikler
2. **Otomatik araçla genişlet** → featuretools / polinom
3. **Özellik seçimi uygula** → boyutu yönet
4. **Leakage kontrol et** → özellikle aggregation'larda
5. **CV içinde pipeline** → her fold ayrı fit

---

## 💡 Bağlantılar
- [[FE - Özellik Türetme]]
- [[FE - Özellik Seçimi Yöntemleri]]
- [[FE - Data Leakage ve Pipeline Doğruluğu]]

## ❓ Sorular / Anlamadıklarım


## 🔗 Kaynaklar
- Featuretools Documentation (featuretools.com)
- tsfresh Documentation
- AutoFeat GitHub
