---
tarih: 2026-08-17
konu: PostgreSQL
etiket: [postgresql, jsonb, array, uuid, veri-tipleri, orta]
kaynak: PostgreSQL Docs
zorluk: orta
---

## 📌 Özet

PostgreSQL'in güçlü veri tipleri onu diğer ilişkisel veritabanlarından ayırır: JSONB NoSQL esnekliği sağlar, Arrays çok değerli sütunları saklar, UUID dağıtık sistemlerde benzersiz kimlik sağlar. Bu tipler doğru kullanılırsa şema tasarımını büyük ölçüde basitleştirir.

---

## 🧠 Detay

### JSONB vs JSON

```sql
-- JSON: metin olarak saklar, her sorguda parse eder
-- JSONB: binary format, index destekli, DAHA HIZLI

-- JSONB tercih et
CREATE TABLE ml_modeller (
    id UUID DEFAULT gen_random_uuid(),
    model_adi VARCHAR(100),
    hiperparametreler JSONB,   -- ← JSONB
    metrikler JSONB,
    olusturma_tarihi TIMESTAMPTZ DEFAULT NOW()
);

INSERT INTO ml_modeller (model_adi, hiperparametreler, metrikler)
VALUES (
    'xgboost_v1',
    '{"n_estimators": 500, "learning_rate": 0.05, "max_depth": 6}',
    '{"accuracy": 0.923, "f1": 0.891, "auc": 0.967}'
);
```

### JSONB Sorgulama Operatörleri

```sql
-- -> : JSON obje döner (metin değil)
SELECT hiperparametreler -> 'n_estimators' FROM ml_modeller;
-- Sonuç: 500

-- ->> : Metin döner
SELECT hiperparametreler ->> 'learning_rate' FROM ml_modeller;
-- Sonuç: '0.05'

-- #> : Nested path (obje)
SELECT metrikler #> '{details, train_loss}' FROM ml_modeller;

-- #>> : Nested path (metin)
SELECT metrikler #>> '{details, train_loss}' FROM ml_modeller;

-- @> : İçeriyor mu? (contains)
SELECT * FROM ml_modeller
WHERE hiperparametreler @> '{"n_estimators": 500}';

-- ? : Anahtar var mı?
SELECT * FROM ml_modeller
WHERE hiperparametreler ? 'learning_rate';

-- ?| : Herhangi biri var mı?
SELECT * FROM ml_modeller
WHERE hiperparametreler ?| ARRAY['learning_rate', 'max_features'];

-- ?& : Hepsi var mı?
SELECT * FROM ml_modeller
WHERE hiperparametreler ?& ARRAY['n_estimators', 'max_depth'];
```

### JSONB Güncelleme

```sql
-- Yeni alan ekle
UPDATE ml_modeller
SET metrikler = metrikler || '{"precision": 0.894}'::jsonb
WHERE model_adi = 'xgboost_v1';

-- Alan sil
UPDATE ml_modeller
SET metrikler = metrikler - 'train_loss'
WHERE model_adi = 'xgboost_v1';

-- Nested güncelleme
UPDATE ml_modeller
SET hiperparametreler = jsonb_set(
    hiperparametreler,
    '{learning_rate}',
    '0.01'::jsonb
);
```

### JSONB ile Analitik

```sql
-- JSON dizisini satırlara aç
CREATE TABLE deney_sonuclari (
    deney_id INT,
    iterasyonlar JSONB   -- [{"iter": 1, "loss": 0.9}, {"iter": 2, "loss": 0.7}]
);

-- jsonb_array_elements ile aç
SELECT
    deney_id,
    (elem->>'iter')::INT AS iterasyon,
    (elem->>'loss')::FLOAT AS kayip
FROM deney_sonuclari,
     jsonb_array_elements(iterasyonlar) AS elem;

-- jsonb_object_keys ile anahtar listesi
SELECT DISTINCT jsonb_object_keys(hiperparametreler)
FROM ml_modeller;
```

### JSONB Index

```sql
-- GIN index — @>, ?, ?|, ?& operatörleri için
CREATE INDEX idx_hiper_gin ON ml_modeller USING GIN (hiperparametreler);

-- Belirli bir path için
CREATE INDEX idx_accuracy ON ml_modeller
USING BTREE ((metrikler->>'accuracy'));
```

