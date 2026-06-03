---
tarih: 2026-05-28
konu: MSSQL
etiket: ["mssql", "join", "inner-join", "left-join", "ilişki"]
kaynak: Microsoft Dokümantasyon
zorluk: başlangıç
---

## 📌 Özet
JOIN, iki veya daha fazla tabloyu ortak bir sütun üzerinden birleştirir. Doğru JOIN türü seçimi sorgunun hem doğruluğunu hem de performansını etkiler.

## 🧠 Detay

### JOIN Türleri

```sql
-- INNER JOIN → Sadece eşleşen kayıtlar
SELECT m.Ad, m.Soyad, s.SiparisNo, s.Tutar
FROM Musteriler m
INNER JOIN Siparisler s ON m.MusteriID = s.MusteriID

-- LEFT JOIN → Sol tablonun tümü + eşleşen sağ
SELECT m.Ad, m.Soyad, s.SiparisNo
FROM Musteriler m
LEFT JOIN Siparisler s ON m.MusteriID = s.MusteriID
-- Siparişsiz müşteriler de gelir, SiparisNo NULL olur

-- RIGHT JOIN → Sağ tablonun tümü + eşleşen sol
SELECT m.Ad, s.SiparisNo
FROM Musteriler m
RIGHT JOIN Siparisler s ON m.MusteriID = s.MusteriID

-- FULL OUTER JOIN → Her iki tablonun tümü
SELECT m.Ad, s.SiparisNo
FROM Musteriler m
FULL OUTER JOIN Siparisler s ON m.MusteriID = s.MusteriID

-- CROSS JOIN → Kartezyen çarpım (dikkatli kullan!)
SELECT m.Ad, u.UrunAdi
FROM Musteriler m
CROSS JOIN Urunler u
```

### Görsel Özet
```
INNER JOIN:    A ∩ B
LEFT JOIN:     A tümü + B eşleşen
RIGHT JOIN:    B tümü + A eşleşen
FULL OUTER:    A ∪ B
```

### Çoklu JOIN
```sql
SELECT
    m.Ad + ' ' + m.Soyad AS Musteri,
    s.SiparisNo,
    s.SiparisTarih,
    u.UrunAdi,
    sd.Miktar,
    sd.BirimFiyat,
    sd.Miktar * sd.BirimFiyat AS ToplamTutar
FROM Musteriler m
INNER JOIN Siparisler s ON m.MusteriID = s.MusteriID
INNER JOIN SiparisDetay sd ON s.SiparisID = sd.SiparisID
INNER JOIN Urunler u ON sd.UrunID = u.UrunID
WHERE s.SiparisTarih >= '2024-01-01'
ORDER BY s.SiparisTarih DESC
```

### Eşleşmeyen Kayıtları Bul
```sql
-- Hiç sipariş vermemiş müşteriler
SELECT m.Ad, m.Soyad
FROM Musteriler m
LEFT JOIN Siparisler s ON m.MusteriID = s.MusteriID
WHERE s.SiparisID IS NULL

-- Müşterisi olmayan siparişler
SELECT s.SiparisNo
FROM Siparisler s
LEFT JOIN Musteriler m ON s.MusteriID = m.MusteriID
WHERE m.MusteriID IS NULL
```

### Self JOIN
```sql
-- Yöneticilerin altındaki çalışanlar
SELECT
    c.Ad AS Calisan,
    y.Ad AS Yonetici
FROM Calisanlar c
LEFT JOIN Calisanlar y ON c.YoneticiID = y.CalisanID
```

## 💡 Bağlantılar
- [[MSSQL - Temel SQL Komutları]]
- [[MSSQL - Agregasyon ve GROUP BY]]
- [[MSSQL - Alt Sorgular ve CTE]]

## ❓ Sorular / Anlamadıklarım
- INNER JOIN ile WHERE ile filtrelemek performans açısından fark yaratır mı?
- Çok sayıda JOIN performansı nasıl etkiler?

## 🔗 Kaynaklar
- https://learn.microsoft.com/tr-tr/sql/t-sql/queries/from-transact-sql
