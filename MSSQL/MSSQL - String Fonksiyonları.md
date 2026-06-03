---
tarih: 2026-05-28
konu: MSSQL
etiket: ["mssql", "string", "metin", "fonksiyon"]
kaynak: Microsoft Dokümantasyon
zorluk: başlangıç
---

## 📌 Özet
T-SQL zengin string fonksiyonları sunar. Metin temizleme, birleştirme, arama ve formatlama işlemleri için kullanılır.

## 🧠 Detay

### Temel Metin Fonksiyonları
```sql
SELECT
    LEN('Merhaba')             AS Uzunluk,        -- 7
    UPPER('merhaba')           AS BuyukHarf,       -- MERHABA
    LOWER('MERHABA')           AS KucukHarf,       -- merhaba
    LTRIM('  metin  ')         AS SolBoslukSil,
    RTRIM('  metin  ')         AS SagBoslukSil,
    TRIM('  metin  ')          AS TumBoslukSil,    -- SQL 2017+
    REVERSE('abc')             AS Tersine          -- cba
```

### Birleştirme
```sql
-- Concat
SELECT Ad + ' ' + Soyad AS TamAd FROM Musteriler
SELECT CONCAT(Ad, ' ', Soyad) AS TamAd FROM Musteriler  -- NULL güvenli
SELECT CONCAT_WS(' ', Ad, SegAdı, Soyad) AS TamAd       -- ayraçlı

-- STRING_AGG (grup birleştirme)
SELECT
    DepartmanID,
    STRING_AGG(Ad, ', ') WITHIN GROUP (ORDER BY Ad) AS Calisanlar
FROM Calisanlar
GROUP BY DepartmanID
```

### Arama ve Kesme
```sql
SELECT
    CHARINDEX('SQL', 'Microsoft SQL Server')     AS KonumBul,    -- 11
    PATINDEX('%[0-9]%', 'abc123def')             AS SayiKonum,
    SUBSTRING('Merhaba Dünya', 9, 5)             AS Kes,          -- Dünya
    LEFT('Merhaba', 3)                           AS Sol,          -- Mer
    RIGHT('Merhaba', 3)                          AS Sag           -- aba
```

### Değiştirme ve Temizleme
```sql
SELECT
    REPLACE('Merhaba Dünya', 'Dünya', 'SQL')     AS Degistir,
    STUFF('Merhaba', 3, 2, 'XX')                 AS Yerles,       -- MeXXaba
    REPLICATE('*', 5)                            AS Tekrar        -- *****

-- Özel karakter temizleme
SELECT REPLACE(REPLACE(Telefon, '-', ''), ' ', '') AS TelTemiz
FROM Musteriler
```

### Formatlama
```sql
SELECT
    FORMAT(GETDATE(), 'dd.MM.yyyy')              AS TurkTarih,
    FORMAT(12345.67, 'N2', 'tr-TR')              AS ParaFormat,   -- 12.345,67
    STR(3.14159, 6, 2)                           AS SayiMetin,    -- '  3.14'
    CAST(123 AS VARCHAR(10))                      AS IntToStr,
    CONVERT(VARCHAR(10), GETDATE(), 104)          AS TarihStr      -- 28.05.2026
```

### NULL İşlemleri
```sql
SELECT
    ISNULL(Email, 'Bilinmiyor')                  AS EmailGoster,
    COALESCE(Telefon, Cep, Email, 'Yok')         AS IletisimBilgi,
    NULLIF(Departman, 'Tanımsız')                AS DeptNull
FROM Musteriler
```

### STRING_SPLIT
```sql
-- Virgülle ayrılmış değerleri satırlara böl
SELECT value
FROM STRING_SPLIT('İstanbul,Ankara,İzmir', ',')

-- Pratik kullanım
SELECT m.*
FROM Musteriler m
INNER JOIN STRING_SPLIT(@SehirListesi, ',') s ON m.Sehir = s.value
```

## 💡 Bağlantılar
- [[MSSQL - Temel SQL Komutları]]
- [[MSSQL - Tarih ve Zaman Fonksiyonları]]
- [[MSSQL - Kullanıcı Tanımlı Fonksiyonlar]]

## ❓ Sorular / Anlamadıklarım
- CHARINDEX ile PATINDEX arasındaki fark nedir?
- STRING_AGG SQL Server'ın hangi versiyonundan itibaren var?

## 🔗 Kaynaklar
- https://learn.microsoft.com/tr-tr/sql/t-sql/functions/string-functions-transact-sql
