# MSSQL - Query Execution Plan ve Performance Tuning

## 📌 Özet
SQL Server'da sorgu performansını optimize etmek için yürütme planlarını (Execution Plans) anlamak ve analiz etmek kritik bir yetkinliktir. Yürütme planları, veritabanı motorunun bir sorguyu en verimli şekilde sonuçlandırmak için seçtiği yolu (Index Seek, Scan, Join tipleri vb.) görselleştirir. Performans iyileştirme sürecinde mantıksal okumaları (logical reads) minimize etmek ve yanlış indeks kullanımından kaynaklanan maliyetli "Index Scan" operasyonlarını "Index Seek" ile değiştirmek hedeflenir. Bu döküman, darboğazları tespit etme ve yürütme planlarını okuma tekniklerine odaklanmaktadır.

## 🧠 Detay

```mermaid
graph TD
    "Sorgu (T-SQL)" --> "Parser"
    "Parser" --> "Algebrizer"
    "Algebrizer" --> "Optimizer"
    "Optimizer" --> "Execution Plan"
    "Execution Plan" --> "Storage Engine"
    "Storage Engine" --> "Sonuç Kümesi"
    
    subgraph "Tuning Operasyonları"
        "Execution Plan" -- "Analiz" --> "Index Seek (Verimli)"
        "Execution Plan" -- "Darboğaz" --> "Index Scan (Maliyetli)"
        "Index Scan (Maliyetli)" -- "Optimizasyon" --> "Missing Index Önerisi"
    end
```

### Temel Kavramlar

1. **Index Seek vs Index Scan**:
   - **Index Seek**: Veritabanı motorunun B-Tree yapısını kullanarak doğrudan ilgili kayıtlara ulaşmasıdır. Genellikle daha verimlidir.
   - **Index Scan**: Belirli bir kriter olmaksızın indeksin tamamının taranmasıdır. Büyük tablolarda performans sorununa yol açar.

2. **Logical Reads**: Verinin bellekten (Buffer Pool) kaç sayfa olarak okunduğunu ifade eder. Bu değerin düşük olması performansın yüksek olduğunu gösterir.

3. **Key Lookup**: Seçilen bir indekste bulunmayan sütunların, ana tablodan (Clustered Index) çekilmesi işlemidir. Fazladan I/O maliyeti yaratır.

### Kod Örnekleri ve Analiz

Sorgu performansını analiz etmek için istatistikleri açma:

```sql
-- I/O ve Zaman istatistiklerini etkinleştirme
SET STATISTICS IO ON;
SET STATISTICS TIME ON;

-- Analiz edilecek sorgu
SELECT BusinessEntityID, FirstName, LastName
FROM Person.Person
WHERE LastName = 'Sánchez';

-- İstatistikleri kapatma
SET STATISTICS IO OFF;
SET STATISTICS TIME OFF;
```

Eksik indeksleri tespit etmek için kullanılan DMV (Dynamic Management View) örneği:

```sql
SELECT 
    migs.avg_total_user_cost * (migs.avg_user_impact / 100.0) * (migs.user_seeks + migs.user_scans) AS [Potansiyel_Kazanç],
    mid.statement AS [Tablo_Adı],
    mid.equality_columns,
    mid.inequality_columns,
    mid.included_columns
FROM sys.dm_db_missing_index_group_stats AS migs
INNER JOIN sys.dm_db_missing_index_groups AS mig AS mig.index_group_handle = migs.group_handle
INNER JOIN sys.dm_db_missing_index_details AS mid ON mid.index_handle = mig.index_handle
ORDER BY [Potansiyel_Kazanç] DESC;
```

## 🔗 İlgili Notlar
- [[MSSQL - Veri Ambarı ve Columnstore Indexing]]
- [[MSSQL - İleri Transaction İzolasyon Seviyeleri]]
