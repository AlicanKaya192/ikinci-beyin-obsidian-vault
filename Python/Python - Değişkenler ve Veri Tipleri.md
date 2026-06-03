---
tarih: 2026-05-28
konu: Python
etiket: ["python", "değişkenler", "veri-tipleri", "temel"]
kaynak: Python Resmi Dokümantasyon
zorluk: başlangıç
---

## 📌 Özet
Python'da değişkenler, veri saklamak için kullanılan isimlendirilmiş alanlardır. Python dinamik tipli bir dil olduğundan tür belirtmeye gerek yoktur; tip otomatik atanır.

## 🧠 Detay

### Değişken Tanımlama
```python
isim = "Ahmet"
yas = 25
pi = 3.14
aktif = True
```

### Temel Veri Tipleri

| Tip | Açıklama | Örnek |
|-----|----------|-------|
| `int` | Tam sayı | `42` |
| `float` | Ondalıklı sayı | `3.14` |
| `str` | Metin | `"merhaba"` |
| `bool` | Mantıksal | `True / False` |
| `NoneType` | Boş değer | `None` |

### Tip Öğrenme ve Dönüştürme
```python
x = 42
print(type(x))       # <class 'int'>

# Tip dönüştürme
sayi = int("10")     # str → int
metin = str(3.14)    # float → str
ondalik = float(5)   # int → float
```

### Çoklu Atama
```python
a, b, c = 1, 2, 3
x = y = z = 0        # hepsi 0
```

### Değişken İsimlendirme Kuralları
- Harf veya `_` ile başlamalı
- Boşluk içeremez
- Büyük/küçük harf duyarlıdır: `isim ≠ Isim`
- `snake_case` kullanımı önerilir: `kullanici_adi`

## 💡 Bağlantılar
- [[Python - Operatörler]]
- [[Python - String İşlemleri]]
- [[Python - Listeler]]

## ❓ Sorular / Anlamadıklarım
- `int` ile `float` arasında hangi durumlarda hangisini kullanmalıyım?
- `None` ile `False` arasındaki fark nedir?

## 🔗 Kaynaklar
- https://docs.python.org/tr/3/library/stdtypes.html
