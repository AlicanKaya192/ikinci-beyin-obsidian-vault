---
tarih: 2025-01-01
konu: MongoDB Şema Tasarımı, Embedding vs Referencing, Design Patterns
etiket: [mongodb, şema, veri-modelleme, embedding, referencing, design-pattern]
kaynak:
zorluk: ⭐⭐⭐⭐
---

## 📌 Özet

MongoDB'de şema tasarımı uygulama erişim kalıplarına göre yapılır — SQL'deki normalleştirme kuralları doğrudan geçmez. "Birlikte erişilen veriyi birlikte sakla" temel prensibidir.

---

## 🧠 Detay

### Temel Karar: Embed mi, Reference mı?

```
Embed (Göm) Seç:                Reference Seç:
✅ Birlikte her zaman okunuyor   ✅ Bağımsız erişim gerekli
✅ "has-a" ilişkisi (1:1, 1:few) ✅ Çok-çok ilişki (many:many)
✅ Alt belge tek başına anlamsız ✅ Alt veri büyür (16MB limit)
✅ Atomic güncelleme önemli      ✅ Sık güncellenen alt veri
✅ Okuma yoğun uygulama          ✅ Yazma yoğun uygulama
```

### Tasarım Kalıpları (Design Patterns)

#### 1. Embedded Document Pattern
```javascript
// Kullanıcı + adresleri birlikte
{
  "_id": ObjectId(),
  "ad": "Ali Yılmaz",
  "email": "ali@example.com",
  "adresler": [
    { "tip": "ev",  "sehir": "İstanbul", "ilce": "Kadıköy" },
    { "tip": "iş",  "sehir": "İstanbul", "ilce": "Levent" }
  ]
}
// ✅ Tek sorguda tüm adresler
// ❌ Adres sayısı çok artarsa sorun
```

#### 2. Subset Pattern — Büyük Array'den Alt Küme
```javascript
// ❌ Sorun: Her üründe 1000 yorum
{
  "_id": 1,
  "ad": "Laptop",
  "yorumlar": [/* 1000 yorum — belge şişer */]
}

// ✅ Çözüm: Son N yorumu belgede, kalanını ayrı koleksiyonda
{
  "_id": 1,
  "ad": "Laptop",
  "son_yorumlar": [/* Son 10 yorum */]  // Hızlı görüntüleme
}
// Tüm yorumlar için → yorumlar koleksiyonu
```

#### 3. Extended Reference Pattern — Sık Okunan Alanları Kopyala
```javascript
// Sipariş belgesi — tam JOIN yerine kritik alanlar kopyalanır
{
  "_id": ObjectId(),
  "musteri_id": ObjectId("usr1"),
  // Müşteri adı kopyalanır → JOIN gerekmez
  "musteri_ad": "Ali Yılmaz",
  "musteri_email": "ali@example.com",
  "tutar": 1500,
  "urunler": [
    {
      "urun_id": ObjectId("prd1"),
      "urun_ad": "Laptop",     // Kopyalandı
      "adet": 1,
      "birim_fiyat": 1500      // Sipariş anındaki fiyat
    }
  ]
}
// ✅ Tek sorguda tüm sipariş
// ⚠️ Müşteri adı değişirse eskiler güncellenmez (kasıtlı)
```

#### 4. Bucket Pattern — Zaman Serisi / IoT Verisi
```javascript
// ❌ Her ölçüm ayrı belge (milyonlarca belge)
{ "sensor_id": 1, "tarih": ISODate("2024-01-01T10:00:00"), "deger": 22.5 }
{ "sensor_id": 1, "tarih": ISODate("2024-01-01T10:01:00"), "deger": 22.6 }
// ...

// ✅ Saatlik bucket — ölçümler gruplandı
{
  "sensor_id": 1,
  "baslangic": ISODate("2024-01-01T10:00:00"),
  "bitis":     ISODate("2024-01-01T11:00:00"),
  "olcum_sayisi": 60,
  "ozet": { "min": 22.1, "max": 23.5, "ort": 22.8 },
  "olcumler": [
    { "dk": 0,  "deger": 22.5 },
    { "dk": 1,  "deger": 22.6 },
    // ...60 ölçüm
  ]
}
// ✅ 60x daha az belge, daha hızlı aggregation
```

#### 5. Computed Pattern — Hesaplanan Değerleri Sakla
```javascript
// ❌ Her sorguda yorum puanını hesapla
db.urunler.aggregate([{ $group: { _id: "$_id", ort_puan: { $avg: "$yorumlar.puan" } } }])

// ✅ Ortalama puanı belgede sakla, yorum eklenince güncelle
{
  "_id": ObjectId(),
  "ad": "Laptop",
  "yorum_sayisi": 150,
  "ort_puan": 4.7,    // Her yorum eklenince güncelle
  "toplam_puan": 705
}
// Güncelleme:
db.urunler.updateOne(
  { _id: urunId },
  {
    $inc: { yorum_sayisi: 1, toplam_puan: yeniPuan },
    $set: { ort_puan: (mevcutToplam + yeniPuan) / (mevcutSayi + 1) }
  }
)
```