---

### ARRAY Tipi

```sql
-- Array sütun oluşturma
CREATE TABLE makale (
    id SERIAL,
    baslik TEXT,
    etiketler TEXT[],           -- string dizisi
    puanlar INTEGER[],          -- int dizisi
    embedding FLOAT4[],         -- ML embedding (pgvector yoksa)
    matris FLOAT4[][]           -- 2D array
);

INSERT INTO makale (baslik, etiketler, puanlar)
VALUES (
    'PostgreSQL ile Vektör Arama',
    ARRAY['postgresql', 'pgvector', 'ai'],
    ARRAY[5, 4, 5, 3]
);
```

### Array Operatörleri

```sql
-- Element var mı?
SELECT * FROM makale WHERE 'postgresql' = ANY(etiketler);

-- Tümü var mı? (contains @>)
SELECT * FROM makale WHERE etiketler @> ARRAY['postgresql', 'ai'];

-- Kesişim var mı? (overlap &&)
SELECT * FROM makale WHERE etiketler && ARRAY['postgresql', 'mysql'];

-- Array fonksiyonları
SELECT
    array_length(etiketler, 1) AS etiket_sayisi,
    array_to_string(etiketler, ', ') AS etiketler_str,
    etiketler[1] AS ilk_etiket,   -- 1-indexed!
    etiketler[2:3] AS dilim
FROM makale;

-- UNNEST: Array'i satırlara çevir
SELECT baslik, unnest(etiketler) AS etiket
FROM makale;
```

---

### UUID

```sql
-- UUID extension (PostgreSQL 13'te built-in)
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";  -- eski yöntem
-- veya
SELECT gen_random_uuid();  -- PostgreSQL 13+ built-in

-- UUID primary key (auto-generate)
CREATE TABLE kullanicilar (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    olusturma_tarihi TIMESTAMPTZ DEFAULT NOW()
);

-- Insert — id otomatik üretilir
INSERT INTO kullanicilar (email)
VALUES ('user@example.com')
RETURNING id;
```

### UUID vs SERIAL — Ne Zaman Hangisi?

| | UUID | SERIAL/BIGSERIAL |
|---|------|-----------------|
| **Benzersizlik** | Evrensel (global unique) | Sadece tablo içinde |
| **Dağıtık sistemler** | ✅ Güvenli | ❌ Çakışma riski |
| **Sıralama** | ❌ Kronolojik değil | ✅ Artan sıra |
| **Boyut** | 16 byte | 4/8 byte |
| **İndeks performansı** | ⚠️ Düşük (random) | ✅ Yüksek |
| **URL'de kullanım** | ✅ Tahmin edilemez | ❌ Sıralı = güvenlik riski |

```sql
-- Sıralı UUID: ULID benzeri (UUIDv7 — PostgreSQL 17+)
-- Hem unique hem kronolojik
```

---

### Diğer Güçlü Tipler

```sql
-- TIMESTAMPTZ (timezone-aware) — her zaman UTC sakla
created_at TIMESTAMPTZ DEFAULT NOW()

-- INTERVAL — zaman farkı
SELECT NOW() - INTERVAL '7 days';
SELECT AGE(NOW(), '2024-01-01'::DATE);

-- NUMERIC(precision, scale) — finansal hesaplama
tutar NUMERIC(12, 4)   -- 00000000.0000

-- INET / CIDR — IP adresi
ip_adresi INET          -- '192.168.1.1'
ag_adresi CIDR          -- '192.168.1.0/24'

-- TSVECTOR — Full text search için
CREATE TABLE belgeler (
    id SERIAL,
    icerik TEXT,
    search_vector TSVECTOR GENERATED ALWAYS AS
        (to_tsvector('turkish', icerik)) STORED
);
```

---

## 💡 Bağlantılar
- [[PG - İndeksleme Stratejileri (B-tree, GIN, GiST, BRIN)]]
- [[PG - Full-Text Search ve Tsvector]]
- [[PG - pgvector ile Vektör Arama ve AI Entegrasyonu]]
- [[MongoDB - CRUD İşlemleri]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- [PostgreSQL JSON Functions](https://www.postgresql.org/docs/current/functions-json.html)
- [PostgreSQL Array Types](https://www.postgresql.org/docs/current/arrays.html)
