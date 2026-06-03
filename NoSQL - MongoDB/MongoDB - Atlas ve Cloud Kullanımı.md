---
tarih: 2025-01-01
konu: MongoDB Atlas, Cloud, Atlas Search, Data API, Serverless
etiket: [mongodb, atlas, cloud, atlas-search, data-api, serverless]
kaynak:
zorluk: ⭐⭐⭐
---

## 📌 Özet

MongoDB Atlas, MongoDB'nin tam yönetilen bulut servisidir. Otomatik ölçeklendirme, yedekleme, Atlas Search (Lucene tabanlı) ve Data API gibi özellikler sunar.

---

## 🧠 Detay

### Atlas Tier'ları

| Tier | RAM | Depolama | Kullanım |
|---|---|---|---|
| **M0** | 512MB | 512MB | Ücretsiz, geliştirme |
| **M2/M5** | 1-2GB | 2-5GB | Küçük uygulama |
| **M10+** | 2GB+ | Esneklik | Production |
| **Serverless** | Otomatik | Kullanım başına | Değişken yük |
| **Dedicated** | 8GB+ | TB+ | Kurumsal |

### Atlas CLI

```bash
# Kurulum
brew install mongodb-atlas-cli  # Mac
# veya: npm install -g @mongodb-js/atlas-cli

# Giriş
atlas auth login

# Organizasyon listesi
atlas organizations list

# Proje oluştur
atlas projects create myProject

# Cluster oluştur
atlas clusters create myCluster \
  --provider AWS \
  --region EU_WEST_1 \
  --tier M10 \
  --mdbVersion 7.0

# Cluster listele
atlas clusters list

# Bağlantı bilgisi al
atlas clusters connectionStrings describe myCluster

# Cluster sil
atlas clusters delete myCluster --force

# Kullanıcı oluştur
atlas dbusers create \
  --username appUser \
  --password "şifre!" \
  --role readWriteAnyDatabase

# IP erişim listesi
atlas accessLists create $(curl -s ifconfig.me)/32 --type ipAddress
atlas accessLists create 0.0.0.0/0 --type ipAddress  # Tüm IP (dikkat!)
```

### Atlas Bağlantısı (Python)

```python
from pymongo import MongoClient
from pymongo.server_api import ServerApi

# SRV bağlantı string (Atlas panelinden kopyala)
uri = "mongodb+srv://user:pass@cluster0.abc123.mongodb.net/"

client = MongoClient(
    uri,
    server_api=ServerApi("1"),
    tls=True,
    tlsAllowInvalidCertificates=False
)

# Bağlantı testi
try:
    client.admin.command("ping")
    print("Atlas'a bağlanıldı!")
except Exception as e:
    print(f"Bağlantı hatası: {e}")

db = client.mydb
```

### Atlas Search (Lucene Tabanlı Full-Text)

```javascript
// Search indeksi (Atlas UI veya CLI'dan oluşturulur)
// Mapping örneği:
{
  "mappings": {
    "dynamic": true,
    "fields": {
      "baslik": {
        "type": "string",
        "analyzer": "lucene.turkish"
      },
      "icerik": {
        "type": "string",
        "analyzer": "lucene.turkish"
      },
      "fiyat": {
        "type": "number"
      },
      "tarih": {
        "type": "date"
      }
    }
  }
}
```

```javascript
// Atlas Search sorguları ($search aşaması)

// Basit metin arama
db.urunler.aggregate([
  {
    $search: {
      index: "default",
      text: {
        query: "laptop hızlı",
        path: ["baslik", "aciklama"],
        fuzzy: { maxEdits: 1 }  // Yazım toleransı
      }
    }
  },
  {
    $project: {
      ad: 1,
      fiyat: 1,
      score: { $meta: "searchScore" }  // Alaka skoru
    }
  },
  { $sort: { score: { $meta: "searchScore" } } }
])

// Otomatik tamamlama
db.urunler.aggregate([
  {
    $search: {
      index: "autocomplete_idx",
      autocomplete: {
        query: "lap",
        path: "ad",
        fuzzy: { maxEdits: 1 }
      }
    }
  },
  { $limit: 5 },
  { $project: { ad: 1 } }
])

// Bileşik sorgu (metin + filtre)
db.urunler.aggregate([
  {
    $search: {
      compound: {
        must: [{
          text: {
            query: "laptop",
            path: "baslik"
          }
        }],
        filter: [{
          range: {
            path: "fiyat",
            gte: 1000,
            lte: 30000
          }
        }],
        should: [{
          text: {
            query: "Dell",
            path: "marka",
            score: { boost: { value: 2 } }
          }
        }]
      }
    }
  }
])

// searchMeta — faceted search
db.urunler.aggregate([
  {
    $searchMeta: {
      facet: {
        operator: {
          text: { query: "laptop", path: "baslik" }
        },
        facets: {
          kategoriFacet: {
            type: "string",
            path: "kategori",
            numBuckets: 10
          },
          fiyatFacet: {
            type: "number",
            path: "fiyat",
            boundaries: [0, 500, 1000, 5000, 50000]
          }
        }
      }
    }
  }
])
```

