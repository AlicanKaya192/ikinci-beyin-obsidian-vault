---
tarih: 2025-01-01
konu: MongoDB Monitoring, Yönetim Komutları, Docker, Atlas, Yedekleme
etiket: [mongodb, monitoring, yönetim, docker, atlas, yedekleme, mongodump]
kaynak:
zorluk: ⭐⭐⭐
---

## 📌 Özet

MongoDB yönetimi: performans izleme, yedekleme/geri yükleme, Docker ile çalıştırma ve MongoDB Atlas (cloud) kullanımı.

---

## 🧠 Detay

### Sistem İstatistikleri

```javascript
// Sunucu durumu
db.serverStatus()
db.serverStatus().opcounters    // Operasyon sayaçları
db.serverStatus().connections   // Aktif/mevcut bağlantılar
db.serverStatus().memory        // RAM kullanımı
db.serverStatus().wiredTiger.cache  // Cache hit ratio

// Veritabanı istatistikleri
db.stats()
// storageSize, dataSize, indexSize, collections, objects

// Koleksiyon istatistikleri
db.col.stats()
db.col.totalSize()
db.col.storageSize()
db.col.totalIndexSize()

// Top — En fazla sorgu alan koleksiyonlar
db.adminCommand({ top: 1 })

// CurrentOp — Çalışan işlemler
db.currentOp()
db.currentOp({ "active": true })
db.currentOp({ "secs_running": { "$gt": 5 } })  // 5 saniyedir çalışanlar

// Uzun çalışan işlemi öldür
db.killOp(12345)  // opId
```

### Profiler

```javascript
// Yavaş sorguları logla
db.setProfilingLevel(1, { slowms: 100 })  // 100ms üstü
db.setProfilingLevel(2)                    // Hepsi

// Profil verilerini gör
db.system.profile
  .find({ "ns": { $ne: "mydb.system.profile" } })
  .sort({ ts: -1 })
  .limit(10)

db.system.profile.find({ millis: { $gt: 200 } })
  .sort({ millis: -1 })

// Kapat
db.setProfilingLevel(0)
db.getProfilingStatus()
```

### Yedekleme ve Geri Yükleme

```bash
# ─── mongodump ─────────────────────────────
# Tüm veritabanı
mongodump --out /backup/$(date +%Y%m%d)

# Belirli veritabanı
mongodump --db mydb --out /backup/

# Belirli koleksiyon
mongodump --db mydb --collection urunler --out /backup/

# Kimlik doğrulamalı
mongodump \
  --host localhost:27017 \
  --username admin \
  --password şifre \
  --authenticationDatabase admin \
  --db mydb \
  --out /backup/

# Sıkıştırılmış
mongodump --db mydb --archive=/backup/mydb.archive --gzip

# ─── mongorestore ──────────────────────────
# Tüm veritabanı
mongorestore /backup/20240101/

# Belirli DB
mongorestore --db mydb /backup/20240101/mydb/

# Sıkıştırılmış
mongorestore --archive=/backup/mydb.archive --gzip

# Drop ve yeniden yükle
mongorestore --drop --db mydb /backup/mydb/

# ─── mongoexport / mongoimport ─────────────
# JSON export
mongoexport --db mydb --collection urunler --out urunler.json
mongoexport --db mydb --collection urunler --type csv \
  --fields ad,fiyat,stok --out urunler.csv

# JSON import
mongoimport --db mydb --collection urunler --file urunler.json
mongoimport --db mydb --collection urunler --type csv \
  --headerline --file urunler.csv

# Upsert modunda import
mongoimport --db mydb --collection urunler --mode upsert \
  --upsertFields ad --file urunler.json
```

### Docker ile MongoDB

```yaml
# docker-compose.yml
version: '3.9'

services:
  mongodb:
    image: mongo:7.0
    container_name: mongodb
    restart: unless-stopped
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: şifre123
      MONGO_INITDB_DATABASE: mydb
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db
      - mongo_config:/data/configdb
      - ./init-mongo.js:/docker-entrypoint-initdb.d/init-mongo.js:ro
    command: mongod --auth

  mongo-express:  # Web UI
    image: mongo-express:latest
    restart: unless-stopped
    ports:
      - "8081:8081"
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: admin
      ME_CONFIG_MONGODB_ADMINPASSWORD: şifre123
      ME_CONFIG_MONGODB_URL: mongodb://admin:şifre123@mongodb:27017/
      ME_CONFIG_BASICAUTH_USERNAME: webadmin
      ME_CONFIG_BASICAUTH_PASSWORD: webşifre
    depends_on:
      - mongodb

volumes:
  mongo_data:
  mongo_config:
```

