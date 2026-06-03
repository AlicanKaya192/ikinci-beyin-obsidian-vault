---
tarih: 2025-01-01
konu: PyMongo, Motor (Async), Python ile MongoDB, Bağlantı Havuzu
etiket: [mongodb, pymongo, python, motor, async, bağlantı]
kaynak:
zorluk: ⭐⭐⭐
---

## 📌 Özet

PyMongo, MongoDB'nin resmi Python sürücüsüdür. Motor ise async/await destekli async sürücüdür. FastAPI ve Django ile entegrasyon örnekleri dahildir.

---

## 🧠 Detay

### Kurulum ve Bağlantı

```bash
pip install pymongo                    # Senkron
pip install motor                      # Async (asyncio)
pip install pymongo[srv]               # SRV (Atlas) desteği
pip install "pymongo[srv,tls]"         # TLS + SRV
```

```python
from pymongo import MongoClient
from pymongo.server_api import ServerApi

# Basit bağlantı
client = MongoClient("mongodb://localhost:27017")

# Kimlik doğrulamalı
client = MongoClient("mongodb://user:pass@localhost:27017/mydb?authSource=admin")

# Atlas (Cloud)
client = MongoClient(
    "mongodb+srv://user:pass@cluster.mongodb.net/",
    server_api=ServerApi("1")
)

# Bağlantı havuzu ayarları
client = MongoClient(
    "mongodb://localhost:27017",
    maxPoolSize=50,          # Maksimum bağlantı
    minPoolSize=5,           # Minimum bağlantı
    maxIdleTimeMS=30000,     # 30sn boşta kalırsa kapat
    connectTimeoutMS=5000,   # Bağlantı timeout
    serverSelectionTimeoutMS=5000,
    socketTimeoutMS=20000
)

# Veritabanı ve koleksiyon
db  = client["mydb"]          # veya client.mydb
col = db["urunler"]           # veya db.urunler

# Bağlantıyı kapat
client.close()

# Context manager ile (önerilen)
with MongoClient("mongodb://localhost:27017") as client:
    db = client.mydb
    # işlemler...
```

### CRUD — PyMongo

```python
from pymongo import MongoClient, ASCENDING, DESCENDING
from bson import ObjectId
from datetime import datetime

client = MongoClient("mongodb://localhost:27017")
db = client.mydb
col = db.urunler

# ─── INSERT ──────────────────────────────
result = col.insert_one({
    "ad": "Laptop",
    "fiyat": 25000,
    "stok": 50,
    "olusturma": datetime.now()
})
print(result.inserted_id)  # ObjectId(...)

result = col.insert_many([
    {"ad": "Klavye", "fiyat": 500},
    {"ad": "Mouse",  "fiyat": 300}
])
print(result.inserted_ids)

# ─── FIND ────────────────────────────────
# Tek belge
belge = col.find_one({"ad": "Laptop"})
belge = col.find_one({"_id": ObjectId("64f1a2b3...")})

# Çoklu belge
belgeler = col.find({"fiyat": {"$gt": 1000}})
for b in belgeler:
    print(b["ad"], b["fiyat"])

# Projeksiyon
belgeler = col.find(
    {"kategori": "elektronik"},
    {"ad": 1, "fiyat": 1, "_id": 0}  # Sadece ad ve fiyat
)

# Sıralama, limit, skip
belgeler = (col.find()
    .sort("fiyat", DESCENDING)
    .skip(10)
    .limit(20))

# Liste olarak al
liste = list(col.find({"aktif": True}))

# Sayma
sayi = col.count_documents({"kategori": "elektronik"})
hizli_sayi = col.estimated_document_count()

# Distinct
kategoriler = col.distinct("kategori")

# ─── UPDATE ──────────────────────────────
# Tek güncelleme
result = col.update_one(
    {"ad": "Laptop"},
    {
        "$set": {"fiyat": 27000},
        "$inc": {"stok": -1}
    }
)
print(result.matched_count, result.modified_count)

# Çoklu güncelleme
result = col.update_many(
    {"kategori": "elektronik"},
    {"$set": {"kdv": 0.18}}
)

# Upsert
col.update_one(
    {"email": "ali@example.com"},
    {"$set": {"ad": "Ali", "aktif": True}},
    upsert=True
)

# findOneAndUpdate
guncellendi = col.find_one_and_update(
    {"_id": ObjectId("...")},
    {"$inc": {"stok": -1}},
    return_document=True  # Güncel belgeyi döndür
)

# ─── DELETE ──────────────────────────────
col.delete_one({"ad": "Eski Ürün"})
col.delete_many({"stok": 0})

silindi = col.find_one_and_delete({"_id": ObjectId("...")})

# ─── BULK WRITE ──────────────────────────
from pymongo import InsertOne, UpdateOne, DeleteOne

col.bulk_write([
    InsertOne({"ad": "Yeni Ürün", "fiyat": 100}),
    UpdateOne({"ad": "Laptop"}, {"$inc": {"stok": 10}}),
    DeleteOne({"stok": 0})
], ordered=False)
```

