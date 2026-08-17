---
tarih: 2026-08-17
konu: PostgreSQL
etiket: [postgresql, partitioning, performans, büyük-veri, ileri]
kaynak: PostgreSQL Docs
zorluk: ileri
---

## 📌 Özet

Partitioning, büyük tabloları mantıksal alt tablolara böler. Doğru yapılandırıldığında sorgu performansını 10-100x artırır, arşivleme ve bakım operasyonlarını kolaylaştırır. PostgreSQL 10+ ile declarative partitioning çok güçlendi.

---

## 🧠 Detay

### Partition Türleri

| Tür | Ne Zaman | Örnek |
|-----|---------|-------|
| **RANGE** | Tarih, sayısal aralık | Log tablosu (aylık) |
| **LIST** | Kategorik, az sayıda değer | Ülke, bölge, durum |
| **HASH** | Eşit dağılım, lookup | Kullanıcı ID'ye göre |

### RANGE Partitioning — Log Tablosu

```sql
-- Ana tablo (partition parent — doğrudan veri tutmaz)
CREATE TABLE model_loglari (
    id BIGSERIAL,
    log_tarihi TIMESTAMPTZ NOT NULL,
    model_adi VARCHAR(100),
    tahmin FLOAT,
    gercek FLOAT,
    gecikme_ms INT
) PARTITION BY RANGE (log_tarihi);

-- Partition'lar oluştur
CREATE TABLE model_loglari_2025_01
    PARTITION OF model_loglari
    FOR VALUES FROM ('2025-01-01') TO ('2025-02-01');

CREATE TABLE model_loglari_2025_02
    PARTITION OF model_loglari
    FOR VALUES FROM ('2025-02-01') TO ('2025-03-01');

-- Index — her partition'da kendi index'i olur
CREATE INDEX ON model_loglari_2025_01 (model_adi);
CREATE INDEX ON model_loglari_2025_02 (model_adi);

-- Ana tablodan sorgula — PostgreSQL doğru partition'ı seçer
SELECT * FROM model_loglari
WHERE log_tarihi >= '2025-01-15'
  AND log_tarihi < '2025-01-16';
-- → Sadece 2025-01 partition'ını okur (partition pruning)
```

### Otomatik Partition Oluşturma (pg_partman)

```sql
-- pg_partman extension ile otomatik partition yönetimi
CREATE EXTENSION IF NOT EXISTS pg_partman;

-- Haftalık partition oluştur
SELECT create_parent(
    p_parent_table := 'public.model_loglari',
    p_control := 'log_tarihi',
    p_interval := '1 month',
    p_premake := 3   -- önceden 3 ay oluştur
);

-- Eski partition'ları temizle (retention)
UPDATE partman.part_config
SET retention = '12 months',
    retention_keep_table = false  -- tabloyu sil
WHERE parent_table = 'public.model_loglari';

-- Bakım (cron ile çalıştır)
SELECT partman.run_maintenance();
```

### LIST Partitioning

```sql
-- Ülke bazlı partition
CREATE TABLE satis_kayitlari (
    id BIGSERIAL,
    ulke VARCHAR(3) NOT NULL,
    tutar NUMERIC(10,2),
    tarih DATE
) PARTITION BY LIST (ulke);

CREATE TABLE satis_tr PARTITION OF satis_kayitlari
    FOR VALUES IN ('TUR');

CREATE TABLE satis_eu PARTITION OF satis_kayitlari
    FOR VALUES IN ('DEU', 'FRA', 'GBR', 'ITA', 'ESP');

CREATE TABLE satis_diger PARTITION OF satis_kayitlari
    DEFAULT;   -- eşleşmeyen tüm değerler buraya
```

### HASH Partitioning

