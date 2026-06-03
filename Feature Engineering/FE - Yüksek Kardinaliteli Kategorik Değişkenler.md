---
tarih: 2025-01-01
konu: Yüksek Kardinalite, Rare Encoding, Embedding, Gruplandırma
etiket: [feature-engineering, yüksek-kardinalite, rare-encoding, embedding, gruplandırma]
kaynak:
zorluk: ⭐⭐⭐
---

## 📌 Özet

Yüzlerce veya binlerce kategorisi olan değişkenler (ürün kodu, şehir, kullanıcı ID) için özel teknikler gerekir. One-hot boyut patlamasına, Label encoding anlamsız sıralamalara yol açar.

---

## 🧠 Detay

### Kardinalite Tespiti

```python
for col in df.select_dtypes('object').columns:
    n = df[col].nunique()
    print(f"{col}: {n} benzersiz değer")

# Yüksek kardinalite eşiği: genellikle > 10-20
high_cardinality = [col for col in cat_cols if df[col].nunique() > 15]
```

---

### 1. Nadir Kategorileri Birleştirme (Rare Encoding)

```python
def rare_encoder(df, col, threshold=0.01):
    """
    Belirli eşiğin altındaki kategorileri 'Rare' olarak birleştir
    """
    freq = df[col].value_counts(normalize=True)
    rare_cats = freq[freq < threshold].index
    df[col] = df[col].apply(lambda x: 'Rare' if x in rare_cats else x)
    return df

df = rare_encoder(df, 'sehir', threshold=0.01)
```

---

### 2. Frekans / Count Encoding

```python
freq_map = df['sehir'].value_counts(normalize=True)
df['sehir_freq'] = df['sehir'].map(freq_map)

# Train'den map, test'e uygula
freq_map = X_train['sehir'].value_counts(normalize=True)
X_train['sehir_freq'] = X_train['sehir'].map(freq_map)
X_test['sehir_freq']  = X_test['sehir'].map(freq_map).fillna(0)
```

---

### 3. Target Encoding + Smoothing

```python
import category_encoders as ce

# Smoothing: küçük gruplarda genel ortalamaya çeker
enc = ce.TargetEncoder(
    cols=['sehir'],
    smoothing=10,       # Büyükse daha çok smoothing
    handle_unknown='value',
    handle_missing='value'
)
enc.fit(X_train, y_train)

X_train_enc = enc.transform(X_train)
X_test_enc  = enc.transform(X_test)
```

**K-Fold Target Encoding** (data leakage'a karşı):

```python
from sklearn.model_selection import KFold

def kfold_target_encode(df, col, target, n_splits=5):
    df[f'{col}_te'] = np.nan
    kf = KFold(n_splits=n_splits, shuffle=True, random_state=42)
    global_mean = df[target].mean()

    for tr_idx, val_idx in kf.split(df):
        tr = df.iloc[tr_idx]
        mapping = tr.groupby(col)[target].mean()
        df.loc[df.index[val_idx], f'{col}_te'] = (
            df.iloc[val_idx][col].map(mapping).fillna(global_mean)
        )
    return df

df = kfold_target_encode(df, 'sehir', 'hedef')
```

---

### 4. Gruplama / Bölme

```python
# Anlamlı gruplara indir
sehir_bolge = {
    'İstanbul': 'Marmara', 'Bursa': 'Marmara', 'Kocaeli': 'Marmara',
    'Ankara': 'İç Anadolu', 'Konya': 'İç Anadolu',
    'İzmir': 'Ege', 'Manisa': 'Ege',
    # ...
}
df['bolge'] = df['sehir'].map(sehir_bolge).fillna('Diğer')

# İlk karaktere göre gruplama (kısa kod vb.)
df['kategori_ana'] = df['kategori_kodu'].str[:2]

# Regex ile gruplama
df['urun_ailesi'] = df['urun_kodu'].str.extract(r'^([A-Z]{2})')
```

---

### 5. Varlık Gömmesi (Entity Embedding) — Sinir Ağı

Kategorileri düşük boyutlu yoğun vektöre dönüştürür.

```python
import tensorflow as tf
from tensorflow.keras.layers import Embedding, Flatten, Dense
from tensorflow.keras.models import Model

n_categories = df['sehir'].nunique()
embedding_dim = min(50, (n_categories + 1) // 2)  # Kural: min(50, (n+1)//2)

# Model içinde embedding
input_sehir = tf.keras.Input(shape=(1,))
emb_sehir = Embedding(n_categories, embedding_dim)(input_sehir)
flat_sehir = Flatten()(emb_sehir)
# ...

# Gömme vektörlerini çıkart
embedding_weights = model.get_layer('embedding').get_weights()[0]
# embedding_weights: (n_categories, embedding_dim)
```

**FastAI kütüphanesiyle daha kolay** tabular embedding.

---

### 6. Hashing Trick

```python
import category_encoders as ce
from sklearn.feature_extraction import FeatureHasher

# category_encoders
enc = ce.HashingEncoder(cols=['sehir'], n_components=16)
X_hashed = enc.fit_transform(X)

# sklearn FeatureHasher
hasher = FeatureHasher(n_features=16, input_type='string')
X_hashed = hasher.transform(df['sehir'].values.reshape(-1, 1))
```

---

### 7. CatBoost Encoding

```python
enc = ce.CatBoostEncoder(cols=['sehir'])
enc.fit(X_train, y_train)
X_train_enc = enc.transform(X_train)
```

---

### 8. Aggregate + Merge (Çok Tablolu Veri)

```python
# Müşteri bazlı özet istatistikler
musteri_agg = df.groupby('musteri_id').agg(
    toplam_siparis=('siparis_id', 'count'),
    toplam_harcama=('tutar', 'sum'),
    ort_harcama=('tutar', 'mean'),
    max_harcama=('tutar', 'max'),
    son_siparis_gun=('tarih', lambda x: (pd.Timestamp.now() - x.max()).days)
).reset_index()

df = df.merge(musteri_agg, on='musteri_id', how='left')
```

---

### Özet: Kardinaliteye Göre Strateji

| Kardinalite | Strateji |
|---|---|
| < 5 | One-Hot |
| 5-15 | One-Hot veya Target Enc. |
| 15-100 | Target/Frequency/Binary |
| 100-10K | Rare + Target Enc. + Smoothing |
| > 10K | Hashing, Embedding, Agg Features |
| ID tipi | Asla doğrudan kullanma → Agg features |

---

## 💡 Bağlantılar
- [[FE - Encoding Yöntemleri]]
- [[FE - Özellik Türetme]]
- [[ML - Veri Ön İşleme Pipeline]]

## ❓ Sorular / Anlamadıklarım


## 🔗 Kaynaklar
- category_encoders Documentation
- Guo & Berkhahn - Entity Embeddings of Categorical Variables (paper)
