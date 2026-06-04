---
tarih: 2025-01-01
konu: Data Leakage, Pipeline Doğruluğu, Train-Test Sızıntısı
etiket: [feature-engineering, data-leakage, pipeline, train-test, sızıntı]
kaynak:
zorluk: ⭐⭐⭐
---

## 📌 Özet

Data Leakage: Modelin eğitim sırasında test verisinden veya hedef değişkenden bilgi sızması. Gerçek dünyada başarısız olan fakat lab ortamında harika görünen modellerin en sık sebebidir.

---

## 🧠 Detay

### Leakage Türleri

#### 1. Train-Test Leakage
Test seti bilgisinin eğitim setine sızması.

```python
# ❌ YANLIŞ — scaler tüm veriyi görüyor
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)  # X = train + test birleşik!
X_train_sc, X_test_sc = train_test_split(X_scaled)

# ✅ DOĞRU
X_train, X_test, y_train, y_test = train_test_split(X, y)
scaler = StandardScaler()
X_train_sc = scaler.fit_transform(X_train)
X_test_sc  = scaler.transform(X_test)  # sadece transform!
```

#### 2. Target Leakage
Hedef değişkeni dolaylı olarak içeren özellikler.

```python
# ❌ YANLIŞ — kredi onayı tahmininde ödeme durumu var!
features = ['yas', 'gelir', 'odeme_durumu', 'kredi_skoru']
# 'odeme_durumu' zaten tahmin sonrası oluşan bir bilgi!

# ✅ DOĞRU — sadece başvuru anındaki bilgiler
features = ['yas', 'gelir', 'meslek', 'egitim']
```

#### 3. Temporal Leakage (Zaman Sızıntısı)

```python
# ❌ YANLIŞ — gelecek bilgisi var
df['7_gun_sonraki_satis'] = df.groupby('urun')['satis'].shift(-7)
# Bu özellik tahmin edilecek değişkenin geleceği!

# ❌ YANLIŞ — zaman serisinde rastgele bölme
X_train, X_test = train_test_split(df)  # Gelecek geçmişte!

# ✅ DOĞRU — kronolojik bölme
cutoff = '2024-01-01'
train = df[df['tarih'] < cutoff]
test  = df[df['tarih'] >= cutoff]

# Lag özellikleri — geçmiş bilgisi kullan
df['satis_lag7'] = df['satis'].shift(7)  # 7 gün ÖNCESİ ✅
```

#### 4. Group Leakage
Aynı grubun hem train hem test'te olması.

```python
# ❌ YANLIŞ — aynı hasta train ve test'te
train_test_split(df, random_state=42)

# ✅ DOĞRU — hasta bazlı bölme
from sklearn.model_selection import GroupShuffleSplit

gss = GroupShuffleSplit(n_splits=1, test_size=0.2, random_state=42)
train_idx, test_idx = next(gss.split(X, y, groups=df['hasta_id']))
```

---

### Cross-Validation'da Leakage

```python
# ❌ YANLIŞ — CV dışında preprocessing
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)  # Tüm fold'lar birbirini görüyor
scores = cross_val_score(model, X_scaled, y, cv=5)

# ✅ DOĞRU — Pipeline ile
from sklearn.pipeline import Pipeline

pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('imputer', SimpleImputer()),
    ('model', LogisticRegression())
])
scores = cross_val_score(pipeline, X, y, cv=5)
# Her fold: train üzerinde fit, val üzerinde transform
```

---

### SMOTE ve Leakage

```python
# ❌ YANLIŞ — SMOTE sonra bölme
smote = SMOTE()
X_res, y_res = smote.fit_resample(X, y)
X_train, X_test = train_test_split(X_res, y_res)
# Sentezlenen test verisi train'den türedi!

# ✅ DOĞRU — imblearn Pipeline
from imblearn.pipeline import Pipeline as ImbPipeline

pipeline = ImbPipeline([
    ('smote', SMOTE()),
    ('model', RandomForestClassifier())
])
# SMOTE sadece train fold'una uygulanır
```

---

### Target Encoding ve Leakage

```python
# ❌ YANLIŞ — tüm veri üzerinde target encoding
enc = TargetEncoder()
df['sehir_enc'] = enc.fit_transform(df[['sehir']], df['hedef'])

# ✅ DOĞRU 1 — sadece train
enc.fit(X_train[['sehir']], y_train)
X_train['sehir_enc'] = enc.transform(X_train[['sehir']])
X_test['sehir_enc']  = enc.transform(X_test[['sehir']])

# ✅ DOĞRU 2 — K-Fold target encoding (CV içinde)
enc = ce.TargetEncoder(cols=['sehir'])
pipeline = Pipeline([('enc', enc), ('model', model)])
cross_val_score(pipeline, X_train, y_train, cv=5)
```

---

### Leakage Tespiti

```python
# 1. Aşırı yüksek CV skoru → şüphe
# Gerçekçi olmayan ROC-AUC > 0.99 → leakage kontrolü yap

# 2. Özellik önemi → hedefle neredeyse mükemmel korelasyon
corr = df.corr()['hedef'].abs().sort_values(ascending=False)
print(corr.head(10))

# 3. Train-test arasında büyük performans farkı
train_score = model.score(X_train, y_train)
test_score  = model.score(X_test, y_test)
print(f"Train: {train_score:.4f}, Test: {test_score:.4f}")
# Fark > 0.10 → overfitting veya leakage

# 4. Zaman serisinde gelecekten özellik kontrol
# Sütun isimleri: 'sonraki_', 'gelecek_', 'next_' → bayrak!
```

---

### Doğru Pipeline Şablonu

```python
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.impute import SimpleImputer
from sklearn.model_selection import cross_val_score

num_cols = ['yas', 'gelir']
cat_cols = ['sehir', 'meslek']

num_pipeline = Pipeline([
    ('imputer', SimpleImputer(strategy='median')),
    ('scaler', StandardScaler())
])

cat_pipeline = Pipeline([
    ('imputer', SimpleImputer(strategy='most_frequent')),
    ('ohe', OneHotEncoder(handle_unknown='ignore', sparse_output=False))
])

preprocessor = ColumnTransformer([
    ('num', num_pipeline, num_cols),
    ('cat', cat_pipeline, cat_cols)
])

full_pipeline = Pipeline([
    ('preprocessor', preprocessor),
    ('model', RandomForestClassifier(random_state=42))
])

# CV — her fold ayrı fit
cv_scores = cross_val_score(full_pipeline, X, y, cv=5, scoring='roc_auc')
print(f"CV ROC-AUC: {cv_scores.mean():.4f} ± {cv_scores.std():.4f}")

# Final model — train üzerinde fit, test'te değerlendir
full_pipeline.fit(X_train, y_train)
test_score = full_pipeline.score(X_test, y_test)
```

---

## 💡 Bağlantılar
- [[FE - Giriş ve Genel Bakış]]
- [[FE - Encoding Yöntemleri]]
- [[FE - Dengesiz Veri Seti İşleme]]
- [[FE - Feature Store ve MLOps]]
- [[ML - Veri Ön İşleme Pipeline]]
- [[ML - Overfitting ve Underfitting]]

## ❓ Sorular / Anlamadıklarım


## 🔗 Kaynaklar
- Kaggle: Data Leakage (free course)
- sklearn Pipeline Documentation
