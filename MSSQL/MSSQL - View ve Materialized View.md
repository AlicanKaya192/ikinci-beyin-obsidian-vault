---
tarih: 2026-05-28
konu: MSSQL
etiket: ["mssql", "view", "görünüm", "indexed-view"]
kaynak: Microsoft Dokümantasyon
zorluk: orta
---

## 📌 Özet
View, karmaşık sorguları basitleştiren sanal tablolardır. Indexed view ise sonuçları fiziksel olarak saklayan MSSQL'e özgü özellikte performans sağlar.

## 🧠 Detay

### View Oluşturma
```sql
CREATE VIEW vw_MusteriSipariOzet
AS
SELECT
    m.MusteriID,
    m.Ad + ' ' + m.Soyad      AS TamAd,
    m.Sehir,
    COUNT(s.SiparisID)         AS SiparisSayisi,
    SUM(s.Tutar)               AS ToplamTutar,
    MAX(s.SiparisTarih)        AS SonSiparisTarih
FROM Musteriler m
LEFT JOIN Siparisler s ON m.MusteriID = s.MusteriID
WHERE m.Aktif = 1
GROUP BY m.MusteriID, m.Ad, m.Soyad, m.Sehir
GO

-- Kullanım
SELECT * FROM vw_MusteriSipariOzet WHERE Sehir = 'İstanbul'
SELECT TamAd, ToplamTutar FROM vw_MusteriSipariOzet ORDER BY ToplamTutar DESC
```

### View Güncelleme
```sql
-- Basit view'lar güncellenebilir (tek tablo, GROUP BY yok)
CREATE VIEW vw_AktifMusteriler
AS
SELECT MusteriID, Ad, Soyad, Email, Sehir
FROM Musteriler
WHERE Aktif = 1

-- Bu view üzerinden INSERT/UPDATE yapılabilir
UPDATE vw_AktifMusteriler SET Sehir = 'Ankara' WHERE MusteriID = 5

-- WITH CHECK OPTION → view koşulunu korur
CREATE VIEW vw_IstanbulMusteriler
AS
SELECT * FROM Musteriler WHERE Sehir = 'İstanbul'
WITH CHECK OPTION   -- başka şehir eklenemez
```

### Indexed View (Materialized View)
```sql
-- Zorunlu: SCHEMABINDING
CREATE VIEW vw_AylikSatis
WITH SCHEMABINDING
AS
SELECT
    YEAR(SiparisTarih)  AS Yil,
    MONTH(SiparisTarih) AS Ay,
    COUNT_BIG(*)        AS SiparisSayisi,   -- COUNT_BIG zorunlu
    SUM(Tutar)          AS ToplamTutar
FROM dbo.Siparisler   -- dbo. prefix zorunlu
GROUP BY YEAR(SiparisTarih), MONTH(SiparisTarih)
GO

-- Clustered index → fiziksel depolama
CREATE UNIQUE CLUSTERED INDEX IX_vw_AylikSatis
ON vw_AylikSatis (Yil, Ay)
```

### View Yönetimi
```sql
-- Değiştir
ALTER VIEW vw_MusteriSipariOzet
AS
-- yeni sorgu

-- Sil
DROP VIEW IF EXISTS vw_EskiGorunum

-- Şifrele (kaynak kodu gizle)
CREATE VIEW vw_Gizli WITH ENCRYPTION
AS SELECT ...

-- Bağımlılık listesi
SELECT * FROM sys.sql_expression_dependencies
WHERE referencing_id = OBJECT_ID('vw_MusteriSipariOzet')
```

## 💡 Bağlantılar
- [[MSSQL - Alt Sorgular ve CTE]]
- [[MSSQL - Index ve Performans]]
- [[MSSQL - Analitik Sorgular]]

## ❓ Sorular / Anlamadıklarım
- View ile CTE arasında ne zaman hangisi tercih edilir?
- Indexed view ne zaman otomatik kullanılır?

## 🔗 Kaynaklar
- https://learn.microsoft.com/tr-tr/sql/relational-databases/views/views
