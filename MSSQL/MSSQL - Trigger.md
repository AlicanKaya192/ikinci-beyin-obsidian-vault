---
tarih: 2026-05-28
konu: MSSQL
etiket: ["mssql", "trigger", "tetikleyici", "dml-trigger"]
kaynak: Microsoft Dokümantasyon
zorluk: orta
---

## 📌 Özet
Trigger, tabloda INSERT, UPDATE veya DELETE işlemi gerçekleştiğinde otomatik çalışan özel stored procedure'dür. Audit log, veri bütünlüğü ve otomatik işlemler için kullanılır.

## 🧠 Detay

### AFTER Trigger (DML)
```sql
-- Değişiklik geçmişi kaydı
CREATE TRIGGER trg_Musteriler_Update
ON Musteriler
AFTER UPDATE
AS
BEGIN
    SET NOCOUNT ON

    INSERT INTO MusteriDegisiklikLog
        (MusteriID, EskiEmail, YeniEmail, DegistirmeTarih, DegistirenKullanici)
    SELECT
        d.MusteriID,
        d.Email AS EskiEmail,
        i.Email AS YeniEmail,
        GETDATE(),
        SYSTEM_USER
    FROM deleted d  -- eski değerler
    INNER JOIN inserted i ON d.MusteriID = i.MusteriID  -- yeni değerler
    WHERE d.Email <> i.Email   -- sadece email değiştiyse
END
GO
```

### INSTEAD OF Trigger
```sql
-- Silme işlemini engelleyip soft-delete yap
CREATE TRIGGER trg_Musteriler_SoftDelete
ON Musteriler
INSTEAD OF DELETE
AS
BEGIN
    SET NOCOUNT ON

    UPDATE Musteriler
    SET Aktif = 0, SilmeTarih = GETDATE()
    WHERE MusteriID IN (SELECT MusteriID FROM deleted)
END
```

### inserted ve deleted Tabloları
```sql
-- INSERT → sadece inserted dolu
-- DELETE → sadece deleted dolu
-- UPDATE → her ikisi de dolu

CREATE TRIGGER trg_Stok_Guncelle
ON SiparisDetay
AFTER INSERT, UPDATE, DELETE
AS
BEGIN
    -- Eklenen veya güncellenen kayıtlar
    UPDATE Urunler
    SET StokAdet -= i.Miktar
    FROM Urunler u
    INNER JOIN inserted i ON u.UrunID = i.UrunID

    -- Silinen veya güncelleme öncesi
    UPDATE Urunler
    SET StokAdet += d.Miktar
    FROM Urunler u
    INNER JOIN deleted d ON u.UrunID = d.UrunID
END
```

### DDL Trigger (Yapısal Değişiklikleri İzle)
```sql
CREATE TRIGGER trg_DDL_Izle
ON DATABASE
FOR DROP_TABLE, ALTER_TABLE
AS
BEGIN
    INSERT INTO DDLLog (Olay, KullaniciAdi, Tarih, TSQLKomut)
    VALUES (
        EVENTDATA().value('(/EVENT_INSTANCE/EventType)[1]', 'NVARCHAR(100)'),
        SYSTEM_USER,
        GETDATE(),
        EVENTDATA().value('(/EVENT_INSTANCE/TSQLCommand)[1]', 'NVARCHAR(MAX)')
    )
END
```

### Trigger Yönetimi
```sql
-- Devre dışı bırak
DISABLE TRIGGER trg_Musteriler_Update ON Musteriler

-- Aktif et
ENABLE TRIGGER trg_Musteriler_Update ON Musteriler

-- Sil
DROP TRIGGER IF EXISTS trg_Musteriler_Update

-- Listele
SELECT name, type_desc FROM sys.triggers
```

## 💡 Bağlantılar
- [[MSSQL - Stored Procedure]]
- [[MSSQL - Transaction ve Hata Yönetimi]]
- [[MSSQL - Tablo Oluşturma ve DDL]]

## ❓ Sorular / Anlamadıklarım
- Trigger performansı nasıl etkiler?
- AFTER ve INSTEAD OF trigger ne zaman hangisi?

## 🔗 Kaynaklar
- https://learn.microsoft.com/tr-tr/sql/t-sql/statements/create-trigger-transact-sql
