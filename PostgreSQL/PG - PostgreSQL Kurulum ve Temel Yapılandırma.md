---
tarih: 2026-08-17
konu: PostgreSQL
etiket: [postgresql, kurulum, psql, yapılandırma, başlangıç]
kaynak: PostgreSQL Official Docs
zorluk: başlangıç
---

## 📌 Özet

PostgreSQL kurulumu, temel yapılandırma, `psql` CLI kullanımı ve güvenli bağlantı kurma. Hem yerel geliştirme hem Docker tabanlı kurulum ele alınır.

---

## 🧠 Detay

### Kurulum Yöntemleri

#### Docker ile (Önerilen — Geliştirme)

```bash
# En hızlı başlangıç
docker run -d \
  --name pg-dev \
  -e POSTGRES_PASSWORD=mysecretpassword \
  -e POSTGRES_USER=devuser \
  -e POSTGRES_DB=mydb \
  -p 5432:5432 \
  -v pg-data:/var/lib/postgresql/data \
  postgres:16

# Bağlan
docker exec -it pg-dev psql -U devuser -d mydb
```

#### Docker Compose ile

```yaml
# docker-compose.yml
services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_USER: devuser
      POSTGRES_PASSWORD: mysecretpassword
      POSTGRES_DB: mydb
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql

  pgadmin:
    image: dpage/pgadmin4
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@admin.com
      PGADMIN_DEFAULT_PASSWORD: admin
    ports:
      - "5050:80"

volumes:
  postgres_data:
```

#### Windows'ta Yerel Kurulum

```
1. postgresql.org/download → Windows installer
2. Kurulum sırasında:
   - Port: 5432 (varsayılan)
   - Superuser: postgres
   - Parola: belirle ve not al
3. pgAdmin 4 → GUI ile yönetim
4. PATH'e ekle: C:\Program Files\PostgreSQL\16\bin
```

#### Linux (Ubuntu/Debian)

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib

# Servis başlat
sudo systemctl start postgresql
sudo systemctl enable postgresql

# postgres kullanıcısına geç
sudo -i -u postgres
psql
```

---

### psql Temel Komutlar

```sql
-- Bağlan
psql -h localhost -p 5432 -U devuser -d mydb

-- psql içinde meta komutlar (\ ile başlar)
\l              -- Veritabanları listele
\c mydb         -- Veritabanına bağlan
\dt             -- Tabloları listele
\d tablo_adi    -- Tablo yapısını göster
\du             -- Kullanıcıları listele
\i dosya.sql    -- SQL dosyasını çalıştır
\timing         -- Sorgu süresini göster
\e              -- Editörde sorgu yaz
\q              -- Çık
```

### İlk Veritabanı ve Kullanıcı

```sql
-- postgres süper kullanıcı olarak
CREATE DATABASE data_science_db;
CREATE USER ds_user WITH ENCRYPTED PASSWORD 'güçlü_parola';
GRANT ALL PRIVILEGES ON DATABASE data_science_db TO ds_user;

-- Veritabanına bağlan
\c data_science_db

-- Schema oluştur
CREATE SCHEMA analytics;
GRANT ALL ON SCHEMA analytics TO ds_user;
```

### postgresql.conf Temel Ayarlar

```ini
# /etc/postgresql/16/main/postgresql.conf
# (veya Docker'da environment variable ile)

# Bellek ayarları
shared_buffers = 256MB          # RAM'in %25'i
effective_cache_size = 1GB      # RAM'in %75'i
work_mem = 16MB                 # Karmaşık sorgular için

# Bağlantı
max_connections = 100
listen_addresses = 'localhost'  # Prodda: '*'

# Loglama
log_statement = 'all'          # Geliştirmede
log_duration = on
```

### pg_hba.conf — Erişim Kontrolü

```
# Dosya: /etc/postgresql/16/main/pg_hba.conf
# FORMAT: TYPE  DATABASE  USER  ADDRESS  METHOD

# Yerel bağlantılar
local   all     postgres              peer
local   all     all                   md5

# Uzak bağlantılar
host    all     all    127.0.0.1/32   md5
host    all     all    0.0.0.0/0      md5   # Prodda kısıtla!
```

### Bağlantı String Formatları

```python
# Python - psycopg2
import psycopg2
conn = psycopg2.connect(
    host="localhost",
    port=5432,
    database="mydb",
    user="devuser",
    password="mysecretpassword"
)

# URL formatı
DATABASE_URL = "postgresql://devuser:password@localhost:5432/mydb"

# .env dosyasında sakla (git'e commit etme!)
# DB_URL=postgresql://user:pass@host:5432/db
```

### Güvenli Parola Yönetimi

```bash
# pgpass dosyası — interaktif parola sormaması için
# Linux: ~/.pgpass | Windows: %APPDATA%\postgresql\pgpass.conf
# FORMAT: hostname:port:database:username:password
localhost:5432:mydb:devuser:mysecretpassword

chmod 600 ~/.pgpass  # Linux'ta izin ayarla
```

### Faydalı Araçlar

| Araç | Açıklama | Kullanım |
|------|----------|----------|
| **pgAdmin 4** | GUI yönetim | Görsel sorgu yazma |
| **DBeaver** | Evrensel DB client | Çoklu DB yönetimi |
| **psql** | CLI | Script ve otomasyon |
| **TablePlus** | macOS/Windows GUI | Modern arayüz |
| **DataGrip** | JetBrains IDE | Profesyonel |

---

## 💡 Bağlantılar
- [[00 - PostgreSQL Giriş ve Yol Haritası]]
- [[PG - SQL Temelleri ve PostgreSQL Sözdizimi]]
- [[Docker - Container Temelleri]]
- [[PG - PostgreSQL ile Python (psycopg2, SQLAlchemy, asyncpg)]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- [PostgreSQL Download](https://www.postgresql.org/download/)
- [Docker Hub postgres](https://hub.docker.com/_/postgres)
- [psql Cheat Sheet](https://www.postgresqltutorial.com/postgresql-cheat-sheet/)
