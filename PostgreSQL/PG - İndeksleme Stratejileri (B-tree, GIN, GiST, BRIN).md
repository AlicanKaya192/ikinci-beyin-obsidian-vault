---
tarih: 2026-08-17
konu: PostgreSQL
etiket: [postgresql, index, b-tree, gin, gist, performans, orta]
kaynak: PostgreSQL Docs, Use The Index Luke
zorluk: orta
---

## 📌 Özet

İndeks, veritabanı performansının en kritik unsuru. Doğru indeks türünü seçmek sorguyu 100x hızlandırabilir; yanlış indeks yazma performansını düşürür. PostgreSQL 8 farklı indeks türü sunar — en önemlisi B-tree, GIN, GiST ve BRIN.

---

## 🧠 Detay

### İndeks Türleri Özeti

| Tür | Kullanım Alanı | Operatörler |
|-----|---------------|-------------|
| **B-tree** | Genel amaçlı, eşitlik, aralık | `=, <, >, <=, >=, BETWEEN, LIKE 'abc%'` |
| **GIN** | Tam metin, diziler, JSONB | `@>, ?, &&, @@` |
| **GiST** | Geometrik, full-text, range | `<<, &&, @>` |
| **BRIN** | Çok büyük tablolar, fiziksel korelasyon | Aralık sorguları |
| **Hash** | Sadece eşitlik | `=` |
| **SP-GiST** | Coğrafi, IP, telefon | Uzay bölümleme |

---

### B-tree Index (Varsayılan)

```sql
-- Temel kullanım
CREATE INDEX idx_kullanici_email ON kullanicilar (email);

-- Composite index (çoklu sütun)
CREATE INDEX idx_siparis_tarih_durum ON siparisler (tarih, durum);
-- ÖNEMLİ: Sütun sırası önemli — sol baştan kullanılır

-- Partial index (koşullu — çok verimli)
CREATE INDEX idx_aktif_kullanici ON kullanicilar (email)
WHERE aktif = true;
-- Sadece aktif kullanıcılar için index → küçük, hızlı

-- Unique index
CREATE UNIQUE INDEX idx_unique_email ON kullanicilar (email);

-- Expression index
CREATE INDEX idx_email_lower ON kullanicilar (LOWER(email));
-- Bu sorgu index kullanır:
SELECT * FROM kullanicilar WHERE LOWER(email) = 'user@example.com';
```

### B-tree Kullanmaz — Dikkat

```sql
-- ❌ Wildcard başta LIKE kullanmaz
WHERE email LIKE '%@gmail.com'   -- Full scan

-- ✅ Wildcard sonda kullanır
WHERE email LIKE 'ahmet%'        -- Index kullanır

-- ❌ Fonksiyon sarmalı (expression index olmadan)
WHERE UPPER(ad) = 'ALİ'          -- Full scan
-- ✅ Çözüm: Expression index ekle
CREATE INDEX ON kullanicilar (UPPER(ad));
```

---

### GIN Index — JSON ve Full-Text

```sql
-- JSONB için GIN
CREATE INDEX idx_meta_gin ON urunler USING GIN (meta_verisi);

-- Bu sorgular GIN kullanır:
SELECT * FROM urunler WHERE meta_verisi @> '{"renk": "mavi"}';
SELECT * FROM urunler WHERE meta_verisi ? 'stok';
SELECT * FROM urunler WHERE meta_verisi ?| ARRAY['renk', 'boyut'];

-- Dizi için GIN
CREATE INDEX idx_etiket_gin ON makaleler USING GIN (etiketler);
SELECT * FROM makaleler WHERE etiketler @> ARRAY['python'];
SELECT * FROM makaleler WHERE etiketler && ARRAY['python', 'sql'];

-- Full-text search için GIN
CREATE INDEX idx_icerik_fts ON belgeler USING GIN (search_vector);
-- veya
CREATE INDEX idx_fts ON belgeler USING GIN (to_tsvector('turkish', icerik));
```

---

### GiST Index — Geometrik ve Range

```sql
-- Range type için GiST
CREATE TABLE rezervasyonlar (
    id SERIAL,
    oda_id INT,
    sure DATERANGE
);
CREATE INDEX idx_sure_gist ON rezervasyonlar USING GiST (sure);

-- Çakışma kontrolü
SELECT * FROM rezervasyonlar
WHERE sure && '[2025-01-01, 2025-01-07]'::daterange;

-- PostGIS ile coğrafi index
CREATE INDEX idx_konum_gist ON lokasyonlar USING GiST (konum);
SELECT * FROM lokasyonlar
WHERE ST_DWithin(konum, ST_MakePoint(29.0, 41.0)::geography, 5000);
```

