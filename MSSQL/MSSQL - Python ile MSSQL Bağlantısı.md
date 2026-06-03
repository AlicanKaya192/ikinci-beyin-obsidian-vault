---
tarih: 2026-05-28
konu: MSSQL
etiket: ["mssql", "python", "pyodbc", "sqlalchemy", "pandas"]
kaynak: Microsoft / PyODBC Dokümantasyon
zorluk: orta
---

## 📌 Özet
Python ile MSSQL entegrasyonu, veri analizi ve otomasyon projelerinde sık kullanılır. pyodbc doğrudan bağlantı, SQLAlchemy ORM desteği, pandas ise kolay veri transferi sağlar.

## 🧠 Detay

### Kurulum
```bash
pip install pyodbc sqlalchemy pandas
# ODBC Driver 18 for SQL Server kurulu olmalı
```

### pyodbc ile Bağlantı
```python
import pyodbc

conn = pyodbc.connect(
    "DRIVER={ODBC Driver 18 for SQL Server};"
    "SERVER=localhost\\SQLEXPRESS;"   # veya SERVER=localhost,1433
    "DATABASE=NorthWind;"
    "UID=sa;"
    "PWD=sifre123;"
    "TrustServerCertificate=yes;"
)

cursor = conn.cursor()

# Sorgu çalıştır
cursor.execute("SELECT TOP 10 * FROM Musteriler WHERE Sehir = ?", ('İstanbul',))
rows = cursor.fetchall()

for row in rows:
    print(row.Ad, row.Soyad)

cursor.close()
conn.close()
```

### SQLAlchemy ile Bağlantı
```python
from sqlalchemy import create_engine, text
import urllib

params = urllib.parse.quote_plus(
    "DRIVER={ODBC Driver 18 for SQL Server};"
    "SERVER=localhost;"
    "DATABASE=NorthWind;"
    "UID=sa;PWD=sifre123;"
    "TrustServerCertificate=yes;"
)

engine = create_engine(f"mssql+pyodbc:///?odbc_connect={params}")

with engine.connect() as con:
    result = con.execute(text("SELECT COUNT(*) FROM Musteriler"))
    print(result.fetchone()[0])
```

### Pandas ile Veri Transferi
```python
import pandas as pd

# MSSQL → DataFrame
df = pd.read_sql(
    "SELECT * FROM Musteriler WHERE Aktif = 1",
    engine
)

# Parametreli sorgu
df = pd.read_sql(
    "SELECT * FROM Siparisler WHERE MusteriID = %(id)s",
    engine,
    params={"id": 5}
)

# Stored Procedure çağır
df = pd.read_sql("EXEC sp_MusteriRapor @Yil = 2024", engine)

# DataFrame → MSSQL
df.to_sql(
    "YeniTablo",
    engine,
    if_exists="append",   # replace, append, fail
    index=False,
    chunksize=1000
)
```

### Bulk Insert
```python
import pyodbc
from io import StringIO

# Hızlı toplu insert
conn.autocommit = False
cursor = conn.cursor()
cursor.fast_executemany = True

data = [(1, 'Ali', 'İstanbul'), (2, 'Ayşe', 'Ankara')]
cursor.executemany(
    "INSERT INTO Musteriler (ID, Ad, Sehir) VALUES (?, ?, ?)",
    data
)
conn.commit()
```

### Context Manager
```python
from contextlib import contextmanager

@contextmanager
def get_connection():
    conn = pyodbc.connect(CONNECTION_STRING)
    try:
        yield conn
        conn.commit()
    except Exception:
        conn.rollback()
        raise
    finally:
        conn.close()

with get_connection() as conn:
    cursor = conn.cursor()
    cursor.execute("UPDATE Musteriler SET Aktif = 0 WHERE MusteriID = ?", (5,))
```

## 💡 Bağlantılar
- [[DS - SQL ve Veritabanı Kullanımı]]
- [[MSSQL - Stored Procedure]]
- [[DS - Pandas Veri Okuma ve Yazma]]

## ❓ Sorular / Anlamadıklarım
- Windows Authentication ile bağlantı nasıl kurulur?
- Büyük veri transferinde to_sql yavaş olursa ne yapmalıyım?

## 🔗 Kaynaklar
- https://learn.microsoft.com/tr-tr/sql/connect/python/pyodbc/python-sql-driver-pyodbc
- https://docs.sqlalchemy.org/en/20/dialects/mssql.html
