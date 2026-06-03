---
tarih: 2025-01-01
konu: MongoDB İndeksler, Compound, Text, Geospatial, Explain, Performans
etiket: [mongodb, index, performans, explain, compound-index, text-index]
kaynak:
zorluk: ⭐⭐⭐⭐
---

## 📌 Özet

İndeks olmayan sorgular tüm koleksiyonu tarar (COLLSCAN) — büyük veride yavaştır. Doğru indeks seçimi sorgu süresini milisaniyelerden mikrosaniyelere indirir.

---

## 🧠 Detay

### İndeks Türleri

```javascript
// ─── 1. Single Field Index ─────────────────
db.col.createIndex({ fiyat: 1 })    // Artan
db.col.createIndex({ fiyat: -1 })   // Azalan

// ─── 2. Compound Index ─────────────────────
db.col.createIndex({ kategori: 1, fiyat: -1 })
// Prefix kuralı: Bu indeks şu sorguları karşılar:
// { kategori }
// { kategori, fiyat }
// Ama KARŞILAMAZ: { fiyat } (prefix değil)

// ─── 3. Multikey Index (Array) ─────────────
db.col.createIndex({ etiketler: 1 })  // Array alana otomatik multikey

// ─── 4. Text Index ─────────────────────────
db.col.createIndex({ aciklama: "text" })
db.col.createIndex({ baslik: "text", aciklama: "text" })
db.col.createIndex(
  { baslik: "text", aciklama: "text" },
  { weights: { baslik: 10, aciklama: 3 }, default_language: "turkish" }
)

// ─── 5. 2dsphere (Coğrafi) ─────────────────
db.col.createIndex({ konum: "2dsphere" })

// ─── 6. Hashed Index ───────────────────────
db.col.createIndex({ kullanici_id: "hashed" })  // Sharding için

// ─── 7. Wildcard Index ─────────────────────
db.col.createIndex({ "ozellikler.$**": 1 })  // Tüm alt alanlar
db.col.createIndex({ "$**": 1 })              // Tüm alanlar
```

### İndeks Seçenekleri

```javascript
// Unique indeks
db.col.createIndex({ email: 1 }, { unique: true })

// Sparse indeks (alan yoksa dahil etme)
db.col.createIndex({ indirim: 1 }, { sparse: true })

// TTL indeks — otomatik belge silme
db.col.createIndex(
  { olusturma_tarihi: 1 },
  { expireAfterSeconds: 3600 }  // 1 saat sonra sil
)
db.col.createIndex(
  { silinme_tarihi: 1 },
  { expireAfterSeconds: 0 }  // silinme_tarihi gelince sil
)

// Partial indeks — koşullu indeks
db.col.createIndex(
  { fiyat: 1 },
  { partialFilterExpression: { stok: { $gt: 0 } } }
)
// Sadece stok > 0 olan belgeleri indeksle → daha küçük indeks

// Case-insensitive (büyük/küçük harf duyarsız)
db.col.createIndex(
  { ad: 1 },
  { collation: { locale: "tr", strength: 2 } }
)

// İndeks adı
db.col.createIndex({ fiyat: 1 }, { name: "fiyat_artan_idx" })

// Background build (eski sürümler — artık varsayılan)
db.col.createIndex({ alan: 1 }, { background: true })
```

### İndeks Yönetimi

```javascript
// İndekleri listele
db.col.getIndexes()
db.col.indexStats()

// İndeks sil
db.col.dropIndex("fiyat_1")           // İsme göre
db.col.dropIndex({ fiyat: 1 })        // Spec'e göre
db.col.dropIndexes()                   // Tümünü sil (_id hariç)

// İndeks yeniden oluştur
db.col.reIndex()

// İndeks boyutu
db.col.stats().indexSizes
db.col.totalIndexSize()
```

### explain() — Sorgu Planı Analizi

```javascript
// Temel plan
db.col.find({ fiyat: { $gt: 1000 } }).explain()

// Yürütme istatistikleri
db.col.find({ fiyat: { $gt: 1000 } }).explain("executionStats")

// Tüm planlar
db.col.find({ fiyat: { $gt: 1000 } }).explain("allPlansExecution")

// Aggregation
db.col.aggregate([...]).explain("executionStats")
```