#### 6. Outlier Pattern — Nadir Büyük Durumlar
```javascript
// Çoğu ürün: az yorum
// Bazı ürün: milyonlarca yorum (outlier)
{
  "_id": ObjectId(),
  "ad": "Laptop",
  "yorumlar": [/* İlk 1000 yorum */],
  "cok_yorum_var": true  // Flag
}
// cok_yorum_var ise → yorumlar koleksiyonundan çek
```

#### 7. Schema Versioning Pattern
```javascript
// Şema değişikliklerini yönet
{
  "_id": ObjectId(),
  "schema_version": 2,  // Bu belge hangi versiyonda?
  "ad": "Ali",
  // v2 alanları:
  "tam_ad": { "isim": "Ali", "soyad": "Yılmaz" }
  // v1'de: "ad": "Ali Yılmaz"
}

// Uygulama katmanında versiyon kontrolü
function kullaniciyiOku(doc) {
  if (doc.schema_version === 1) {
    return migrateV1toV2(doc)
  }
  return doc
}
```

### İlişki Tipleri

#### Bire-Bir (1:1)
```javascript
// Seçenek A: Embed (genellikle tercih)
{ "_id": 1, "ad": "Ali", "profil": { "bio": "...", "website": "..." } }

// Seçenek B: Reference (büyük veya nadiren okunuyorsa)
// kullanicilar: { _id: 1, ad: "Ali" }
// profiller:    { kullanici_id: 1, bio: "...", website: "..." }
```

#### Bire-Çok (1:N)
```javascript
// N küçükse → Embed
{ "siparis_id": 1, "kalemler": [{ ... }, { ... }] }

// N büyükse → Reference (N tarafında parent ID)
// siparisler:  { _id: 1, musteri_id: 100 }
// yorumlar:    { _id: 10, urun_id: 1, metin: "..." }
```

#### Çoka-Çok (M:N)
```javascript
// Öğrenci ↔ Ders

// Seçenek A: Her iki tarafta referans dizisi
// ogrenciler: { _id: 1, dersler: [101, 102, 103] }
// dersler:    { _id: 101, ogrenciler: [1, 2, 3] }

// Seçenek B: Ara koleksiyon (SQL gibi)
// kayitlar: { _id: 1, ogrenci_id: 1, ders_id: 101, tarih: ... }
```

### MongoDB Schema Validation

```javascript
// Şema doğrulaması ekle
db.createCollection("kullanicilar", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["ad", "email", "yas"],
      properties: {
        ad: {
          bsonType: "string",
          minLength: 2,
          maxLength: 100,
          description: "Zorunlu string, 2-100 karakter"
        },
        email: {
          bsonType: "string",
          pattern: "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$",
          description: "Geçerli email formatı"
        },
        yas: {
          bsonType: "int",
          minimum: 0,
          maximum: 150
        },
        rol: {
          enum: ["admin", "kullanici", "moderator"],
          description: "Geçerli roller"
        }
      }
    }
  },
  validationAction: "error",   // "warn" — sadece uyar
  validationLevel: "strict"    // "moderate" — sadece insert
})

// Mevcut koleksiyona doğrulama ekle
db.runCommand({
  collMod: "kullanicilar",
  validator: { $jsonSchema: { ... } }
})
```

### Koleksiyon Tasarım Kontrol Listesi

```
Tasarım Öncesi Sorular:
□ Uygulama nasıl veri okuyacak? (Erişim kalıpları)
□ Yazma / okuma oranı nedir?
□ Veri ne kadar büyüyecek?
□ Hangi sorgular en sık çalışacak?
□ Latency mi throughput mu önemli?
□ Atomicity (tek belge güncellemesi) gerekli mi?

Karar Kontrol:
□ İlişki 1:1 veya 1:few → Embed düşün
□ İlişki 1:many (büyük) → Reference kullan
□ 16MB belge sınırına dikkat
□ Array sınırsız büyüyebilir mi? → Bucket/Subset
□ Sık hesaplanan değer var mı? → Computed pattern
□ Şema değişebilir mi? → Versioning
```

---

## 💡 Bağlantılar
- [[MongoDB - Giriş ve Temel Kavramlar]]
- [[MongoDB - CRUD İşlemleri]]
- [[MongoDB - Aggregation Pipeline]]
- [[MongoDB - Replikasyon ve Sharding]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- mongodb.com/docs/manual/data-modeling/
- mongodb.com/blog/post/building-with-patterns
