---
tarih: 2026-08-17
konu: PostgreSQL
etiket: [postgresql, sql, window-functions, cte, analitik, orta]
kaynak: PostgreSQL Docs, Mode Analytics
zorluk: orta
---

## 📌 Özet

Window Functions ve CTE (Common Table Expressions), SQL'i analitik güçlendiren özelliklerdir. GROUP BY'ın aksine, Window Functions satırları gruplara indirgemez — her satırı ayrı tutar ve üzerine hesaplama yapar. DS mülakatlarında en sık sorulan SQL konularıdır.

---

## 🧠 Detay

### Window Functions Anatomisi

```sql
fonksiyon_adi(argüman) OVER (
    PARTITION BY bölümleme_sütunu
    ORDER BY sıralama_sütunu
    ROWS/RANGE BETWEEN başlangıç AND bitiş
)
```

| Parça | Açıklama |
|-------|----------|
| `OVER ()` | Window function olduğunu belirtir — zorunlu |
| `PARTITION BY` | Pencereyi bu sütuna göre böler (GROUP BY gibi ama satırları korur) |
| `ORDER BY` | Pencere içi sıralama — sıralı fonksiyonlar için gerekli |
| `ROWS BETWEEN` | Fiziksel satır aralığı |
| `RANGE BETWEEN` | Değer aralığı |

---

### Sıralama Fonksiyonları

```sql
-- Örnek tablo
CREATE TABLE satislar (
    id SERIAL,
    urun VARCHAR(50),
    kategori VARCHAR(30),
    tutar DECIMAL(10,2),
    tarih DATE
);

-- ROW_NUMBER: Benzersiz sıra numarası
SELECT
    urun,
    kategori,
    tutar,
    ROW_NUMBER() OVER (PARTITION BY kategori ORDER BY tutar DESC) AS sira
FROM satislar;

-- RANK: Aynı değere aynı sıra, sonraki atlar (1,1,3)
SELECT urun, tutar,
    RANK() OVER (ORDER BY tutar DESC) AS rank
FROM satislar;

-- DENSE_RANK: Aynı değere aynı sıra, atlamaz (1,1,2)
SELECT urun, tutar,
    DENSE_RANK() OVER (ORDER BY tutar DESC) AS dense_rank
FROM satislar;

-- NTILE: N eşit parçaya böl (yüzdelik dilimleme)
SELECT urun, tutar,
    NTILE(4) OVER (ORDER BY tutar) AS ceyrek  -- Q1, Q2, Q3, Q4
FROM satislar;
```

### Analitik Fonksiyonlar

```sql
-- LAG/LEAD: Önceki/sonraki satıra bak
SELECT
    tarih,
    tutar,
    LAG(tutar, 1) OVER (ORDER BY tarih) AS onceki_gun,
    LEAD(tutar, 1) OVER (ORDER BY tarih) AS sonraki_gun,
    tutar - LAG(tutar, 1) OVER (ORDER BY tarih) AS degisim
FROM satislar;

-- FIRST_VALUE / LAST_VALUE
SELECT
    kategori,
    urun,
    tutar,
    FIRST_VALUE(urun) OVER (
        PARTITION BY kategori ORDER BY tutar DESC
    ) AS en_pahali_urun
FROM satislar;

-- NTH_VALUE: N'inci değer
SELECT
    urun, tutar,
    NTH_VALUE(tutar, 2) OVER (
        PARTITION BY kategori ORDER BY tutar DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS ikinci_en_yuksek
FROM satislar;
```

### Kümülatif ve Kayan Ortalama

```sql
-- Kümülatif toplam (Running Total)
SELECT
    tarih,
    tutar,
    SUM(tutar) OVER (ORDER BY tarih) AS kumulatif_toplam
FROM satislar;

-- 7 günlük kayan ortalama
SELECT
    tarih,
    tutar,
    AVG(tutar) OVER (
        ORDER BY tarih
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS ort_7_gun
FROM satislar;

-- Kayan toplam — her kategoride
SELECT
    kategori,
    tarih,
    tutar,
    SUM(tutar) OVER (
        PARTITION BY kategori
        ORDER BY tarih
        ROWS UNBOUNDED PRECEDING
    ) AS kategori_kumulatif
FROM satislar;
```

### Percentile ve Quantile

```sql
-- Yüzdelik değer hesapla
SELECT
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY tutar) AS medyan,
    PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY tutar) AS q1,
    PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY tutar) AS q3,
    PERCENTILE_CONT(0.90) WITHIN GROUP (ORDER BY tutar) AS p90
FROM satislar;

-- Window ile yüzdelik dilim (her satır için)
SELECT
    urun,
    tutar,
    PERCENT_RANK() OVER (ORDER BY tutar) AS yuzdelik_konum,
    CUME_DIST() OVER (ORDER BY tutar) AS kumulatif_dagilim
FROM satislar;
```

---

### CTE — Common Table Expressions

