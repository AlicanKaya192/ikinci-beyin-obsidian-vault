# MSSQL - Veri Ambarı ve Columnstore Indexing

## 📌 Özet
Modern veri ambarı mimarilerinde analitik sorguların (OLAP) performansını artırmak için Columnstore Indexing teknolojisi kullanılmaktadır. Geleneksel satır bazlı (rowstore) depolamanın aksine, verileri sütun bazlı gruplandırarak yüksek sıkıştırma oranları ve hızlı toplama (aggregation) işlemleri sunar. Clustered Columnstore tüm tabloyu sütun bazlı saklarken, Non-clustered Columnstore hibrit bir yapı sunarak hem OLTP hem de OLAP ihtiyaçlarını karşılar. Bu döküman, sütun bazlı depolama stratejilerini ve Batch Mode yürütme avantajlarını incelemektedir.

## 🧠 Detay

```mermaid
graph TD
    "Satır Bazlı Depolama (Rowstore)" -- "OLTP" --> "B-Tree Yapısı"
    "Sütun Bazlı Depolama (Columnstore)" -- "OLAP" --> "Sıkıştırılmış Segmentler"
    
    subgraph "Columnstore Bileşenleri"
        "Delta Store" -- "Geçici Kayıtlar" --> "Row Groups"
        "Row Groups" -- "Sıkıştırma" --> "Compressed Segments"
    end
    
    "Compressed Segments" -- "Avantaj" --> "Yüksek Sıkıştırma"
    "Compressed Segments" -- "Avantaj" --> "Batch Mode Processing"
```

### Clustered vs Non-Clustered Columnstore

1. **Clustered Columnstore Index (CCI)**:
   - Tablonun fiziksel depolama biçimidir.
   - Büyük veri ambarı tabloları (Fact Tables) için idealdir.
   - Tüm tabloyu kapsar ve başka bir Clustered Index ile aynı anda bulunamaz.

2. **Non-Clustered Columnstore Index (NCCI)**:
   - Satır bazlı bir tablo üzerine eklenir.
   - Operasyonel veritabanlarında analitik sorguları hızlandırmak için (Real-time Analytics) kullanılır.
   - Belirli sütunları içerebilir.

### Kod Örnekleri

Büyük bir tablo için Clustered Columnstore Index oluşturma:

```sql
-- Mevcut bir tabloyu Columnstore yapısına dönüştürme
CREATE CLUSTERED COLUMNSTORE INDEX CCI_FactSales
ON Sales.FactSales;
```

Filtreli Non-clustered Columnstore Index örneği:

```sql
-- Sadece son 1 yıla ait veriler için analitik indeks oluşturma
CREATE NONCLUSTERED COLUMNSTORE INDEX NCCI_SalesAnalytics
ON Sales.Orders (OrderDate, CustomerID, TotalAmount)
WHERE OrderDate >= '2023-01-01';
```

Sıkıştırma oranlarını kontrol etme:

```sql
SELECT 
    object_name(object_id) AS TableName,
    index_id,
    partition_number,
    compression_delay,
    columnstore_delete_bitmap_units
FROM sys.columnstore_row_groups;
```

## 🔗 İlgili Notlar
- [[MSSQL - Query Execution Plan ve Performance Tuning]]
- [[FE - Veri Ambarı ve ETL Stratejileri]]
