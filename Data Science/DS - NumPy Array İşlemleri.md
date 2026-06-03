---
tarih: 2026-05-28
konu: Data Science
etiket: ["data-science", "numpy", "array", "şekillendirme"]
kaynak: NumPy Resmi Dokümantasyon
zorluk: orta
---

## 📌 Özet
NumPy array'leri yeniden şekillendirme, birleştirme, filtreleme ve vektörel işlemler NumPy'ın en güçlü yönleridir.

## 🧠 Detay

### Şekillendirme
```python
import numpy as np

a = np.arange(12)        # [0..11]
b = a.reshape(3, 4)      # 3x4 matris
c = b.reshape(2, 6)
d = b.flatten()          # 1D'ye düzleştir
e = b.T                  # transpoze
```

### Birleştirme ve Bölme
```python
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

np.concatenate([a, b])        # [1,2,3,4,5,6]
np.vstack([a, b])             # dikey birleştir
np.hstack([a, b])             # yatay birleştir

# Bölme
np.split(a, 3)                # 3 parçaya böl
```

### Boolean Maskeleme
```python
a = np.array([10, 25, 3, 47, 8, 31])

maske = a > 20
print(maske)        # [F, T, F, T, F, T]
print(a[maske])     # [25 47 31]

# Kısa yol
print(a[a > 20])    # [25 47 31]
print(a[(a > 10) & (a < 40)])  # [25 31]
```

### Fancy Indexing
```python
a = np.array([10, 20, 30, 40, 50])
indeksler = [0, 2, 4]
print(a[indeksler])   # [10 30 50]
```

### Broadcasting
```python
# Farklı boyuttaki arraylerle işlem
a = np.array([[1, 2, 3],
              [4, 5, 6]])
b = np.array([10, 20, 30])

print(a + b)
# [[11, 22, 33],
#  [14, 25, 36]]
```

### Kopyalama
```python
a = np.array([1, 2, 3])
b = a        # referans! a değişirse b de değişir
c = a.copy() # gerçek kopya
```

## 💡 Bağlantılar
- [[DS - NumPy Temel Kullanım]]
- [[DS - NumPy Matematiksel İşlemler]]

## ❓ Sorular / Anlamadıklarım
- Broadcasting kuralları tam olarak nasıl çalışır?
- `reshape(-1, 1)` neden sık kullanılır?

## 🔗 Kaynaklar
- https://numpy.org/doc/stable/user/basics.broadcasting.html