```sql
-- Veriyi eşit dağıt (analitik yük dengeleme)
CREATE TABLE kullanici_aktiviteleri (
    id BIGSERIAL,
    kullanici_id BIGINT NOT NULL,
    aksiyon VARCHAR(50),
    tarih TIMESTAMPTZ
) PARTITION BY HASH (kullanici_id);

-- 8 eşit partition
CREATE TABLE kullanici_aktiviteleri_0
    PARTITION OF kullanici_aktiviteleri
    FOR VALUES WITH (MODULUS 8, REMAINDER 0);

CREATE TABLE kullanici_aktiviteleri_1
    PARTITION OF kullanici_aktiviteleri
    FOR VALUES WITH (MODULUS 8, REMAINDER 1);
-- ... 2'den 7'ye kadar devam et
```

### Partition Pruning — Doğrulama

```sql
-- Partition pruning çalışıyor mu?
EXPLAIN (ANALYZE, FORMAT text)
SELECT * FROM model_loglari
WHERE log_tarihi >= '2025-01-01'
  AND log_tarihi < '2025-02-01';

-- Çıktıda gör:
-- Partitions selected: 1 of 12   ← sadece ilgili partition
-- Partitions excluded: 11 of 12
```

### Partition'ı Detach / Attach

```sql
-- Eski partition'ı ana tablodan ayır (hızlı, lock yok)
ALTER TABLE model_loglari
    DETACH PARTITION model_loglari_2024_01 CONCURRENTLY;

-- Bağımsız tabloya dönüştü — arşivle veya sil
ALTER TABLE model_loglari_2024_01
    RENAME TO model_loglari_2024_01_arsiv;

-- Yeni partition ekle
CREATE TABLE model_loglari_2025_12
    PARTITION OF model_loglari
    FOR VALUES FROM ('2025-12-01') TO ('2026-01-01');

-- Mevcut tabloyu partition olarak ekle
ALTER TABLE model_loglari_2025_12
    ATTACH PARTITION model_loglari CONCURRENTLY
    FOR VALUES FROM ('2025-12-01') TO ('2026-01-01');
```

### DS Kullanım Senaryosu — Feature Store

```sql
-- Kullanıcı feature'ları — günlük partition
CREATE TABLE kullanici_ozellikleri (
    hesaplama_tarihi DATE NOT NULL,
    kullanici_id BIGINT NOT NULL,
    son_7_gun_alisveris NUMERIC,
    son_30_gun_aktif_gun INT,
    toplam_harcama NUMERIC,
    PRIMARY KEY (hesaplama_tarihi, kullanici_id)
) PARTITION BY RANGE (hesaplama_tarihi);

-- Önerilir: pg_partman ile otomasyon
-- Avantaj: Her günlük hesaplama kendi partition'ında
--          Eski veriler DROP PARTITION ile hızla temizlenir
--          Query sadece ilgili tarih dilimini okur

-- Örnek: Son 7 günün feature'ını al
SELECT *
FROM kullanici_ozellikleri
WHERE hesaplama_tarihi = CURRENT_DATE - 1
  AND kullanici_id = 12345;
-- → Sadece dünün partition'ı okunur
```

### Performans Karşılaştırması

```
Durum: 1 milyar satır log tablosu, tarih bazlı sorgu

Partition yok (tek tablo):
  → Seq Scan: ~180 saniye
  → Index Scan: ~15 saniye

Aylık partition (24 partition, 2 yıl):
  → Index Scan (1 partition): ~0.3 saniye
  → Oran: 50x hız artışı

Ek avantajlar:
  → Eski partition'ı DROP PARTITION ile anında sil (VACUUM gerekmez)
  → Backup: Aylık partition ayrı backup edilebilir
  → I/O: Sık sorgu partition'ı önbellekte tutulur
```

---

## 💡 Bağlantılar
- [[PG - Performans Optimizasyonu ve EXPLAIN ANALYZE]]
- [[PG - İndeksleme Stratejileri (B-tree, GIN, GiST, BRIN)]]
- [[MLOps - Feature Store Tasarımı]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- [PostgreSQL Table Partitioning](https://www.postgresql.org/docs/current/ddl-partitioning.html)
- [pg_partman](https://github.com/pgpartman/pg_partman)
