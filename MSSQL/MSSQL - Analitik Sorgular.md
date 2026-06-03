---
tarih: 2026-05-28
konu: MSSQL
etiket: ["mssql", "analitik", "raporlama", "pivot", "data-analysis"]
kaynak: Microsoft Dokümantasyon
zorluk: ileri
---

## 📌 Özet
MSSQL'in analitik sorgu özellikleri — PIVOT, UNPIVOT, pencere fonksiyonları ve veri analizi teknikleri — iş zekası ve raporlama için güçlü araçlar sunar.

## 🧠 Detay

### PIVOT
```sql
-- Satırları sütuna çevir
SELECT *
FROM (
    SELECT Kategori, Ay, Satis
    FROM AylikSatislar
) AS Kaynak
PIVOT (
    SUM(Satis)
    FOR Ay IN ([Ocak], [Subat], [Mart], [Nisan],
               [Mayis], [Haziran])
) AS PivotSonuc

-- Dinamik PIVOT
DECLARE @Sutunlar NVARCHAR(MAX) = ''
DECLARE @SQL NVARCHAR(MAX)

SELECT @Sutunlar += ',' + QUOTENAME(Ay)
FROM (SELECT DISTINCT Ay FROM AylikSatislar) t

SET @Sutunlar = STUFF(@Sutunlar, 1, 1, '')

SET @SQL = '
SELECT * FROM (
    SELECT Kategori, Ay, Satis FROM AylikSatislar
) src
PIVOT (SUM(Satis) FOR Ay IN (' + @Sutunlar + ')) pvt'

EXEC sp_executesql @SQL
```

### UNPIVOT
```sql
-- Sütunları satıra çevir
SELECT Musteri, Donem, Satis
FROM SatisTabloPivot
UNPIVOT (
    Satis FOR Donem IN (Q1, Q2, Q3, Q4)
) AS UnpivotSonuc
```

### Büyüme Analizi
```sql
WITH AylikSatis AS (
    SELECT
        YEAR(SiparisTarih)  AS Yil,
        MONTH(SiparisTarih) AS Ay,
        SUM(Tutar) AS ToplamSatis
    FROM Siparisler
    GROUP BY YEAR(SiparisTarih), MONTH(SiparisTarih)
)
SELECT
    Yil, Ay, ToplamSatis,
    LAG(ToplamSatis) OVER (ORDER BY Yil, Ay) AS OncekiAy,
    ToplamSatis - LAG(ToplamSatis) OVER (ORDER BY Yil, Ay) AS Degisim,
    ROUND(100.0 * (ToplamSatis - LAG(ToplamSatis) OVER (ORDER BY Yil, Ay))
        / NULLIF(LAG(ToplamSatis) OVER (ORDER BY Yil, Ay), 0), 2) AS YuzdeDegisim
FROM AylikSatis
```

### ABC Analizi
```sql
WITH UrunSatis AS (
    SELECT UrunID, SUM(Tutar) AS ToplamSatis
    FROM SiparisDetay
    GROUP BY UrunID
),
YuzdeHesap AS (
    SELECT UrunID, ToplamSatis,
        SUM(ToplamSatis) OVER (ORDER BY ToplamSatis DESC
            ROWS UNBOUNDED PRECEDING) AS KumulatifSatis,
        SUM(ToplamSatis) OVER () AS GenelToplam
    FROM UrunSatis
)
SELECT UrunID, ToplamSatis,
    ROUND(100.0 * KumulatifSatis / GenelToplam, 2) AS KumulatifYuzde,
    CASE
        WHEN KumulatifSatis / GenelToplam <= 0.80 THEN 'A'
        WHEN KumulatifSatis / GenelToplam <= 0.95 THEN 'B'
        ELSE 'C'
    END AS ABCSinif
FROM YuzdeHesap
ORDER BY ToplamSatis DESC
```

### Cohort Analizi
```sql
-- Müşteri kayıt ayına göre sipariş davranışı
WITH IlkSiparis AS (
    SELECT MusteriID, MIN(SiparisTarih) AS IlkSiparisTarih,
        DATEFROMPARTS(YEAR(MIN(SiparisTarih)), MONTH(MIN(SiparisTarih)), 1) AS Cohort
    FROM Siparisler GROUP BY MusteriID
)
SELECT
    FORMAT(i.Cohort, 'yyyy-MM') AS CohortAyi,
    DATEDIFF(MONTH, i.Cohort, DATEFROMPARTS(YEAR(s.SiparisTarih), MONTH(s.SiparisTarih), 1)) AS Ay,
    COUNT(DISTINCT s.MusteriID) AS AktifMusteri
FROM IlkSiparis i
INNER JOIN Siparisler s ON i.MusteriID = s.MusteriID
GROUP BY i.Cohort, DATEDIFF(MONTH, i.Cohort, DATEFROMPARTS(YEAR(s.SiparisTarih), MONTH(s.SiparisTarih), 1))
ORDER BY i.Cohort, Ay
```

## 💡 Bağlantılar
- [[MSSQL - Pencere Fonksiyonları]]
- [[MSSQL - Alt Sorgular ve CTE]]
- [[MSSQL - Agregasyon ve GROUP BY]]

## ❓ Sorular / Anlamadıklarım
- Dinamik PIVOT SQL injection'a karşı nasıl korunur?
- Cohort analizini daha verimli yapmanın yolu?

## 🔗 Kaynaklar
- https://learn.microsoft.com/tr-tr/sql/t-sql/queries/from-using-pivot-and-unpivot
