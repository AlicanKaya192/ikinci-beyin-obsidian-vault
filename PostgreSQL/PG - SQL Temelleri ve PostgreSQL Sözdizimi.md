---
tarih: 2026-08-17
konu: PostgreSQL
etiket: [postgresql, sql, temel, select, join, subquery, başlangıç]
kaynak: PostgreSQL Tutorial
zorluk: başlangıç
---

## 📌 Özet

PostgreSQL'de SQL temelleri: SELECT, JOIN, GROUP BY, subquery, CASE ve temel DDL komutları. MSSQL bilgisi olanlar için öne çıkan farklar vurgulanmıştır. Vault'ta MSSQL notları detaylı olduğundan bu not PostgreSQL'e özgü noktalara odaklanır.

---

## 🧠 Detay

### DDL — Tablo Oluşturma

```sql
-- PostgreSQL'e özgü tipler
CREATE TABLE ml_sonuclari (
    id          BIGSERIAL PRIMARY KEY,           -- Auto-increment
    tahmin_id   UUID DEFAULT gen_random_uuid(),  -- UUID
    model_adi   VARCHAR(100) NOT NULL,
    tahmin      FLOAT8 NOT NULL,                 -- DOUBLE PRECISION
    gercek      FLOAT8,
    meta_veri   JSONB,                           -- Native JSON
    etiketler   TEXT[],                          -- Array
    tarih       TIMESTAMPTZ DEFAULT NOW(),       -- Timezone-aware
    CONSTRAINT ck_tahmin CHECK (tahmin BETWEEN 0 AND 1)
);

-- Enum tip
CREATE TYPE model_durumu AS ENUM ('taslak', 'egitimde', 'aktif', 'pasif');

ALTER TABLE ml_sonuclari
    ADD COLUMN durum model_durumu DEFAULT 'taslak';
```

### SELECT Temelleri

```sql
-- Temel sorgular
SELECT * FROM kullanicilar;
SELECT id, email, ad FROM kullanicilar;
SELECT DISTINCT sehir FROM kullanicilar;

-- Koşullar
SELECT * FROM kullanicilar
WHERE aktif = true
  AND kayit_tarihi >= '2024-01-01'
  AND sehir IN ('İstanbul', 'Ankara', 'İzmir');

-- LIKE / ILIKE (case-insensitive)
SELECT * FROM kullanicilar WHERE email ILIKE '%@gmail.com';
SELECT * FROM kullanicilar WHERE ad LIKE 'Ali%';

-- IS NULL / IS NOT NULL
SELECT * FROM kullanicilar WHERE telefon IS NULL;

-- BETWEEN
SELECT * FROM siparisler WHERE tutar BETWEEN 100 AND 500;

-- LIMIT ve OFFSET (sayfalama)
SELECT * FROM urunler ORDER BY fiyat DESC LIMIT 10 OFFSET 20;
```

### Aggregate Fonksiyonlar

```sql
SELECT
    kategori,
    COUNT(*) AS urun_sayisi,
    SUM(stok) AS toplam_stok,
    AVG(fiyat) AS ort_fiyat,
    MIN(fiyat) AS min_fiyat,
    MAX(fiyat) AS max_fiyat,
    STDDEV(fiyat) AS fiyat_std,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY fiyat) AS medyan
FROM urunler
GROUP BY kategori
HAVING COUNT(*) > 5
ORDER BY ort_fiyat DESC;
```

### JOIN Türleri

```sql
-- INNER JOIN
SELECT s.id, k.email, s.tutar
FROM siparisler s
INNER JOIN kullanicilar k ON s.kullanici_id = k.id;

-- LEFT JOIN (eşleşmeyenler de gelir — NULL ile)
SELECT k.email, COUNT(s.id) AS siparis_sayisi
FROM kullanicilar k
LEFT JOIN siparisler s ON k.id = s.kullanici_id
GROUP BY k.email;

-- FULL OUTER JOIN
SELECT k.email, s.id AS siparis_id
FROM kullanicilar k
FULL OUTER JOIN siparisler s ON k.id = s.kullanici_id;

-- CROSS JOIN (kartezyen çarpım)
SELECT a.renk, b.boyut FROM renkler a CROSS JOIN boyutlar b;

-- SELF JOIN (hiyerarşi)
SELECT c.ad AS calisan, m.ad AS yonetici
FROM calisanlar c
LEFT JOIN calisanlar m ON c.yonetici_id = m.id;
```

