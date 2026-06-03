---
tarih: 2025-01-01
konu: Dengesiz Veri Seti, SMOTE, Oversampling, Undersampling, Class Weight
etiket: [feature-engineering, dengesiz-veri, imbalanced, SMOTE, oversampling, class-weight]
kaynak:
zorluk: ⭐⭐⭐
---

## 📌 Özet

Sınıflar arasında büyük fark varken (örn. %99 normal, %1 dolandırıcılık) model çoğunluk sınıfını tahmin ederek yüksek accuracy elde eder ama azınlık sınıfını görmez. Dengeleme gerekir.

---

## 🧠 Detay

### Sorunun Tespiti

```python
# Sınıf dağılımı
print(df['hedef'].value_counts())
print(df['hedef'].value_counts(normalize=True) * 100)

# Görsel
import matplotlib.pyplot as plt
df['hedef'].value_counts().plot(kind='bar')
```

**Dengesizlik oranı**: %5'in altı → ciddi sorun.

### Yanlış Metrik: Accuracy

%1 pozitif sınıfta hep 0 tahmin edersen → **%99 accuracy ama model işe yaramaz!**

**Doğru metrikler**:
- Precision, Recall, F1-Score
- ROC-AUC, PR-AUC
- Balanced Accuracy

---

### 1. Class Weight Ayarı (En Basit)

```python
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import RandomForestClassifier
from sklearn.utils.class_weight import compute_class_weight
import numpy as np

# Otomatik ağırlık
model = LogisticRegression(class_weight='balanced')
model = RandomForestClassifier(class_weight='balanced')

# Manuel ağırlık
weights = compute_class_weight('balanced',
                                classes=np.unique(y),
                                y=y)
class_weight = dict(zip(np.unique(y), weights))
model = LogisticRegression(class_weight=class_weight)

# XGBoost için
scale_pos_weight = sum(y == 0) / sum(y == 1)
import xgboost as xgb
model = xgb.XGBClassifier(scale_pos_weight=scale_pos_weight)
```

---

### 2. Oversampling

#### Random Oversampling

```python
from imblearn.over_sampling import RandomOverSampler

ros = RandomOverSampler(random_state=42)
X_res, y_res = ros.fit_resample(X_train, y_train)
```

Azınlık sınıfı rastgele kopyalar → overfitting riski.

#### SMOTE (Synthetic Minority Over-sampling Technique) ⭐

```python
from imblearn.over_sampling import SMOTE, ADASYN, BorderlineSMOTE

# Standart SMOTE
smote = SMOTE(sampling_strategy=0.5,  # azınlık/çoğunluk oranı
              k_neighbors=5,
              random_state=42)
X_res, y_res = smote.fit_resample(X_train, y_train)

# Sınır bölgesine odaklanan
bsmote = BorderlineSMOTE(random_state=42)
X_res, y_res = bsmote.fit_resample(X_train, y_train)

# Uyarlanabilir sentez
adasyn = ADASYN(random_state=42)
X_res, y_res = adasyn.fit_resample(X_train, y_train)
```

**SMOTE nasıl çalışır?**
1. Azınlık sınıfından örnek al
2. K en yakın komşusunu bul
3. Komşular arasında interpolasyon ile yeni örnek üret

#### SMOTE + Kategorik: SMOTENC

```python
from imblearn.over_sampling import SMOTENC

# categorical_features: kategorik sütunların indeksleri
smotenc = SMOTENC(categorical_features=[0, 2, 5], random_state=42)
X_res, y_res = smotenc.fit_resample(X_train, y_train)
```

---

### 3. Undersampling

#### Random Undersampling

```python
from imblearn.under_sampling import RandomUnderSampler

rus = RandomUnderSampler(sampling_strategy=0.5, random_state=42)
X_res, y_res = rus.fit_resample(X_train, y_train)
```

Çoğunluk sınıfından örnekleri rastgele siler → bilgi kaybı.

#### Tomek Links

```python
from imblearn.under_sampling import TomekLinks

tl = TomekLinks()
X_res, y_res = tl.fit_resample(X_train, y_train)
```

Sınıf sınırındaki çoğunluk örneklerini siler → temizleme.

#### Edited Nearest Neighbours (ENN)

```python
from imblearn.under_sampling import EditedNearestNeighbours

enn = EditedNearestNeighbours()
X_res, y_res = enn.fit_resample(X_train, y_train)
```

---

### 4. Kombinasyon

```python
from imblearn.combine import SMOTETomek, SMOTEENN

# SMOTE + Tomek Links temizleme
pipeline = SMOTETomek(random_state=42)
X_res, y_res = pipeline.fit_resample(X_train, y_train)

# SMOTE + ENN temizleme
pipeline = SMOTEENN(random_state=42)
X_res, y_res = pipeline.fit_resample(X_train, y_train)
```

---

### 5. Pipeline ile Dengeleme

```python
from imblearn.pipeline import Pipeline as ImbPipeline
from sklearn.preprocessing import StandardScaler
from sklearn.ensemble import RandomForestClassifier

pipeline = ImbPipeline([
    ('scaler', StandardScaler()),
    ('smote', SMOTE(random_state=42)),  # Sadece train'e uygulanır
    ('model', RandomForestClassifier(random_state=42))
])

# CV ile
from sklearn.model_selection import StratifiedKFold, cross_val_score
cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
scores = cross_val_score(pipeline, X_train, y_train,
                         cv=cv, scoring='f1')
```

**⚠️ ÖNEMLİ**: SMOTE **sadece training set'e** uygulanmalı, test'e değil!

---

### 6. Eşik (Threshold) Ayarı

```python
from sklearn.metrics import precision_recall_curve
import matplotlib.pyplot as plt

probs = model.predict_proba(X_test)[:, 1]
precision, recall, thresholds = precision_recall_curve(y_test, probs)

# F1 maksimum eşik
f1_scores = 2 * precision * recall / (precision + recall + 1e-10)
optimal_threshold = thresholds[np.argmax(f1_scores)]

y_pred_opt = (probs >= optimal_threshold).astype(int)
```

---

### Strateji Özeti

| Durum | Yöntem |
|---|---|
| Hızlı çözüm | class_weight='balanced' |
| Az veri | SMOTE |
| Çok veri | Undersampling |
| Veri + temizlik | SMOTETomek |
| Tree modeli | scale_pos_weight / class_weight |
| İnce ayar | Threshold optimizasyonu |

---

### Değerlendirme Metrikleri

```python
from sklearn.metrics import (
    classification_report, confusion_matrix,
    roc_auc_score, average_precision_score,
    balanced_accuracy_score
)

print(classification_report(y_test, y_pred, digits=4))
print(f"ROC-AUC:          {roc_auc_score(y_test, probs):.4f}")
print(f"PR-AUC:           {average_precision_score(y_test, probs):.4f}")
print(f"Balanced Accuracy:{balanced_accuracy_score(y_test, y_pred):.4f}")
```

---

## 💡 Bağlantılar
- [[FE - Giriş ve Genel Bakış]]
- [[ML - Dengesiz Veri Seti Yönetimi]]
- [[ML - Model Değerlendirme Metrikleri]]

## ❓ Sorular / Anlamadıklarım


## 🔗 Kaynaklar
- imbalanced-learn Documentation (imbalanced-learn.org)
- SMOTE Paper (Chawla et al., 2002)