### Aggregation — PyMongo

```python
# Pipeline
pipeline = [
    {"$match": {"kategori": "elektronik", "fiyat": {"$gt": 500}}},
    {"$group": {
        "_id": "$marka",
        "urun_sayisi": {"$sum": 1},
        "ort_fiyat":   {"$avg": "$fiyat"},
        "max_fiyat":   {"$max": "$fiyat"}
    }},
    {"$sort": {"ort_fiyat": -1}},
    {"$limit": 10}
]

for sonuc in col.aggregate(pipeline):
    print(sonuc)

# allowDiskUse
list(col.aggregate(pipeline, allowDiskUse=True))
```

### İndeks — PyMongo

```python
# Tek alan
col.create_index("email", unique=True)
col.create_index([("fiyat", ASCENDING)])
col.create_index([("fiyat", DESCENDING)])

# Compound
col.create_index([("kategori", ASCENDING), ("fiyat", DESCENDING)])

# TTL
col.create_index("olusturma", expireAfterSeconds=3600)

# Text
col.create_index([("ad", "text"), ("aciklama", "text")])

# Seçeneklerle
col.create_index(
    [("email", ASCENDING)],
    unique=True,
    sparse=True,
    name="email_unique_idx"
)

# İndeks listesi
for idx in col.list_indexes():
    print(idx)

# İndeks sil
col.drop_index("email_1")
col.drop_indexes()
```

### Motor — Async MongoDB (FastAPI için)

```python
# pip install motor
import motor.motor_asyncio
from fastapi import FastAPI
from bson import ObjectId

app = FastAPI()

# Async client
client = motor.motor_asyncio.AsyncIOMotorClient("mongodb://localhost:27017")
db = client.mydb
col = db.urunler

@app.get("/urunler")
async def urun_listesi(limit: int = 10):
    belgeler = []
    async for belge in col.find().limit(limit):
        belge["_id"] = str(belge["_id"])  # ObjectId → string
        belgeler.append(belge)
    return belgeler

@app.get("/urunler/{id}")
async def urun_getir(id: str):
    belge = await col.find_one({"_id": ObjectId(id)})
    if not belge:
        return {"hata": "Bulunamadı"}
    belge["_id"] = str(belge["_id"])
    return belge

@app.post("/urunler")
async def urun_ekle(urun: dict):
    result = await col.insert_one(urun)
    return {"id": str(result.inserted_id)}

@app.put("/urunler/{id}")
async def urun_guncelle(id: str, guncelleme: dict):
    await col.update_one(
        {"_id": ObjectId(id)},
        {"$set": guncelleme}
    )
    return {"durum": "güncellendi"}

@app.delete("/urunler/{id}")
async def urun_sil(id: str):
    await col.delete_one({"_id": ObjectId(id)})
    return {"durum": "silindi"}
```

### Pydantic + Motor (Modern Pattern)

```python
from pydantic import BaseModel, Field
from bson import ObjectId
from typing import Optional

class PyObjectId(ObjectId):
    @classmethod
    def __get_validators__(cls):
        yield cls.validate

    @classmethod
    def validate(cls, v):
        if not ObjectId.is_valid(v):
            raise ValueError("Geçersiz ObjectId")
        return ObjectId(v)

class UrunModel(BaseModel):
    id: Optional[PyObjectId] = Field(alias="_id")
    ad: str
    fiyat: float
    stok: int = 0

    class Config:
        json_encoders = {ObjectId: str}
        populate_by_name = True

# Kullanım
urun = UrunModel(ad="Laptop", fiyat=25000.0)
await col.insert_one(urun.model_dump(by_alias=True, exclude_none=True))
```

### Hata Yönetimi

```python
from pymongo.errors import (
    ConnectionFailure, OperationFailure,
    DuplicateKeyError, ServerSelectionTimeoutError
)

try:
    col.insert_one({"email": "mevcut@example.com"})
except DuplicateKeyError:
    print("Email zaten mevcut")
except ServerSelectionTimeoutError:
    print("MongoDB'ye bağlanılamadı")
except OperationFailure as e:
    print(f"İşlem hatası: {e.code} — {e.details}")
except Exception as e:
    print(f"Beklenmedik hata: {e}")
```

---

## 💡 Bağlantılar
- [[MongoDB - CRUD İşlemleri]]
- [[MongoDB - Aggregation Pipeline]]
- [[MongoDB - Transactions ve ACID]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- pymongo.readthedocs.io
- motor.readthedocs.io
- mongodb.com/docs/drivers/python/
