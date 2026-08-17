---
tarih: 2026-08-17
konu: PostgreSQL
etiket: [postgresql, performans, explain, analyze, optimizasyon, ileri]
kaynak: PostgreSQL Docs, pganalyze
zorluk: ileri
---

## 📌 Özet

Yavaş sorgular production'da gizlice maliyete dönüşür. EXPLAIN ANALYZE sorgunun nasıl çalıştığını açar; query planner kararlarını, cost tahminlerini ve gerçek execution süresini gösterir. Bu not, yavaş sorguyu tespit etmekten çözüme götüren süreci açıklar.

---

## 🧠 Detay

### EXPLAIN Seçenekleri

```sql
-- Sadece plan (çalıştırmaz)
EXPLAIN SELECT * FROM siparisler WHERE kullanici_id = 42;

-- Plan + gerçek süre (çalıştırır!)
EXPLAIN ANALYZE SELECT * FROM siparisler WHERE kullanici_id = 42;

-- Tam detay (production için ideal)
EXPLAIN (
    ANALYZE true,
    BUFFERS true,    -- önbellek kullanımı
    FORMAT text,     -- text | json | xml | yaml
    VERBOSE true     -- sütun listesi
)
SELECT s.*, k.ad
FROM siparisler s
JOIN kullanicilar k ON s.kullanici_id = k.id
WHERE s.tarih >= '2025-01-01';
```

### EXPLAIN Çıktısını Okuma

```
Seq Scan on siparisler  (cost=0.00..45231.00 rows=1000000 width=80)
                                 ↑         ↑       ↑          ↑
                              başlangıç  bitiş  satır tahmini  satır genişliği (byte)

Sonrası (ANALYZE ile):
  actual time=0.034..892.432 rows=987654 loops=1
  ↑                   ↑           ↑         ↑
  ilk satır süresi   toplam süre  gerçek satır  döngü sayısı

Buffers: shared hit=1234 read=5678
                   ↑ önbellekte     ↑ diskten okunan
```

### Scan Türleri — İyiden Kötüye

```sql
-- 1. Index Only Scan (MÜKEMMEL)
-- Tablo okumadan, sadece index'ten yanıt
-- Tablo heap'e hiç dokunmaz
Index Only Scan using idx_email on kullanicilar
  Index Cond: (email = 'user@example.com')
  Heap Fetches: 0  ← mükemmel

-- 2. Index Scan (İYİ)
-- Index var, tabloya da bakıyor (ihtiyaç duyduğunda)
Index Scan using idx_tarih on siparisler
  Index Cond: (tarih >= '2025-01-01')

-- 3. Bitmap Heap Scan (MAKUL — büyük sonuç için)
-- Önce index ile konumları toplar, sonra toplu okur
Bitmap Heap Scan on siparisler
  Recheck Cond: (kullanici_id = 42)
  -> Bitmap Index Scan on idx_kullanici

-- 4. Seq Scan (KÖTÜ — büyük tabloda)
-- Tabloyu baştan sona okur — index yok veya planner tercih etmedi
Seq Scan on siparisler
  Filter: (kullanici_id = 42)
  Rows Removed by Filter: 999000  ← 1M satırdan 1 bulduk
```

### Join Stratejileri

```sql
-- Hash Join (büyük tablolar)
Hash Join
  Hash Cond: (s.kullanici_id = k.id)
  -> Seq Scan on siparisler s
  -> Hash
       -> Seq Scan on kullanicilar k

-- Nested Loop (küçük tablolar, index varsa)
Nested Loop
  -> Index Scan on kullanicilar k
       Index Cond: (id = 42)
  -> Index Scan on siparisler s
       Index Cond: (kullanici_id = k.id)

-- Merge Join (her iki tablo sıralıysa)
Merge Join
  Merge Cond: (s.kullanici_id = k.id)
  -> Sort on kullanici_id
  -> Sort on id
```

### pg_stat_statements — Yavaş Sorgular

