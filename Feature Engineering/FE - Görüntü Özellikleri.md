---
tarih: 2026-05-28
konu: Feature Engineering
etiket: [feature-engineering, görüntü, HOG, CNN, OpenCV, transfer-learning]
kaynak: OpenCV / scikit-image Dokümantasyon
zorluk: ⭐⭐⭐
---

## 📌 Özet
Görüntülerden anlamlı sayısal özellikler çıkarmak, klasik bilgisayarla görme (HOG, LBP, renk histogramı) ile derin öğrenme (CNN embedding) yöntemleriyle yapılır. Transfer learning ile önceden eğitilmiş ağlardan özellik çıkarmak en güçlü yaklaşımdır.

---

## 🧠 Detay

### Temel Görüntü İstatistikleri

```python
import cv2
import numpy as np
from PIL import Image
import pandas as pd

def goruntu_istatistik(img_yolu):
    """Ham görüntü istatistikleri"""
    img = cv2.imread(img_yolu)
    img_gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
    img_rgb = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)

    ozellikler = {}

    # Boyut
    ozellikler['yukseklik'] = img.shape[0]
    ozellikler['genislik'] = img.shape[1]
    ozellikler['en_boy_orani'] = img.shape[1] / img.shape[0]
    ozellikler['toplam_piksel'] = img.shape[0] * img.shape[1]

    # Gri tonlama istatistikleri
    ozellikler['gray_mean'] = img_gray.mean()
    ozellikler['gray_std'] = img_gray.std()
    ozellikler['gray_min'] = img_gray.min()
    ozellikler['gray_max'] = img_gray.max()

    # RGB kanal istatistikleri
    for i, kanal in enumerate(['r', 'g', 'b']):
        ozellikler[f'{kanal}_mean'] = img_rgb[:,:,i].mean()
        ozellikler[f'{kanal}_std'] = img_rgb[:,:,i].std()

    # Parlaklık ve kontrast
    ozellikler['parlaklik'] = img_gray.mean()
    ozellikler['kontrast'] = img_gray.std()

    return ozellikler

df_stats = pd.DataFrame([goruntu_istatistik(p) for p in goruntu_yollari])
```

### Renk Histogramı

```python
def renk_histogrami(img_yolu, bins=32):
    """Her kanal için renk histogramı"""
    img = cv2.imread(img_yolu)
    img_rgb = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)

    ozellikler = []
    for kanal in range(3):
        hist = cv2.calcHist([img_rgb], [kanal], None,
                           [bins], [0, 256])
        hist = cv2.normalize(hist, hist).flatten()
        ozellikler.extend(hist)

    return np.array(ozellikler)
# → 3 * bins = 96 özellik

X_renk = np.array([renk_histogrami(p) for p in goruntu_yollari])
```

### HOG (Histogram of Oriented Gradients)

```python
from skimage.feature import hog
from skimage import io, color, transform

def hog_ozellik(img_yolu, boyut=(128, 128)):
    """HOG özellikleri — şekil ve kenar bilgisi"""
    img = io.imread(img_yolu)
    img_gray = color.rgb2gray(img) if len(img.shape) == 3 else img
    img_resized = transform.resize(img_gray, boyut)

    ozellikler, hog_img = hog(
        img_resized,
        orientations=9,           # Gradyan yön sayısı
        pixels_per_cell=(8, 8),   # Hücre boyutu
        cells_per_block=(2, 2),   # Blok başına hücre
        visualize=True,
        feature_vector=True
    )
    return ozellikler

X_hog = np.array([hog_ozellik(p) for p in goruntu_yollari])
print(f"HOG özellik boyutu: {X_hog.shape[1]}")
```

### LBP (Local Binary Patterns) — Doku Analizi

```python
from skimage.feature import local_binary_pattern

def lbp_ozellik(img_yolu, P=8, R=1, method='uniform'):
    """LBP — doku özellikleri"""
    img = io.imread(img_yolu, as_gray=True)
    img_resized = transform.resize(img, (128, 128))

    lbp = local_binary_pattern(img_resized, P, R, method)

    # Histogram
    n_bins = P + 2 if method == 'uniform' else 2**P
    hist, _ = np.histogram(lbp, bins=n_bins, range=(0, n_bins),
                           density=True)
    return hist

X_lbp = np.array([lbp_ozellik(p) for p in goruntu_yollari])
```

