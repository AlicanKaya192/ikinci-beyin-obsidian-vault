---
tarih: 2026-05-28
konu: Python
etiket: ["python", "koşullar", "if-elif-else", "temel"]
kaynak: Python Resmi Dokümantasyon
zorluk: başlangıç
---

## 📌 Özet
Koşullu ifadeler, belirli bir koşula göre farklı kod bloklarının çalıştırılmasını sağlar. Python'da `if`, `elif` ve `else` anahtar kelimeleri kullanılır.

## 🧠 Detay

### Temel if-elif-else
```python
yas = 20

if yas < 18:
    print("Çocuk")
elif yas < 65:
    print("Yetişkin")
else:
    print("Yaşlı")
```

### İç İçe Koşullar
```python
puan = 75

if puan >= 50:
    if puan >= 85:
        print("Pekiyi")
    elif puan >= 70:
        print("İyi")
    else:
        print("Geçti")
else:
    print("Kaldı")
```

### Tek Satır (Ternary)
```python
x = 10
sonuc = "pozitif" if x > 0 else "negatif"
print(sonuc)  # pozitif
```

### Mantıksal Operatörlerle Koşul
```python
yas = 25
vatandas = True

if yas >= 18 and vatandas:
    print("Oy kullanabilir")

sehir = "İstanbul"
if sehir == "İstanbul" or sehir == "Ankara":
    print("Büyükşehir")
```

### Truthy / Falsy Değerler
```python
# False sayılanlar: 0, "", [], {}, None, False
# True sayılanlar: diğer her şey

liste = []
if liste:
    print("Dolu")
else:
    print("Boş")  # Bu çalışır
```

## 💡 Bağlantılar
- [[Python - Operatörler]]
- [[Python - Döngüler (for-while)]]
- [[Python - Fonksiyonlar]]

## ❓ Sorular / Anlamadıklarım
- `match-case` (Python 3.10+) ile `if-elif` arasında ne fark var?
- Truthy/Falsy değerleri nerede dikkat etmeliyim?

## 🔗 Kaynaklar
- https://docs.python.org/tr/3/tutorial/controlflow.html
