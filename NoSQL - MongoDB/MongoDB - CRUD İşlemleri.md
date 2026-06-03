---
tarih: 2025-01-01
konu: MongoDB CRUD, insertOne, find, updateOne, deleteOne
etiket: [mongodb, crud, insert, find, update, delete, nosql]
kaynak:
zorluk: ⭐⭐
---

## 📌 Özet

MongoDB'de temel veri işlemleri: Create (insertOne/Many), Read (find/findOne), Update (updateOne/Many), Delete (deleteOne/Many). Her işlem döküman bazında çalışır.

---

## 🧠 Detay

### CREATE — Belge Ekleme

```javascript
// Tek belge ekle
db.urunler.insertOne({
  ad: "Laptop",
  fiyat: 25000,
  stok: 50,
  kategori: "elektronik",
  olusturma: new Date()
})
// Döner: { acknowledged: true, insertedId: ObjectId("...") }

// Çoklu belge ekle
db.urunler.insertMany([
  { ad: "Klavye", fiyat: 500, stok: 200 },
  { ad: "Mouse",  fiyat: 300, stok: 150 },
  { ad: "Monitör",fiyat: 8000, stok: 30  }
])
// Döner: { acknowledged: true, insertedIds: { "0": ..., "1": ..., "2": ... } }

// Hata durumunda devam et (ordered: false)
db.urunler.insertMany(belgeler, { ordered: false })

// _id manuel belirleme
db.urunler.insertOne({ _id: "laptop-001", ad: "Laptop" })
```

---

### READ — Belge Okuma

```javascript
// Tüm belgeleri getir
db.urunler.find()
db.urunler.find({})

// Güzel formatlama
db.urunler.find().pretty()  // Eski mongosh
// Yeni mongosh'ta otomatik formatlı

// Tek belge (ilk eşleşen)
db.urunler.findOne({ ad: "Laptop" })

// Koşulla sorgula
db.urunler.find({ kategori: "elektronik" })
db.urunler.find({ fiyat: 25000 })

// Projeksiyon — hangi alanlar gelsin
db.urunler.find(
  { kategori: "elektronik" },  // filtre
  { ad: 1, fiyat: 1, _id: 0 } // projeksiyon (1=dahil, 0=hariç)
)

// Sıralama
db.urunler.find().sort({ fiyat: 1 })   // Artan
db.urunler.find().sort({ fiyat: -1 })  // Azalan
db.urunler.find().sort({ kategori: 1, fiyat: -1 })  // Çoklu

// Limit ve Skip (sayfalama)
db.urunler.find().limit(10)
db.urunler.find().skip(20).limit(10)   // Sayfa 3 (0-indeksli, sayfa 2)

// Sayma
db.urunler.countDocuments({ kategori: "elektronik" })
db.urunler.estimatedDocumentCount()  // Yaklaşık (hızlı)

// Tekil değerleri getir
db.urunler.distinct("kategori")
// ["elektronik", "giyim", "kitap"]
```

---

### UPDATE — Belge Güncelleme

