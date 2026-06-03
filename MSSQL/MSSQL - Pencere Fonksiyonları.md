---
tarih: 2026-05-28
konu: MSSQL
etiket: ["mssql", "pencere-fonksiyonu", "window-function", "rank", "over"]
kaynak: Microsoft Dokümantasyon
zorluk: orta
---

## 📌 Özet
Pencere fonksiyonları (Window Functions), GROUP BY'dan farklı olarak satırları daraltmadan hesaplama yapar. Sıralama, kümülatif toplam ve hareketli ortalama için vazgeçilmezdir.

## 🧠 Detay

### Sıralama Fonksiyonları
```sql
SELECT
    Ad, Soyad, Maas,
    ROW_NUMBER() OVER (ORDER BY Maas DESC)    AS SiraNo,
    RANK()       OVER (ORDER BY Maas DESC)    AS Rank,       -- eşitlerde boşluk bırakır
    DENSE_RANK() OVER (ORDER BY Maas DESC)    AS DenseRank,  -- eşitlerde boşluk bırakmaz
    NTILE(4)     OVER (ORDER BY Maas DESC)    AS Ceyrek      -- 4 gruba böler
FROM Calisanlar
```

### PARTITION BY (Grup İçi Sıralama)
```sql
-- Her departmanda maaş sıralaması
SELECT
    DepartmanAdi,
    Ad, Soyad, Maas,
    RANK() OVER (PARTITION BY DepartmanID ORDER BY Maas DESC) AS DeptIciRank
FROM Calisanlar c
INNER JOIN Departmanlar d ON c.DepartmanID = d.DepartmanID

-- Her kategoride en pahalı 3 ürün
SELECT *
FROM (
    SELECT
        Kategori, UrunAdi, Fiyat,
        ROW_NUMBER() OVER (PARTITION BY Kategori ORDER BY Fiyat DESC) AS Sira
    FROM Urunler
) AS Ranked
WHERE Sira <= 3
```

### Agregasyon Pencere Fonksiyonları
```sql
SELECT
    SiparisTarih,
    Tutar,
    SUM(Tutar)   OVER (ORDER BY SiparisTarih)                    AS KumulatifToplam,
    AVG(Tutar)   OVER (ORDER BY SiparisTarih ROWS 6 PRECEDING)   AS HareketliOrtalama7Gun,
    SUM(Tutar)   OVER (PARTITION BY YEAR(SiparisTarih))          AS YillikToplam,
    Tutar * 1.0 / SUM(Tutar) OVER (PARTITION BY YEAR(SiparisTarih)) AS YillikPay
FROM Siparisler
```

### LAG ve LEAD (Önceki/Sonraki Satır)
```sql
SELECT
    SiparisTarih,
    Tutar,
    LAG(Tutar, 1, 0)  OVER (ORDER BY SiparisTarih) AS OncekiGun,
    LEAD(Tutar, 1, 0) OVER (ORDER BY SiparisTarih) AS SonrakiGun,
    Tutar - LAG(Tutar) OVER (ORDER BY SiparisTarih) AS GunlukDegisim
FROM GunlukSatislar
```

### FIRST_VALUE ve LAST_VALUE
```sql
SELECT
    Ad, Maas,
    FIRST_VALUE(Ad) OVER (ORDER BY Maas DESC) AS EnYuksekMaasliKisi,
    LAST_VALUE(Ad)  OVER (ORDER BY Maas DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS EnDusukMaasliKisi
FROM Calisanlar
```

### ROWS vs RANGE
```sql
-- ROWS → fiziksel satır sayısı
ROWS BETWEEN 3 PRECEDING AND CURRENT ROW

-- RANGE → değer aralığı
RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```

## 💡 Bağlantılar
- [[MSSQL - Agregasyon ve GROUP BY]]
- [[MSSQL - Alt Sorgular ve CTE]]
- [[MSSQL - Analitik Sorgular]]

## ❓ Sorular / Anlamadıklarım
- RANK() ile DENSE_RANK() hangi durumda hangisi kullanılır?
- ROWS UNBOUNDED PRECEDING ne anlama gelir?

## 🔗 Kaynaklar
- https://learn.microsoft.com/tr-tr/sql/t-sql/functions/ranking-functions-transact-sql
