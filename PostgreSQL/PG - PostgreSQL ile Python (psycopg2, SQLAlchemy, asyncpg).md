---
tarih: 2026-08-17
konu: PostgreSQL
etiket: [postgresql, python, psycopg2, sqlalchemy, asyncpg, data-science, orta]
kaynak: SQLAlchemy Docs, psycopg2 Docs
zorluk: orta
---

## 📌 Özet

Python ile PostgreSQL bağlantısı için üç ana yaklaşım: **psycopg2** (düşük seviye, hızlı), **SQLAlchemy** (ORM + query builder, en yaygın), **asyncpg** (async uygulamalar için). DS workflow'larında pandas ile entegrasyon kritik.

---

## 🧠 Detay

### Kütüphane Seçimi

```mermaid
graph TD
    A[Python + PostgreSQL] --> B{Kullanım Amacı}
    B -->|Basit sorgular| C[psycopg2]
    B -->|ORM + Büyük proje| D[SQLAlchemy]
    B -->|FastAPI / async| E[asyncpg + SQLAlchemy async]
    B -->|Data Science / pandas| F[pandas + SQLAlchemy]
    B -->|Yalnızca okuma| G[polars + ConnectorX]
```

### psycopg2 — Temel Kullanım

```python
import psycopg2
import psycopg2.extras   # DictCursor için

# Bağlantı
conn = psycopg2.connect(
    host="localhost",
    port=5432,
    dbname="mydb",
    user="devuser",
    password="password"
)

# Context manager ile (önerilen)
with psycopg2.connect(DSN) as conn:
    with conn.cursor(cursor_factory=psycopg2.extras.DictCursor) as cur:
        cur.execute("SELECT * FROM kullanicilar WHERE aktif = %s", (True,))
        rows = cur.fetchall()
        for row in rows:
            print(row['email'])  # DictCursor → dict gibi erişim
```

### Güvenli Parameterize Sorgu

```python
# ✅ Doğru — SQL injection korumalı
cur.execute(
    "SELECT * FROM kullanicilar WHERE email = %s AND aktif = %s",
    (email, True)
)

# ❌ Yanlış — SQL injection riski!
cur.execute(f"SELECT * FROM kullanicilar WHERE email = '{email}'")

# Batch insert — executemany
kullanicilar = [
    ("ali@example.com", "Ali"),
    ("veli@example.com", "Veli")
]
cur.executemany(
    "INSERT INTO kullanicilar (email, ad) VALUES (%s, %s)",
    kullanicilar
)
conn.commit()

# execute_values — çok daha hızlı toplu insert
from psycopg2.extras import execute_values
execute_values(
    cur,
    "INSERT INTO kullanicilar (email, ad) VALUES %s",
    kullanicilar
)
```

### psycopg2 + .env ile Güvenli Bağlantı

```python
# .env
# DB_URL=postgresql://user:pass@localhost:5432/mydb

from dotenv import load_dotenv
import os
import psycopg2
from urllib.parse import urlparse

load_dotenv()
url = urlparse(os.getenv("DB_URL"))

conn = psycopg2.connect(
    host=url.hostname,
    port=url.port,
    dbname=url.path[1:],   # /mydb → mydb
    user=url.username,
    password=url.password
)
```

---

### SQLAlchemy — ORM ve Core

```python
from sqlalchemy import create_engine, text, Column, Integer, String
from sqlalchemy.orm import DeclarativeBase, Session

# Engine oluştur
engine = create_engine(
    "postgresql+psycopg2://user:pass@localhost/mydb",
    pool_size=10,
    max_overflow=20,
    echo=False   # True → SQL loglar
)

# ORM Model
class Base(DeclarativeBase):
    pass

class Kullanici(Base):
    __tablename__ = "kullanicilar"

    id = Column(Integer, primary_key=True)
    email = Column(String(255), unique=True, nullable=False)
    ad = Column(String(100))

# Tablo oluştur
Base.metadata.create_all(engine)

# CRUD
with Session(engine) as session:
    # Create
    yeni = Kullanici(email="ali@example.com", ad="Ali")
    session.add(yeni)
    session.commit()

    # Read
    kullanici = session.query(Kullanici).filter_by(email="ali@example.com").first()

    # Update
    kullanici.ad = "Ali Veli"
    session.commit()

    # Delete
    session.delete(kullanici)
    session.commit()
```

