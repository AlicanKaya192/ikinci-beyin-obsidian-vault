---
tarih: 2026-08-17
konu: PostgreSQL
etiket: [postgresql, replikasyon, yüksek-erişilebilirlik, ha, production, ileri]
kaynak: PostgreSQL Docs, Patroni
zorluk: ileri
---

## 📌 Özet

Production'da tek bir PostgreSQL sunucusu kabul edilemez risk. Streaming replication ile primer-replica mimarisi, Patroni ile otomatik failover, pgBouncer ile connection pooling — yüksek erişilebilir PostgreSQL cluster kurma rehberi.

---

## 🧠 Detay

### Replikasyon Türleri

```
Fiziksel Replikasyon (Streaming):
  → WAL (Write-Ahead Log) byte byte kopyalar
  → Replica, primer'ın aynası (okuma için)
  → Standby sunucu olarak çalışır

Mantıksal Replikasyon (Logical):
  → Tablo bazlı, seçici
  → Farklı PostgreSQL versiyonları arası
  → Sütun filtresi, satır filtresi mümkün
```

### Streaming Replication Kurulumu

```bash
# PRIMARY sunucuda

# postgresql.conf
wal_level = replica
max_wal_senders = 3
wal_keep_size = 1GB

# pg_hba.conf — replica için izin
host replication replicator 192.168.1.102/32 scram-sha-256

# Replication kullanıcısı
CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD 'güçlü_parola';
```

```bash
# REPLICA sunucuda

# Basebackup al (primary'dan)
pg_basebackup -h 192.168.1.101 -U replicator \
  -D /var/lib/postgresql/16/main \
  -P -Xs -R   # -R: standby.signal ve recovery.conf oluşturur

# postgresql.conf (replica)
hot_standby = on   # Okuma sorguları kabul et

# Başlat
sudo systemctl start postgresql

# Kontrol
psql -c "SELECT * FROM pg_stat_replication;"
```

### Replikasyon Durumu İzleme

```sql
-- Primary'da
SELECT
    client_addr,
    state,          -- streaming / catchup / startup
    sent_lsn,
    write_lsn,
    flush_lsn,
    replay_lsn,
    write_lag,      -- gecikme
    flush_lag,
    replay_lag
FROM pg_stat_replication;

-- Replica'da
SELECT
    status,         -- streaming | startup | catchup
    received_lsn,
    last_msg_receipt_time,
    latest_end_lsn,
    latest_end_time
FROM pg_stat_wal_receiver;
```

### Failover — Manuel

```bash
# Primary çöktü, replica'yı primary'a yükselt

# Replica sunucuda
pg_ctl promote -D /var/lib/postgresql/16/main

# veya
touch /var/lib/postgresql/16/main/failover.signal
```

### Patroni — Otomatik Failover

```yaml
# patroni.yml — Her sunucuda
scope: postgres-cluster
namespace: /service/
name: pg1  # pg2, pg3 diğer sunucularda

restapi:
  listen: 0.0.0.0:8008
  connect_address: 192.168.1.101:8008

etcd:   # Consensus: etcd | ZooKeeper | Consul
  hosts: 192.168.1.100:2379

bootstrap:
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 30
    maximum_lag_on_failover: 1048576  # 1MB

postgresql:
  listen: 0.0.0.0:5432
  connect_address: 192.168.1.101:5432
  data_dir: /var/lib/postgresql/16/main
  authentication:
    replication:
      username: replicator
      password: güçlü_parola
    superuser:
      username: postgres
      password: postgres_parola
```

```bash
# Patroni cluster durumu
patronictl -c /etc/patroni/patroni.yml list

# Output:
# + Cluster: postgres-cluster --------+----+-----------+
# | Member | Host           | Role    | State   | TL |
# +--------+----------------+---------+---------+----+
# | pg1    | 192.168.1.101  | Leader  | running | 1  |
# | pg2    | 192.168.1.102  | Replica | running | 1  |
# | pg3    | 192.168.1.103  | Replica | running | 1  |

# Manuel failover
patronictl -c patroni.yml failover postgres-cluster --master pg1 --candidate pg2
```

### pgBouncer — Connection Pooling

```ini
# pgbouncer.ini
[databases]
mydb = host=127.0.0.1 port=5432 dbname=mydb

[pgbouncer]
listen_port = 6432
listen_addr = *
auth_type = scram-sha-256
auth_file = /etc/pgbouncer/userlist.txt
pool_mode = transaction   # transaction | session | statement
max_client_conn = 1000
default_pool_size = 20
```

```
Mimari:
  App (1000 bağlantı) → pgBouncer → PostgreSQL (20 bağlantı)
  
  session mode:  Kullanıcı bağlantısı boyunca aynı PG bağlantısı
  transaction:   Her transaction farklı PG bağlantısı (önerilen)
  statement:     Her sorgu farklı (prepared statement ile çalışmaz)
```

### HAProxy ile Trafik Yönlendirme

```
Client
  │
  ├─→ HAProxy :5432 (yazma) ──→ Primary
  └─→ HAProxy :5433 (okuma) ──→ Replica Round-Robin
```

```ini
# haproxy.cfg
frontend postgres_write
    bind *:5432
    default_backend postgres_primary

backend postgres_primary
    option httpchk GET /primary   # Patroni REST API'si
    server pg1 192.168.1.101:5432 check port 8008
    server pg2 192.168.1.102:5432 check port 8008 backup
    server pg3 192.168.1.103:5432 check port 8008 backup

frontend postgres_read
    bind *:5433
    default_backend postgres_replicas

backend postgres_replicas
    balance roundrobin
    option httpchk GET /replica
    server pg2 192.168.1.102:5432 check port 8008
    server pg3 192.168.1.103:5432 check port 8008
```

### Yedekleme — pgBackRest

```bash
# Kurulum
sudo apt install pgbackrest

# /etc/pgbackrest.conf
[global]
repo1-path=/var/lib/pgbackrest
repo1-retention-full=2
repo1-cipher-type=aes-256-cbc  # şifreli yedek
repo1-cipher-pass=yedek_sifresi

[mydb]
pg1-path=/var/lib/postgresql/16/main

# İlk tam yedek
pgbackrest --stanza=mydb stanza-create
pgbackrest --stanza=mydb backup --type=full

# Artımlı yedek (günlük cron)
pgbackrest --stanza=mydb backup --type=incr

# Geri yükle
pgbackrest --stanza=mydb restore --recovery-option="recovery_target_time=2025-01-15 12:00:00"
```

---

## 💡 Bağlantılar
- [[PG - Performans Optimizasyonu ve EXPLAIN ANALYZE]]
- [[K8s - StatefulSet ile Veritabanı Yönetimi]]
- [[Monitoring - Prometheus ve Grafana ile Altyapı İzleme]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- [Patroni GitHub](https://github.com/patroni/patroni)
- [pgBackRest](https://pgbackrest.org/)
- [PostgreSQL Replication Docs](https://www.postgresql.org/docs/current/high-availability.html)
