---
tarih: 2025-01-01
konu: Özellik Seçimi, Filter, Wrapper, Embedded, RFE, SHAP
etiket: [feature-engineering, özellik-seçimi, feature-selection, RFE, SHAP, filter]
kaynak:
zorluk: ⭐⭐⭐
---

## 📌 Özet

Tüm özellikler faydalı değildir. Gereksiz özellikler modeli yavaşlatır, overfitting yaratır ve yorumlamayı zorlaştırır. Özellik seçimi en bilgilendirici alt kümeyi bulmaktır.

---

## 🧠 Detay

### Özellik Seçim Kategorileri

```
Özellik Seçimi
├── Filter Yöntemleri (Model bağımsız, hızlı)
│   ├── Korelasyon
│   ├── Chi-kare
│   ├── ANOVA F-testi
│   └── Mutual Information
├── Wrapper Yöntemleri (Model tabanlı, yavaş)
│   ├── RFE (Backward)
│   ├── Forward Selection
│   └── Exhaustive Search
└── Embedded Yöntemler (Modele gömülü)
    ├── Lasso (L1)
    ├── Tree Feature Importance
    └── SHAP Values
```

---

### 1. Filter Yöntemleri

#### Varyans Eşiği

```python
from sklearn.feature_selection import VarianceThreshold

# Varyansı 0 olan (sabit) özellikleri kaldır
sel = VarianceThreshold(threshold=0.0)
X_reduced = sel.fit_transform(X)

# Eşiği artır
sel = VarianceThreshold(threshold=0.01)
```

#### Korelasyon Tabanlı

```python
import pandas as pd
import numpy as np

# Hedefle korelasyon (sayısal hedef)
corr_with_target = df.corr()['hedef'].abs().sort_values(ascending=False)

# Özellikler arası yüksek korelasyon → birini çıkar
def drop_correlated(df, threshold=0.90):
    corr_matrix = df.corr().abs()
    upper = corr_matrix.where(
        np.triu(np.ones(corr_matrix.shape), k=1).astype(bool)
    )
    to_drop = [col for col in upper.columns if any(upper[col] > threshold)]
    return df.drop(columns=to_drop)

df_reduced = drop_correlated(df, threshold=0.90)
```

#### İstatistiksel Testler

```python
from sklearn.feature_selection import (
    SelectKBest, chi2, f_classif, f_regression, mutual_info_classif
)

# Sınıflandırma için — kategorik özellik
sel_chi2 = SelectKBest(chi2, k=10)
X_chi2 = sel_chi2.fit_transform(X_cat, y)

# Sınıflandırma için — sayısal özellik
sel_f = SelectKBest(f_classif, k=10)
X_f = sel_f.fit_transform(X_num, y)

# Mutual Information (doğrusal olmayan ilişkileri de yakalar)
sel_mi = SelectKBest(mutual_info_classif, k=10)
X_mi = sel_mi.fit_transform(X, y)

# Regresyon için
sel_freg = SelectKBest(f_regression, k=10)

# Seçilen özellik isimleri
selected = sel_f.get_support(indices=True)
selected_features = X.columns[selected]
```

---

### 2. Wrapper Yöntemleri

#### RFE (Recursive Feature Elimination)

```python
from sklearn.feature_selection import RFE, RFECV
from sklearn.ensemble import RandomForestClassifier

# Belirtilen sayıda özellik seç
rfe = RFE(estimator=RandomForestClassifier(n_estimators=100),
          n_features_to_select=10, step=1)
rfe.fit(X_train, y_train)

selected_features = X.columns[rfe.support_]
feature_ranking = pd.Series(rfe.ranking_, index=X.columns).sort_values()

# CV ile otomatik k seçimi
rfecv = RFECV(estimator=RandomForestClassifier(), cv=5, scoring='accuracy')
rfecv.fit(X_train, y_train)
optimal_k = rfecv.n_features_
```

#### SequentialFeatureSelector

```python
from sklearn.feature_selection import SequentialFeatureSelector

# Forward selection
sfs = SequentialFeatureSelector(
    RandomForestClassifier(n_estimators=50),
    n_features_to_select=10,
    direction='forward',
    cv=3
)
sfs.fit(X_train, y_train)
selected = X.columns[sfs.get_support()]
```