```sql
-- Extension kur (postgresql.conf: shared_preload_libraries = 'pg_stat_statements')
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- En yavaş 10 sorgu
SELECT
    LEFT(query, 100) AS sorgu,
    calls AS cagri_sayisi,
    ROUND(total_exec_time::numeric / calls, 2) AS ort_ms,
    ROUND(total_exec_time::numeric, 0) AS toplam_ms,
    rows / calls AS ort_satir
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;

-- Cache hit oranı (<%90 ise RAM artır)
SELECT
    sum(heap_blks_read) AS disk_okunan,
    sum(heap_blks_hit) AS onbellekten,
    ROUND(
        100.0 * sum(heap_blks_hit) /
        NULLIF(sum(heap_blks_hit + heap_blks_read), 0),
        2
    ) AS hit_orani_yuzde
FROM pg_statio_user_tables;
```

### Planner Statistiklerini Güncelle

```sql
-- Planner yanlış tahmin yapıyorsa
ANALYZE tablo_adi;        -- istatistikleri güncelle
ANALYZE;                  -- tüm tabloları güncelle

-- İstatistik hedefini artır (varsayılan 100)
ALTER TABLE siparisler ALTER COLUMN kullanici_id SET STATISTICS 500;
ANALYZE siparisler;

-- Tablo istatistikleri
SELECT
    relname,
    n_live_tup AS canli_satir,
    n_dead_tup AS olu_satir,
    last_analyze,
    last_autovacuum
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC;
```

### VACUUM ve Table Bloat

```sql
-- Dead tuple'ları temizle (MVCC sonrası)
VACUUM ANALYZE siparisler;

-- Tam sıkıştırma (tablo kilitler — dikkatli kullan)
VACUUM FULL siparisler;

-- Autovacuum ayarları (postgresql.conf)
autovacuum = on
autovacuum_vacuum_threshold = 50
autovacuum_analyze_threshold = 50
autovacuum_vacuum_scale_factor = 0.02   -- %2 dead tuple → tetikle

-- Bloat kontrolü
SELECT
    tablename,
    pg_size_pretty(pg_total_relation_size(tablename::regclass)) AS toplam,
    pg_size_pretty(pg_relation_size(tablename::regclass)) AS tablo,
    pg_size_pretty(
        pg_total_relation_size(tablename::regclass) -
        pg_relation_size(tablename::regclass)
    ) AS index_boyutu
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(tablename::regclass) DESC
LIMIT 20;
```

### Pratik Optimizasyon Süreci

```
1. Yavaş sorguyu bul
   → pg_stat_statements veya log

2. EXPLAIN ANALYZE çalıştır
   → Hangi node en çok zaman alıyor?
   → Rows estimate vs actual büyük fark var mı?

3. Teşhis et
   Seq Scan büyük tabloda      → İndeks ekle
   Yanlış rows estimate        → ANALYZE çalıştır
   Büyük Hash Join             → join_collapse_limit ayarla
   Yüksek Heap Fetches         → Covering index ekle
   Parallelism yok (>1M satır) → max_parallel_workers_per_gather artır

4. Çözümü uygula ve tekrar ölç
   → Süre karşılaştır

5. Monitoring ekle
   → pg_stat_statements ile haftalık rapor
```

### postgresql.conf Performans Ayarları

```ini
# Bellek
shared_buffers = 4GB          # RAM'in 25%'i
effective_cache_size = 12GB   # RAM'in 75%'i
work_mem = 64MB               # Karmaşık sort/hash için
maintenance_work_mem = 1GB    # VACUUM, CREATE INDEX için

# Paralel sorgu
max_parallel_workers_per_gather = 4
max_parallel_workers = 8
parallel_tuple_cost = 0.1

# WAL (yazma performansı)
wal_buffers = 64MB
checkpoint_completion_target = 0.9
wal_compression = on

# Planner
default_statistics_target = 100   # 500'e kadar artırılabilir
random_page_cost = 1.1             # SSD için (HDD varsayılan: 4.0)
effective_io_concurrency = 200     # SSD için
```

---

## 💡 Bağlantılar
- [[PG - İndeksleme Stratejileri (B-tree, GIN, GiST, BRIN)]]
- [[PG - Partitioning ve Tablo Bölümleme]]
- [[Monitoring - Prometheus ve Grafana ile Altyapı İzleme]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- [pganalyze EXPLAIN Visualizer](https://explain.dalibo.com/)
- [PostgreSQL Performance Tuning](https://wiki.postgresql.org/wiki/Performance_Optimization)
- [PGTune](https://pgtune.leopard.in.ua/)
