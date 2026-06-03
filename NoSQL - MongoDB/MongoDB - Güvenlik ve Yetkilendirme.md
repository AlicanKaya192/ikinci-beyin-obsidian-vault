---
tarih: 2025-01-01
konu: MongoDB Güvenlik, Authentication, Authorization, TLS, RBAC
etiket: [mongodb, güvenlik, authentication, authorization, rbac, tls, kullanıcı]
kaynak:
zorluk: ⭐⭐⭐
---

## 📌 Özet

MongoDB varsayılan olarak güvenlik kapalıdır. Production'da mutlaka: kimlik doğrulama, yetkilendirme, TLS/SSL, ağ kısıtlaması ve audit logging etkinleştirilmelidir.

---

## 🧠 Detay

### Authentication Aktifleştirme

```yaml
# /etc/mongod.conf
security:
  authorization: enabled
  javascriptEnabled: false  # $where ve MapReduce'u devre dışı bırak
```

```bash
# Komut satırından
mongod --auth --bind_ip 127.0.0.1
```

### Kullanıcı Yönetimi

```javascript
// Admin veritabanında admin kullanıcı oluştur
use admin

db.createUser({
  user: "adminUser",
  pwd: "güçlü_şifre_123!",
  roles: [
    { role: "userAdminAnyDatabase", db: "admin" },
    { role: "readWriteAnyDatabase", db: "admin" },
    "clusterAdmin"
  ]
})

// Uygulama kullanıcısı — sadece kendi DB'sine erişim
use mydb

db.createUser({
  user: "appUser",
  pwd: "app_güçlü_şifre!",
  roles: [
    { role: "readWrite", db: "mydb" }
  ],
  passwordDigestor: "server",
  customData: { department: "backend" }
})

// Salt okunur kullanıcı (raporlama)
db.createUser({
  user: "reportUser",
  pwd: "rapor_şifre!",
  roles: [{ role: "read", db: "mydb" }]
})

// Kullanıcı listele
db.getUsers()
use admin
db.system.users.find()

// Kullanıcı güncelle
db.updateUser("appUser", {
  pwd: "yeni_şifre!",
  roles: [
    { role: "readWrite", db: "mydb" },
    { role: "read", db: "raporlar" }
  ]
})

// Şifre değiştir
db.changeUserPassword("appUser", "yeni_şifre!")

// Kullanıcı sil
db.dropUser("eskiUser")

// Tüm kullanıcılar
db.runCommand({ usersInfo: 1 })
```

### Built-in Roller

```javascript
// Veritabanı Rolleri:
"read"              // Okuma
"readWrite"         // Okuma + Yazma
"dbAdmin"           // İndeks, istatistik, profil
"userAdmin"         // Kullanıcı yönetimi
"dbOwner"           // readWrite + dbAdmin + userAdmin

// Admin Rolleri (anyDatabase):
"readAnyDatabase"
"readWriteAnyDatabase"
"userAdminAnyDatabase"
"dbAdminAnyDatabase"

// Cluster Rolleri:
"clusterAdmin"      // Tam cluster yönetimi
"clusterManager"
"clusterMonitor"    // Sadece monitoring

// Özel Roller:
"backup"            // Yedekleme
"restore"           // Geri yükleme
"root"              // Süper admin (dikkatli!)
```

### Custom Rol Oluşturma

```javascript
use admin

db.createRole({
  role: "urunYoneticisi",
  privileges: [
    {
      resource: { db: "mydb", collection: "urunler" },
      actions: ["find", "insert", "update"]
      // Silme yetkisi yok!
    },
    {
      resource: { db: "mydb", collection: "kategoriler" },
      actions: ["find"]
    }
  ],
  roles: []  // Miras alınan roller
})

// Role rol devret
db.grantRolesToUser("appUser", [{ role: "urunYoneticisi", db: "admin" }])
db.revokeRolesFromUser("appUser", [{ role: "urunYoneticisi", db: "admin" }])
```

