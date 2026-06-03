---
tarih: 2026-05-28
konu: MSSQL
etiket: ["mssql", "stored-procedure", "sp", "prosedür"]
kaynak: Microsoft Dokümantasyon
zorluk: orta
---

## 📌 Özet
Stored Procedure, sunucuda saklanan ve tekrar kullanılabilen SQL kod bloklarıdır. Performans, güvenlik ve kod tekrarını önleme açısından avantajlıdır.

## 🧠 Detay

### Temel Stored Procedure
```sql
CREATE PROCEDURE sp_MusteriGetir
    @MusteriID INT
AS
BEGIN
    SET NOCOUNT ON   -- "N rows affected" mesajını kapat

    SELECT
        MusteriID, Ad, Soyad, Email, Sehir
    FROM Musteriler
    WHERE MusteriID = @MusteriID
      AND Aktif = 1
END
GO

-- Çalıştırma
EXEC sp_MusteriGetir @MusteriID = 5
EXEC sp_MusteriGetir 5
```

### Parametreler
```sql
CREATE PROCEDURE sp_SiparisAra
    @Baslangic    DATE = NULL,
    @Bitis        DATE = NULL,
    @MinTutar     DECIMAL(10,2) = 0,
    @SehirFiltre  NVARCHAR(50) = NULL
AS
BEGIN
    SET NOCOUNT ON

    SELECT s.SiparisNo, m.Ad, m.Soyad, s.Tutar, s.SiparisTarih
    FROM Siparisler s
    INNER JOIN Musteriler m ON s.MusteriID = m.MusteriID
    WHERE (@Baslangic IS NULL OR s.SiparisTarih >= @Baslangic)
      AND (@Bitis     IS NULL OR s.SiparisTarih <= @Bitis)
      AND s.Tutar >= @MinTutar
      AND (@SehirFiltre IS NULL OR m.Sehir = @SehirFiltre)
END
GO

EXEC sp_SiparisAra
    @Baslangic = '2024-01-01',
    @Bitis = '2024-12-31',
    @MinTutar = 500
```

### OUTPUT Parametresi
```sql
CREATE PROCEDURE sp_MusteriEkle
    @Ad        NVARCHAR(50),
    @Soyad     NVARCHAR(50),
    @Email     NVARCHAR(100),
    @YeniID    INT OUTPUT
AS
BEGIN
    INSERT INTO Musteriler (Ad, Soyad, Email)
    VALUES (@Ad, @Soyad, @Email)

    SET @YeniID = SCOPE_IDENTITY()
END
GO

-- Kullanım
DECLARE @YeniMusteriID INT
EXEC sp_MusteriEkle
    @Ad = 'Ahmet',
    @Soyad = 'Yılmaz',
    @Email = 'a@b.com',
    @YeniID = @YeniMusteriID OUTPUT

SELECT @YeniMusteriID AS OluşturulanID
```

### Hata Yönetimi
```sql
CREATE PROCEDURE sp_GüvenliIslem
    @MusteriID INT,
    @Tutar     DECIMAL(10,2)
AS
BEGIN
    SET NOCOUNT ON
    BEGIN TRY
        BEGIN TRANSACTION

        UPDATE Hesaplar SET Bakiye -= @Tutar
        WHERE MusteriID = @MusteriID

        INSERT INTO IslemLog (MusteriID, Tutar, Tarih)
        VALUES (@MusteriID, @Tutar, GETDATE())

        COMMIT TRANSACTION
    END TRY
    BEGIN CATCH
        ROLLBACK TRANSACTION
        THROW   -- hatayı yeniden fırlat
    END CATCH
END
```

### ALTER ve DROP
```sql
ALTER PROCEDURE sp_MusteriGetir
    @MusteriID INT
AS
BEGIN
    -- güncellenmiş kod
END

DROP PROCEDURE IF EXISTS sp_EskiProsedur
```

## 💡 Bağlantılar
- [[MSSQL - Fonksiyonlar ve UDF]]
- [[MSSQL - Transaction ve Hata Yönetimi]]
- [[MSSQL - Değişkenler ve Kontrol Akışı]]

## ❓ Sorular / Anlamadıklarım
- SET NOCOUNT ON neden önemli?
- SP ile view arasındaki temel fark nedir?

## 🔗 Kaynaklar
- https://learn.microsoft.com/tr-tr/sql/t-sql/statements/create-procedure-transact-sql