### CNN Embedding (Transfer Learning) ⭐

```python
import tensorflow as tf
from tensorflow.keras.applications import ResNet50, EfficientNetB0, VGG16
from tensorflow.keras.applications.resnet50 import preprocess_input
from tensorflow.keras.preprocessing.image import load_img, img_to_array

def cnn_ozellik_cikart(img_yollari, model_adi='resnet50',
                        boyut=(224, 224), batch_size=32):
    """Önceden eğitilmiş CNN ile özellik çıkarımı"""

    # Model seç (son sınıflandırma katmanı olmadan)
    if model_adi == 'resnet50':
        temel_model = ResNet50(weights='imagenet', include_top=False,
                               pooling='avg')
    elif model_adi == 'efficientnet':
        temel_model = EfficientNetB0(weights='imagenet', include_top=False,
                                      pooling='avg')

    def goruntu_yukle(yol):
        img = load_img(yol, target_size=boyut)
        arr = img_to_array(img)
        return preprocess_input(arr)

    # Batch işleme
    embeddingler = []
    for i in range(0, len(img_yollari), batch_size):
        batch_yollar = img_yollari[i:i+batch_size]
        batch = np.array([goruntu_yukle(p) for p in batch_yollar])
        batch_embed = temel_model.predict(batch, verbose=0)
        embeddingler.append(batch_embed)

    return np.vstack(embeddingler)

# ResNet50 → 2048 özellik
X_cnn = cnn_ozellik_cikart(goruntu_yollari, model_adi='resnet50')
print(f"CNN embedding boyutu: {X_cnn.shape}")  # (n, 2048)
```

### Boyut İndirgeme (Yüksek Boyutlu Embeddingler İçin)

```python
from sklearn.decomposition import PCA
from sklearn.preprocessing import StandardScaler
import umap

# PCA ile 2048 → 100 boyut
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X_cnn)

pca = PCA(n_components=100, random_state=42)
X_pca = pca.fit_transform(X_scaled)
print(f"Açıklanan varyans: {pca.explained_variance_ratio_.sum():.2%}")

# UMAP ile görselleştirme (2D)
reducer = umap.UMAP(n_components=2, random_state=42)
X_2d = reducer.fit_transform(X_scaled)
```

### Pratik ML Pipeline

```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.ensemble import GradientBoostingClassifier

# HOG + Geleneksel ML
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('model', GradientBoostingClassifier(n_estimators=200))
])
pipeline.fit(X_hog, y)

# CNN embedding + ML (derin öğrenmesiz yaklaşım)
pipeline_cnn = Pipeline([
    ('pca', PCA(n_components=200)),
    ('scaler', StandardScaler()),
    ('model', GradientBoostingClassifier())
])
pipeline_cnn.fit(X_cnn, y)
```

### Hangi Yöntem Ne Zaman?

| Yöntem | Kullanım | Boyut | Hız |
|--------|----------|-------|-----|
| Renk Histogramı | Renk bilgisi önemli | ~100 | ⚡ Hızlı |
| HOG | Şekil/kenar tespiti | ~1000-8000 | ⚡ Hızlı |
| LBP | Doku analizi | ~50 | ⚡ Hızlı |
| CNN (ResNet) | Genel görüntü | 2048 | 🐢 Yavaş |
| CNN (EfficientNet) | Yüksek doğruluk | 1280 | 🐢 Yavaş |

---

## 💡 Bağlantılar
- [[FE - Giriş ve Genel Bakış]]
- [[FE - Özellik Seçimi Yöntemleri]]
- [[ML - Sinir Ağları Temel]]

## ❓ Sorular / Anlamadıklarım
- ResNet vs EfficientNet transfer learning'de ne zaman hangisi?
- Fine-tuning ile feature extraction arasındaki fark?

## 🔗 Kaynaklar
- https://scikit-image.org/docs/stable/api/skimage.feature.html
- https://keras.io/api/applications/
