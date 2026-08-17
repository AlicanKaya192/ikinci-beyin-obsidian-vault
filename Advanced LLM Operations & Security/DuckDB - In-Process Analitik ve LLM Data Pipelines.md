---
tarih: 2026-06-08
konu: Advanced LLM Operations & Security
etiket: [duckdb, analitik, veri, llm, rag, olap, sql]
kaynak: DuckDB Docs, MotherDuck Blog
zorluk: başlangıç-orta
---

## 📌 Özet

**DuckDB**, sunucu gerektirmeden çalışan (in-process), kolon-tabanlı (columnar) bir analitik veritabanıdır. Pandas'ın yerini almak için değil, onu tamamlamak için tasarlanmıştır. LLM ve RAG pipeline'larında **hızlı veri hazırlama** için giderek daha fazla tercih edilmektedir.

> "The SQLite of OLAP" — yerel geliştirmede BigQuery/Spark'a gerek kalmadan milyonlarca satırı saniyelerde analiz edersin.

---

## 🧠 Detay

### Neden DuckDB?

| Özellik | Pandas | DuckDB |
|---|---|---|
| Çalışma şekli | Satır (row) bazlı | Kolon (columnar) bazlı |
| 1M satır filtre | ~500ms | ~20ms |
| Bellek kullanımı | Tüm veriyi RAM'e yükler | Streaming okuma |
| SQL desteği | df.groupby() vb. | Tam SQL |
| Dosya formatı | CSV, Parquet | CSV, Parquet, JSON, Excel, Arrow |

### Kurulum ve Temel Kullanım

```python
pip install duckdb
```

```python
import duckdb

# Bağlantı — dosyasız (in-memory) ya da dosyalı
con = duckdb.connect()  # in-memory
# con = duckdb.connect("analytics.duckdb")  # kalıcı

# Parquet dosyasını doğrudan sorgula — kopyalamadan!
result = con.execute("""
    SELECT date_trunc('month', event_date) AS ay,
           COUNT(*) AS istek_sayisi,
           AVG(token_count) AS ort_token
    FROM 'llm_logs/*.parquet'
    WHERE model = 'claude-opus-4-5'
    GROUP BY 1
    ORDER BY 1
""").df()  # pandas DataFrame olarak döner
```

### Pandas ile Birlikte Kullanım

```python
import pandas as pd
import duckdb

df = pd.read_csv("embeddings.csv")

# DuckDB direkt pandas DF'yi görebilir
result = duckdb.query("""
    SELECT chunk_id, content,
           list_distance(embedding, [0.1, 0.2, ...]) AS distance
    FROM df
    ORDER BY distance
    LIMIT 10
""").df()
```

### LLM / RAG Pipeline'ında DuckDB

```python
import duckdb
from anthropic import Anthropic

con = duckdb.connect("knowledge_base.duckdb")

# 1. Dökümanları yükle
con.execute("""
    CREATE TABLE IF NOT EXISTS documents AS
    SELECT * FROM read_json_auto('docs/*.json')
""")

# 2. Basit keyword arama (vektör DB olmadan prototip için)
def retrieve_context(query: str, top_k: int = 3):
    results = con.execute("""
        SELECT content, source
        FROM documents
        WHERE content ILIKE ?
        LIMIT ?
    """, [f"%{query}%", top_k]).fetchall()
    return results

# 3. LLM'e gönder
client = Anthropic()

def rag_answer(question: str):
    context = retrieve_context(question)
    context_text = "\n".join([row[0] for row in context])
    
    response = client.messages.create(
        model="claude-sonnet-4-6",
        system="Sadece verilen bağlamı kullanarak yanıtla.",
        messages=[{
            "role": "user",
            "content": f"Bağlam:\n{context_text}\n\nSoru: {question}"
        }],
        max_tokens=500
    )
    return response.content[0].text
```

### LLM Log Analizi

```python
# API çağrı loglarını analiz et
con.execute("""
    CREATE TABLE llm_calls AS
    FROM read_ndjson_auto('logs/api_calls.jsonl')
""")

# Günlük maliyet raporu
cost_report = con.execute("""
    SELECT
        DATE(created_at) AS gun,
        model,
        SUM(usage.input_tokens) AS toplam_input,
        SUM(usage.output_tokens) AS toplam_output,
        SUM(usage.input_tokens * 0.000003 + 
            usage.output_tokens * 0.000015) AS tahmini_maliyet_usd
    FROM llm_calls
    GROUP BY 1, 2
    ORDER BY 1 DESC
""").df()
```

---

## 💡 Bağlantılar
- [[LLM - RAG (Retrieval Augmented Generation) Mimarisi]]
- [[Context Caching ve AI Economics - Maliyet Optimizasyonu]]
- [[DS - Büyük Veri ile Çalışma (Polars ve Dask)]]
- [[DE - ETL vs ELT Stratejileri]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- [DuckDB Resmi Belgeleri](https://duckdb.org/docs/)
- [MotherDuck Blog](https://motherduck.com/blog/)
