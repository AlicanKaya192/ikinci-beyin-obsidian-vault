---
tarih: 2026-05-28
konu: MSSQL
etiket: ["mssql", "ddl", "create-table", "alter", "constraint"]
kaynak: Microsoft Dokümantasyon
zorluk: başlangıç
---

## 📌 Özet
DDL komutları veritabanı nesnelerinin yapısını oluşturur ve değiştirir. Doğru veri tipleri ve kısıtlamalar veri bütünlüğünü sağlar.

## 🧠 Detay

### Veri Tipleri
```sql
-- Sayısal
INT, BIGINT, SMALLINT, TINYINT
DECIMAL(10,2), NUMERIC(10,2)  -- hassas ondalık
FLOAT, REAL                    -- yaklaşık
BIT                            -- 0/1

-- Metin
VARCHAR(100)     -- değişken uzunluk (max 8000)
NVARCHAR(100)    -- unicode (Türkçe için N prefix)
CHAR(10)         -- sabit uzunluk
NVARCHAR(MAX)    -- büyük metin

-- Tarih
DATE             -- sadece tarih
TIME             -- sadece saat
DATETIME         -- tarih + saat
DATETIME2        -- daha hassas
DATETIMEOFFSET   -- timezone'lu

-- Diğer
UNIQUEIDENTIFIER -- GUID
VARBINARY(MAX)   -- binary veri
XML
```

### Tablo Oluşturma
```sql
CREATE TABLE Musteriler (
    MusteriID     INT           IDENTITY(1,1) PRIMARY KEY,
    MusteriKodu   VARCHAR(20)   NOT NULL UNIQUE,
    Ad            NVARCHAR(50)  NOT NULL,
    Soyad         NVARCHAR(50)  NOT NULL,
    Email         NVARCHAR(100) NULL,
    Telefon       VARCHAR(20)   NULL,
    DogumTarihi   DATE          NULL,
    Sehir         NVARCHAR(50)  NULL,
    Aktif         BIT           NOT NULL DEFAULT 1,
    OlusturmaTarih DATETIME2    NOT NULL DEFAULT GETDATE(),
    GuncellenmeTarih DATETIME2  NULL,

    CONSTRAINT CK_Yas CHECK (DogumTarihi < GETDATE()),
    CONSTRAINT FK_Musteriler_Sehirler
        FOREIGN KEY (SehirID) REFERENCES Sehirler(SehirID)
)
```

### ALTER TABLE
```sql
-- Sütun ekle
ALTER TABLE Musteriler
ADD PuanBakiyesi DECIMAL(10,2) DEFAULT 0

-- Sütun değiştir
ALTER TABLE Musteriler
ALTER COLUMN Email NVARCHAR(200)

-- Sütun sil
ALTER TABLE Musteriler
DROP COLUMN EskiSutun

-- Kısıtlama ekle
ALTER TABLE Musteriler
ADD CONSTRAINT UQ_Email UNIQUE (Email)

-- Kısıtlama sil
ALTER TABLE Musteriler
DROP CONSTRAINT UQ_Email
```

### Index Oluşturma
```sql
-- Clustered (sadece 1 tane olabilir)
CREATE CLUSTERED INDEX IX_Musteriler_MusteriKodu
ON Musteriler (MusteriKodu)

-- Non-Clustered
CREATE NONCLUSTERED INDEX IX_Musteriler_Sehir
ON Musteriler (Sehir)
INCLUDE (Ad, Soyad, Email)  -- kapsayan sütunlar

-- Unique index
CREATE UNIQUE INDEX UX_Musteriler_Email
ON Musteriler (Email) WHERE Email IS NOT NULL
```

### DROP ve TRUNCATE
```sql
DROP TABLE IF EXISTS GeciciTablo    -- SQL 2016+
TRUNCATE TABLE LogTablosu           -- hızlı temizle
DROP DATABASE TestDB                -- veritabanı sil (dikkat!)
```

## 💡 Bağlantılar
- [[MSSQL - Index ve Performans]]
- [[MSSQL - Kısıtlamalar ve Veri Bütünlüğü]]
- [[MSSQL - Geçici Tablolar ve Değişkenler]]

## ❓ Sorular / Anlamadıklarım
- VARCHAR ile NVARCHAR ne zaman hangisi kullanılmalı?
- IDENTITY sütunlarında boşluklar neden oluşur?

## 🔗 Kaynaklar
- https://learn.microsoft.com/tr-tr/sql/t-sql/statements/create-table-transact-sql
