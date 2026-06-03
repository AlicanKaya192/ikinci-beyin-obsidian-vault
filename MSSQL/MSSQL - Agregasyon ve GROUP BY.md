---
tarih: 2026-05-28
konu: MSSQL
etiket: ["mssql", "agregasyon", "group-by", "having", "aggregate"]
kaynak: Microsoft Dokümantasyon
zorluk: başlangıç
---

## 📌 Özet
Agregasyon fonksiyonları veri özetleme için kullanılır. GROUP BY satırları gruplara böler, HAVING ise grup bazında filtreleme yapar.

## 🧠 Detay

### Temel Agregasyon Fonksiyonları
```sql
SELECT
    COUNT(*)              AS ToplamKayit,
    COUNT(Email)          AS EmailliKayit,    -- NULL'ları saymaz
    COUNT(DISTINCT Sehir) AS FarkliSehir,
    SUM(Tutar)            AS ToplamTutar,
    AVG(Tutar)            AS OrtTutar,
    MIN(Tutar)            AS MinTutar,
    MAX(Tutar)            AS MaxTutar,
    STDEV(Tutar)          AS StandartSapma
FROM Siparisler
WHERE SiparisTarih >= '2024-01-01'
```

### GROUP BY
```sql
-- Şehre göre müşteri sayısı
SELECT Sehir, COUNT(*) AS MusteriSayisi
FROM Musteriler
GROUP BY Sehir
ORDER BY MusteriSayisi DESC

-- Ürün bazında satış özeti
SELECT
    u.UrunAdi,
    COUNT(sd.SiparisDetayID)     AS SatisAdedi,
    SUM(sd.Miktar)               AS ToplamMiktar,
    SUM(sd.Miktar * sd.BirimFiyat) AS ToplamCiro,
    AVG(sd.BirimFiyat)           AS OrtFiyat
FROM SiparisDetay sd
INNER JOIN Urunler u ON sd.UrunID = u.UrunID
GROUP BY u.UrunAdi
ORDER BY ToplamCiro DESC
```

### HAVING (Grup Filtresi)
```sql
-- 5'ten fazla sipariş veren müşteriler
SELECT MusteriID, COUNT(*) AS SiparisSayisi
FROM Siparisler
GROUP BY MusteriID
HAVING COUNT(*) > 5

-- Toplam cirosu 10.000 TL üzeri ürünler
SELECT u.UrunAdi, SUM(sd.Miktar * sd.BirimFiyat) AS Ciro
FROM SiparisDetay sd
INNER JOIN Urunler u ON sd.UrunID = u.UrunID
GROUP BY u.UrunAdi
HAVING SUM(sd.Miktar * sd.BirimFiyat) > 10000
ORDER BY Ciro DESC
```

### WHERE vs HAVING
```sql
-- WHERE → gruplama öncesi filtreler
-- HAVING → gruplama sonrası filtreler

SELECT Sehir, COUNT(*) AS Sayi
FROM Musteriler
WHERE Aktif = 1          -- önce aktif olanları al
GROUP BY Sehir
HAVING COUNT(*) >= 10    -- sonra 10+ olanları filtrele
```

### ROLLUP ve CUBE (Alt Toplamlar)
```sql
-- Kategori ve ürün bazında + ara toplamlar
SELECT
    Kategori,
    UrunAdi,
    SUM(Tutar) AS Toplam
FROM Satislar
GROUP BY ROLLUP(Kategori, UrunAdi)

-- GROUPING() ile NULL kontrolü
SELECT
    CASE WHEN GROUPING(Kategori) = 1 THEN 'GENEL TOPLAM'
         ELSE Kategori END AS Kategori,
    SUM(Tutar) AS Toplam
FROM Satislar
GROUP BY ROLLUP(Kategori)
```

## 💡 Bağlantılar
- [[MSSQL - Temel SQL Komutları]]
- [[MSSQL - Pencere Fonksiyonları]]
- [[MSSQL - JOIN İşlemleri]]

## ❓ Sorular / Anlamadıklarım
- COUNT(*) ile COUNT(sütun) ne zaman farklı sonuç verir?
- ROLLUP ile CUBE arasındaki fark nedir?

## 🔗 Kaynaklar
- https://learn.microsoft.com/tr-tr/sql/t-sql/queries/select-group-by-transact-sql