```javascript
// init-mongo.js — Container başlangıcında çalışır
db = db.getSiblingDB("mydb")

db.createUser({
  user: "appUser",
  pwd: "app_şifre!",
  roles: [{ role: "readWrite", db: "mydb" }]
})

db.urunler.insertMany([
  { ad: "Laptop", fiyat: 25000, stok: 50 },
  { ad: "Klavye", fiyat: 500,   stok: 200 }
])

db.urunler.createIndex({ ad: 1 }, { unique: true })
```

```bash
# Replica Set — Docker ile
docker network create mongo-net

docker run -d --name mongo1 --network mongo-net \
  -p 27017:27017 mongo:7.0 mongod --replSet rs0

docker run -d --name mongo2 --network mongo-net \
  -p 27018:27017 mongo:7.0 mongod --replSet rs0

docker run -d --name mongo3 --network mongo-net \
  -p 27019:27017 mongo:7.0 mongod --replSet rs0

# Replica set başlat
docker exec -it mongo1 mongosh --eval "
rs.initiate({
  _id: 'rs0',
  members: [
    { _id: 0, host: 'mongo1:27017' },
    { _id: 1, host: 'mongo2:27017' },
    { _id: 2, host: 'mongo3:27017' }
  ]
})"
```

### MongoDB Atlas (Cloud)

```bash
# Atlas CLI
brew install mongodb-atlas-cli  # Mac
atlas auth login

# Cluster oluştur (M0 = ücretsiz tier)
atlas clusters create myCluster \
  --provider AWS --region EU_WEST_1 \
  --tier M10 --mdbVersion 7.0

# Bağlantı string al
atlas clusters connectionStrings describe myCluster

# Kullanıcı ekle
atlas dbusers create --username appUser --password "şifre!" \
  --role readWriteAnyDatabase

# IP whitelist
atlas accessLists create 0.0.0.0/0 --type ipAddress  # Dikkat: Herkese açık!
atlas accessLists create $(curl -s ifconfig.me)/32 --type ipAddress
```

### Yönetim İpuçları

```javascript
// Koleksiyon compact et (boşlukları temizle)
db.runCommand({ compact: "urunler" })

// Koleksiyon dönüştür (capped)
db.runCommand({
  convertToCapped: "loglar",
  size: 10485760  // 10MB
})

// Validate
db.urunler.validate()
db.urunler.validate({ full: true })

// Repair (WiredTiger)
// mongod --repair

// Free monitoring (Atlas olmayan)
db.enableFreeMonitoring()
db.getFreeMonitoringStatus()

// Koleksiyon yeniden adlandır
db.urunler.renameCollection("products")
db.adminCommand({ renameCollection: "mydb.urunler", to: "mydb.products" })
```

### Önemli Config Parametreleri

```yaml
# /etc/mongod.conf
storage:
  dbPath: /var/lib/mongodb
  journal:
    enabled: true
  wiredTiger:
    engineConfig:
      cacheSizeGB: 2      # RAM'in %50'si (varsayılan)

systemLog:
  destination: file
  path: /var/log/mongodb/mongod.log
  logAppend: true
  verbosity: 0            # 0-5 (0=minimal)

operationProfiling:
  mode: slowOp
  slowOpThresholdMs: 100

net:
  port: 27017
  bindIp: 127.0.0.1
  maxIncomingConnections: 1000000

replication:
  replSetName: "rs0"      # Replica set adı

setParameter:
  ttlMonitorSleepSecs: 60    # TTL kontrol sıklığı
```

---

## 💡 Bağlantılar
- [[MongoDB - Replikasyon ve Sharding]]
- [[MongoDB - Güvenlik ve Yetkilendirme]]
- [[MongoDB - İndeksler ve Performans]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- mongodb.com/docs/manual/administration/
- mongodb.com/docs/atlas/
