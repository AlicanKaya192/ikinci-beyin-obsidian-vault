---
tarih: 2025-01-01
konu: MongoDB Change Streams, Gerçek Zamanlı Veri, Event-Driven
etiket: [mongodb, change-stream, gerçek-zamanlı, event-driven, websocket]
kaynak:
zorluk: ⭐⭐⭐⭐
---

## 📌 Özet

Change Streams, MongoDB'deki veri değişikliklerini gerçek zamanlı olarak dinlemenizi sağlar. Oplog üzerine inşa edilmiştir ve Replica Set / Sharded Cluster gerektirir.

---

## 🧠 Detay

### Change Stream Nedir?

```
Uygulama                 MongoDB
    │                       │
    │── watch() ────────────►│
    │                       │ (Oplog izler)
    │◄── insert event ───────│
    │◄── update event ───────│
    │◄── delete event ───────│
```

- SQL'deki trigger'ın MongoDB karşılığı
- CDC (Change Data Capture) için ideal
- Replica Set veya Sharded Cluster zorunlu

### Temel Kullanım (mongosh)

```javascript
// Koleksiyon düzeyinde dinle
const stream = db.siparisler.watch()
while (stream.hasNext()) {
  const event = stream.next()
  printjson(event)
}

// Veritabanı düzeyinde tüm koleksiyonları dinle
const dbStream = db.watch()

// Cluster düzeyinde (admin db'de)
const clusterStream = db.getMongo().watch()
```

### Event Yapısı

```javascript
// insert eventi
{
  _id: { _data: "resume_token..." },  // Resume token
  operationType: "insert",             // insert | update | replace | delete | drop | rename
  clusterTime: Timestamp(1704067200, 1),
  ns: { db: "mydb", coll: "siparisler" },
  documentKey: { _id: ObjectId("...") },
  fullDocument: {                      // Eklenen/değiştirilen belge
    _id: ObjectId("..."),
    musteri: "Ali",
    tutar: 500
  }
}

// update eventi
{
  operationType: "update",
  documentKey: { _id: ObjectId("...") },
  updateDescription: {
    updatedFields: { "durum": "kargoda" },
    removedFields: [],
    truncatedArrays: []
  }
  // fullDocument: null (varsayılan — tam belge için updateLookup gerekir)
}
```

### Pipeline ile Filtreleme

```javascript
// Sadece belirli olayları dinle
const pipeline = [
  {
    $match: {
      $or: [
        { operationType: "insert" },
        { operationType: "update" }
      ]
    }
  }
]
const stream = db.siparisler.watch(pipeline)

// Belirli alanlar değişince tetikle
const pipeline2 = [
  {
    $match: {
      operationType: "update",
      "updateDescription.updatedFields.durum": { $exists: true }
    }
  }
]

// Tam belgeyi döndür (updateLookup)
const stream3 = db.col.watch(pipeline, {
  fullDocument: "updateLookup",         // Update'te tam belge dahil et
  fullDocumentBeforeChange: "whenAvailable"  // Değişim öncesi belge (5.3+)
})
```

### Resume Token — Kaldığı Yerden Devam

```javascript
// Token'ı kaydet
let resumeToken = null

const stream = db.col.watch()
while (stream.hasNext()) {
  const event = stream.next()
  resumeToken = event._id  // Her event'ten token al
  processEvent(event)
}

// Uygulama çökmüşse kaldığı yerden devam et
const newStream = db.col.watch([], {
  resumeAfter: resumeToken
  // veya:
  // startAtOperationTime: Timestamp(1704067200, 1)
})
```

### Python ile Change Stream

```python
import pymongo
from pymongo import MongoClient
import threading

client = MongoClient("mongodb://localhost:27017/?replicaSet=rs0")
db = client.mydb
col = db.siparisler

def dinle_siparisler():
    pipeline = [
        {"$match": {
            "operationType": {"$in": ["insert", "update"]},
        }}
    ]

    with col.watch(
        pipeline,
        full_document="updateLookup"
    ) as stream:
        for event in stream:
            op = event["operationType"]
            belge = event.get("fullDocument", {})

            if op == "insert":
                print(f"Yeni sipariş: {belge.get('musteri')} - {belge.get('tutar')}₺")
                bildirim_gonder(belge)

            elif op == "update":
                degisimler = event["updateDescription"]["updatedFields"]
                if "durum" in degisimler:
                    print(f"Durum değişti: {degisimler['durum']}")
                    email_gonder(belge)

# Ayrı thread'de çalıştır
thread = threading.Thread(target=dinle_siparisler, daemon=True)
thread.start()
```

### FastAPI + Motor ile WebSocket

```python
import asyncio
import motor.motor_asyncio
from fastapi import FastAPI, WebSocket
from typing import List

app = FastAPI()
client = motor.motor_asyncio.AsyncIOMotorClient("mongodb://localhost:27017")
db = client.mydb

# Aktif WebSocket bağlantıları
active_connections: List[WebSocket] = []

@app.websocket("/ws/siparisler")
async def siparis_stream(websocket: WebSocket):
    await websocket.accept()
    active_connections.append(websocket)

    try:
        pipeline = [{"$match": {"operationType": "insert"}}]
        async with db.siparisler.watch(pipeline) as stream:
            async for event in stream:
                belge = event["fullDocument"]
                belge["_id"] = str(belge["_id"])

                # Tüm bağlı istemcilere yayınla
                for conn in active_connections:
                    await conn.send_json(belge)
    except Exception:
        active_connections.remove(websocket)
```

### Kullanım Senaryoları

```python
# 1. Cache Invalidation
async def cache_invalidator():
    async with db.urunler.watch() as stream:
        async for event in stream:
            urun_id = str(event["documentKey"]["_id"])
            await redis.delete(f"urun:{urun_id}")
            print(f"Cache temizlendi: {urun_id}")

# 2. Audit Log
async def audit_logger():
    async with db.watch() as stream:
        async for event in stream:
            await db.audit_log.insert_one({
                "operasyon": event["operationType"],
                "koleksiyon": event["ns"]["coll"],
                "belge_id": event["documentKey"]["_id"],
                "zaman": event["clusterTime"],
                "degisimler": event.get("updateDescription")
            })

# 3. Elasticsearch Sync
async def es_syncer():
    async with db.urunler.watch(
        full_document="updateLookup"
    ) as stream:
        async for event in stream:
            op = event["operationType"]
            doc = event.get("fullDocument")
            doc_id = str(event["documentKey"]["_id"])

            if op in ("insert", "update", "replace"):
                await es.index(index="urunler", id=doc_id, document=doc)
            elif op == "delete":
                await es.delete(index="urunler", id=doc_id)
```

### Konfigürasyon Seçenekleri

```javascript
db.col.watch(pipeline, {
  fullDocument: "updateLookup",        // update'te tam belge
  fullDocumentBeforeChange: "off",     // "whenAvailable" | "required"
  resumeAfter: token,                  // Token'dan devam
  startAfter: token,                   // Token sonrasından başla
  startAtOperationTime: timestamp,     // Belirli zamandan başla
  maxAwaitTimeMS: 10000,              // Max bekleme süresi (ms)
  batchSize: 100                       // Batch büyüklüğü
})
```

---

## 💡 Bağlantılar
- [[MongoDB - Replikasyon ve Sharding]]
- [[MongoDB - Python ile PyMongo]]
- [[MongoDB - Transactions ve ACID]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- mongodb.com/docs/manual/changeStreams/
- motor.readthedocs.io/en/stable/tutorial-asyncio.html