---

### BRIN Index — Büyük Tablolar

```sql
-- Çok büyük tablolarda (100M+ satır) fiziksel sıralı veriler için
-- Küçük boyut, hızlı oluşturma, arama yavaş ama sıralı veri için makul
CREATE INDEX idx_log_tarih_brin ON log_kayitlari
USING BRIN (log_tarihi) WITH (pages_per_range = 128);

-- Koşul: log_tarihi sütunu fiziksel olarak sıralı eklenmelidir
-- (Zaman damgası, autoincrement ID — doğal sıralı)
```

---

### EXPLAIN ANALYZE — İndeks Kullanımını Doğrula

```sql
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT * FROM siparisler
WHERE kullanici_id = 42 AND durum = 'aktif';

-- Çıktı yorumu:
-- Seq Scan        → İndeks YOK veya kullanılmıyor (kötü)
-- Index Scan      → İndeks var, kullanılıyor (iyi)
-- Index Only Scan → Tablo okumadan sadece indeksten cevap (mükemmel)
-- Bitmap Scan     → Büyük sonuç kümesi, verimli
```

### Hangi Sorgular Kaçırıyor?

```sql
-- Yavaş sorguları bul (pg_stat_statements gerekli)
SELECT query, calls, total_exec_time, mean_exec_time, rows
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 20;

-- Index kullanılmayanları bul
SELECT relname, seq_scan, seq_tup_read, idx_scan
FROM pg_stat_user_tables
WHERE seq_scan > idx_scan
ORDER BY seq_scan DESC;

-- Kullanılmayan indexleri bul (disk israfı!)
SELECT schemaname, tablename, indexname, idx_scan
FROM pg_stat_user_indexes
WHERE idx_scan = 0
ORDER BY pg_relation_size(indexrelid) DESC;
```

### Covering Index (INCLUDE)

```sql
-- Index Only Scan sağlamak için
-- idx üzerinde olmayan ama sorguda lazım olan sütunları INCLUDE et
CREATE INDEX idx_siparis_kullanici ON siparisler (kullanici_id)
INCLUDE (tutar, tarih, durum);

-- Bu sorgu artık tablo'ya hiç bakmaz:
SELECT tutar, tarih, durum
FROM siparisler
WHERE kullanici_id = 42;
```

### İndeks Bakımı

```sql
-- Index bloat — zaman içinde şişer
-- REINDEX ile yeniden oluştur (tablo kitlenir)
REINDEX INDEX idx_kullanici_email;

-- CONCURRENTLY — tablo kilitlemeden (önerilir)
REINDEX INDEX CONCURRENTLY idx_kullanici_email;

-- Tüm tablo indexlerini yenile
REINDEX TABLE CONCURRENTLY kullanicilar;

-- Index boyutunu kontrol et
SELECT
    indexname,
    pg_size_pretty(pg_relation_size(indexname::regclass)) AS boyut
FROM pg_indexes
WHERE tablename = 'kullanicilar';
```

### DS için Index Stratejisi

```python
# Bir ML serving tablosu için index planı
index_plani = """
-- Feature tablosu (çok okunan)
CREATE INDEX idx_feature_kullanici ON features (kullanici_id);
CREATE INDEX idx_feature_tarih ON features (hesaplama_tarihi);
CREATE INDEX idx_feature_gist ON features USING GIN (ozellikler_jsonb);

-- Tahmin sonuçları (sürekli yazılan)
-- Az index! Yazma performansını düşürür
CREATE INDEX idx_tahmin_model ON tahminler (model_id, tarih);

-- Log tablosu (BRIN ile büyük tablo)
CREATE INDEX idx_log_brin ON model_loglari USING BRIN (log_tarihi);
"""
```

---

## 💡 Bağlantılar
- [[PG - Performans Optimizasyonu ve EXPLAIN ANALYZE]]
- [[PG - PostgreSQL Veri Tipleri (JSONB, Arrays, UUID)]]
- [[PG - Full-Text Search ve Tsvector]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- [Use The Index, Luke](https://use-the-index-luke.com/)
- [PostgreSQL Index Types](https://www.postgresql.org/docs/current/indexes-types.html)
- [pganalyze Index Advisor](https://pganalyze.com/)
