---
tarih: 2025-01-01
konu: MongoDB Aggregation Pipeline, $match, $group, $lookup, $unwind
etiket: [mongodb, aggregation, pipeline, group, lookup, unwind, project]
kaynak:
zorluk: ⭐⭐⭐⭐
---

## 📌 Özet

Aggregation Pipeline, MongoDB'nin en güçlü özelliğidir. Veriler bir pipeline'dan geçerek dönüşür, gruplandırılır, filtrelenir ve birleştirilir. SQL'deki GROUP BY + JOIN + HAVING kombinasyonunun karşılığıdır.

---

## 🧠 Detay

### Pipeline Konsepti

```javascript
db.koleksiyon.aggregate([
  { $aşama1: { ... } },  // Belgeler aşamadan geçer
  { $aşama2: { ... } },  // Her aşama öncekinin çıktısını alır
  { $aşama3: { ... } }   // Son aşama sonuç döner
])
```

```
Koleksiyon
    │
    ▼
[$match]   → Filtrele (index kullanır — ilk koy!)
    │
    ▼
[$group]   → Grupla ve hesapla
    │
    ▼
[$sort]    → Sırala
    │
    ▼
[$limit]   → Sınırla
    │
    ▼
Sonuç
```

---

### Temel Aşamalar

#### $match — Filtreleme
```javascript
// find() gibi, ama pipeline içinde
{ $match: { kategori: "elektronik", fiyat: { $gt: 1000 } } }

// ⭐ Pipeline'ın başına koy → index kullanır, veri azaltır
```

#### $project — Alan Seçimi / Hesaplama
```javascript
{ $project: {
  ad: 1,
  fiyat: 1,
  _id: 0,
  // Yeni alan hesapla
  kdvli_fiyat: { $multiply: ["$fiyat", 1.18] },
  // String birleştir
  tam_ad: { $concat: ["$ad", " - ", "$marka"] },
  // Koşullu
  indirimli: { $cond: { if: { $gt: ["$indirim", 0] }, then: true, else: false } }
}}
```

#### $group — Gruplama ve Aggregation
```javascript
{ $group: {
  _id: "$kategori",           // Neye göre grupla (_id zorunlu)
  toplam_urun: { $sum: 1 },   // Sayım
  toplam_stok: { $sum: "$stok" },
  ort_fiyat:   { $avg: "$fiyat" },
  max_fiyat:   { $max: "$fiyat" },
  min_fiyat:   { $min: "$fiyat" },
  urunler:     { $push: "$ad" },        // Array'e ekle
  benzersiz:   { $addToSet: "$marka" }  // Tekrarsız array
}}

// Tek grup (tüm koleksiyon)
{ $group: { _id: null, toplam: { $sum: "$fiyat" } } }

// Çoklu alan ile grup
{ $group: {
  _id: { kategori: "$kategori", marka: "$marka" },
  adet: { $sum: 1 }
}}
```

#### $sort — Sıralama
```javascript
{ $sort: { ort_fiyat: -1, ad: 1 } }  // -1: azalan, 1: artan
```

#### $limit ve $skip
```javascript
{ $limit: 10 }
{ $skip: 20 }
```

#### $unwind — Array'i Düzleştir
```javascript
// Belge: { ad: "Laptop", etiketler: ["a", "b", "c"] }
{ $unwind: "$etiketler" }
// → 3 ayrı belge: { ad: "Laptop", etiketler: "a" }
//                 { ad: "Laptop", etiketler: "b" }
//                 { ad: "Laptop", etiketler: "c" }

// Boş array'ı koru
{ $unwind: { path: "$etiketler", preserveNullAndEmptyArrays: true } }

// İndeks ekle
{ $unwind: { path: "$etiketler", includeArrayIndex: "idx" } }
```

---

### $lookup — JOIN İşlemi

```javascript
// Siparişlere müşteri bilgisi ekle
{ $lookup: {
  from: "musteriler",          // Birleştirilecek koleksiyon
  localField: "musteri_id",    // Bu koleksiyondaki alan
  foreignField: "_id",         // Dış koleksiyondaki alan
  as: "musteri_bilgisi"        // Sonuç array adı
}}
// musteri_bilgisi: [{ _id, ad, email, ... }]

// Sonucu düzleştir (tek belge ise)
{ $unwind: "$musteri_bilgisi" }

// Pipeline ile gelişmiş lookup
{ $lookup: {
  from: "urunler",
  let: { siparis_id: "$_id", min_tutar: "$min_tutar" },
  pipeline: [
    { $match: { $expr: { $and: [
      { $eq: ["$$siparis_id", "$siparis_id"] },
      { $gt: ["$fiyat", "$$min_tutar"] }
    ]}}},
    { $project: { ad: 1, fiyat: 1 } }
  ],
  as: "pahalı_urunler"
}}
```

---

### $addFields ve $set

```javascript
// Mevcut belgeye yeni alan ekle (project'ten farkı: tüm alanlar korunur)
{ $addFields: {
  kdvli_fiyat: { $multiply: ["$fiyat", 1.18] },
  yil: { $year: "$olusturma" }
}}

// $set — $addFields'in takma adı (MongoDB 4.2+)
{ $set: { toplam: { $sum: "$urunler.fiyat" } } }

// $unset — alan kaldır
{ $unset: ["gizli_alan", "gecici"] }
```

