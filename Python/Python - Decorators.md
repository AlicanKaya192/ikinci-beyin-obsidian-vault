---
tarih: 2026-05-28
konu: Python
etiket: ["python", "decorator", "fonksiyonel", "ileri"]
kaynak: Python Resmi Dokümantasyon
zorluk: ileri
---

## 📌 Özet
Decorator'lar, bir fonksiyonun davranışını değiştirmeden ona ek özellik kazandıran fonksiyonlardır. `@` sembolü ile kullanılır. Loglama, zamanlama, yetki kontrolü gibi durumlarda idealdir.

## 🧠 Detay

### Temel Decorator
```python
def log_dekorator(fonksiyon):
    def sarici(*args, **kwargs):
        print(f"{fonksiyon.__name__} çalıştı")
        sonuc = fonksiyon(*args, **kwargs)
        print(f"{fonksiyon.__name__} bitti")
        return sonuc
    return sarici

@log_dekorator
def topla(a, b):
    return a + b

topla(3, 5)
# topla çalıştı
# topla bitti
```

### functools.wraps
```python
import functools

def log(func):
    @functools.wraps(func)    # metadata korur
    def wrapper(*args, **kwargs):
        print(f"Çağrıldı: {func.__name__}")
        return func(*args, **kwargs)
    return wrapper
```

### Parametreli Decorator
```python
def tekrarla(n):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(n):
                func(*args, **kwargs)
        return wrapper
    return decorator

@tekrarla(3)
def merhaba():
    print("Merhaba!")

merhaba()  # 3 kez yazdırır
```

### Zamanlama Decorator'ı
```python
import time

def zamanlayici(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        baslangic = time.time()
        sonuc = func(*args, **kwargs)
        sure = time.time() - baslangic
        print(f"{func.__name__}: {sure:.4f} saniye")
        return sonuc
    return wrapper

@zamanlayici
def uzun_islem():
    time.sleep(1)
```

### Sınıf Decorator'ları
```python
# @property, @classmethod, @staticmethod
# bunlar da birer decorator'dır!
```

## 💡 Bağlantılar
- [[Python - Fonksiyonlar]]
- [[Python - OOP - Kapsülleme]]
- [[Python - Lambda ve Fonksiyonel Programlama]]

## ❓ Sorular / Anlamadıklarım
- `functools.wraps` olmadan ne kaybolur?
- Birden fazla decorator üst üste kullanılırsa sıra nasıl işler?

## 🔗 Kaynaklar
- https://docs.python.org/tr/3/glossary.html#term-decorator