### Subquery

```sql
-- WHERE'de subquery
SELECT * FROM kullanicilar
WHERE id IN (
    SELECT DISTINCT kullanici_id FROM siparisler
    WHERE tarih >= '2025-01-01'
);

-- FROM'da subquery (derived table)
SELECT ort_siparis.kullanici_id, ort_siparis.ort_tutar
FROM (
    SELECT kullanici_id, AVG(tutar) AS ort_tutar
    FROM siparisler
    GROUP BY kullanici_id
) AS ort_siparis
WHERE ort_tutar > 500;

-- Correlated subquery
SELECT k.email,
    (SELECT COUNT(*) FROM siparisler s WHERE s.kullanici_id = k.id) AS siparis_sayisi
FROM kullanicilar k;

-- EXISTS
SELECT * FROM kullanicilar k
WHERE EXISTS (
    SELECT 1 FROM siparisler s
    WHERE s.kullanici_id = k.id AND s.tutar > 1000
);
```

### CASE

```sql
-- Arama CASE
SELECT
    ad,
    puan,
    CASE
        WHEN puan >= 90 THEN 'Mükemmel'
        WHEN puan >= 70 THEN 'İyi'
        WHEN puan >= 50 THEN 'Orta'
        ELSE 'Zayıf'
    END AS degerlendirme
FROM ogrenciler;

-- Basit CASE
SELECT
    durum,
    CASE durum
        WHEN 'aktif' THEN '✅'
        WHEN 'pasif' THEN '❌'
        ELSE '❓'
    END AS ikon
FROM kullanicilar;

-- Aggregate içinde CASE (koşullu sayma)
SELECT
    COUNT(*) FILTER (WHERE aktif = true) AS aktif_sayi,
    COUNT(*) FILTER (WHERE aktif = false) AS pasif_sayi
    -- PostgreSQL'e özgü FILTER sözdizimi — MSSQL'de yok
FROM kullanicilar;
```

### String ve Tarih Fonksiyonları

```sql
-- String
SELECT
    UPPER(email), LOWER(ad),
    TRIM('  metin  '),
    LPAD('42', 5, '0'),        -- '00042'
    CONCAT(ad, ' ', soyad),    -- veya ad || ' ' || soyad
    SPLIT_PART('a@b.com', '@', 2),  -- 'b.com'
    REGEXP_REPLACE(metin, '\d+', 'X')
FROM kullanicilar;

-- Tarih
SELECT
    NOW(),
    CURRENT_DATE,
    DATE_TRUNC('month', NOW()),          -- ayın başı
    DATE_PART('dow', NOW()),             -- haftanın günü (0=Pazar)
    EXTRACT(YEAR FROM siparis_tarihi),
    siparis_tarihi + INTERVAL '30 days',
    AGE(siparis_tarihi),                 -- şu ana kadar geçen süre
    TO_CHAR(siparis_tarihi, 'YYYY-MM-DD HH24:MI')
FROM siparisler;
```

### INSERT, UPDATE, DELETE

```sql
-- INSERT ... RETURNING (MSSQL'de OUTPUT)
INSERT INTO kullanicilar (email, ad)
VALUES ('ali@example.com', 'Ali')
RETURNING id, email;

-- UPSERT (INSERT ON CONFLICT)
INSERT INTO kullanicilar (email, ad)
VALUES ('ali@example.com', 'Ali Yeni')
ON CONFLICT (email)
DO UPDATE SET
    ad = EXCLUDED.ad,
    guncelleme = NOW();

-- UPDATE ... RETURNING
UPDATE siparisler
SET durum = 'gönderildi'
WHERE id = 42
RETURNING id, durum, guncelleme;

-- DELETE ile JOIN (PostgreSQL sözdizimi)
DELETE FROM siparis_satirlari ss
USING siparisler s
WHERE ss.siparis_id = s.id
  AND s.iptal = true;
```

---

## 💡 Bağlantılar
- [[PG - İleri SQL — Window Functions ve CTE]]
- [[MSSQL - SQL Temel Sorgular (SELECT, WHERE, ORDER BY, TOP)]]
- [[MSSQL - JOIN İşlemleri (INNER, LEFT, RIGHT, FULL OUTER)]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- [PostgreSQL Tutorial](https://www.postgresqltutorial.com/)
- [SQLZoo - PostgreSQL](https://sqlzoo.net/)
