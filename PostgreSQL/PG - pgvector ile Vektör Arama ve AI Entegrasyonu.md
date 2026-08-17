---
tarih: 2026-08-17
konu: PostgreSQL
etiket: [postgresql, pgvector, vektör, ai, rag, embedding, ileri]
kaynak: pgvector GitHub, Supabase Docs
zorluk: ileri
---

## 📌 Özet

pgvector, PostgreSQL'e vektör benzerlik araması ekler. Özel bir vektör veritabanı kurmadan (Pinecone, Qdrant) RAG, semantik arama ve öneri sistemi için embedding depolayabilirsin. Mevcut PostgreSQL altyapına AI eklemek için en hızlı yol.

---

## 🧠 Detay

### pgvector Kurulumu

```bash
# Docker ile (en kolay)
docker run -d \
  --name pg-vector \
  -e POSTGRES_PASSWORD=pass \
  -p 5432:5432 \
  pgvector/pgvector:pg16   # ← pgvector dahil resmi image

# Mevcut PostgreSQL'e ekle (Ubuntu)
sudo apt install postgresql-16-pgvector

# Extension aktifleştir
CREATE EXTENSION IF NOT EXISTS vector;
```

### Embedding Tablosu

```sql
-- OpenAI text-embedding-3-small: 1536 boyut
-- Nomic embed: 768 boyut
-- all-MiniLM-L6-v2: 384 boyut

CREATE TABLE belgeler (
    id BIGSERIAL PRIMARY KEY,
    baslik TEXT NOT NULL,
    icerik TEXT NOT NULL,
    embedding VECTOR(1536),   -- boyutu modele göre ayarla
    kaynak VARCHAR(255),
    meta JSONB,
    olusturma_tarihi TIMESTAMPTZ DEFAULT NOW()
);
```

### Python ile Embedding Ekle

```python
from openai import OpenAI
import psycopg2
import psycopg2.extras
import numpy as np

client = OpenAI()

def embedding_al(metin: str) -> list[float]:
    response = client.embeddings.create(
        model="text-embedding-3-small",
        input=metin
    )
    return response.data[0].embedding

# Belge ekle
def belge_ekle(conn, baslik: str, icerik: str):
    embedding = embedding_al(icerik)
    with conn.cursor() as cur:
        cur.execute(
            """
            INSERT INTO belgeler (baslik, icerik, embedding)
            VALUES (%s, %s, %s)
            RETURNING id
            """,
            (baslik, icerik, embedding)
        )
        return cur.fetchone()[0]

# Batch ekleme
from psycopg2.extras import execute_values

def belgeler_toplu_ekle(conn, belgeler: list[dict]):
    embeddings = [embedding_al(b['icerik']) for b in belgeler]
    data = [(b['baslik'], b['icerik'], e)
            for b, e in zip(belgeler, embeddings)]
    with conn.cursor() as cur:
        execute_values(
            cur,
            "INSERT INTO belgeler (baslik, icerik, embedding) VALUES %s",
            data
        )
    conn.commit()
```

### Benzerlik Araması

```sql
-- En yakın 5 belge (cosine distance — küçük = daha benzer)
SELECT
    id,
    baslik,
    icerik,
    1 - (embedding <=> '[0.1, 0.2, ...]'::vector) AS benzerlik_skoru
FROM belgeler
ORDER BY embedding <=> '[0.1, 0.2, ...]'::vector
LIMIT 5;
```

```
pgvector Operatörleri:
  <->  L2 distance (Euclidean)
  <#>  Negative inner product (dot product için)
  <=>  Cosine distance
```

### Python ile Semantik Arama

```python
def semantik_ara(conn, sorgu: str, k: int = 5) -> list[dict]:
    sorgu_embedding = embedding_al(sorgu)

    with conn.cursor(cursor_factory=psycopg2.extras.RealDictCursor) as cur:
        cur.execute(
            """
            SELECT
                id,
                baslik,
                icerik,
                1 - (embedding <=> %s::vector) AS skor
            FROM belgeler
            ORDER BY embedding <=> %s::vector
            LIMIT %s
            """,
            (sorgu_embedding, sorgu_embedding, k)
        )
        return cur.fetchall()

# Kullanım
sonuclar = semantik_ara(conn, "Python'da makine öğrenmesi nasıl yapılır?")
for s in sonuclar:
    print(f"{s['baslik']} — skor: {s['skor']:.3f}")
```

