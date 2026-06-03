---
tarih: 2026-05-28
konu: MSSQL
etiket: ["mssql", "transaction", "try-catch", "hata-yönetimi", "acid"]
kaynak: Microsoft Dokümantasyon
zorluk: orta
---

## 📌 Özet
Transaction, ya tamamen gerçekleşen ya da hiç gerçekleşmeyen işlem gruplarıdır. ACID prensiplerine dayanır. TRY/CATCH ile hata yönetimi sağlanır.

## 🧠 Detay

### ACID Prensipleri
```
Atomicity    → Ya hepsi ya hiçbiri
Consistency  → Veri bütünlüğü korunur
Isolation    → İşlemler birbirini etkilemez
Durability   → Commit sonrası kalıcıdır
```

### Temel Transaction
```sql
BEGIN TRANSACTION

    UPDATE Hesaplar SET Bakiye -= 1000 WHERE HesapID = 1
    UPDATE Hesaplar SET Bakiye += 1000 WHERE HesapID = 2

IF @@ERROR <> 0
    ROLLBACK TRANSACTION
ELSE
    COMMIT TRANSACTION
```

### TRY/CATCH ile Transaction
```sql
CREATE PROCEDURE sp_ParaTransferi
    @GonderenID INT,
    @AliciID    INT,
    @Tutar      DECIMAL(10,2)
AS
BEGIN
    SET NOCOUNT ON

    BEGIN TRY
        BEGIN TRANSACTION

            -- Bakiye kontrolü
            IF (SELECT Bakiye FROM Hesaplar WHERE HesapID = @GonderenID) < @Tutar
                THROW 50001, 'Yetersiz bakiye!', 1

            UPDATE Hesaplar SET Bakiye -= @Tutar WHERE HesapID = @GonderenID
            UPDATE Hesaplar SET Bakiye += @Tutar WHERE HesapID = @AliciID

            INSERT INTO IslemLog (GonderenID, AliciID, Tutar, Tarih)
            VALUES (@GonderenID, @AliciID, @Tutar, GETDATE())

        COMMIT TRANSACTION

    END TRY
    BEGIN CATCH
        IF @@TRANCOUNT > 0
            ROLLBACK TRANSACTION

        -- Hata bilgisi
        SELECT
            ERROR_NUMBER()    AS HataNo,
            ERROR_MESSAGE()   AS HataMesaji,
            ERROR_SEVERITY()  AS Siddet,
            ERROR_LINE()      AS Satir

        THROW   -- hatayı yeniden fırlat
    END CATCH
END
```

### SAVEPOINT
```sql
BEGIN TRANSACTION

    INSERT INTO Tablo1 VALUES (1)
    SAVE TRANSACTION Nokta1   -- kayıt noktası

    INSERT INTO Tablo2 VALUES (2)

    -- Sadece Nokta1'den bu yana geri al
    ROLLBACK TRANSACTION Nokta1

    -- Tablo1 insert korunur

COMMIT TRANSACTION
```

### İzolasyon Seviyeleri
```sql
-- Dirty read engelle (varsayılan)
SET TRANSACTION ISOLATION LEVEL READ COMMITTED

-- En yüksek izolasyon
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE

-- Snapshot izolasyon (row versioning)
SET TRANSACTION ISOLATION LEVEL SNAPSHOT

-- Kilitlenmeyi önlemek için hint
SELECT * FROM Musteriler WITH (NOLOCK)    -- dirty read kabul
SELECT * FROM Musteriler WITH (READPAST)  -- kilitli satırları atla
```

### Deadlock İzleme
```sql
-- Aktif kilitler
SELECT * FROM sys.dm_exec_requests WHERE blocking_session_id > 0

-- Deadlock trace flag
DBCC TRACEON(1222, -1)
```

## 💡 Bağlantılar
- [[MSSQL - Stored Procedure]]
- [[MSSQL - Index ve Performans]]
- [[MSSQL - Değişkenler ve Kontrol Akışı]]

## ❓ Sorular / Anlamadıklarım
- NOLOCK hint ne zaman güvenli, ne zaman riskli?
- Deadlock nasıl çözülür?

## 🔗 Kaynaklar
- https://learn.microsoft.com/tr-tr/sql/t-sql/language-elements/transactions-transact-sql
