---
tarih: 2026-08-17
konu: PostgreSQL
etiket: [postgresql, mssql, sql-server, karşılaştırma, geçiş, orta]
kaynak: PostgreSQL Docs, Microsoft SQL Docs
zorluk: orta
---

## 📌 Özet

Vault'ta MSSQL kapsamlı ele alınıyor. Bu not, MSSQL bilen birinin PostgreSQL'e geçişini kolaylaştırmak için iki sistemin sözdizimi, özellik ve felsefe farklarını karşılaştırır.

---

## 🧠 Detay

### Genel Karşılaştırma

| Özellik | PostgreSQL | MSSQL |
|---------|-----------|-------|
| **Lisans** | Açık kaynak (BSD) | Ticari (Microsoft) |
| **Maliyet** | Ücretsiz | Pahalı (Enterprise: $15K+/çekirdek) |
| **Platform** | Linux, Windows, macOS | Windows öncelikli, Linux var |
| **JSON** | JSONB (Native, index) | JSON (metin, sınırlı) |
| **Array** | Native array tipi | Yok (tablo tabanlı çözüm) |
| **Extensibility** | Extension sistemi | CLR integration |
| **Full-text** | Dahili (tsvector) | Full-text index (güçlü) |
| **Partitioning** | Native (pg10+) | Partition table |
| **Community** | Çok güçlü | Microsoft desteği |
| **Cloud** | RDS, Aurora, Supabase | Azure SQL, AWS RDS |

---

### Sözdizimi Farkları

```sql
-- ── TABLO OLUŞTURMA ──

-- MSSQL
CREATE TABLE kullanicilar (
    id INT IDENTITY(1,1) PRIMARY KEY,
    email NVARCHAR(255) NOT NULL,
    ad NVARCHAR(100),
    olusturma DATETIME2 DEFAULT GETDATE()
);

-- PostgreSQL
CREATE TABLE kullanicilar (
    id SERIAL PRIMARY KEY,          -- veya BIGSERIAL, IDENTITY
    email VARCHAR(255) NOT NULL,    -- TEXT de kullanılabilir
    ad VARCHAR(100),
    olusturma TIMESTAMPTZ DEFAULT NOW()
);
```

```sql
-- ── STRING FONKSİYONLARI ──

-- MSSQL           → PostgreSQL
LEN(metin)         → LENGTH(metin)  veya  char_length(metin)
CHARINDEX('a', s)  → POSITION('a' IN s)  veya  strpos(s, 'a')
SUBSTRING(s,1,3)   → SUBSTRING(s FROM 1 FOR 3)
ISNULL(val, def)   → COALESCE(val, def)
GETDATE()          → NOW()  veya  CURRENT_TIMESTAMP
TOP 10             → LIMIT 10
NOLOCK (hint)      → yok (MVCC gerek yok)
```

```sql
-- ── DATE FONKSİYONLARI ──

-- MSSQL
SELECT DATEADD(day, -7, GETDATE())
SELECT DATEDIFF(day, baslangic, bitis)
SELECT YEAR(tarih), MONTH(tarih), DAY(tarih)

-- PostgreSQL
SELECT NOW() - INTERVAL '7 days'
SELECT tarih2 - tarih1   -- gün olarak integer döner
SELECT EXTRACT(YEAR FROM tarih), EXTRACT(MONTH FROM tarih)
-- veya
SELECT DATE_PART('year', tarih)
SELECT DATE_TRUNC('month', tarih)   -- ayın başına yuvarla
```

```sql
-- ── CONDITIONAL ──

-- MSSQL
SELECT IIF(skor > 50, 'Geçti', 'Kaldı')

-- PostgreSQL (IIF yok)
SELECT CASE WHEN skor > 50 THEN 'Geçti' ELSE 'Kaldı' END
-- veya
SELECT (CASE skor > 50 WHEN true THEN 'Geçti' ELSE 'Kaldı' END)
```

```sql
-- ── STRING AGGREGATİON ──

-- MSSQL
SELECT STRING_AGG(ad, ', ') WITHIN GROUP (ORDER BY ad)

-- PostgreSQL
SELECT STRING_AGG(ad, ', ' ORDER BY ad)
-- veya
SELECT ARRAY_TO_STRING(ARRAY_AGG(ad ORDER BY ad), ', ')
```

```sql
-- ── PIVOT / CROSSTAB ──

-- MSSQL PIVOT
SELECT *
FROM (SELECT ay, kategori, tutar FROM satislar) src
PIVOT (SUM(tutar) FOR kategori IN ([Elektronik], [Giyim])) pvt;

-- PostgreSQL (tablefunc extension)
CREATE EXTENSION IF NOT EXISTS tablefunc;
SELECT * FROM crosstab(
    'SELECT ay, kategori, SUM(tutar) FROM satislar GROUP BY 1,2 ORDER BY 1,2'
) AS ct(ay TEXT, elektronik NUMERIC, giyim NUMERIC);

-- veya FILTER ile manuel pivot
SELECT
    ay,
    SUM(tutar) FILTER (WHERE kategori = 'Elektronik') AS elektronik,
    SUM(tutar) FILTER (WHERE kategori = 'Giyim') AS giyim
FROM satislar
GROUP BY ay;
```

### Stored Procedure Farkları

```sql
-- MSSQL
CREATE PROCEDURE aktif_kullanici_getir @gun_sayisi INT = 30
AS
BEGIN
    SELECT * FROM kullanicilar
    WHERE son_giris >= DATEADD(day, -@gun_sayisi, GETDATE());
END;

EXEC aktif_kullanici_getir @gun_sayisi = 7;

-- ────────────────────────────────

-- PostgreSQL (procedure veya function)
CREATE OR REPLACE FUNCTION aktif_kullanici_getir(gun_sayisi INT DEFAULT 30)
RETURNS TABLE(id INT, email TEXT, son_giris TIMESTAMPTZ)
LANGUAGE sql
AS $$
    SELECT id, email, son_giris
    FROM kullanicilar
    WHERE son_giris >= NOW() - (gun_sayisi || ' days')::INTERVAL;
$$;

SELECT * FROM aktif_kullanici_getir(7);
```

### Geçiş Araçları

```bash
# pgloader — MSSQL'den PostgreSQL'e veri geçişi
pgloader mssql://user:pass@host/database postgresql://user:pass@host/database

# Şema geçişi için önce:
# 1. MSSQL şemasını export et
# 2. Manuel uyarlama yap (data type, identity → serial, vb.)
# 3. pgloader ile veri taşı
```

### MSSQL'e Özgü — PostgreSQL Alternatifleri

| MSSQL Özellik | PostgreSQL Alternatif |
|---------------|----------------------|
| SQL Server Agent | pg_cron extension |
| SSRS (Raporlama) | Metabase, Superset, Grafana |
| SSIS (ETL) | dbt, Airbyte |
| Always On AG | Patroni + streaming replication |
| Linked Server | Foreign Data Wrapper (FDW) |
| XML support | XML functions (benzer) |
| Columnstore Index | TimescaleDB, cstore_fdw |

---

## 💡 Bağlantılar
- [[00 - PostgreSQL Giriş ve Yol Haritası]]
- [[MSSQL - SQL Temel Sorgular (SELECT, WHERE, ORDER BY, TOP)]]
- [[MSSQL - Window Functions ve CTE (Ortak Tablo İfadeleri)]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- [pgloader Documentation](https://pgloader.io/)
- [PostgreSQL vs SQL Server](https://www.enterprisedb.com/blog/microsoft-sql-server-mssql-vs-postgresql-comparison-2021-detailed)
