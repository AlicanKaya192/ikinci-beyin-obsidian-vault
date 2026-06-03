---
tarih: 2026-05-28
konu: Python
etiket: ["python", "regex", "regular-expression", "metin", "ileri"]
kaynak: Python Resmi Dokümantasyon
zorluk: ileri
---

## 📌 Özet
Regular Expression (regex), metin içinde desen aramak, eşleştirmek ve değiştirmek için kullanılan güçlü bir araçtır. Python'da `re` modülü ile kullanılır.

## 🧠 Detay

### Temel Karakterler
| Desen | Anlam |
|-------|-------|
| `.` | Herhangi bir karakter |
| `\d` | Rakam [0-9] |
| `\w` | Harf/rakam/alt çizgi |
| `\s` | Boşluk |
| `^` | Başlangıç |
| `$` | Bitiş |
| `*` | 0 veya daha fazla |
| `+` | 1 veya daha fazla |
| `?` | 0 veya 1 |

### Temel Fonksiyonlar
```python
import re

metin = "Python 3.11 çıktı, Python 3.12 geliyor"

# Eşleşme var mı?
re.search(r"Python", metin)

# Tüm eşleşmeleri bul
re.findall(r"Python \d+\.\d+", metin)
# ['Python 3.11', 'Python 3.12']

# Değiştir
re.sub(r"Python", "Dil", metin)

# Baştan eşleşme
re.match(r"Python", metin)
```

### Pratik Örnekler
```python
# E-posta doğrulama
email_pattern = r"[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}"
email = "ornek@gmail.com"
if re.match(email_pattern, email):
    print("Geçerli email")

# Telefon numarası
tel_pattern = r"(\+90|0)?[\s-]?5\d{2}[\s-]?\d{3}[\s-]?\d{4}"

# Tüm sayıları bul
metin = "Fiyat 100 TL, indirim 20 TL"
sayilar = re.findall(r"\d+", metin)
# ['100', '20']

# Grup yakalama
tarih = "2026-05-28"
m = re.match(r"(\d{4})-(\d{2})-(\d{2})", tarih)
yil, ay, gun = m.groups()
```

## 💡 Bağlantılar
- [[Python - String İşlemleri]]
- [[Python - Dosya İşlemleri]]

## ❓ Sorular / Anlamadıklarım
- `re.match` ile `re.search` arasındaki fark nedir?
- `r"..."` (raw string) neden kullanılır?

## 🔗 Kaynaklar
- https://docs.python.org/tr/3/library/re.html
- https://regex101.com (test aracı)
