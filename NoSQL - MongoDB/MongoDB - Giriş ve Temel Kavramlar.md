---
tarih: 2025-01-01
konu: MongoDB Giriş, NoSQL vs SQL, Temel Kavramlar, BSON
etiket: [mongodb, nosql, döküman, bson, koleksiyon, temel]
kaynak:
zorluk: ⭐
---

## 📌 Özet

MongoDB, JSON benzeri belgeler (document) halinde veri saklayan NoSQL veritabanıdır. Esnek şema, yatay ölçeklenebilirlik ve hızlı geliştirme döngüsü temel avantajlarıdır.

---

## 🧠 Detay

### SQL vs MongoDB Karşılaştırması

| SQL | MongoDB | Açıklama |
|---|---|---|
| Database | Database | Veritabanı |
| Table | Collection | Veri grubu |
| Row / Record | Document | Tek veri birimi |
| Column | Field | Veri alanı |
| Primary Key | `_id` | Benzersiz tanımlayıcı |
| JOIN | $lookup / Embedding | İlişki yönetimi |
| INDEX | Index | Arama hızlandırıcı |
| Schema | Flexible Schema | Yapı tanımı |

### NoSQL Ne Zaman Kullanılır?

```
✅ MongoDB için ideal:
- Değişken yapıda veriler (farklı ürün özellikleri)
- İç içe (nested) veri yapıları (blog: yazar + yorumlar + etiketler)
- Hızlı prototipleme ve iterasyon
- Yatay ölçeklenme gereksinimi (sharding)
- Gerçek zamanlı analitik
- Coğrafi veri (geospatial)

❌ SQL daha iyi:
- Karmaşık JOIN'ler ve ilişkisel bütünlük şartı
- Güçlü ACID garantisi gereken finansal işlemler
- Sabit ve iyi tanımlı şema
- Raporlama ve BI araçları entegrasyonu
```

### BSON (Binary JSON)

MongoDB verisi aslında BSON (Binary JSON) formatında saklanır.

**JSON'a ek BSON türleri:**

```javascript
{
  // String
  isim: "Ahmet",

  // Number (Int32, Int64, Double, Decimal128)
  yas: 30,
  maas: NumberDecimal("15000.50"),
  buyuk_sayi: NumberLong("9007199254740993"),

  // Boolean
  aktif: true,

  // Date
  kayit: new Date("2024-01-15"),
  kayit_ts: ISODate("2024-01-15T10:30:00Z"),

  // Array
  hobiler: ["kitap", "yüzme", "kod"],

  // Embedded Document
  adres: { il: "İstanbul", ilce: "Kadıköy" },

  // ObjectId (_id için)
  _id: ObjectId("507f1f77bcf86cd799439011"),

  // Null
  silinme_tarihi: null,

  // Binary
  fotograf: BinData(0, "...base64..."),

  // Regular Expression
  pattern: /^ahmet/i,

  // Timestamp (internal)
  ts: Timestamp(1704067200, 1)
}
```

### MongoDB Mimarisi

```
MongoDB Cluster
├── mongos (Query Router)         ← İstemci buraya bağlanır
│   ├── Config Servers (3x)      ← Metadata: hangi veri nerede
│   └── Shard 1: Replica Set     ← Veri parçası 1
│       ├── Primary              ← Yazma işlemi
│       ├── Secondary            ← Okuma + Failover
│       └── Secondary            ← Okuma + Failover
│   └── Shard 2: Replica Set     ← Veri parçası 2
│       ├── Primary
│       └── Secondary x2
```

### Temel Kavramlar

#### Document (Belge)
```javascript
// Tek bir kayıt
{
  "_id": ObjectId("64f1a2b3c4d5e6f7a8b9c0d1"),
  "ad": "Laptop",
  "fiyat": 25000,
  "stok": 150,
  "ozellikler": {
    "marka": "Dell",
    "ram": "16GB",
    "islemci": "Intel i7"
  },
  "etiketler": ["elektronik", "bilgisayar", "iş"],
  "olusturma": ISODate("2024-01-15T09:00:00Z")
}
```

#### Collection (Koleksiyon)
Aynı türde belgelerin grubu. SQL'deki tablo gibi ama **esnek şema** — her belge farklı alanlara sahip olabilir.

#### ObjectId
```javascript
ObjectId("507f1f77bcf86cd799439011")
// └─ 4 byte: timestamp
//    └─ 5 byte: random (machine + process)
//       └─ 3 byte: counter

// Zaman bilgisi içerir:
ObjectId("507f1f77bcf86cd799439011").getTimestamp()
// ISODate("2012-10-17T20:46:31Z")
```

### Kurulum ve Bağlantı

```bash
# Docker ile hızlı başlat
docker run -d \
  --name mongodb \
  -p 27017:27017 \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=password \
  -v mongo_data:/data/db \
  mongo:7.0

# mongosh ile bağlan
mongosh "mongodb://admin:password@localhost:27017"

# URI formatı
mongodb://[username:password@]host[:port][/database][?options]
mongodb+srv://user:pass@cluster.mongodb.net/mydb  # Atlas (SRV)
```

### İlk Adımlar (mongosh)

```javascript
// Veritabanı seç / oluştur (ilk yazma ile oluşur)
use mydb

// Mevcut veritabanları
show dbs

// Mevcut koleksiyonlar
show collections

// Veritabanı bilgisi
db.stats()

// Yardım
db.help()
db.collection.help()
```

### Veri Modelleme Stratejileri

#### 1. Embedding (Gömme) — Birlikte kullanılan veriler
```javascript
// Blog yazısı + yorumlar aynı belgede
{
  "_id": ObjectId(),
  "baslik": "Docker Nedir?",
  "icerik": "...",
  "yorumlar": [
    { "yazar": "Ali", "metin": "Harika yazı!", "tarih": ISODate() },
    { "yazar": "Veli", "metin": "Teşekkürler", "tarih": ISODate() }
  ]
}
// ✅ Tek sorguda tüm veri
// ❌ Belge büyürse (16MB limit) sorun
```

#### 2. Referencing (Referans) — Ayrı koleksiyonlar
```javascript
// Kullanıcı belgesi
{ "_id": ObjectId("usr1"), "ad": "Ali" }

// Sipariş belgesi
{ "_id": ObjectId(), "musteri_id": ObjectId("usr1"), "tutar": 500 }

// ✅ Esneklik, tekrar yok
// ❌ $lookup (JOIN) gerekir
```

---

## 💡 Bağlantılar
- [[MongoDB - CRUD İşlemleri]]
- [[MongoDB - Sorgulama ve Operatörler]]
- [[MongoDB - Aggregation Pipeline]]
- [[MongoDB - İndeksler ve Performans]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- mongodb.com/docs/manual/
- mongodb.com/try/download/community
