---
tarih: 2025-01-01
konu: MongoDB Replikasyon, Replica Set, Sharding, Yatay Ölçeklendirme
etiket: [mongodb, replikasyon, replica-set, sharding, ölçeklendirme, failover]
kaynak:
zorluk: ⭐⭐⭐⭐
---

## 📌 Özet

Replikasyon yüksek erişilebilirlik (HA) sağlar, sharding ise yatay ölçeklendirme. Replica set: aynı verinin kopyaları. Sharded cluster: veri parçaları farklı sunucularda.

---

## 🧠 Detay

### Replica Set

```
Replica Set (3 Node — minimum önerilen)

Primary            Secondary          Secondary
┌───────────┐      ┌───────────┐      ┌───────────┐
│ Reads &   │      │ Reads     │      │ Reads     │
│ Writes    │─────►│ (Optional)│      │ (Optional)│
│           │      │ Failover  │      │ Failover  │
└───────────┘      └───────────┘      └───────────┘
      │                  │                  │
      └──────── Oplog Replication ──────────┘
```

**Oplog**: Primary'nin yaptığı tüm değişiklikleri kaydeden capped collection. Secondary'ler oplogu sürekli okur ve uygular.

#### Replica Set Kurulumu

```javascript
// Primary'de başlat
rs.initiate({
  _id: "myReplicaSet",
  members: [
    { _id: 0, host: "mongo1:27017", priority: 2 },  // Priority yüksek → Primary tercih
    { _id: 1, host: "mongo2:27017", priority: 1 },
    { _id: 2, host: "mongo3:27017", priority: 1 }
  ]
})

// Durum kontrolü
rs.status()
rs.conf()
rs.isMaster()  // Primary mi?

// Üye ekle
rs.add("mongo4:27017")
rs.add({ host: "mongo4:27017", priority: 0, hidden: true })

// Üye kaldır
rs.remove("mongo4:27017")

// Arbiter ekle (veri yok, oy var — çift sayıda node için)
rs.addArb("mongo-arb:27017")

// Stepdown (Primary'yi değiştir)
rs.stepDown(60)  // 60 saniye Primary olmama
```

#### Replica Set Node Tipleri

