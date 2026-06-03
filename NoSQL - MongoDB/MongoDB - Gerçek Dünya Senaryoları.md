---
tarih: 2025-01-01
konu: MongoDB Gerçek Dünya, E-Ticaret, Blog, IoT Örnekleri, Anti-Pattern
etiket: [mongodb, senaryo, e-ticaret, blog, iot, best-practices, anti-pattern]
kaynak:
zorluk: ⭐⭐⭐
---

## 📌 Özet

Gerçek dünya MongoDB şema örnekleri: E-Ticaret (ürün kataloğu + sipariş), Blog sistemi ve IoT sensör verisi. Yaygın anti-pattern'ler ve çözümleri.

---

## 🧠 Detay

### Senaryo 1: E-Ticaret Şeması

```javascript
// ─── Ürün Kataloğu ───────────────────────
// Her ürün farklı özelliklere sahip → Esnek şema mükemmel!
{
  "_id": ObjectId(),
  "sku": "DELL-XPS-13-001",
  "ad": "Dell XPS 13 Laptop",
  "slug": "dell-xps-13",
  "marka": "Dell",
  "kategori": ["elektronik", "bilgisayar", "laptop"],
  "fiyat": {
    "liste": 35000,
    "satis": 29999,
    "para_birimi": "TRY"
  },
  "stok": {
    "adet": 45,
    "rezerve": 3,
    "minimum_uyari": 5
  },
  "ozellikler": {           // Kategoriye göre değişen alanlar
    "islemci": "Intel i7-1250U",
    "ram": "16GB LPDDR5",
    "depolama": "512GB NVMe SSD",
    "ekran": "13.4 FHD+",
    "batarya": "13 saat"
  },
  "resimler": [
    { "url": "s3://bucket/xps13-1.jpg", "sira": 1, "tip": "ana" },
    { "url": "s3://bucket/xps13-2.jpg", "sira": 2, "tip": "yan" }
  ],
  "ozet_yorumlar": {        // Computed pattern
    "toplam": 127,
    "ortalama": 4.6,
    "dagilim": { "5": 89, "4": 28, "3": 7, "2": 2, "1": 1 }
  },
  "aktif": true,
  "olusturma": ISODate("2024-01-01"),
  "guncelleme": ISODate("2024-06-15")
}

// ─── Sipariş ─────────────────────────────
// Extended Reference Pattern: kritik müşteri bilgileri kopyalanır
{
  "_id": ObjectId(),
  "siparis_no": "ORD-2024-001234",
  "musteri": {
    "_id": ObjectId("usr1"),          // Referans
    "ad": "Ali Yılmaz",               // Kopyalandı — müşteri değişse bile sipariş doğru kalır
    "email": "ali@example.com",
    "telefon": "+90 555 123 4567"
  },
  "kalemler": [
    {
      "urun_id": ObjectId("prd1"),
      "sku": "DELL-XPS-13-001",       // Kopyalandı
      "ad": "Dell XPS 13 Laptop",     // Kopyalandı
      "adet": 1,
      "birim_fiyat": 29999,           // Sipariş anındaki fiyat!
      "toplam": 29999
    }
  ],
  "teslimat_adresi": {
    "isim": "Ali Yılmaz",
    "adres": "Kadıköy Mah. ...",
    "sehir": "İstanbul",
    "posta_kodu": "34710"
  },
  "odeme": {
    "yontem": "kredi_karti",
    "son_4_hane": "4242",
    "durum": "onaylandi"
  },
  "tutarlar": {
    "ara_toplam": 29999,
    "kargo": 0,
    "vergi": 5400,
    "toplam": 35399
  },
  "durum": "kargoda",
  "durum_gecmisi": [                  // Tam geçmiş
    { "durum": "beklemede", "tarih": ISODate("2024-06-15T10:00:00Z") },
    { "durum": "onaylandi",  "tarih": ISODate("2024-06-15T10:05:00Z") },
    { "durum": "kargoda",    "tarih": ISODate("2024-06-15T14:30:00Z") }
  ],
  "kargo_no": "YK-123456789",
  "olusturma": ISODate("2024-06-15T10:00:00Z")
}
```

### Senaryo 2: Blog / İçerik Sistemi

```javascript
// ─── Yazı (Post) ─────────────────────────
{
  "_id": ObjectId(),
  "baslik": "Docker ile MongoDB Çalıştırma",
  "slug": "docker-ile-mongodb",
  "icerik": "...",
  "ozet": "Docker Compose ile ...",
  "yazar": {
    "_id": ObjectId("usr1"),
    "ad": "Ahmet Demir",
    "profil_foto": "s3://..."
  },
  "etiketler": ["docker", "mongodb", "devops"],
  "kategori": "backend",
  "durum": "yayinlandi",   // taslak | incelemede | yayinlandi
  "son_yorumlar": [        // Subset Pattern: sadece son 5 yorum
    { "_id": ObjectId(), "yazar": "Ali", "metin": "Harika!", "tarih": ISODate() }
  ],
  "istatistikler": {       // Computed Pattern
    "yorum_sayisi": 24,
    "gorunum": 1850,
    "begeni": 67
  },
  "seo": {
    "meta_title": "Docker MongoDB Kurulum Rehberi",
    "meta_desc": "...",
    "og_image": "s3://..."
  },
  "yayinlama_tarihi": ISODate("2024-06-01"),
  "guncelleme": ISODate("2024-06-10")
}

// Tüm yorumlar ayrı koleksiyonda (Referencing)
// yorumlar: { yazi_id, yazar, metin, begeni, tarih }
// → Yorum eklenince hem yorumlar koleksiyonuna hem de
//   yazıdaki son_yorumlar dizisine güncelle
```

