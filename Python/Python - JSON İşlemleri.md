---
tarih: 2026-05-28
konu: Python
etiket: ["python", "json", "veri", "api", "orta"]
kaynak: Python Resmi Dokümantasyon
zorluk: orta
---

## 📌 Özet
JSON (JavaScript Object Notation), veri alışverişinde en yaygın kullanılan format. Python'da `json` modülü ile kolayca okunup yazılabilir. API'larla çalışırken sıklıkla kullanılır.

## 🧠 Detay

### json Modülü
```python
import json

# Python dict → JSON string
veri = {
    "isim": "Ahmet",
    "yas": 30,
    "hobiler": ["python", "data science"]
}

json_str = json.dumps(veri, ensure_ascii=False, indent=2)
print(json_str)

# JSON string → Python dict
geri = json.loads(json_str)
print(geri["isim"])   # Ahmet
```

### Dosyaya Yazma ve Okuma
```python
# Dosyaya yaz
with open("veri.json", "w", encoding="utf-8") as f:
    json.dump(veri, f, ensure_ascii=False, indent=2)

# Dosyadan oku
with open("veri.json", "r", encoding="utf-8") as f:
    okunan = json.load(f)
```

### API'dan JSON Alma
```python
import requests

yanit = requests.get("https://api.example.com/data")
veri = yanit.json()   # otomatik parse eder
print(veri["sonuc"])
```

### İç İçe JSON Gezinme
```python
veri = {
    "kullanici": {
        "isim": "Ali",
        "adres": {
            "sehir": "İstanbul",
            "ilce": "Kadıköy"
        }
    }
}

sehir = veri["kullanici"]["adres"]["sehir"]
# veya güvenli
sehir = veri.get("kullanici", {}).get("adres", {}).get("sehir")
```

### JSON ↔ Python Tip Eşleşmesi
| JSON | Python |
|------|--------|
| object | dict |
| array | list |
| string | str |
| number | int/float |
| true/false | True/False |
| null | None |

## 💡 Bağlantılar
- [[Python - Sözlükler (Dictionary)]]
- [[Python - Dosya İşlemleri]]
- [[Data Science - API Kullanımı]]

## ❓ Sorular / Anlamadıklarım
- `ensure_ascii=False` ne işe yarar?
- Büyük JSON dosyaları nasıl verimli işlenir?

## 🔗 Kaynaklar
- https://docs.python.org/tr/3/library/json.html
