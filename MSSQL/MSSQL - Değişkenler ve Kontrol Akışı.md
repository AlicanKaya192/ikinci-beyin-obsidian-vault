---
tarih: 2026-05-28
konu: MSSQL
etiket: ["mssql", "kontrol-akışı", "if-else", "while", "cursor", "t-sql"]
kaynak: Microsoft Dokümantasyon
zorluk: orta
---

## 📌 Özet
T-SQL programlama yapıları (IF/ELSE, WHILE, CASE, CURSOR) stored procedure ve batch işlemlerinde iş mantığı uygulamak için kullanılır.

## 🧠 Detay

### IF / ELSE
```sql
DECLARE @Bakiye DECIMAL(10,2) = 500

IF @Bakiye >= 1000
    PRINT 'Altın müşteri'
ELSE IF @Bakiye >= 500
BEGIN
    PRINT 'Gümüş müşteri'
    UPDATE Musteriler SET Kategori = 'Gumus' WHERE Bakiye = @Bakiye
END
ELSE
    PRINT 'Standart müşteri'
```

### CASE İfadesi
```sql
-- Basit CASE
SELECT Ad,
    CASE Kategori
        WHEN 'A' THEN 'Premium'
        WHEN 'B' THEN 'Standart'
        ELSE 'Diğer'
    END AS KategoriAdi
FROM Musteriler

-- Arama CASE (daha esnek)
SELECT Ad, Yas,
    CASE
        WHEN Yas < 18 THEN 'Çocuk'
        WHEN Yas BETWEEN 18 AND 30 THEN 'Genç'
        WHEN Yas BETWEEN 31 AND 60 THEN 'Yetişkin'
        ELSE 'Yaşlı'
    END AS YasGrubu,
    CASE
        WHEN Bakiye > 10000 THEN Bakiye * 0.1
        WHEN Bakiye > 5000  THEN Bakiye * 0.05
        ELSE 0
    END AS Bonus
FROM Musteriler
```

### WHILE Döngüsü
```sql
DECLARE @Sayac INT = 1
DECLARE @Toplam INT = 0

WHILE @Sayac <= 100
BEGIN
    SET @Toplam += @Sayac
    SET @Sayac += 1

    IF @Sayac = 50
        BREAK       -- döngüden çık

    IF @Sayac % 2 = 0
        CONTINUE    -- bir sonraki iterasyona geç
END

PRINT 'Toplam: ' + CAST(@Toplam AS VARCHAR)
```

### CURSOR (Set-based tercih et!)
```sql
-- Cursor genellikle yavaştır, set-based tercih edilir
DECLARE @MusteriID INT
DECLARE @Ad        NVARCHAR(50)

DECLARE cursor_Musteriler CURSOR FOR
    SELECT MusteriID, Ad FROM Musteriler WHERE Aktif = 1

OPEN cursor_Musteriler
FETCH NEXT FROM cursor_Musteriler INTO @MusteriID, @Ad

WHILE @@FETCH_STATUS = 0
BEGIN
    PRINT 'İşleniyor: ' + @Ad
    -- satır bazlı işlemler

    FETCH NEXT FROM cursor_Musteriler INTO @MusteriID, @Ad
END

CLOSE cursor_Musteriler
DEALLOCATE cursor_Musteriler
```

### IIF ve CHOOSE
```sql
-- IIF → kısa CASE
SELECT Ad, IIF(Aktif = 1, 'Aktif', 'Pasif') AS Durum
FROM Musteriler

-- CHOOSE → index bazlı seçim
SELECT CHOOSE(MONTH(GETDATE()), 'Ocak','Şubat','Mart','Nisan',
    'Mayıs','Haziran','Temmuz','Ağustos','Eylül','Ekim','Kasım','Aralık')
    AS AyAdi
```

### GOTO ve RETURN
```sql
-- RETURN → SP'den erken çık
IF @Bakiye < 0
BEGIN
    RAISERROR('Negatif bakiye!', 16, 1)
    RETURN
END
```

## 💡 Bağlantılar
- [[MSSQL - Stored Procedure]]
- [[MSSQL - Geçici Tablolar ve Değişkenler]]
- [[MSSQL - Transaction ve Hata Yönetimi]]

## ❓ Sorular / Anlamadıklarım
- Cursor yerine set-based çözüm her zaman mümkün mü?
- WHILE döngüsü ne zaman kaçınılmazdır?

## 🔗 Kaynaklar
- https://learn.microsoft.com/tr-tr/sql/t-sql/language-elements/control-of-flow
