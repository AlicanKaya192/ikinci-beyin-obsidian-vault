# MSSQL - İleri Transaction İzolasyon Seviyeleri

## 📌 Özet
Veritabanı sistemlerinde eşzamanlılık (concurrency) ve veri bütünlüğü arasındaki denge, işlem izolasyon seviyeleri (Isolation Levels) ile kurulur. Standart seviyelerin ötesinde, MSSQL'in sunduğu "Snapshot Isolation" ve "Read Committed Snapshot (RCSI)" gibi satır sürümleme (row versioning) tabanlı mekanizmalar, okuyucuların yazıcıları engellemesini önleyerek yüksek performans sağlar. Bu döküman, Dirty Reads, Non-repeatable Reads ve Phantom Reads gibi anomallikleri ve bunlara karşı kullanılan ileri düzey izolasyon stratejilerini ele almaktadır.

## 🧠 Detay

```mermaid
graph LR
    "Transaction A" -- "Update" --> "Row Versioning (tempdb)"
    "Transaction B" -- "Read" --> "Snapshot of Data"
    "Row Versioning (tempdb)" -- "Provides" --> "Snapshot of Data"
    
    subgraph "İzolasyon Seviyeleri ve Korumalar"
        "Read Uncommitted" --> "Hız Öncelikli (Kirli Okuma)"
        "Read Committed" --> "Varsayılan (Bloklama Olabilir)"
        "Repeatable Read" --> "Satır Kilitleme"
        "Serializable" --> "Tam İzolasyon (Range Locks)"
        "Snapshot" --> "Versiyonlama (No Blocking)"
    end
```

### İzolasyon Seviyeleri ve Etkileri

| Seviye | Dirty Read | Non-Repeatable Read | Phantom Read |
| :--- | :---: | :---: | :---: |
| **Read Uncommitted** | İzin Verir | İzin Verir | İzin Verir |
| **Read Committed** | Önler | İzin Verir | İzin Verir |
| **Repeatable Read** | Önler | Önler | İzin Verir |
| **Serializable** | Önler | Önler | Önler |
| **Snapshot** | Önler | Önler | Önler |

### Row Versioning ve Snapshot İzolasyonu

Snapshot izolasyonu, verinin bir kopyasını `tempdb` üzerinde tutarak okuma işlemlerinin bloklanmadan devam etmesini sağlar. Yazma işlemleri hala birbirini bekler, ancak okuyucular ve yazıcılar birbirini engellemez.

### Kod Örnekleri

Veritabanı düzeyinde Read Committed Snapshot (RCSI) özelliğini açma:

```sql
-- Veritabanını tek kullanıcı moduna alıp RCSI'yı etkinleştirme
ALTER DATABASE [TargetDB] SET SINGLE_USER WITH ROLLBACK IMMEDIATE;
ALTER DATABASE [TargetDB] SET READ_COMMITTED_SNAPSHOT ON;
ALTER DATABASE [TargetDB] SET MULTI_USER;
```

Sorgu bazlı izolasyon seviyesi belirleme:

```sql
-- Transaction içerisinde Snapshot İzolasyonu kullanma
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;

BEGIN TRANSACTION;
    -- Bu sorgu, transaction başladığı andaki veri setini görür
    SELECT * FROM Sales.Orders WHERE OrderID = 5000;
COMMIT TRANSACTION;
```

## 🔗 İlgili Notlar
- [[MSSQL - Query Execution Plan ve Performance Tuning]]
- [[MSSQL - Güvenlik, Audit ve Row-Level Security]]