**Kritik alanlar:**
```javascript
{
  queryPlanner: {
    winningPlan: {
      stage: "IXSCAN",    // ✅ Index Scan (iyi)
      // "COLLSCAN"       // ❌ Collection Scan (kötü)
      // "FETCH"          // İndeks + belge çekme
      // "SORT"           // Bellekte sıralama (dikkat!)
      // "PROJECTION"     // Alan filtresi
      indexName: "fiyat_1"
    }
  },
  executionStats: {
    nReturned: 42,          // Dönen belge sayısı
    totalDocsExamined: 42,  // ✅ İncelenen = Dönen (ideal)
    // totalDocsExamined: 50000  // ❌ Çok fazla inceleme
    totalKeysExamined: 42,
    executionTimeMillis: 2  // Çalışma süresi (ms)
  }
}
```

### İndeks Stratejisi — ESR Kuralı

**E**quality → **S**ort → **R**ange

```javascript
// Sorgu: { kategori: "elektronik", fiyat: { $gt: 500 } }
//        sırala: { olusturma: -1 }

// ❌ Yanlış sıra
db.col.createIndex({ fiyat: 1, kategori: 1, olusturma: -1 })

// ✅ ESR kuralına göre
db.col.createIndex({ kategori: 1, olusturma: -1, fiyat: 1 })
// E: kategori (equality)
// S: olusturma (sort)
// R: fiyat (range)
```

### Covered Query — İndeksten Tam Yanıt

```javascript
// İndeks: { kategori: 1, fiyat: 1, ad: 1 }
// Sorgu: sadece indeksteki alanlar
db.col.find(
  { kategori: "elektronik" },
  { fiyat: 1, ad: 1, _id: 0 }  // _id hariç indeksteki alanlar
)
// executionStats.totalDocsExamined = 0 → Belgeye hiç gitmedi!
```

### Profiler — Yavaş Sorgu Tespiti

```javascript
// Profiler'ı aç (0=off, 1=yavaş, 2=hepsi)
db.setProfilingLevel(1, { slowms: 100 })  // 100ms üstü logla
db.setProfilingLevel(2)                    // Tümünü logla

// Profil verilerini gör
db.system.profile.find().sort({ ts: -1 }).limit(10)

// En yavaş sorgular
db.system.profile.find({ millis: { $gt: 100 } })
  .sort({ millis: -1 })

// Profiler kapat
db.setProfilingLevel(0)

// Mevcut durum
db.getProfilingStatus()
```

### Performans İpuçları

```javascript
// 1. İndeks yoksa COLLSCAN uyarı ver
db.col.find({ fiyat: 1000 }).hint({ $natural: 1 }) // zorla COLLSCAN

// 2. İndeks zorla (query optimizer override)
db.col.find({ fiyat: 1000 }).hint({ fiyat: 1 })
db.col.find({ fiyat: 1000 }).hint("fiyat_1")  // isimle

// 3. Sayfalama — skip yerine range kullan (büyük skip yavaş!)
// ❌ Yavaş: skip(10000).limit(10)
// ✅ Hızlı:
const lastId = ObjectId("...") // Önceki sayfanın son _id'si
db.col.find({ _id: { $gt: lastId } }).limit(10)

// 4. Count yerine countDocuments
db.col.countDocuments({ kategori: "elektronik" })
// Değil: db.col.find(...).count()  (deprecated)

// 5. Projeksiyon kullan — gereksiz alan çekme
db.col.find({}, { aciklama: 0, buyuk_alan: 0 })

// 6. Büyük array'lerden kaçın (16MB limit)
// ❌ Tek belgede 1M yorum
// ✅ Yorumları ayrı koleksiyona al

// 7. Write concern ayarla
db.col.insertOne(doc, { writeConcern: { w: 1 } })    // Hızlı
db.col.insertOne(doc, { writeConcern: { w: "majority" } }) // Güvenli
```

### İndeks Boyut ve Verimlilik

```javascript
// Koleksiyon istatistikleri
db.col.stats()
// storageSize, totalIndexSize, avgObjSize

// İndeks hit oranı (cache'ten mi diskten mi?)
db.serverStatus().wiredTiger.cache

// İndeks kullanım istatistikleri
db.col.aggregate([{ $indexStats: {} }])
// → Her indeks için: accesses.ops (kaç kez kullanıldı)
// Hiç kullanılmayan indeksler → sil (write overhead yaratır)
```

---

## 💡 Bağlantılar
- [[MongoDB - Sorgulama ve Operatörler]]
- [[MongoDB - Aggregation Pipeline]]
- [[MongoDB - Şema Tasarımı ve Veri Modelleme]]
- [[MongoDB - Replikasyon ve Sharding]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- mongodb.com/docs/manual/indexes/
- mongodb.com/docs/manual/reference/explain-results/
