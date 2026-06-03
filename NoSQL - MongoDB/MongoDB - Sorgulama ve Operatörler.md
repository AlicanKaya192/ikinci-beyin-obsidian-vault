---
tarih: 2025-01-01
konu: MongoDB Sorgu Operatörleri, Karşılaştırma, Mantıksal, Array, Regex
etiket: [mongodb, sorgu, operatör, regex, array, karşılaştırma, mantıksal]
kaynak:
zorluk: ⭐⭐⭐
---

## 📌 Özet

MongoDB'nin zengin sorgu dili: karşılaştırma ($gt, $lt), mantıksal ($and, $or), element ($exists, $type), array ($in, $all, $elemMatch) ve metin ($regex, $text) operatörleri.

---

## 🧠 Detay

### Karşılaştırma Operatörleri

```javascript
// $eq — eşit (varsayılan)
db.col.find({ yas: { $eq: 25 } })
db.col.find({ yas: 25 })          // Aynı şey

// $ne — eşit değil
db.col.find({ durum: { $ne: "pasif" } })

// $gt, $gte — büyük, büyük-eşit
db.col.find({ fiyat: { $gt: 1000 } })
db.col.find({ fiyat: { $gte: 1000 } })

// $lt, $lte — küçük, küçük-eşit
db.col.find({ stok: { $lt: 10 } })
db.col.find({ stok: { $lte: 10 } })

// Aralık sorgusu
db.col.find({ fiyat: { $gte: 500, $lte: 5000 } })

// $in — liste içinde
db.col.find({ kategori: { $in: ["elektronik", "bilgisayar"] } })

// $nin — liste dışında
db.col.find({ durum: { $nin: ["pasif", "silindi"] } })
```

### Mantıksal Operatörler

```javascript
// $and — VE koşulu (varsayılan)
db.col.find({
  $and: [
    { kategori: "elektronik" },
    { fiyat: { $lt: 10000 } }
  ]
})
// Kısaltma (aynı alan değilse):
db.col.find({ kategori: "elektronik", fiyat: { $lt: 10000 } })

// $or — VEYA koşulu
db.col.find({
  $or: [
    { kategori: "elektronik" },
    { fiyat: { $lt: 100 } }
  ]
})

// $nor — HİÇBİRİ değil
db.col.find({
  $nor: [
    { durum: "pasif" },
    { stok: 0 }
  ]
})

// $not — değili
db.col.find({ fiyat: { $not: { $gt: 10000 } } })

// Karmaşık kombinasyon
db.col.find({
  $or: [
    { kategori: "elektronik", fiyat: { $lt: 5000 } },
    { kategori: "giyim",      fiyat: { $lt: 500 } }
  ]
})
```

### Element Operatörleri

```javascript
// $exists — alan var mı?
db.col.find({ indirim: { $exists: true } })
db.col.find({ silinme: { $exists: false } })

// $type — alan tipi kontrolü
db.col.find({ fiyat: { $type: "double" } })
db.col.find({ fiyat: { $type: ["int", "double"] } })
// Tipler: "double"(1), "string"(2), "object"(3), "array"(4),
//         "bool"(8), "date"(9), "null"(10), "int"(16), "long"(18)
```

### Array Operatörleri

```javascript
// Array içinde eleman ara
db.col.find({ etiketler: "indirim" })       // Tam eşleşme veya array içinde
db.col.find({ etiketler: { $in: ["indirim", "yeni"] } })  // Birini içersin

// $all — hepsini içermeli
db.col.find({ etiketler: { $all: ["indirim", "elektronik"] } })

// $size — array uzunluğu
db.col.find({ etiketler: { $size: 3 } })

// $elemMatch — array elemanı üzerinde çoklu koşul
db.siparisler.find({
  urunler: {
    $elemMatch: {
      fiyat: { $gt: 100 },
      adet: { $gte: 2 }
    }
  }
})

// Pozisyon bazlı sorgu (dot notation)
db.col.find({ "adresler.0.sehir": "İstanbul" })  // İlk adresin şehri
```

### Embedded Document Sorguları

