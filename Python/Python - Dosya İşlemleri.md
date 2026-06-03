---
tarih: 2026-05-28
konu: Python
etiket: ["python", "dosya", "io", "orta"]
kaynak: Python Resmi Dokümantasyon
zorluk: orta
---

## 📌 Özet
Python'da dosya okuma ve yazma işlemleri `open()` fonksiyonu ile yapılır. `with` bloğu kullanımı dosyaların güvenli şekilde kapatılmasını sağlar.

## 🧠 Detay

### Dosya Açma Modları
| Mod | Açıklama |
|-----|----------|
| `r` | Okuma (varsayılan) |
| `w` | Yazma (sıfırdan yazar) |
| `a` | Ekleme (sonuna ekler) |
| `r+` | Okuma + yazma |

### Dosya Okuma
```python
# Tamamını oku
with open("dosya.txt", "r", encoding="utf-8") as f:
    icerik = f.read()

# Satır satır oku
with open("dosya.txt", "r", encoding="utf-8") as f:
    for satir in f:
        print(satir.strip())

# Tüm satırları liste olarak
with open("dosya.txt", "r", encoding="utf-8") as f:
    satirlar = f.readlines()
```

### Dosya Yazma
```python
# Yeni dosya oluştur / üzerine yaz
with open("yeni.txt", "w", encoding="utf-8") as f:
    f.write("Merhaba Dünya\n")
    f.write("İkinci satır\n")

# Sonuna ekle
with open("yeni.txt", "a", encoding="utf-8") as f:
    f.write("Üçüncü satır\n")
```

### os Modülü ile Dosya İşlemleri
```python
import os

print(os.getcwd())              # mevcut dizin
print(os.listdir("."))          # dizin içeriği
os.makedirs("yeni_klasor", exist_ok=True)
os.remove("dosya.txt")         # dosya sil
print(os.path.exists("dosya")) # var mı?
```

## 💡 Bağlantılar
- [[Python - Hata Yönetimi (try-except)]]
- [[Python - JSON İşlemleri]]
- [[Python - Modüller ve Paketler]]

## ❓ Sorular / Anlamadıklarım
- `encoding="utf-8"` neden önemli, ne zaman atlayabilirim?
- Binary dosyalar nasıl okunur?

## 🔗 Kaynaklar
- https://docs.python.org/tr/3/tutorial/inputoutput.html#reading-and-writing-files