---

### 3. Embedded Yöntemler

#### Lasso (L1 Regularization)

```python
from sklearn.linear_model import Lasso, LassoCV
from sklearn.feature_selection import SelectFromModel

# Optimal alpha bul
lasso_cv = LassoCV(cv=5, random_state=42)
lasso_cv.fit(X_train, y_train)
print(f"Optimal alpha: {lasso_cv.alpha_:.4f}")

# Sıfır olmayan katsayılar
sel = SelectFromModel(lasso_cv, prefit=True)
X_reduced = sel.transform(X)
selected = X.columns[sel.get_support()]
```

#### Tree Feature Importance

```python
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
import matplotlib.pyplot as plt

rf = RandomForestClassifier(n_estimators=200, random_state=42)
rf.fit(X_train, y_train)

importances = pd.Series(rf.feature_importances_, index=X.columns)
importances.sort_values(ascending=False).head(20).plot(kind='bar')
plt.title('Özellik Önemleri')

# Eşiğe göre seç
sel = SelectFromModel(rf, threshold='mean', prefit=True)
X_reduced = sel.transform(X)
```

#### SHAP Değerleri (En Güvenilir)

```python
import shap

# Herhangi bir model
model = RandomForestClassifier().fit(X_train, y_train)
explainer = shap.TreeExplainer(model)
shap_values = explainer.shap_values(X_test)

# Özellik önemleri (ortalama |SHAP|)
shap_importance = pd.DataFrame({
    'feature': X.columns,
    'importance': np.abs(shap_values[1]).mean(axis=0)
}).sort_values('importance', ascending=False)

# Görsel
shap.summary_plot(shap_values[1], X_test, plot_type='bar')
shap.summary_plot(shap_values[1], X_test)  # Beeswarm
```

---

### Permutation Importance

```python
from sklearn.inspection import permutation_importance

result = permutation_importance(
    model, X_test, y_test,
    n_repeats=10, random_state=42
)

perm_imp = pd.Series(
    result.importances_mean,
    index=X.columns
).sort_values(ascending=False)
```

Model-agnostik, güvenilir ama yavaş.

---

### Özellik Seçimi Stratejisi

```python
def feature_selection_pipeline(X_train, y_train, X_test):
    # 1. Sabit özellikleri çıkar
    var_sel = VarianceThreshold(threshold=0.01)
    X_train = var_sel.fit_transform(X_train)
    X_test  = var_sel.transform(X_test)

    # 2. Yüksek korelasyonlu çıkar
    X_train = drop_correlated(pd.DataFrame(X_train), threshold=0.95)

    # 3. MI ile top-K seç
    sel_mi = SelectKBest(mutual_info_classif, k=50)
    X_train = sel_mi.fit_transform(X_train, y_train)
    X_test  = sel_mi.transform(X_test)

    # 4. RFECV ile final seçim
    rfecv = RFECV(RandomForestClassifier(), cv=5, scoring='roc_auc')
    X_train = rfecv.fit_transform(X_train, y_train)
    X_test  = rfecv.transform(X_test)

    return X_train, X_test
```

---

### Hangi Yöntemi Ne Zaman?

| Durum | Yöntem |
|---|---|
| Hızlı ön eleme | Varyans eşiği + Korelasyon |
| Kategorik özellik, sınıflandırma | Chi-kare |
| Doğrusal ilişki | ANOVA F-testi |
| Doğrusal olmayan ilişki | Mutual Information |
| Az özellik, güçlü model | RFE / RFECV |
| Doğrusal model | Lasso |
| Ağaç modeli | Feature Importance / SHAP |
| Model yorumlanabilirliği | SHAP (her zaman iyi) |

---

## 💡 Bağlantılar
- [[FE - Giriş ve Genel Bakış]]
- [[FE - Özellik Türetme]]
- [[ML - Özellik Seçimi]]
- [[ML - Overfitting ve Underfitting]]

## ❓ Sorular / Anlamadıklarım


## 🔗 Kaynaklar
- sklearn.feature_selection Documentation
- SHAP Documentation (shap.readthedocs.io)
