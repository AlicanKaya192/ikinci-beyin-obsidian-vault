---
tarih: 2025-01-01
konu: MongoDB Transactions, ACID, Multi-Document, Session
etiket: [mongodb, transaction, acid, session, çok-belge, atomicity]
kaynak:
zorluk: ⭐⭐⭐⭐
---

## 📌 Özet

MongoDB 4.0+ ile çok belgeli (multi-document) ACID işlemleri desteklenir. Tek belge güncellemeleri her zaman atomiktir; birden fazla belge/koleksiyonu kapsayan işlemler için explicit transaction kullanılır.

---

## 🧠 Detay

### ACID MongoDB'de

| Özellik | Tek Belge | Çok Belge (Transaction) |
|---|---|---|
| **Atomicity** | ✅ Her zaman | ✅ 4.0+ |
| **Consistency** | ✅ | ✅ |
| **Isolation** | ✅ | ✅ Snapshot isolation |
| **Durability** | ✅ | ✅ |

### Tek Belge Atomicity (Transaction Gerektirmez)

```javascript
// Bu işlem atomiktir — kısmen güncelleme olmaz
db.hesaplar.updateOne(
  { _id: ObjectId("hesap1") },
  {
    $inc: { bakiye: -500 },
    $push: { islemler: { tip: "cikis", tutar: 500, tarih: new Date() } }
  }
)

// Array içindeki çoklu güncelleme de atomiktir
db.siparisler.updateOne(
  { _id: ObjectId() },
  {
    $set: { durum: "kargoda" },
    $push: { durum_gecmisi: { durum: "kargoda", tarih: new Date() } },
    $inc: { guncelleme_sayisi: 1 }
  }
)
```

### Multi-Document Transaction

```javascript
// ─── mongosh ───────────────────────────────
const session = db.getMongo().startSession()

session.startTransaction({
  readConcern:  { level: "snapshot" },
  writeConcern: { w: "majority" },
  readPreference: "primary"
})

try {
  const hesaplar = session.getDatabase("banka").getCollection("hesaplar")

  // Transfer işlemi: A'dan B'ye 500 TL
  hesaplar.updateOne(
    { _id: "hesap_A", bakiye: { $gte: 500 } },  // Yeterli bakiye kontrolü
    { $inc: { bakiye: -500 } },
    { session }
  )

  hesaplar.updateOne(
    { _id: "hesap_B" },
    { $inc: { bakiye: 500 } },
    { session }
  )

  // Her iki güncelleme başarılı → commit
  session.commitTransaction()
  print("Transfer başarılı!")

} catch (error) {
  // Herhangi biri başarısız → rollback
  session.abortTransaction()
  print("Transfer iptal:", error.message)

} finally {
  session.endSession()
}
```

### Python ile Transaction (PyMongo)

```python
from pymongo import MongoClient
from pymongo.errors import OperationFailure

client = MongoClient("mongodb://localhost:27017")
db = client.banka

def transfer(hesap_a: str, hesap_b: str, tutar: float):
    with client.start_session() as session:
        with session.start_transaction():
            hesaplar = db.hesaplar

            # A'dan düş
            result_a = hesaplar.update_one(
                {"_id": hesap_a, "bakiye": {"$gte": tutar}},
                {"$inc": {"bakiye": -tutar}},
                session=session
            )

            if result_a.matched_count == 0:
                raise OperationFailure("Yetersiz bakiye")

            # B'ye ekle
            hesaplar.update_one(
                {"_id": hesap_b},
                {"$inc": {"bakiye": tutar}},
                session=session
            )

            # Commit (with bloğu çıkışında otomatik)
            print(f"{tutar} TL transfer tamamlandı")

try:
    transfer("hesap_A", "hesap_B", 500)
except Exception as e:
    print(f"Hata: {e}")  # Otomatik abort
```

### Transaction Kısıtlamaları

```javascript
// ⚠️ Transaction limitleri:
// - Max süre: 60 saniye (maxTransactionLockRequestTimeoutMillis)
// - Max boyut: 16MB (tüm işlem toplamı)
// - DDL işlem yok (collection oluştur/sil)
// - Yeni collection oluşturma yok (zaten varsa tamam)
// - getMore yok cursor üzerinde

// Transaction timeout ayarla
session.startTransaction({ maxCommitTimeMS: 30000 })  // 30 saniye
```

### Read Concern ve Write Concern

```javascript
// Read Concern Seviyeleri:
// "local"     → En son yazılan (replica lag olabilir) — hızlı
// "available" → local gibi ama sharded'da farklı
// "majority"  → Çoğunluk tarafından kabul edilmiş — güvenli
// "snapshot"  → Transaction başındaki anlık görüntü

// Write Concern Seviyeleri:
// w: 0        → Onay bekleme (fire and forget)
// w: 1        → Primary onayı (varsayılan)
// w: "majority" → Çoğunluk onayı
// j: true     → Journal'a yazıldı (disk garantisi)

db.col.insertOne(
  { veri: "önemli" },
  { writeConcern: { w: "majority", j: true, wtimeout: 5000 } }
)
```

### Idempotent Yazma — Retry Mantığı

```javascript
// Transaction'lar transient hata durumunda retry yapılabilir
const maxRetry = 3

async function transactionWithRetry(fn, session) {
  for (let attempt = 0; attempt < maxRetry; attempt++) {
    try {
      session.startTransaction()
      await fn(session)
      await session.commitTransaction()
      return
    } catch (err) {
      await session.abortTransaction()

      // Geçici hata → retry
      if (err.errorLabels?.includes("TransientTransactionError")) {
        console.log(`Retry ${attempt + 1}/${maxRetry}`)
        continue
      }

      // UnknownTransactionCommitResult → commit retry
      if (err.errorLabels?.includes("UnknownTransactionCommitResult")) {
        await session.commitTransaction()
        return
      }

      throw err  // Kalıcı hata → fırlat
    }
  }
  throw new Error("Max retry aşıldı")
}
```

### Ne Zaman Transaction Kullanmalı?

```javascript
// ✅ Transaction gerekli:
// - Para transferi (A'dan düş, B'ye ekle)
// - Stok güncelleme + sipariş oluşturma
// - Birden fazla koleksiyonun atomik güncellenmesi
// - "Hepsi veya hiçbiri" mantığı

// ❌ Transaction gereksiz:
// - Tek belge güncellemesi (zaten atomik)
// - Okuma işlemleri
// - Düşük kritiklik (log yazma, cache güncelleme)

// ✅ Alternatif — İki Aşamalı Commit (Two-Phase Commit):
// Transaction yokken kullanılan classik pattern
// Pending state oluştur → işle → tamamlandı işaretle
```

---

## 💡 Bağlantılar
- [[MongoDB - CRUD İşlemleri]]
- [[MongoDB - Replikasyon ve Sharding]]
- [[MongoDB - Python ile MongoDB (PyMongo)]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- mongodb.com/docs/manual/core/transactions/
- mongodb.com/docs/drivers/python/
