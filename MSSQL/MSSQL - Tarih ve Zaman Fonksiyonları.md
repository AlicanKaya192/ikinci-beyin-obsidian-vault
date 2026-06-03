---
tarih: 2026-05-28
konu: MSSQL
etiket: ["mssql", "tarih", "datetime", "fonksiyon"]
kaynak: Microsoft Dokümantasyon
zorluk: başlangıç
---

## 📌 Özet
T-SQL'de tarih ve zaman işlemleri çok sık kullanılır. Raporlarda dönem filtreleme, süre hesaplama ve formatlama işlemleri için zengin fonksiyon seti sunar.

## 🧠 Detay

### Güncel Tarih/Saat
```sql
SELECT
    GETDATE()          AS SimdikiZaman,       -- 2026-05-28 14:30:00.123
    GETUTCDATE()       AS UTCZaman,
    SYSDATETIME()      AS YuksekPrecision,    -- milisaniye hassas
    CAST(GETDATE() AS DATE)    AS SadeceTarih,
    CAST(GETDATE() AS TIME)    AS SadeceSaat
```

### Tarih Parçaları
```sql
SELECT
    YEAR(GETDATE())    AS Yil,
    MONTH(GETDATE())   AS Ay,
    DAY(GETDATE())     AS Gun,
    DATEPART(WEEK, GETDATE())     AS HaftaNo,
    DATEPART(WEEKDAY, GETDATE())  AS HaftaGunu,    -- 1=Pazar
    DATEPART(HOUR, GETDATE())     AS Saat,
    DATENAME(MONTH, GETDATE())    AS AyAdi,         -- May
    DATENAME(WEEKDAY, GETDATE())  AS GunAdi         -- Wednesday
```

### Tarih Hesaplama
```sql
SELECT
    DATEADD(DAY, 7, GETDATE())     AS BirHaftaSonra,
    DATEADD(MONTH, -1, GETDATE())  AS BirAyOnce,
    DATEADD(YEAR, 1, GETDATE())    AS BirYilSonra,
    DATEDIFF(DAY, '2024-01-01', GETDATE())    AS GecenGun,
    DATEDIFF(MONTH, '2024-01-01', GETDATE())  AS GecenAy,
    DATEDIFF(YEAR, DogumTarihi, GETDATE())    AS Yas
FROM Musteriler
```

### Tarih Dönüşümü
```sql
-- String → Date
SELECT CAST('2026-05-28' AS DATE)
SELECT CONVERT(DATE, '28.05.2026', 104)   -- 104 = Türk formatı

-- Date → String
SELECT FORMAT(GETDATE(), 'dd.MM.yyyy')             -- 28.05.2026
SELECT CONVERT(VARCHAR(10), GETDATE(), 104)        -- 28.05.2026
SELECT FORMAT(GETDATE(), 'dd MMMM yyyy', 'tr-TR')  -- 28 Mayıs 2026
```

### Dönem Filtreleri
```sql
-- Bu ay
WHERE SiparisTarih >= DATEADD(MONTH, DATEDIFF(MONTH, 0, GETDATE()), 0)
  AND SiparisTarih <  DATEADD(MONTH, DATEDIFF(MONTH, 0, GETDATE()) + 1, 0)

-- Bu yıl
WHERE YEAR(SiparisTarih) = YEAR(GETDATE())

-- Son 30 gün
WHERE SiparisTarih >= DATEADD(DAY, -30, GETDATE())

-- Ayın ilk ve son günü
SELECT
    DATEADD(MONTH, DATEDIFF(MONTH, 0, GETDATE()), 0)     AS AyinIlkGunu,
    EOMONTH(GETDATE())                                     AS AyinSonGunu
```

### Çalışma Günü Hesaplama
```sql
-- İki tarih arası hafta içi gün sayısı
SELECT
    (DATEDIFF(DAY, @Baslangic, @Bitis) + 1)
    - (DATEDIFF(WEEK, @Baslangic, @Bitis) * 2)
    - CASE WHEN DATEPART(WEEKDAY, @Baslangic) = 1 THEN 1 ELSE 0 END
    - CASE WHEN DATEPART(WEEKDAY, @Bitis) = 7 THEN 1 ELSE 0 END
    AS CalismaGunSayisi
```

## 💡 Bağlantılar
- [[MSSQL - Temel SQL Komutları]]
- [[MSSQL - String Fonksiyonları]]
- [[MSSQL - Analitik Sorgular]]

## ❓ Sorular / Anlamadıklarım
- GETDATE() ile SYSDATETIME() arasındaki fark ne zaman önemli?
- Tarih sütunlarında index neden önemlidir?

## 🔗 Kaynaklar
- https://learn.microsoft.com/tr-tr/sql/t-sql/functions/date-and-time-data-types-and-functions-transact-sql
