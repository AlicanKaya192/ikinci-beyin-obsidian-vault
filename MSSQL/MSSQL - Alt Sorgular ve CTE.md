---
tarih: 2026-05-28
konu: MSSQL
etiket: ["mssql", "alt-sorgu", "cte", "subquery", "with"]
kaynak: Microsoft Dokümantasyon
zorluk: orta
---

## 📌 Özet
Alt sorgular (subquery), bir sorgu içinde başka bir sorgu çalıştırır. CTE (Common Table Expression), karmaşık sorguları okunabilir parçalara böler.

## 🧠 Detay

### Alt Sorgu Türleri
```sql
-- WHERE içinde alt sorgu (Scalar)
SELECT Ad, Soyad, Maas
FROM Calisanlar
WHERE Maas > (SELECT AVG(Maas) FROM Calisanlar)

-- IN ile alt sorgu
SELECT Ad, Soyad
FROM Musteriler
WHERE MusteriID IN (
    SELECT DISTINCT MusteriID
    FROM Siparisler
    WHERE SiparisTarih >= '2024-01-01'
)

-- EXISTS (daha hızlı)
SELECT Ad, Soyad
FROM Musteriler m
WHERE EXISTS (
    SELECT 1 FROM Siparisler s
    WHERE s.MusteriID = m.MusteriID
    AND s.SiparisTarih >= '2024-01-01'
)
```

### FROM içinde Alt Sorgu
```sql
-- Türetilmiş tablo
SELECT Sehir, OrtYas
FROM (
    SELECT Sehir, AVG(Yas) AS OrtYas
    FROM Musteriler
    GROUP BY Sehir
) AS SehirOzet
WHERE OrtYas > 30
```

### CTE (WITH)
```sql
-- Temel CTE
WITH AktifMusteriler AS (
    SELECT MusteriID, Ad, Soyad
    FROM Musteriler
    WHERE Aktif = 1
),
MusteriSiparisleri AS (
    SELECT MusteriID, COUNT(*) AS SiparisSayisi
    FROM Siparisler
    GROUP BY MusteriID
)
SELECT
    am.Ad,
    am.Soyad,
    ISNULL(ms.SiparisSayisi, 0) AS SiparisSayisi
FROM AktifMusteriler am
LEFT JOIN MusteriSiparisleri ms ON am.MusteriID = ms.MusteriID
ORDER BY SiparisSayisi DESC
```

### Özyinelemeli CTE (Hierarchical)
```sql
-- Organizasyon hiyerarşisi
WITH OrgHiyerarsi AS (
    -- Anchor: En üst yönetici
    SELECT CalisanID, Ad, YoneticiID, 0 AS Seviye
    FROM Calisanlar
    WHERE YoneticiID IS NULL

    UNION ALL

    -- Recursive: Alt çalışanlar
    SELECT c.CalisanID, c.Ad, c.YoneticiID, oh.Seviye + 1
    FROM Calisanlar c
    INNER JOIN OrgHiyerarsi oh ON c.YoneticiID = oh.CalisanID
)
SELECT CalisanID, Ad, Seviye,
       REPLICATE('  ', Seviye) + Ad AS HiyerarsiGorunum
FROM OrgHiyerarsi
ORDER BY Seviye, Ad
```

### CTE vs Alt Sorgu vs Temp Tablo
| Yöntem | Ne Zaman |
|--------|----------|
| Alt Sorgu | Basit, tek kullanım |
| CTE | Okunabilirlik, çok adım |
| Temp Tablo | Büyük veri, tekrar kullanım |
| View | Kalıcı, paylaşılan sorgu |

## 💡 Bağlantılar
- [[MSSQL - Temel SQL Komutları]]
- [[MSSQL - Pencere Fonksiyonları]]
- [[MSSQL - Geçici Tablolar ve Değişkenler]]

## ❓ Sorular / Anlamadıklarım
- EXISTS mi IN mi? Hangisi daha hızlı?
- Özyinelemeli CTE'de sonsuz döngüyü nasıl önlerim?

## 🔗 Kaynaklar
- https://learn.microsoft.com/tr-tr/sql/t-sql/queries/with-common-table-expression-transact-sql
