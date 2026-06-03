---
tarih: 2026-05-28
konu: MSSQL
etiket: ["mssql", "fonksiyon", "udf", "scalar", "table-valued"]
kaynak: Microsoft Dokümantasyon
zorluk: orta
---

## 📌 Özet
Kullanıcı tanımlı fonksiyonlar (UDF), tekrar kullanılabilir hesaplama mantığı oluşturur. Scalar fonksiyonlar tek değer, table-valued fonksiyonlar tablo döndürür.

## 🧠 Detay

### Scalar Fonksiyon
```sql
-- Tek değer döndürür
CREATE FUNCTION fn_YasHesapla (@DogumTarihi DATE)
RETURNS INT
AS
BEGIN
    RETURN DATEDIFF(YEAR, @DogumTarihi, GETDATE())
        - CASE WHEN MONTH(@DogumTarihi) > MONTH(GETDATE())
                OR (MONTH(@DogumTarihi) = MONTH(GETDATE())
                AND DAY(@DogumTarihi) > DAY(GETDATE()))
               THEN 1 ELSE 0 END
END
GO

-- Kullanım
SELECT Ad, Soyad, dbo.fn_YasHesapla(DogumTarihi) AS Yas
FROM Musteriler

-- Örnek: TL formatlama
CREATE FUNCTION fn_ParaFormat (@Tutar DECIMAL(18,2))
RETURNS NVARCHAR(50)
AS
BEGIN
    RETURN FORMAT(@Tutar, 'N2', 'tr-TR') + ' ₺'
END
```

### Inline Table-Valued Fonksiyon
```sql
-- Tablo döndürür (en iyi performans)
CREATE FUNCTION fn_AktifMusteriler (@Sehir NVARCHAR(50) = NULL)
RETURNS TABLE
AS
RETURN (
    SELECT MusteriID, Ad, Soyad, Email, Sehir
    FROM Musteriler
    WHERE Aktif = 1
      AND (@Sehir IS NULL OR Sehir = @Sehir)
)
GO

-- Kullanım
SELECT * FROM dbo.fn_AktifMusteriler('İstanbul')
SELECT * FROM dbo.fn_AktifMusteriler(NULL)  -- hepsi

-- JOIN ile
SELECT m.Ad, s.SiparisNo
FROM dbo.fn_AktifMusteriler('Ankara') m
INNER JOIN Siparisler s ON m.MusteriID = s.MusteriID
```

### Multi-Statement Table-Valued Fonksiyon
```sql
CREATE FUNCTION fn_MusteriOzet (@MinSiparis INT)
RETURNS @Sonuc TABLE (
    MusteriID     INT,
    TamAd         NVARCHAR(100),
    SiparisSayisi INT,
    ToplamTutar   DECIMAL(10,2)
)
AS
BEGIN
    INSERT INTO @Sonuc
    SELECT
        m.MusteriID,
        m.Ad + ' ' + m.Soyad,
        COUNT(s.SiparisID),
        SUM(s.Tutar)
    FROM Musteriler m
    LEFT JOIN Siparisler s ON m.MusteriID = s.MusteriID
    GROUP BY m.MusteriID, m.Ad, m.Soyad
    HAVING COUNT(s.SiparisID) >= @MinSiparis

    RETURN
END
```

### SP vs Fonksiyon
| Özellik | SP | Fonksiyon |
|---------|----|-----------| 
| SELECT içinde kullanım | ❌ | ✅ |
| Veri değiştirme | ✅ | ❌ (scalar) |
| OUTPUT parametre | ✅ | ❌ |
| Hata yönetimi | TRY/CATCH | Sınırlı |

## 💡 Bağlantılar
- [[MSSQL - Stored Procedure]]
- [[MSSQL - Pencere Fonksiyonları]]
- [[MSSQL - String Fonksiyonları]]

## ❓ Sorular / Anlamadıklarım
- Scalar UDF performans sorunu neden yaratır?
- Inline TVF ile Multi-Statement TVF ne zaman hangisi?

## 🔗 Kaynaklar
- https://learn.microsoft.com/tr-tr/sql/t-sql/statements/create-function-transact-sql
