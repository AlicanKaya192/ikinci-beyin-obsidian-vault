---
tarih: 2026-08-17
konu: PostgreSQL
etiket: [postgresql, full-text-search, tsvector, tsquery, nlp, orta]
kaynak: PostgreSQL Docs
zorluk: orta
---

## 📌 Özet

PostgreSQL'in yerleşik Full-Text Search (FTS), küçük-orta ölçekli arama ihtiyaçları için Elasticsearch kurmadan güçlü metin arama sağlar. LIKE sorgusundan çok daha hızlı ve akıllı: stemming, ranking, Türkçe desteği.

---

## 🧠 Detay

### FTS Temelleri

```sql
-- to_tsvector: metni arama vektörüne çevir
SELECT to_tsvector('english', 'The quick brown fox jumped over the lazy dog');
-- 'brown':3 'dog':9 'fox':4 'jump':5 'lazi':8 'quick':2
-- Stop words kaldırıldı (the, over), stemming uygulandı (jumped→jump)

-- to_tsquery: arama sorgusunu parse et
SELECT to_tsquery('english', 'jumping & fox');
-- 'jump' & 'fox'

-- @@ operatörü: eşleşme kontrolü
SELECT to_tsvector('english', 'The fox jumped') @@ to_tsquery('english', 'fox & jump');
-- → TRUE
```

### Tablo Kurulumu

```sql
-- Yöntem 1: Computed column (PostgreSQL 12+)
CREATE TABLE haberler (
    id SERIAL PRIMARY KEY,
    baslik TEXT NOT NULL,
    icerik TEXT NOT NULL,
    yayin_tarihi DATE,
    -- Otomatik güncellenen FTS sütunu
    search_vector TSVECTOR GENERATED ALWAYS AS (
        setweight(to_tsvector('turkish', coalesce(baslik, '')), 'A') ||
        setweight(to_tsvector('turkish', coalesce(icerik, '')), 'B')
    ) STORED
);

-- Başlığa A ağırlığı (daha önemli), içeriğe B
-- setweight: A > B > C > D

-- GIN index
CREATE INDEX idx_haber_fts ON haberler USING GIN (search_vector);
```

```sql
-- Yöntem 2: Trigger ile güncelle (eski PostgreSQL uyumlu)
ALTER TABLE haberler ADD COLUMN search_vector TSVECTOR;

CREATE FUNCTION haberler_fts_guncelle() RETURNS TRIGGER AS $$
BEGIN
    NEW.search_vector :=
        setweight(to_tsvector('turkish', coalesce(NEW.baslik, '')), 'A') ||
        setweight(to_tsvector('turkish', coalesce(NEW.icerik, '')), 'B');
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_haber_fts
    BEFORE INSERT OR UPDATE ON haberler
    FOR EACH ROW EXECUTE FUNCTION haberler_fts_guncelle();
```

### Arama Sorguları

```sql
-- Temel arama
SELECT baslik, yayin_tarihi
FROM haberler
WHERE search_vector @@ to_tsquery('turkish', 'yapay & zeka')
ORDER BY yayin_tarihi DESC;

-- Sıralama (ranking)
SELECT
    baslik,
    ts_rank(search_vector, query) AS skor
FROM haberler,
     to_tsquery('turkish', 'makine & öğrenmesi') AS query
WHERE search_vector @@ query
ORDER BY skor DESC
LIMIT 10;

-- ts_rank_cd: cover density — öbeksel arama için daha iyi
SELECT baslik, ts_rank_cd(search_vector, query) AS skor
FROM haberler, to_tsquery('turkish', 'derin öğrenme') query
WHERE search_vector @@ query
ORDER BY skor DESC;
```

### tsquery Operatörleri

```sql
-- AND: Her iki kelime de olmalı
to_tsquery('turkish', 'python & makine')

-- OR: Birisi olsa yeter
to_tsquery('turkish', 'python | r')

-- NOT: Bu kelime olmamalı
to_tsquery('turkish', 'python & !java')

-- PHRASE: Yan yana olmalı (sıralı)
phraseto_tsquery('turkish', 'makine öğrenmesi')
-- → 'makine' <-> 'öğrenme'  ← bitişik

-- Prefix arama
to_tsquery('turkish', 'pyt:*')  -- pytXXX ile başlayan
```

### Highlight (Vurgulama)

```sql
-- Arama sonuçlarında eşleşen kısımları vurgula
SELECT
    baslik,
    ts_headline(
        'turkish',
        icerik,
        to_tsquery('turkish', 'makine & öğrenmesi'),
        'StartSel=<mark>, StopSel=</mark>, MaxWords=30, MinWords=15, ShortWord=3'
    ) AS ozet
FROM haberler
WHERE search_vector @@ to_tsquery('turkish', 'makine & öğrenmesi');

-- Çıktı örneği:
-- "...Python ile <mark>makine</mark> <mark>öğrenmesi</mark> uygulamaları..."
```

### Türkçe FTS

```sql
-- turkish dictionary var mı kontrol et
SELECT * FROM pg_ts_config WHERE cfgname = 'turkish';

-- Türkçe stopwords ayarla
-- /usr/share/postgresql/16/tsearch_data/turkish.stop dosyasına ekle

-- Test
SELECT to_tsvector('turkish', 'Derin öğrenme ve yapay zeka alanında');
-- 'alan':6 'derin':1 'öğren':2 'yapay':4 'zeka':5
```

### Unaccent — Türkçe Karakter Toleransı

```sql
-- Ğ, ş, ı, ö, ü, ç ile yapılan aramalarda sorun yaşamamak için
CREATE EXTENSION IF NOT EXISTS unaccent;

-- Unaccent ile FTS
to_tsvector('turkish', unaccent('Şırnak şehrine özgü'))

-- Custom text search config
CREATE TEXT SEARCH CONFIGURATION turkish_unaccent (COPY = turkish);
ALTER TEXT SEARCH CONFIGURATION turkish_unaccent
    ALTER MAPPING FOR hword, hword_part, word
    WITH unaccent, turkish_stem;
```

### FTS vs LIKE Performans

```sql
-- LIKE — yavaş, tam metin araması yapmaz
SELECT * FROM haberler WHERE icerik LIKE '%makine öğrenmesi%';
-- → Sequential scan, stem yok, 100K satırda saniyeler sürer

-- FTS — GIN index ile anında
SELECT * FROM haberler WHERE search_vector @@ phraseto_tsquery('turkish', 'makine öğrenmesi');
-- → GIN index scan, stemming, 0.01ms

-- Ne zaman Elasticsearch?
-- 10M+ belge, faceted search, gelişmiş relevance, gerçek zamanlı index
-- Bunlar gerekmiyorsa PostgreSQL FTS yeterli
```

---

## 💡 Bağlantılar
- [[PG - İndeksleme Stratejileri (B-tree, GIN, GiST, BRIN)]]
- [[PG - PostgreSQL Veri Tipleri (JSONB, Arrays, UUID)]]
- [[GenAI - RAG Mimarisi ve Vektör Veritabanları]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- [PostgreSQL Full Text Search](https://www.postgresql.org/docs/current/textsearch.html)
- [FTS Tutorial - Cybertec](https://www.cybertec-postgresql.com/en/postgresql-full-text-search/)