```sql
-- Temel CTE
WITH aylik_ozet AS (
    SELECT
        DATE_TRUNC('month', tarih) AS ay,
        SUM(tutar) AS aylik_toplam,
        COUNT(*) AS islem_sayisi
    FROM satislar
    GROUP BY 1
)
SELECT
    ay,
    aylik_toplam,
    aylik_toplam - LAG(aylik_toplam) OVER (ORDER BY ay) AS degisim,
    ROUND(
        100.0 * (aylik_toplam - LAG(aylik_toplam) OVER (ORDER BY ay))
        / LAG(aylik_toplam) OVER (ORDER BY ay),
        2
    ) AS degisim_yuzde
FROM aylik_ozet;
```

### Zincirleme CTE (Birden fazla WITH)

```sql
WITH
-- Adım 1: Ham hesaplama
ham_veri AS (
    SELECT
        kullanici_id,
        COUNT(DISTINCT tarih) AS aktif_gun,
        SUM(tutar) AS toplam_harcama
    FROM islemler
    WHERE tarih >= CURRENT_DATE - INTERVAL '30 days'
    GROUP BY kullanici_id
),
-- Adım 2: Segmentasyon
segmentler AS (
    SELECT
        kullanici_id,
        aktif_gun,
        toplam_harcama,
        CASE
            WHEN aktif_gun >= 20 AND toplam_harcama >= 5000 THEN 'VIP'
            WHEN aktif_gun >= 10 THEN 'Aktif'
            WHEN aktif_gun >= 3 THEN 'Ziyaretçi'
            ELSE 'Uyuyan'
        END AS segment
    FROM ham_veri
)
-- Adım 3: Özet
SELECT
    segment,
    COUNT(*) AS kullanici_sayisi,
    AVG(aktif_gun) AS ort_aktif_gun,
    AVG(toplam_harcama) AS ort_harcama
FROM segmentler
GROUP BY segment
ORDER BY ort_harcama DESC;
```

### Recursive CTE — Hiyerarşik Veri

```sql
-- Organizasyon şeması (çalışan → yönetici)
CREATE TABLE calisanlar (
    id INT,
    ad VARCHAR(50),
    yonetici_id INT
);

-- Recursive CTE ile hiyerarşi
WITH RECURSIVE hiyerarsi AS (
    -- Base case: üst yöneticiler
    SELECT id, ad, yonetici_id, 0 AS seviye, ARRAY[id] AS yol
    FROM calisanlar
    WHERE yonetici_id IS NULL

    UNION ALL

    -- Recursive: altındakileri bul
    SELECT c.id, c.ad, c.yonetici_id, h.seviye + 1, h.yol || c.id
    FROM calisanlar c
    JOIN hiyerarsi h ON c.yonetici_id = h.id
    WHERE NOT c.id = ANY(h.yol)  -- döngü önleme
)
SELECT
    REPEAT('  ', seviye) || ad AS organizasyon_agaci,
    seviye
FROM hiyerarsi
ORDER BY yol;
```

### Pratik DS Sorusu — Cohort Analizi

```sql
-- Her kullanıcının ilk satın alma ayı = cohort
WITH ilk_alis AS (
    SELECT
        kullanici_id,
        DATE_TRUNC('month', MIN(tarih)) AS cohort_ay
    FROM siparisler
    GROUP BY kullanici_id
),
cohort_veri AS (
    SELECT
        i.cohort_ay,
        DATE_TRUNC('month', s.tarih) AS siparis_ay,
        COUNT(DISTINCT s.kullanici_id) AS kullanici_sayisi
    FROM siparisler s
    JOIN ilk_alis i ON s.kullanici_id = i.kullanici_id
    GROUP BY 1, 2
),
cohort_boyutu AS (
    SELECT cohort_ay, kullanici_sayisi AS baslangic_boyutu
    FROM cohort_veri
    WHERE cohort_ay = siparis_ay
)
SELECT
    cv.cohort_ay,
    EXTRACT(MONTH FROM AGE(cv.siparis_ay, cv.cohort_ay)) AS ay_sonra,
    cv.kullanici_sayisi,
    ROUND(100.0 * cv.kullanici_sayisi / cb.baslangic_boyutu, 1) AS retention_yuzde
FROM cohort_veri cv
JOIN cohort_boyutu cb ON cv.cohort_ay = cb.cohort_ay
ORDER BY 1, 2;
```

---

## 💡 Bağlantılar
- [[00 - PostgreSQL Giriş ve Yol Haritası]]
- [[MSSQL - Window Functions ve CTE (Ortak Tablo İfadeleri)]]
- [[STAT - A-B Testi Tasarımı ve Analizi]]
- [[Interview - Makine Öğrenmesi Temel Sorular]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- [Mode SQL Tutorial - Window Functions](https://mode.com/sql-tutorial/sql-window-functions/)
- [PostgreSQL Window Functions](https://www.postgresql.org/docs/current/tutorial-window.html)
- [Practical SQL - Anthony DeBarros](https://nostarch.com/practical-sql-2nd-edition)
