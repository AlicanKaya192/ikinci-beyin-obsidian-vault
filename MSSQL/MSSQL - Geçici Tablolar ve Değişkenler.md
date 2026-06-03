---
tarih: 2026-05-28
konu: MSSQL
etiket: ["mssql", "geçici-tablo", "temp-table", "değişken", "table-variable"]
kaynak: Microsoft Dokümantasyon
zorluk: orta
---

## 📌 Özet
Geçici tablolar ve tablo değişkenleri ara sonuçları saklamak için kullanılır. Doğru seçim performansı önemli ölçüde etkiler.

## 🧠 Detay

### Değişken Türleri
```sql
-- Scalar değişken
DECLARE @Ad       NVARCHAR(50) = 'Ahmet'
DECLARE @Tutar    DECIMAL(10,2)
DECLARE @Tarih    DATE = GETDATE()
DECLARE @Sayac    INT = 0

SET @Tutar = 1500.00
SELECT @Sayac = COUNT(*) FROM Musteriler

PRINT 'Müşteri sayısı: ' + CAST(@Sayac AS VARCHAR)
```

### Tablo Değişkeni (@table)
```sql
DECLARE @SonucTablosu TABLE (
    MusteriID   INT,
    TamAd       NVARCHAR(100),
    ToplamTutar DECIMAL(10,2)
)

INSERT INTO @SonucTablosu
SELECT
    m.MusteriID,
    m.Ad + ' ' + m.Soyad,
    SUM(s.Tutar)
FROM Musteriler m
INNER JOIN Siparisler s ON m.MusteriID = s.MusteriID
GROUP BY m.MusteriID, m.Ad, m.Soyad

SELECT * FROM @SonucTablosu ORDER BY ToplamTutar DESC
```

### Geçici Tablo (#temp)
```sql
-- Lokal geçici tablo (oturuma özgü)
CREATE TABLE #AktifMusteriler (
    MusteriID INT,
    TamAd     NVARCHAR(100),
    Sehir     NVARCHAR(50)
)

INSERT INTO #AktifMusteriler
SELECT MusteriID, Ad + ' ' + Soyad, Sehir
FROM Musteriler
WHERE Aktif = 1

-- Index eklenebilir!
CREATE INDEX IX_Temp_Sehir ON #AktifMusteriler (Sehir)

SELECT * FROM #AktifMusteriler WHERE Sehir = 'İstanbul'

DROP TABLE IF EXISTS #AktifMusteriler
```

### Global Geçici Tablo (##temp)
```sql
-- Tüm oturumlardan erişilebilir
CREATE TABLE ##PaylasilanSonuc (ID INT, Deger NVARCHAR(100))
-- Tüm bağlantılar kapanınca otomatik silinir
```

### SELECT INTO
```sql
-- Hızlı geçici tablo oluştur
SELECT MusteriID, Ad, Soyad, Email
INTO #FiltreliMusteriler
FROM Musteriler
WHERE Sehir = 'İstanbul' AND Aktif = 1
```

### Karşılaştırma
| Özellik | @TableVar | #TempTable |
|---------|-----------|------------|
| Kapsam | Batch | Oturum |
| Index | Sınırlı | Tam destek |
| İstatistik | Yok | Var |
| Rollback | Etkilenir | Etkilenmez |
| Büyük veri | ❌ | ✅ |

### Dinamik SQL
```sql
DECLARE @Tablo   NVARCHAR(100) = 'Musteriler'
DECLARE @Filtre  NVARCHAR(100) = 'İstanbul'
DECLARE @SQL     NVARCHAR(MAX)

SET @SQL = N'SELECT * FROM ' + QUOTENAME(@Tablo) +
           N' WHERE Sehir = @SehirParam'

EXEC sp_executesql @SQL,
    N'@SehirParam NVARCHAR(100)',
    @SehirParam = @Filtre
```

## 💡 Bağlantılar
- [[MSSQL - Stored Procedure]]
- [[MSSQL - Değişkenler ve Kontrol Akışı]]
- [[MSSQL - Index ve Performans]]

## ❓ Sorular / Anlamadıklarım
- Tablo değişkeni mi geçici tablo mu? Karar kriteri nedir?
- sp_executesql neden EXEC'ten güvenli?

## 🔗 Kaynaklar
- https://learn.microsoft.com/tr-tr/sql/t-sql/data-types/table-transact-sql