### Senaryo 3: IoT / Sensör Verisi — Bucket Pattern

```javascript
// ─── Her ölçüm ayrı belge (Anti-Pattern!) ────
// { sensor_id: "s1", zaman: ISODate(), sicaklik: 22.5 }
// → Milyonlarca küçük belge, yavaş aggregation

// ─── Bucket Pattern (Doğru) ──────────────────
// Her bucket = 1 sensörün 1 saatlik verisi
{
  "_id": ObjectId(),
  "sensor_id": "s1",
  "konum": { "bina": "A", "kat": 3, "oda": "301" },
  "periyot": {
    "baslangic": ISODate("2024-06-15T10:00:00Z"),
    "bitis":     ISODate("2024-06-15T11:00:00Z")
  },
  "olcum_sayisi": 60,
  "ozet": {
    "min_sicaklik": 21.2,
    "max_sicaklik": 23.8,
    "ort_sicaklik": 22.5,
    "min_nem": 45,
    "max_nem": 52
  },
  "olcumler": [            // 60 adet (her dakika)
    { "dk": 0,  "sicaklik": 22.1, "nem": 48, "co2": 412 },
    { "dk": 1,  "sicaklik": 22.3, "nem": 47, "co2": 415 },
    // ... 60 ölçüm
  ],
  "anormallik": false      // Alarm var mı?
}

// Bucket'a yeni ölçüm ekleme (Upsert ile)
db.sensor_verileri.updateOne(
  {
    sensor_id: "s1",
    "periyot.baslangic": ISODate("2024-06-15T10:00:00Z"),
    olcum_sayisi: { $lt: 60 }
  },
  {
    $push: { olcumler: { dk: 15, sicaklik: 22.4, nem: 47, co2: 410 } },
    $inc:  { olcum_sayisi: 1 },
    $min:  { "ozet.min_sicaklik": 22.4 },
    $max:  { "ozet.max_sicaklik": 22.4 }
  },
  { upsert: true }
)
```

### Yaygın Anti-Pattern'ler

```javascript
// ─── Anti-Pattern 1: Sınırsız büyüyen array ──
// ❌ Blog yazısında tüm yorumlar
{ _id: 1, baslik: "...", yorumlar: [/* Binlerce yorum */] }
// Sorun: 16MB limit, her okuma tüm yorumları çeker
// ✅ Yorumları ayrı koleksiyona al, son 5'ini belgede tut

// ─── Anti-Pattern 2: Her şeyi embed ──────────
// ❌ Siparişteki tam ürün belgesi
{ siparis_id: 1, urun: { /* 50 alan tam ürün belgesi */ } }
// ✅ Sadece kritik alanları kopyala (ad, sku, fiyat)

// ─── Anti-Pattern 3: Birçok koleksiyon ───────
// ❌ Kategori başına koleksiyon
// laptoplar, telefonlar, tabletler koleksiyonları
// ✅ Tek urunler koleksiyonu + kategori alanı
// → İndeks basit, sorgu basit

// ─── Anti-Pattern 4: Büyük belge sayısı takibi
// ❌ Aggregation ile her sorgu
// ✅ Computed pattern → sayıyı belgede sakla

// ─── Anti-Pattern 5: Monoton shard key ───────
// ❌ { _id: 1 } shard key → son shard'a yığılır
// ✅ { _id: "hashed" } veya compound key

// ─── Anti-Pattern 6: SQL düşüncesi ───────────
// ❌ Her veriyi normalize et, JOIN yap
// ✅ Erişim kalıplarına göre tasarla, embedding düşün
```

### Best Practices Özeti

```javascript
// 1. Erişim kalıplarını önce belirle
// "Hangi sorgular sık çalışacak?" → Şemayı buna göre tasarla

// 2. İndeks stratejisi
db.col.createIndex({ kategori: 1, fiyat: -1 })  // ESR kuralı
db.col.createIndex({ ad: "text" }, { default_language: "turkish" })

// 3. Validation şeması ekle (production için şart)
db.createCollection("urunler", {
  validator: { $jsonSchema: {
    required: ["sku", "ad", "fiyat"],
    properties: {
      fiyat: { bsonType: "number", minimum: 0 }
    }
  }}
})

// 4. Büyük okuma için projeksiyon kullan
db.urunler.find({}, { aciklama: 0, buyuk_alan: 0 })  // İstenmeyen alanları çıkar

// 5. Write concern dengesini kur
// Veri kritikliğine göre: w:1 (hızlı) vs w:"majority" (güvenli)

// 6. Connection pool doğru ayarla
const client = new MongoClient(uri, {
  maxPoolSize: 50,
  minPoolSize: 5
})

// 7. Düzenli indeks bakımı
db.col.aggregate([{ $indexStats: {} }])
// Hiç kullanılmayan indeksi sil (write overhead azaltır)
```

---

## 💡 Bağlantılar
- [[MongoDB - Şema Tasarımı ve Veri Modelleme]]
- [[MongoDB - İndeksler ve Performans]]
- [[MongoDB - Aggregation Pipeline]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- mongodb.com/blog/post/building-with-patterns
- mongodb.com/developer/products/mongodb/schema-design-anti-pattern/
