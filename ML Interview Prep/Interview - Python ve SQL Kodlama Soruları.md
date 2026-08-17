---
tarih: 2026-06-08
konu: ML Interview Prep
etiket: [mülakat, interview, python, sql, kodlama, pandas, leetcode]
kaynak: LeetCode, NeetCode, StrataScratch
zorluk: orta
---

## 📌 Özet

DS/ML mülakatlarındaki Python ve SQL soruları; pandas manipülasyonu, pencere fonksiyonları, liste/dict işlemleri ve temel algoritma sorularını kapsar. LeetCode'dan farklı olarak veri odaklıdır.

---

## 🧠 Detay

### Python — Sık Sorulan Kodlama Soruları

**S: Pandas'ta rolling 7-günlük ortalama nasıl hesaplanır?**
```python
import pandas as pd

df = pd.DataFrame({
    "tarih": pd.date_range("2024-01-01", periods=30),
    "satis": [100 + i * 2 + (i % 7) * 5 for i in range(30)]
})
df = df.set_index("tarih")
df["rolling_7"] = df["satis"].rolling(window=7).mean()
```

---

**S: Büyük CSV dosyasını belleğe sığdırmadan oku:**
```python
# Chunk'lar halinde oku
chunks = []
for chunk in pd.read_csv("büyük_dosya.csv", chunksize=10_000):
    # Her chunk için işlemi yap
    filtered = chunk[chunk["değer"] > 0]
    chunks.append(filtered)
df = pd.concat(chunks)

# Alternatif: DuckDB
import duckdb
result = duckdb.query("SELECT * FROM 'büyük_dosya.csv' WHERE değer > 0").df()
```

---

**S: Pandas groupby + custom aggregation:**
```python
df.groupby("kategori").agg(
    ortalama=("satis", "mean"),
    toplam=("satis", "sum"),
    adet=("satis", "count"),
    std_sapma=("satis", "std"),
    # Custom: son 3 değerin ortalaması
    son3_ort=("satis", lambda x: x.tail(3).mean())
)
```

---

**S: Missing value stratejisi — ne zaman ne kullanılır?**
```python
# Sayısal → median (aykırı değerlere karşı dayanıklı)
df["yaş"].fillna(df["yaş"].median(), inplace=True)

# Kategorik → mod (en sık görülen)
df["şehir"].fillna(df["şehir"].mode()[0], inplace=True)

# Zaman serisi → forward fill
df["fiyat"].fillna(method="ffill", inplace=True)

# Çok fazla eksik (>%50) → kolonu düşür
threshold = len(df) * 0.5
df.dropna(thresh=threshold, axis=1, inplace=True)
```

---

**S: List comprehension ile flatten:**
```python
nested = [[1, 2, 3], [4, 5], [6, 7, 8]]
flat = [x for sublist in nested for x in sublist]
# [1, 2, 3, 4, 5, 6, 7, 8]
```

---

**S: Counter ile kelime frekansı:**
```python
from collections import Counter

metin = "data science is great data is fun"
sayac = Counter(metin.split())
en_sik_3 = sayac.most_common(3)
# [('data', 2), ('is', 2), ('science', 1)]
```

---

### SQL — Sık Sorulan Sorular

**S: Hangi müşteriler her ay sipariş vermiş?**
```sql
WITH aylik AS (
    SELECT
        musteri_id,
        DATE_TRUNC('month', siparis_tarihi) AS ay,
        COUNT(*) AS siparis_sayisi
    FROM siparisler
    GROUP BY 1, 2
)
SELECT musteri_id
FROM aylik
GROUP BY musteri_id
HAVING COUNT(DISTINCT ay) = (
    SELECT COUNT(DISTINCT DATE_TRUNC('month', siparis_tarihi))
    FROM siparisler
);
```

---

**S: Running total (kümülatif toplam):**
```sql
SELECT
    tarih,
    satis,
    SUM(satis) OVER (ORDER BY tarih) AS kumulatif_toplam,
    SUM(satis) OVER (
        PARTITION BY kategori
        ORDER BY tarih
    ) AS kategori_kumulatif
FROM satis_tablosu;
```

---

**S: Her kategoride en çok satan ürün (TOP-N per group):**
```sql
WITH ranked AS (
    SELECT
        kategori,
        urun,
        toplam_satis,
        ROW_NUMBER() OVER (
            PARTITION BY kategori
            ORDER BY toplam_satis DESC
        ) AS sira
    FROM urun_satis_ozeti
)
SELECT kategori, urun, toplam_satis
FROM ranked
WHERE sira = 1;
```

---

**S: Retention analizi (kullanıcı tutma):**
```sql
-- Hafta 0'da kayıt olan kullanıcıların Hafta 1'de aktif olma oranı
SELECT
    DATE_TRUNC('week', kayit_tarihi) AS cohort_haftasi,
    COUNT(DISTINCT k.kullanici_id) AS kayit_sayisi,
    COUNT(DISTINCT e.kullanici_id) AS aktif_hafta1,
    COUNT(DISTINCT e.kullanici_id) * 100.0 /
        COUNT(DISTINCT k.kullanici_id) AS retention_pct
FROM kullanicilar k
LEFT JOIN etkinlikler e
    ON k.kullanici_id = e.kullanici_id
    AND e.etkinlik_tarihi BETWEEN
        DATE_TRUNC('week', k.kayit_tarihi) + INTERVAL '7 days'
        AND DATE_TRUNC('week', k.kayit_tarihi) + INTERVAL '13 days'
GROUP BY 1
ORDER BY 1;
```

---

**S: Self-join ile çalışan-yönetici ilişkisi:**
```sql
SELECT
    c.ad AS çalışan,
    m.ad AS yönetici,
    c.maas
FROM çalışanlar c
LEFT JOIN çalışanlar m ON c.yönetici_id = m.id;
```

---

### Hızlı Pandas Cheat Sheet

```python
# Veri keşfi
df.info()          # dtype + null count
df.describe()      # sayısal özet
df.value_counts()  # kategorik dağılım
df.nunique()       # benzersiz değer sayısı

# Filtreleme
df[df["a"] > 5]
df.query("a > 5 and b == 'x'")

# Pivot
df.pivot_table(values="satis", index="ay", columns="kategori", aggfunc="sum")
```

---

## 💡 Bağlantılar
- [[DS - Pandas Temel Kullanım]]
- [[DS - İleri Düzey SQL]]
- [[MSSQL - Pencere Fonksiyonları]]
- [[Interview - Data Science Vaka Çalışmaları]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- [StrataScratch - SQL Mülakat Soruları](https://www.stratascratch.com/)
- [NeetCode - Python Algoritma](https://neetcode.io/)
