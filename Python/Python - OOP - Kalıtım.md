---
tarih: 2026-05-28
konu: Python
etiket: ["python", "oop", "kalıtım", "inheritance", "orta"]
kaynak: Python Resmi Dokümantasyon
zorluk: orta
---

## 📌 Özet
Kalıtım (inheritance), bir sınıfın başka bir sınıfın özelliklerini ve metodlarını devralmasıdır. Kod tekrarını önler ve hiyerarşik yapılar oluşturur.

## 🧠 Detay

### Temel Kalıtım
```python
class Hayvan:
    def __init__(self, isim):
        self.isim = isim

    def ses_cikar(self):
        print("...")

    def tanitim(self):
        print(f"Ben {self.isim}")

class Kopek(Hayvan):
    def ses_cikar(self):     # override
        print("Hav hav!")

    def getir(self):         # yeni metod
        print("Topu getirdi")

class Kedi(Hayvan):
    def ses_cikar(self):
        print("Miyav!")

kopek = Kopek("Karabaş")
kopek.tanitim()      # Hayvan'dan miras → "Ben Karabaş"
kopek.ses_cikar()    # Override → "Hav hav!"
```

### super() Kullanımı
```python
class Calisan:
    def __init__(self, isim, maas):
        self.isim = isim
        self.maas = maas

class Yonetici(Calisan):
    def __init__(self, isim, maas, departman):
        super().__init__(isim, maas)   # üst sınıf __init__
        self.departman = departman

    def tanitim(self):
        print(f"{self.isim} - {self.departman} Müdürü")
```

### Çoklu Kalıtım
```python
class A:
    def metod(self):
        print("A")

class B(A):
    def metod(self):
        print("B")

class C(A):
    def metod(self):
        print("C")

class D(B, C):
    pass

d = D()
d.metod()   # "B" → MRO: D → B → C → A
print(D.__mro__)
```

### isinstance ve issubclass
```python
kopek = Kopek("Rex")
print(isinstance(kopek, Kopek))   # True
print(isinstance(kopek, Hayvan))  # True
print(issubclass(Kopek, Hayvan))  # True
```

## 💡 Bağlantılar
- [[Python - OOP - Sınıflar ve Nesneler]]
- [[Python - OOP - Kapsülleme]]

## ❓ Sorular / Anlamadıklarım
- MRO (Method Resolution Order) nasıl çalışır?
- Çoklu kalıtımda diamond problem nedir?

## 🔗 Kaynaklar
- https://docs.python.org/tr/3/tutorial/classes.html#inheritance
