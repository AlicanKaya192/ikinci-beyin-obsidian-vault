---
tarih: 2026-05-28
konu: MSSQL
etiket: ["mssql", "index", "performans", "query-plan", "optimizasyon"]
kaynak: Microsoft Dokümantasyon
zorluk: ileri
---

## 📌 Özet
Index, sorgu performansını dramatik ölçüde artıran veri yapılarıdır. Doğru index tasarımı ve sorgu optimizasyonu production sistemlerde kritik öneme sahiptir.

## 🧠 Detay

### Index Türleri
```sql
-- Clustered Index → fiziksel sıralama (tablo başına 1)
CREATE CLUSTERED INDEX IX_Siparisler_Tarih
ON Siparisler (SiparisTarih)

-- Non-Clustered Index → ayrı yapı
CREATE NONCLUSTERED INDEX IX_Musteriler_Email
ON Musteriler (Email)

-- Composite Index (sıra önemli!)
CREATE INDEX IX_Siparisler_MusteriTarih
ON Siparisler (MusteriID, SiparisTarih)

-- Include → covering index
CREATE INDEX IX_Siparisler_Cover
ON Siparisler (MusteriID, SiparisTarih)
INCLUDE (Tutar, Durum)
```

### Execution Plan Okuma
```sql
-- Execution plan göster
SET STATISTICS IO ON
SET STATISTICS TIME ON

SELECT * FROM Siparisler WHERE MusteriID = 5

-- Actual Execution Plan: Ctrl+M
-- Estimated Plan: Ctrl+L

-- Önemli operatörler:
-- Index Seek → iyi (hızlı)
-- Index Scan → orta
-- Table Scan → kötü (yavaş, index yok)
-- Key Lookup → iyileştirilebilir (INCLUDE ile)
-- Hash Match → büyük join, index yok
```

### Eksik Index Tespiti
```sql
-- Sistem önerilen indexler
SELECT
    DB_NAME(mid.database_id) AS Veritabani,
    OBJECT_NAME(mid.object_id) AS Tablo,
    mid.equality_columns,
    mid.include_columns,
    migs.avg_user_impact AS TahminiKazanim
FROM sys.dm_db_missing_index_details mid
INNER JOIN sys.dm_db_missing_index_groups mig ON mid.index_handle = mig.index_handle
INNER JOIN sys.dm_db_missing_index_group_stats migs ON mig.index_group_handle = migs.group_handle
ORDER BY migs.avg_user_impact DESC
```

### Index Kullanım İstatistikleri
```sql
SELECT
    OBJECT_NAME(i.object_id) AS Tablo,
    i.name AS IndexAdi,
    ius.user_seeks, ius.user_scans, ius.user_lookups,
    ius.user_updates
FROM sys.indexes i
LEFT JOIN sys.dm_db_index_usage_stats ius
    ON i.object_id = ius.object_id AND i.index_id = ius.index_id
WHERE OBJECT_NAME(i.object_id) = 'Siparisler'
```

### Sorgu Optimizasyon İpuçları
```sql
-- ❌ Sütunda fonksiyon → index kullanılmaz
WHERE YEAR(SiparisTarih) = 2024

-- ✅ Aralık ile → index kullanılır
WHERE SiparisTarih >= '2024-01-01' AND SiparisTarih < '2025-01-01'

-- ❌ Wildcard başta → index kullanılmaz
WHERE Ad LIKE '%met'

-- ✅ Wildcard sonda → index kullanılır
WHERE Ad LIKE 'Ah%'

-- ❌ Implicit conversion
WHERE MusteriID = '5'   -- INT sütuna string

-- ✅ Doğru tip
WHERE MusteriID = 5
```

### Index Bakımı
```sql
-- Fragmentasyon kontrolü
SELECT index_id, avg_fragmentation_in_percent
FROM sys.dm_db_index_physical_stats(DB_ID(), OBJECT_ID('Siparisler'), NULL, NULL, 'SAMPLED')

-- %5-30 → REORGANIZE
ALTER INDEX IX_Siparisler_MusteriTarih ON Siparisler REORGANIZE

-- %30+ → REBUILD
ALTER INDEX ALL ON Siparisler REBUILD WITH (ONLINE = ON)
```

## 💡 Bağlantılar
- [[MSSQL - Tablo Oluşturma ve DDL]]
- [[MSSQL - Analitik Sorgular]]
- [[MSSQL - Alt Sorgular ve CTE]]

## ❓ Sorular / Anlamadıklarım
- Clustered index seçimi neden bu kadar önemli?
- Çok fazla index ne zaman sorun olur?

## 🔗 Kaynaklar
- https://learn.microsoft.com/tr-tr/sql/relational-databases/indexes/indexes
