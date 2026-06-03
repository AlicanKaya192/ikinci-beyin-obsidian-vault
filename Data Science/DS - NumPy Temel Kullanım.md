---
tarih: 2026-05-28
konu: Data Science
etiket: ["data-science", "numpy", "array", "temel"]
kaynak: NumPy Resmi Dokümantasyon
zorluk: başlangıç
---

## 📌 Özet
NumPy, Python'da sayısal hesaplama için temel kütüphanedir. Çok boyutlu diziler (ndarray) ve matematiksel işlemler sunar. Pandas, Scikit-learn gibi kütüphanelerin temeli NumPy'dır.

## 🧠 Detay

### Kurulum ve Import
```python
pip install numpy
import numpy as np
```

### Array Oluşturma
```python
# Listeden array
a = np.array([1, 2, 3, 4, 5])
print(a)           # [1 2 3 4 5]
print(a.dtype)     # int64
print(a.shape)     # (5,)

# 2D array
b = np.array([[1, 2, 3],
              [4, 5, 6]])
print(b.shape)     # (2, 3)

# Hazır arrayler
np.zeros((3, 3))       # sıfırlar
np.ones((2, 4))        # birler
np.eye(3)              # birim matris
np.arange(0, 10, 2)    # [0,2,4,6,8]
np.linspace(0, 1, 5)   # [0, 0.25, 0.5, 0.75, 1]
```

### Temel Özellikler
```python
a = np.array([[1,2,3],[4,5,6]])

print(a.shape)    # (2, 3)
print(a.ndim)     # 2
print(a.size)     # 6
print(a.dtype)    # int64
```

### Indexleme ve Dilimleme
```python
a = np.array([10, 20, 30, 40, 50])
print(a[0])       # 10
print(a[-1])      # 50
print(a[1:4])     # [20 30 40]
print(a[::2])     # [10 30 50]

# 2D
b = np.array([[1,2,3],[4,5,6]])
print(b[0, 1])    # 2
print(b[:, 1])    # [2, 5] → 2. sütun
print(b[1, :])    # [4, 5, 6] → 2. satır
```

## 💡 Bağlantılar
- [[DS - NumPy Array İşlemleri]]
- [[DS - NumPy Matematiksel İşlemler]]
- [[DS - Pandas Temel Kullanım]]

## ❓ Sorular / Anlamadıklarım
- Python listesi ile NumPy array arasındaki hız farkı ne kadar?
- `dtype` neden önemli, ne zaman değiştirmeliyim?

## 🔗 Kaynaklar
- https://numpy.org/doc/stable/user/quickstart.html