```javascript
// Alt alana erişim (dot notation)
db.col.find({ "adres.sehir": "İstanbul" })
db.col.find({ "adres.posta_kodu": { $regex: /^34/ } })

// Exact match (sıra önemli!)
db.col.find({ adres: { sehir: "İstanbul", ilce: "Kadıköy" } })
// ❌ Bu { sehir, ilce, posta } şeklindeyse eşleşmez!

// Esnek alt alan sorgusu — dot notation kullan
db.col.find({ "adres.sehir": "İstanbul", "adres.ilce": "Kadıköy" })
```

### Metin ve Regex Sorguları

```javascript
// $regex — düzenli ifade
db.col.find({ ad: { $regex: /laptop/i } })          // Büyük/küçük duyarsız
db.col.find({ ad: { $regex: "^Dell", $options: "i" } })
db.col.find({ email: { $regex: /@gmail\.com$/ } })

// $text — metin indeksi ile tam metin arama
db.col.createIndex({ aciklama: "text", baslik: "text" })
db.col.find({ $text: { $search: "laptop hızlı" } })
db.col.find({ $text: { $search: "\"tam ifade\"" } })  // Tam ifade
db.col.find({ $text: { $search: "laptop -ağır" } })    // laptop ama ağır değil

// Skora göre sırala
db.col.find(
  { $text: { $search: "laptop" } },
  { score: { $meta: "textScore" } }
).sort({ score: { $meta: "textScore" } })
```

### Coğrafi Sorgular

```javascript
// GeoJSON formatında koordinat
db.mekanlar.insertOne({
  ad: "Ofis",
  konum: {
    type: "Point",
    coordinates: [28.9784, 41.0082]  // [longitude, latitude]
  }
})

// 2dsphere indeksi oluştur
db.mekanlar.createIndex({ konum: "2dsphere" })

// Yakın nokta ara (metre cinsinden)
db.mekanlar.find({
  konum: {
    $near: {
      $geometry: { type: "Point", coordinates: [28.9784, 41.0082] },
      $maxDistance: 5000,  // 5 km
      $minDistance: 100    // 100 m
    }
  }
})

// Belirli alan içinde ara
db.mekanlar.find({
  konum: {
    $geoWithin: {
      $centerSphere: [[28.9784, 41.0082], 10 / 6371]  // 10km yarıçap
    }
  }
})
```

### Projeksiyon Detayları

```javascript
// İstenen alanlar (1 = dahil)
db.col.find({}, { ad: 1, fiyat: 1 })         // _id her zaman gelir
db.col.find({}, { ad: 1, fiyat: 1, _id: 0 }) // _id'yi de çıkar

// İstenmeyen alanlar (0 = hariç) — karıştıramazsın
db.col.find({}, { sifre: 0, gizli: 0 })

// Array dilimle
db.col.find({}, { yorumlar: { $slice: 5 } })       // İlk 5
db.col.find({}, { yorumlar: { $slice: -3 } })       // Son 3
db.col.find({}, { yorumlar: { $slice: [10, 5] } })  // 10'dan sonra 5

// $elemMatch ile projeksiyon
db.col.find(
  { "notlar.ders": "Matematik" },
  { "notlar.$": 1 }   // Sadece eşleşen ilk array elemanı
)
```

### Cursor Metotları

```javascript
const cursor = db.urunler.find({ kategori: "elektronik" })

cursor.sort({ fiyat: -1 })
cursor.limit(10)
cursor.skip(20)
cursor.count()          // Deprecated → countDocuments kullan
cursor.explain()        // Sorgu planı
cursor.explain("executionStats")  // Detaylı plan

// Chaining
db.urunler
  .find({ stok: { $gt: 0 } })
  .sort({ fiyat: 1 })
  .skip(0)
  .limit(20)
  .projection({ ad: 1, fiyat: 1, _id: 0 })

// forEach ile döngü
db.urunler.find().forEach(doc => {
  printjson(doc)
})

// toArray
const arr = db.urunler.find().toArray()
```

### Operatör Hızlı Başvuru

```
Karşılaştırma : $eq $ne $gt $gte $lt $lte $in $nin
Mantıksal     : $and $or $nor $not
Element       : $exists $type
Array         : $all $elemMatch $size
Metin         : $regex $text $where
Güncelleme    : $set $unset $inc $mul $push $pull $addToSet $pop
```

---

## 💡 Bağlantılar
- [[MongoDB - CRUD İşlemleri]]
- [[MongoDB - Aggregation Pipeline]]
- [[MongoDB - İndeksler ve Performans]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- mongodb.com/docs/manual/reference/operator/query/