### Atlas Data API

```bash
# HTTP üzerinden MongoDB sorgulama (serverless uygulamalar için)
# Atlas panelinden Data API aktifleştir

# Find
curl -X POST "https://data.mongodb-api.com/app/<app-id>/endpoint/data/v1/action/find" \
  -H "Content-Type: application/json" \
  -H "api-key: <api-key>" \
  -d '{
    "dataSource": "myCluster",
    "database": "mydb",
    "collection": "urunler",
    "filter": { "kategori": "elektronik" },
    "limit": 10
  }'

# InsertOne
curl -X POST ".../action/insertOne" \
  -H "api-key: <api-key>" \
  -d '{
    "dataSource": "myCluster",
    "database": "mydb",
    "collection": "urunler",
    "document": { "ad": "Laptop", "fiyat": 25000 }
  }'

# UpdateOne
curl -X POST ".../action/updateOne" \
  -H "api-key: <api-key>" \
  -d '{
    "dataSource": "myCluster",
    "database": "mydb",
    "collection": "urunler",
    "filter": { "ad": "Laptop" },
    "update": { "$set": { "fiyat": 27000 } }
  }'
```

### Atlas Triggers

```javascript
// Veritabanı Trigger (UI veya CLI'dan oluştur)
// Tetikleme: siparisler koleksiyonuna insert

exports = async function(changeEvent) {
  const belge = changeEvent.fullDocument
  const db = context.services.get("myCluster").db("mydb")

  // Stok güncelle
  await db.collection("urunler").updateOne(
    { _id: belge.urun_id },
    { $inc: { stok: -belge.adet } }
  )

  // Bildirim gönder
  await context.functions.execute("bildirimGonder", {
    email: belge.musteri_email,
    mesaj: `Siparişiniz alındı: ${belge._id}`
  })
}

// Scheduled Trigger (CRON)
exports = async function() {
  const db = context.services.get("myCluster").db("mydb")

  // Süresi dolmuş oturumları temizle
  const sinir = new Date(Date.now() - 24 * 60 * 60 * 1000)
  const result = await db.collection("oturumlar").deleteMany({
    son_erisim: { $lt: sinir }
  })
  console.log(`${result.deletedCount} oturum temizlendi`)
}
```

### Atlas Monitoring

```javascript
// Atlas panelinden erişilebilen metrikler:
// - Connections (aktif bağlantı sayısı)
// - Opcounters (insert/query/update/delete/s)
// - Network (bytesIn, bytesOut)
// - System CPU/Memory
// - Disk IOPS
// - Replication Lag
// - Cache Hit Ratio

// Atlas Alerts yapılandırması (CLI)
atlas alerts settings create \
  --metricName CONNECTIONS \
  --threshold 1000 \
  --operator GREATER_THAN \
  --notifications EMAIL \
  --notificationEmailAddress admin@company.com
```

### Atlas Backup

```bash
# Cloud Backup (snapshot tabanlı)
# UI'dan: Database → Backup → Take Snapshot

# Atlas CLI ile
atlas backups snapshots create myCluster \
  --desc "Manuel yedek - 2024-01-15"

# Snapshot listele
atlas backups snapshots list myCluster

# Geri yükle
atlas backups restores start automated \
  --clusterName myCluster \
  --snapshotId <snapshot-id> \
  --targetClusterName myRestoredCluster
```

---

## 💡 Bağlantılar
- [[MongoDB - Monitoring Yönetim ve Docker]]
- [[MongoDB - Güvenlik ve Yetkilendirme]]
- [[MongoDB - Python ile PyMongo]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- mongodb.com/docs/atlas/
- mongodb.com/docs/atlas/atlas-search/
- mongodb.com/docs/atlas/data-api/
