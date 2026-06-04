---
tarih: 2026-06-04
konu: Python ile MongoDB (PyMongo) Kullanımı
etiket: [mongodb, python, pymongo, integration]
kaynak: PyMongo Documentation
zorluk: Orta
---

## 📌 Özet
Python, veri bilimi ve web geliştirme dünyasındaki popülaritesiyle MongoDB için en çok tercih edilen dillerden biridir. PyMongo, Python uygulamalarının MongoDB ile iletişim kurmasını sağlayan resmi ve standart sürücüdür. Python'un yerel veri yapısı olan sözlükler (dict), döküman tabanlı yapıya tam uyum sağlar. Bu notta, PyMongo üzerinden CRUD işlemleri ve aggregation pipeline kullanımı incelenmektedir.

## 🧠 Detay

```mermaid
graph LR
    A["Python Uygulaması"] --> B["PyMongo Driver"]
    B --> C["BSON Dönüştürücü"]
    C --> D["MongoDB Server"]
```

### 1. Temel Kullanım
```python
from pymongo import MongoClient
client = MongoClient("mongodb://localhost:27017/")
db = client.test_db
collection = db.users
```

### 2. İşlemler
- **Ekleme:** `collection.insert_one({"name": "Ahmet"})`
- **Sorgulama:** `collection.find({"active": True})`

## 💡 Bağlantılar
- [[Python - Giriş ve Yol Haritası]]
- [[MongoDB - CRUD İşlemleri]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- PyMongo Official Tutorial
- MongoDB for Python Developers