### SQLAlchemy Core (ORM olmadan)

```python
from sqlalchemy import select, insert, update, delete, Table, MetaData

metadata = MetaData()
metadata.reflect(bind=engine)
kullanicilar = metadata.tables['kullanicilar']

with engine.connect() as conn:
    # Select
    stmt = select(kullanicilar).where(kullanicilar.c.aktif == True)
    result = conn.execute(stmt)
    for row in result:
        print(row._mapping)

    # Ham SQL
    result = conn.execute(text("SELECT COUNT(*) FROM kullanicilar"))
    print(result.scalar())
```

---

### pandas + SQLAlchemy — DS Workflow

```python
import pandas as pd
from sqlalchemy import create_engine

engine = create_engine("postgresql+psycopg2://user:pass@localhost/mydb")

# Veritabanından DataFrame'e
df = pd.read_sql_query(
    "SELECT * FROM satislar WHERE tarih >= '2025-01-01'",
    con=engine
)

# Büyük veri için chunk
for chunk in pd.read_sql_query(sql, con=engine, chunksize=10_000):
    process(chunk)

# DataFrame'i veritabanına yaz
df_sonuc.to_sql(
    name="tahmin_sonuclari",
    con=engine,
    schema="analytics",
    if_exists="append",   # append | replace | fail
    index=False,
    method="multi",       # Batch insert
    chunksize=1000
)

# Çok daha hızlı: COPY ile (psycopg2.extras)
from io import StringIO
buffer = StringIO()
df.to_csv(buffer, index=False, header=False)
buffer.seek(0)
with conn.cursor() as cur:
    cur.copy_from(buffer, 'tahmin_sonuclari', sep=',', null='')
conn.commit()
```

---

### asyncpg — Async FastAPI Entegrasyonu

```python
import asyncpg
from fastapi import FastAPI, Depends

app = FastAPI()

# Connection pool
@app.on_event("startup")
async def startup():
    app.state.pool = await asyncpg.create_pool(
        dsn="postgresql://user:pass@localhost/mydb",
        min_size=5,
        max_size=20
    )

@app.on_event("shutdown")
async def shutdown():
    await app.state.pool.close()

# Dependency
async def get_db():
    async with app.state.pool.acquire() as conn:
        yield conn

# Endpoint
@app.get("/kullanicilar/{id}")
async def get_kullanici(id: int, db=Depends(get_db)):
    row = await db.fetchrow(
        "SELECT * FROM kullanicilar WHERE id = $1", id
    )
    return dict(row) if row else {"error": "bulunamadı"}

# asyncpg metotları
# fetchrow  → tek satır
# fetch     → tüm satırlar (liste)
# fetchval  → tek değer
# execute   → DML (INSERT/UPDATE/DELETE)
# executemany → batch
```

### SQLAlchemy 2.0 Async (asyncpg ile)

```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker

engine = create_async_engine(
    "postgresql+asyncpg://user:pass@localhost/mydb",
    echo=False
)
AsyncSessionLocal = sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)

async def get_session():
    async with AsyncSessionLocal() as session:
        yield session
```

---

### Connection Pooling

```python
# SQLAlchemy pool ayarları
engine = create_engine(
    "postgresql+psycopg2://...",
    pool_size=10,          # Kalıcı bağlantı sayısı
    max_overflow=20,       # Geçici ek bağlantı
    pool_timeout=30,       # Bağlantı bekle (saniye)
    pool_recycle=3600,     # Bağlantıyı 1 saatte yenile
    pool_pre_ping=True,    # Önce ping at (stale bağlantı kontrolü)
)

# PgBouncer ile harici pooling (production önerilir)
# app → PgBouncer → PostgreSQL
# transaction mode: her sorgu ayrı bağlantı
# session mode: uygulama oturumu boyunca aynı bağlantı
```

---

## 💡 Bağlantılar
- [[PG - PostgreSQL Kurulum ve Temel Yapılandırma]]
- [[FastAPI - Veritabanı Entegrasyonu (SQLAlchemy & Async)]]
- [[Python - Async Programlama (asyncio, aiohttp)]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- [SQLAlchemy Docs](https://docs.sqlalchemy.org/)
- [asyncpg Docs](https://magicstack.github.io/asyncpg/)
- [psycopg2 Docs](https://www.psycopg.org/docs/)