---

### $facet — Çok Yönlü Aggregation

```javascript
// Tek sorguda birden fazla pipeline
{ $facet: {
  kategoriler: [
    { $group: { _id: "$kategori", adet: { $sum: 1 } } }
  ],
  fiyat_dagilimi: [
    { $bucket: {
      groupBy: "$fiyat",
      boundaries: [0, 100, 500, 1000, 5000, 100000],
      default: "Diğer",
      output: { adet: { $sum: 1 }, ort: { $avg: "$fiyat" } }
    }}
  ],
  en_pahali: [
    { $sort: { fiyat: -1 } },
    { $limit: 5 },
    { $project: { ad: 1, fiyat: 1, _id: 0 } }
  ]
}}
```

---

### $bucket ve $bucketAuto

```javascript
// Manuel sınırlı gruplama
{ $bucket: {
  groupBy: "$yas",
  boundaries: [0, 18, 30, 45, 60, 100],
  default: "Diğer",
  output: {
    adet: { $sum: 1 },
    ortalama_maas: { $avg: "$maas" }
  }
}}

// Otomatik gruplama
{ $bucketAuto: {
  groupBy: "$fiyat",
  buckets: 5  // 5 eşit grup
}}
```

---

### İfade Operatörleri

```javascript
// Aritmetik
$add, $subtract, $multiply, $divide, $mod, $abs, $ceil, $floor, $round

// String
$concat, $toLower, $toUpper, $trim, $substr, $split, $strLenCP, $regexMatch

// Tarih
$year, $month, $dayOfMonth, $hour, $minute, $second,
$dayOfWeek, $dayOfYear, $week, $dateToString, $dateDiff

// Dizi
$size, $slice, $first, $last, $arrayElemAt, $filter, $map, $reduce,
$zip, $range, $indexOfArray, $setUnion, $setIntersection

// Koşullu
$cond, $ifNull, $switch

// Tip dönüşüm
$toInt, $toDouble, $toString, $toDate, $toBool, $convert

// Karşılaştırma ($expr içinde)
$eq, $ne, $gt, $gte, $lt, $lte, $cmp
```

---

### Gerçek Dünya Örnekleri

```javascript
// Örnek 1: Aylık satış raporu
db.siparisler.aggregate([
  { $match: { durum: "tamamlandı", tarih: { $gte: ISODate("2024-01-01") } } },
  { $group: {
    _id: {
      yil: { $year: "$tarih" },
      ay:  { $month: "$tarih" }
    },
    siparis_sayisi: { $sum: 1 },
    toplam_gelir:   { $sum: "$tutar" },
    ort_siparis:    { $avg: "$tutar" }
  }},
  { $sort: { "_id.yil": 1, "_id.ay": 1 } },
  { $project: {
    _id: 0,
    donem: { $dateToString: {
      format: "%Y-%m",
      date: { $dateFromParts: { year: "$_id.yil", month: "$_id.ay", day: 1 } }
    }},
    siparis_sayisi: 1,
    toplam_gelir:   { $round: ["$toplam_gelir", 2] }
  }}
])

// Örnek 2: En çok satan ürünler (JOIN ile)
db.siparis_kalemleri.aggregate([
  { $group: { _id: "$urun_id", toplam_satis: { $sum: "$adet" } } },
  { $sort: { toplam_satis: -1 } },
  { $limit: 10 },
  { $lookup: {
    from: "urunler",
    localField: "_id",
    foreignField: "_id",
    as: "urun"
  }},
  { $unwind: "$urun" },
  { $project: { ad: "$urun.ad", toplam_satis: 1, _id: 0 } }
])

// Örnek 3: Kullanıcı aktivite özeti
db.etkinlikler.aggregate([
  { $match: { tarih: { $gte: ISODate("2024-01-01") } } },
  { $group: {
    _id: "$kullanici_id",
    etkinlik_sayisi: { $sum: 1 },
    son_etkinlik:    { $max: "$tarih" },
    tipler:          { $addToSet: "$tip" }
  }},
  { $lookup: { from: "kullanicilar", localField: "_id", foreignField: "_id", as: "kullanici" } },
  { $unwind: "$kullanici" },
  { $project: {
    _id: 0,
    kullanici_adi: "$kullanici.ad",
    etkinlik_sayisi: 1,
    son_etkinlik: 1,
    etkinlik_tipi_sayisi: { $size: "$tipler" }
  }},
  { $sort: { etkinlik_sayisi: -1 } }
])
```

---

### Performans İpuçları

```javascript
// ✅ $match ve $sort → pipeline başına al
// ✅ $match'ta indexed alanlar kullan
// ✅ $project ile erken alan azalt
// ✅ allowDiskUse: true → büyük veri
db.col.aggregate([...], { allowDiskUse: true })

// ✅ $limit → mümkün olduğunca erken
// ❌ $group'tan önce $sort (gereksiz yük)
// ❌ Büyük $lookup'ları pipeline sonuna koyma
```

---

## 💡 Bağlantılar
- [[MongoDB - CRUD İşlemleri]]
- [[MongoDB - İndeksler ve Performans]]
- [[MongoDB - Şema Tasarımı ve Veri Modelleme]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- mongodb.com/docs/manual/aggregation/
- mongodb.com/docs/manual/reference/operator/aggregation/