### RAG Pipeline

```python
from openai import OpenAI

client = OpenAI()

def rag_cevap(conn, soru: str) -> str:
    # 1. İlgili belgeleri bul
    ilgili = semantik_ara(conn, soru, k=3)

    # 2. Context hazırla
    context = "\n\n".join([
        f"[{r['baslik']}]\n{r['icerik']}"
        for r in ilgili
    ])

    # 3. LLM'e gönder
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {
                "role": "system",
                "content": f"Sadece aşağıdaki bilgilere dayanarak cevap ver:\n\n{context}"
            },
            {"role": "user", "content": soru}
        ]
    )
    return response.choices[0].message.content

# Kullanım
cevap = rag_cevap(conn, "pgvector nasıl kurulur?")
print(cevap)
```

### IVFFlat Index — Hızlı Benzerlik Araması

```sql
-- Milyonlarca vektör için index şart
-- IVFFlat: Yaklaşık en yakın komşu (ANN)
-- Önce verini ekle, sonra index oluştur

-- Cosine distance için
CREATE INDEX idx_embedding_cosine ON belgeler
USING ivfflat (embedding vector_cosine_ops)
WITH (lists = 100);   -- sqrt(satır sayısı) ≈ iyi başlangıç

-- L2 için
CREATE INDEX idx_embedding_l2 ON belgeler
USING ivfflat (embedding vector_l2_ops)
WITH (lists = 100);

-- HNSW Index (pgvector 0.5+) — daha hızlı ve daha iyi recall
CREATE INDEX idx_embedding_hnsw ON belgeler
USING hnsw (embedding vector_cosine_ops)
WITH (m = 16, ef_construction = 64);
-- m: bağlantı sayısı (16-64), ef_construction: build kalitesi

-- Arama kalitesini artır (query sırasında)
SET hnsw.ef_search = 100;   -- yüksek = yavaş ama daha iyi recall
```

### Hibrit Arama — Full-Text + Semantik

```sql
-- Full-text + vektör arama birleştir (en güçlü yaklaşım)
WITH semantik AS (
    SELECT id, 1 - (embedding <=> %s::vector) AS skor
    FROM belgeler
    ORDER BY embedding <=> %s::vector
    LIMIT 50
),
fulltext AS (
    SELECT id, ts_rank(search_vector, to_tsquery('turkish', %s)) AS skor
    FROM belgeler
    WHERE search_vector @@ to_tsquery('turkish', %s)
    LIMIT 50
)
SELECT
    b.id, b.baslik,
    COALESCE(s.skor, 0) * 0.7 + COALESCE(f.skor, 0) * 0.3 AS hibrit_skor
FROM belgeler b
LEFT JOIN semantik s ON b.id = s.id
LEFT JOIN fulltext f ON b.id = f.id
WHERE s.id IS NOT NULL OR f.id IS NOT NULL
ORDER BY hibrit_skor DESC
LIMIT 10;
```

### SQLAlchemy + pgvector

```python
from sqlalchemy.orm import DeclarativeBase, mapped_column, Mapped
from pgvector.sqlalchemy import Vector

class Base(DeclarativeBase):
    pass

class Belge(Base):
    __tablename__ = "belgeler"

    id: Mapped[int] = mapped_column(primary_key=True)
    baslik: Mapped[str]
    icerik: Mapped[str]
    embedding: Mapped[list] = mapped_column(Vector(1536))

# Sorgulama
from sqlalchemy import select, func

with Session(engine) as session:
    sorgu_vec = embedding_al("Python pandas")
    sonuclar = session.scalars(
        select(Belge)
        .order_by(Belge.embedding.cosine_distance(sorgu_vec))
        .limit(5)
    ).all()
```

---

## 💡 Bağlantılar
- [[GenAI - RAG Mimarisi ve Vektör Veritabanları]]
- [[PG - PostgreSQL ile Python (psycopg2, SQLAlchemy, asyncpg)]]
- [[PG - İndeksleme Stratejileri (B-tree, GIN, GiST, BRIN)]]
- [[DL - LoRA ve Parameter-Efficient Fine-Tuning (PEFT)]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- [pgvector GitHub](https://github.com/pgvector/pgvector)
- [Supabase pgvector Guide](https://supabase.com/docs/guides/ai)
- [pgvector SQLAlchemy](https://github.com/pgvector/pgvector-python)
