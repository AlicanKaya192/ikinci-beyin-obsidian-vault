---
tarih: 2026-05-28
konu: Machine Learning
etiket: ["ml", "kmeans", "kümeleme", "gözetimsiz"]
kaynak: Scikit-learn Dokümantasyon
zorluk: orta
---

## 📌 Özet
K-Means, veriyi K kümeye bölen en popüler gözetimsiz öğrenme algoritmasıdır. Her küme, üyelerinin ortalama noktası (centroid) ile temsil edilir.

## 🧠 Detay

### Temel Kullanım
```python
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler
import matplotlib.pyplot as plt

# Ölçekleme önemli
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

kmeans = KMeans(
    n_clusters=3,
    init="k-means++",    # akıllı başlangıç
    n_init=10,           # farklı başlangıçları dene
    max_iter=300,
    random_state=42
)

kmeans.fit(X_scaled)
etiketler = kmeans.labels_
merkezler = kmeans.cluster_centers_
```

### Optimal K Seçimi — Elbow Yöntemi
```python
inertia_listesi = []
K_araligi = range(1, 11)

for k in K_araligi:
    km = KMeans(n_clusters=k, random_state=42, n_init=10)
    km.fit(X_scaled)
    inertia_listesi.append(km.inertia_)

plt.plot(K_araligi, inertia_listesi, "bo-")
plt.xlabel("K (Küme sayısı)")
plt.ylabel("Inertia")
plt.title("Elbow Yöntemi")
# Dirsek noktası → optimal K
```

### Silhouette Skoru
```python
from sklearn.metrics import silhouette_score

skorlar = []
for k in range(2, 11):
    km = KMeans(n_clusters=k, random_state=42, n_init=10)
    etiket = km.fit_predict(X_scaled)
    skor = silhouette_score(X_scaled, etiket)
    skorlar.append(skor)
    print(f"K={k}: Silhouette={skor:.3f}")

# En yüksek skor → optimal K
```

### Kümeleri Görselleştirme
```python
# 2D için
plt.scatter(X_scaled[:, 0], X_scaled[:, 1],
    c=etiketler, cmap="tab10", alpha=0.7)
plt.scatter(merkezler[:, 0], merkezler[:, 1],
    c="red", marker="X", s=200, label="Merkezler")
plt.legend()
plt.title("K-Means Kümeleme")
```

### Küme Analizi
```python
import pandas as pd

df["kume"] = etiketler
# Küme özeti
df.groupby("kume").mean()
df.groupby("kume").size()
```

## 💡 Bağlantılar
- [[ML - Makine Öğrenmesine Giriş]]
- [[ML - PCA - Boyut İndirgeme]]
- [[DS - Veri Dönüşümleri]]

## ❓ Sorular / Anlamadıklarım
- K-Means küresel olmayan kümeleri bulabilir mi?
- DBSCAN ile K-Means'in farkı nedir?

## 🔗 Kaynaklar
- https://scikit-learn.org/stable/modules/clustering.html