### TLS/SSL Yapılandırması

```yaml
# /etc/mongod.conf
net:
  tls:
    mode: requireTLS
    certificateKeyFile: /etc/mongodb/server.pem
    CAFile: /etc/mongodb/ca.pem
    allowConnectionsWithoutCertificates: false
```

```bash
# TLS ile bağlan
mongosh --tls \
  --tlsCertificateKeyFile client.pem \
  --tlsCAFile ca.pem \
  "mongodb://localhost:27017"

# Python ile TLS
client = MongoClient(
    "mongodb://localhost:27017",
    tls=True,
    tlsCertificateKeyFile="/path/to/client.pem",
    tlsCAFile="/path/to/ca.pem"
)
```

### Ağ Güvenliği

```yaml
# /etc/mongod.conf — sadece belirli IP'lere izin ver
net:
  port: 27017
  bindIp: 127.0.0.1,10.0.0.5  # Localhost + uygulama sunucusu

# Güvenlik Duvarı (ufw)
# sudo ufw allow from 10.0.0.0/8 to any port 27017
# sudo ufw deny 27017
```

### Field Level Encryption (FLE)

```python
# Hassas alanları şifreli sakla
from pymongo.encryption import ClientEncryption
from pymongo.encryption_options import AutoEncryptionOpts

# Key vault
key_vault_namespace = "encryption.__keyVault"

# KMS sağlayıcı (yerel anahtar)
kms_providers = {
    "local": {
        "key": b"\x00" * 96  # Production'da güvenli anahtar!
    }
}

# Auto encryption
auto_encryption_opts = AutoEncryptionOpts(
    kms_providers=kms_providers,
    key_vault_namespace=key_vault_namespace,
    schema_map={
        "mydb.hastalar": {
            "bsonType": "object",
            "encryptMetadata": {
                "keyId": [key_id],
                "algorithm": "AEAD_AES_256_CBC_HMAC_SHA_512-Deterministic"
            },
            "properties": {
                "tc_kimlik": {
                    "encrypt": {
                        "bsonType": "string",
                        "algorithm": "AEAD_AES_256_CBC_HMAC_SHA_512-Deterministic"
                    }
                },
                "teshis": {
                    "encrypt": {
                        "bsonType": "string",
                        "algorithm": "AEAD_AES_256_CBC_HMAC_SHA_512-Random"
                    }
                }
            }
        }
    }
)

client = MongoClient(
    "mongodb://localhost:27017",
    auto_encryption_opts=auto_encryption_opts
)
```

### Audit Logging

```yaml
# /etc/mongod.conf (MongoDB Enterprise)
auditLog:
  destination: file
  format: JSON
  path: /var/log/mongodb/audit.json
  filter: |
    {
      atype: {
        $in: ["authenticate", "createUser", "dropUser",
              "createCollection", "dropCollection",
              "find", "insert", "update", "remove"]
      }
    }
```

### Güvenlik Kontrol Listesi

```
□ Kimlik doğrulama aktif mi? (--auth)
□ Admin kullanıcısı güçlü şifre ile oluşturuldu mu?
□ Uygulama için minimum yetkili kullanıcı var mı?
□ TLS/SSL aktif mi?
□ bindIp ile ağ kısıtlaması var mı?
□ Güvenlik duvarı 27017'yi dışarıya kapatıyor mu?
□ MongoDB güncel versiyonda mı?
□ $where ve server-side JS kapalı mı?
□ Audit logging aktif mi? (Enterprise)
□ Düzenli yedekleme var mı?
```

---

## 💡 Bağlantılar
- [[MongoDB - Replikasyon ve Sharding]]
- [[MongoDB - Monitoring ve Yönetim]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- mongodb.com/docs/manual/security/
- mongodb.com/docs/manual/tutorial/enable-authentication/