| Tip | Veri | Oy | Primary Olabilir | Kullanım |
|---|---|---|---|---|
| Normal | ✅ | ✅ | ✅ | Standart |
| Priority 0 | ✅ | ✅ | ❌ | DR site |
| Hidden | ✅ | ✅ | ❌ | Raporlama |
| Delayed | ✅ | ✅ | ❌ | Felaket kurtarma (gecikmeli kopya) |
| Arbiter | ❌ | ✅ | ❌ | Sadece oy (çift node'da) |

```javascript
// Hidden + Delayed üye yapılandırması
rs.reconfig({
  _id: "myRS",
  members: [
    { _id: 0, host: "mongo1:27017" },
    { _id: 1, host: "mongo2:27017" },
    { _id: 2, host: "mongo3:27017",
      priority: 0,
      hidden: true,
      secondaryDelaySecs: 3600  // 1 saat gecikmeli — hata kurtarma
    }
  ]
})
```

#### Bağlantı ve Read Preference

```javascript
// Replica set bağlantısı
const uri = "mongodb://mongo1:27017,mongo2:27017,mongo3:27017/?replicaSet=myRS"

// Read preference ayarları:
// "primary"            → Sadece primary (varsayılan)
// "primaryPreferred"   → Primary yoksa secondary
// "secondary"          → Sadece secondary (replica lag!)
// "secondaryPreferred" → Secondary yoksa primary
// "nearest"            → En düşük latency

db.col.find().readPref("secondary")

// Python'da:
from pymongo import MongoClient, ReadPreference
client = MongoClient(uri, readPreference="secondaryPreferred")
```

---

### Sharding (Yatay Ölçeklendirme)

```
Sharded Cluster Mimarisi:

Uygulama
    │
    ▼
mongos (Query Router)  ←── Config Servers (3x replica set)
    │                        Metadata: hangi shard nerede
    ├──► Shard 1 (Replica Set)
    │     Veri: A-M
    ├──► Shard 2 (Replica Set)
    │     Veri: N-Z
    └──► Shard 3 (Replica Set)
          Veri: Yeni eklenenler
```

#### Shard Key Seçimi

```javascript
// Shard key seçimi kritik — sonradan değiştirmek çok zor!

// ✅ İyi shard key özellikleri:
// - Yüksek kardinalite (çok farklı değer)
// - Eşit dağılım (hot shard olmasın)
// - Sorgu kalıplarıyla uyumlu

// ❌ Kötü shard key:
// - Monoton artan (_id, timestamp) → Son sharda yığılır
// - Düşük kardinalite (true/false) → 2 chunk max
// - Nadiren kullanılan alan

// Örnekler:
db.adminCommand({ shardCollection: "mydb.col", key: { kullanici_id: "hashed" } })  // Hash → Eşit dağılım
db.adminCommand({ shardCollection: "mydb.col", key: { sehir: 1, _id: 1 } })  // Compound ranged
```

#### Sharding Kurulumu

```javascript
// Config server ve shard'ları başlat (her biri replica set)

// mongos'ta sharding aktifleştir
sh.enableSharding("mydb")

// Koleksiyonu shard et
sh.shardCollection(
  "mydb.siparisler",
  { musteri_id: "hashed" }  // Hash shard key → eşit dağılım
)

// Ranged shard key
sh.shardCollection(
  "mydb.urunler",
  { kategori: 1, _id: 1 }   // Önce kategori, sonra _id
)

// Durum
sh.status()
sh.isBalancerRunning()

// Manuel chunk taşıma
sh.moveChunk("mydb.col", { musteri_id: "user123" }, "shard2")

// Balancer kontrolü
sh.stopBalancer()
sh.startBalancer()
```

#### Zone Sharding (Coğrafi Dağılım)

```javascript
// Türkiye verisi → TR shard'ına, Avrupa → EU shard'ına
sh.addShardTag("shard1", "TR")
sh.addShardTag("shard2", "EU")

sh.addTagRange(
  "mydb.kullanicilar",
  { ulke: "TR", _id: MinKey },
  { ulke: "TR", _id: MaxKey },
  "TR"
)
sh.addTagRange(
  "mydb.kullanicilar",
  { ulke: "DE", _id: MinKey },
  { ulke: "DE", _id: MaxKey },
  "EU"
)
```

---

### Replica Set vs Sharding Karar

```
Ne zaman Replica Set yeterli?
✅ Veri boyutu tek sunucuya sığıyor
✅ Yüksek erişilebilirlik (HA) istiyorum
✅ Okuma performansını artırmak istiyorum
✅ Disaster recovery gerekiyor

Ne zaman Sharding gerekli?
✅ Veri boyutu tek sunucuyu aştı
✅ Yazma kapasitesi tek primary yetersiz
✅ TB / PB düzeyinde veri
✅ Bölgesel veri yerleşimi (GDPR vb.)
```

---

### Monitoring

```javascript
// Replica set lag
rs.status().members.forEach(m => {
  if (m.state === 2) {  // Secondary
    print(m.name, "lag:", m.optimeDate)
  }
})

// Oplog boyutu ve yeterliliği
rs.printReplicationInfo()
rs.printSecondaryReplicationInfo()

// Shard boyutları
sh.status(true)

// Sunucu istatistikleri
db.serverStatus()
db.serverStatus().opcounters  // insert, query, update, delete/saniye
```

---

## 💡 Bağlantılar
- [[MongoDB - Transactions ve ACID]]
- [[MongoDB - Güvenlik ve Yetkilendirme]]
- [[MongoDB - Monitoring ve Yönetim]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- mongodb.com/docs/manual/replication/
- mongodb.com/docs/manual/sharding/