```javascript
// ── Güncelleme Operatörleri ───────────────

// $set — alan ekle/güncelle
db.urunler.updateOne(
  { ad: "Laptop" },             // filtre
  { $set: { fiyat: 27000, guncelleme: new Date() } }
)

// $unset — alanı sil
db.urunler.updateOne(
  { ad: "Laptop" },
  { $unset: { eski_alan: "" } }
)

// $inc — sayısal artır/azalt
db.urunler.updateOne(
  { ad: "Laptop" },
  { $inc: { stok: -5, satis_sayisi: 5 } }
)

// $mul — çarp
db.urunler.updateOne(
  { ad: "Laptop" },
  { $mul: { fiyat: 1.1 } }  // %10 zam
)

// $rename — alan adını değiştir
db.urunler.updateMany(
  {},
  { $rename: { "eski_ad": "yeni_ad" } }
)

// $min / $max — karşılaştırmalı set
db.urunler.updateOne(
  { ad: "Laptop" },
  { $min: { min_fiyat: 20000 } }  // Mevcut değerden küçükse güncelle
)

// ── Array Operatörleri ────────────────────

// $push — array'e eleman ekle
db.urunler.updateOne(
  { ad: "Laptop" },
  { $push: { etiketler: "indirim" } }
)

// $push + $each — birden fazla eleman
db.urunler.updateOne(
  { ad: "Laptop" },
  { $push: { etiketler: { $each: ["yeni", "popüler"] } } }
)

// $addToSet — tekrarsız ekleme
db.urunler.updateOne(
  { ad: "Laptop" },
  { $addToSet: { etiketler: "elektronik" } }  // Yoksa ekler, varsa eklemez
)

// $pull — koşula göre sil
db.urunler.updateOne(
  { ad: "Laptop" },
  { $pull: { etiketler: "eski" } }
)

// $pop — ilk/son elemanı sil
db.urunler.updateOne({ ad: "Laptop" }, { $pop: { etiketler: 1 } })   // Son
db.urunler.updateOne({ ad: "Laptop" }, { $pop: { etiketler: -1 } })  // İlk

// ── Çoklu Güncelleme ─────────────────────

// Birden fazla belge güncelle
db.urunler.updateMany(
  { kategori: "elektronik" },
  { $set: { kdv: 0.18 } }
)

// ── Upsert — Yoksa Ekle, Varsa Güncelle ──
db.urunler.updateOne(
  { ad: "Yeni Ürün" },
  { $set: { fiyat: 100, stok: 50 } },
  { upsert: true }
)

// ── findOneAndUpdate — Güncelle ve Döndür ─
const onceki = db.urunler.findOneAndUpdate(
  { ad: "Laptop" },
  { $inc: { stok: -1 } },
  {
    returnDocument: "after",  // "before" → güncelleme öncesi
    projection: { ad: 1, stok: 1 }
  }
)

// ── replaceOne — Tüm belgeyi değiştir ────
db.urunler.replaceOne(
  { ad: "Eski Laptop" },
  { ad: "Yeni Laptop", fiyat: 30000, stok: 20 }
  // _id korunur, diğer tüm alanlar değişir
)
```

---

### DELETE — Belge Silme

```javascript
// Tek belge sil (ilk eşleşen)
db.urunler.deleteOne({ ad: "Laptop" })

// Çoklu belge sil
db.urunler.deleteMany({ stok: 0 })

// Tüm koleksiyonu temizle (şema korunur)
db.urunler.deleteMany({})

// findOneAndDelete — Sil ve döndür
const silinen = db.urunler.findOneAndDelete(
  { ad: "Laptop" },
  { projection: { ad: 1, fiyat: 1 } }
)

// Koleksiyonu tamamen sil (şema dahil)
db.urunler.drop()

// Veritabanını sil
db.dropDatabase()
```

---

### Bulk Write — Toplu İşlem

```javascript
db.urunler.bulkWrite([
  { insertOne: { document: { ad: "Tablet", fiyat: 8000 } } },
  { updateOne: {
      filter: { ad: "Laptop" },
      update: { $inc: { stok: -1 } }
  }},
  { deleteOne: { filter: { stok: 0 } } },
  { replaceOne: {
      filter: { ad: "Eski Model" },
      replacement: { ad: "Yeni Model", fiyat: 5000 },
      upsert: true
  }}
], { ordered: false })  // Hata durumunda diğerleri devam et
```

---

### CRUD Özet Tablosu

| İşlem | Tek | Çoklu |
|---|---|---|
| **Create** | `insertOne()` | `insertMany()` |
| **Read** | `findOne()` | `find()` |
| **Update** | `updateOne()` | `updateMany()` |
| **Delete** | `deleteOne()` | `deleteMany()` |
| **Replace** | `replaceOne()` | — |
| **Upsert** | `updateOne({upsert:true})` | `updateMany({upsert:true})` |

---

## 💡 Bağlantılar
- [[MongoDB - Giriş ve Temel Kavramlar]]
- [[MongoDB - Sorgulama ve Operatörler]]
- [[MongoDB - Aggregation Pipeline]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- mongodb.com/docs/manual/crud/
- mongodb.com/docs/manual/reference/operator/update/
