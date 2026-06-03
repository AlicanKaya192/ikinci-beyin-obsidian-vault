---
tarih: 2026-05-28
konu: Python
etiket: ["python", "string", "metin", "temel"]
kaynak: Python Resmi Dokümantasyon
zorluk: başlangıç
---

## 📌 Özet
String (metin), Python'da en çok kullanılan veri tipidir. Tırnak işaretleri içinde tanımlanır ve üzerinde onlarca hazır metod bulunur.

## 🧠 Detay

### Tanımlama ve f-string
```python
isim = "Ahmet"
soyisim = 'Yılmaz'
cok_satir = """Bu
çok satırlı
bir metindir."""

# f-string (en çok kullanılan)
yas = 30
print(f"{isim} {soyisim}, {yas} yaşında")
```

### Temel Metodlar
```python
metin = "  Merhaba Dünya  "

print(metin.upper())       # MERHABA DÜNYA
print(metin.lower())       # merhaba dünya
print(metin.strip())       # "Merhaba Dünya" (boşluk siler)
print(metin.replace("Dünya", "Python"))
print(metin.split())       # ['Merhaba', 'Dünya']
```

### Arama ve Kontrol
```python
metin = "Python programlama"

print(metin.startswith("Python"))  # True
print(metin.endswith("lama"))      # True
print(metin.find("program"))       # 7 (index)
print("Python" in metin)           # True
print(metin.count("a"))            # 3
```

### Dilimleme
```python
s = "Python"
print(s[0])     # P
print(s[-1])    # n
print(s[1:4])   # yth
print(s[::-1])  # nohtyP (ters)
```

### Birleştirme
```python
kelimeler = ["Python", "çok", "güzel"]
print(" ".join(kelimeler))    # Python çok güzel
print("-".join(kelimeler))    # Python-çok-güzel
```

### Formatlama
```python
pi = 3.14159
print(f"{pi:.2f}")        # 3.14
print(f"{1000000:,}")     # 1,000,000
print(f"{'sağa':>10}")    # sağa hizalama
```

## 💡 Bağlantılar
- [[Python - Değişkenler ve Veri Tipleri]]
- [[Python - Listeler]]
- [[Python - Regular Expressions]]

## ❓ Sorular / Anlamadıklarım
- `format()` ile f-string arasındaki fark nedir?
- Unicode ve Türkçe karakter sorunları nasıl çözülür?

## 🔗 Kaynaklar
- https://docs.python.org/tr/3/library/stdtypes.html#string-methods
