---
tarih: 2026-05-28
konu: MSSQL
etiket: ["mssql", "sql", "temel", "ddl", "dml"]
kaynak: Microsoft Dokümantasyon
zorluk: başlangıç
---

## 📌 Özet
SQL (Structured Query Language), ilişkisel veritabanlarını yönetmek için kullanılan standarttır. MSSQL, Microsoft'un SQL Server ürününde kullanılan T-SQL diyalektidir.

## 🧠 Detay

### SQL Kategorileri
| Kategori | Açıklama | Komutlar |
|----------|----------|----------|
| DDL | Yapı tanımlama | CREATE, ALTER, DROP |
| DML | Veri işleme | SELECT, INSERT, UPDATE, DELETE |
| DCL | Yetki kontrolü | GRANT, REVOKE |
| TCL | İşlem kontrolü | BEGIN, COMMIT, ROLLBACK |

### SELECT
```sql
-- Tüm sütunlar
SELECT * FROM Musteriler

-- Belirli sütunlar
SELECT MusteriID, Ad, Soyad, Email
FROM Musteriler

-- Alias (takma ad)
SELECT Ad + ' ' + Soyad AS TamAd,
       Email AS ElektronikPosta
FROM Musteriler

-- Distinct (tekrarsız)
SELECT DISTINCT Sehir FROM Musteriler

-- Top N kayıt
SELECT TOP 10 * FROM Siparisler
SELECT TOP 10 PERCENT * FROM Siparisler
```

### WHERE
```sql
SELECT * FROM Musteriler
WHERE Sehir = 'İstanbul'

-- Karşılaştırma operatörleri
WHERE Yas > 25
WHERE Yas BETWEEN 18 AND 65
WHERE Sehir IN ('İstanbul', 'Ankara', 'İzmir')
WHERE Ad LIKE 'Ah%'        -- Ah ile başlayanlar
WHERE Ad LIKE '%met'       -- met ile bitenler
WHERE Email IS NULL
WHERE Email IS NOT NULL
WHERE NOT Aktif = 1
```

### ORDER BY
```sql
SELECT * FROM Musteriler
ORDER BY Soyad ASC, Ad DESC

-- Sütun numarasıyla
SELECT Ad, Soyad, Yas FROM Musteriler
ORDER BY 3 DESC   -- 3. sütun (Yas) azalan
```

### INSERT
```sql
-- Tek kayıt
INSERT INTO Musteriler (Ad, Soyad, Email, Sehir)
VALUES ('Ahmet', 'Yılmaz', 'ahmet@mail.com', 'İstanbul')

-- Çok kayıt
INSERT INTO Musteriler (Ad, Soyad)
VALUES ('Ali', 'Kaya'), ('Ayşe', 'Demir'), ('Veli', 'Çelik')
```

### UPDATE
```sql
UPDATE Musteriler
SET Email = 'yeni@mail.com', Sehir = 'Ankara'
WHERE MusteriID = 5

-- Dikkat: WHERE olmadan tüm kayıtlar güncellenir!
```

### DELETE
```sql
DELETE FROM Musteriler
WHERE MusteriID = 5

-- Tüm tabloyu temizle (TRUNCATE daha hızlı)
TRUNCATE TABLE GeciiciListe
```

## 💡 Bağlantılar
- [[MSSQL - JOIN İşlemleri]]
- [[MSSQL - Agregasyon ve GROUP BY]]
- [[MSSQL - WHERE ve Filtre Operatörleri]]

## ❓ Sorular / Anlamadıklarım
- DELETE ile TRUNCATE arasındaki fark nedir?
- SELECT * kullanımı neden performans sorunu yaratır?

## 🔗 Kaynaklar
- https://learn.microsoft.com/tr-tr/sql/t-sql/queries/select-transact-sql
