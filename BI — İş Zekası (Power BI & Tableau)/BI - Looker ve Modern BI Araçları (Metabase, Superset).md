---
tarih: 2026-06-08
konu: BI — İş Zekası (Power BI & Tableau)
etiket: [bi, looker, metabase, superset, self-serve, analytics]
kaynak: Looker Docs, Metabase Docs, Apache Superset
zorluk: başlangıç-orta
---

## 📌 Özet

Power BI ve Tableau dışında modern veri ekiplerinin yaygın kullandığı BI araçları: **Looker** (Google'ın kurumsal tercihi), **Metabase** (self-serve, kolay kurulum) ve **Apache Superset** (açık kaynak, güçlü). Seçim; ekip büyüklüğüne, teknik kapasiteye ve bütçeye göre değişir.

---

## 🧠 Detay

### Araç Karşılaştırması

| Özellik | Looker | Metabase | Superset | Power BI |
|---|---|---|---|---|
| **Fiyat** | Pahalı (Google Cloud) | Freemium | Açık kaynak | Microsoft lisans |
| **Kurulum** | SaaS | Self-hosted / Cloud | Self-hosted | Desktop + Cloud |
| **Teknik seviye** | Orta (LookML) | Düşük (GUI) | Orta (SQL) | Düşük-Orta |
| **SQL desteği** | LookML üzerinden | Native SQL | Native SQL | Power Query |
| **Embedding** | Mükemmel | İyi | İyi | Sınırlı |
| **Kaynak** | Kapalı | Açık kaynak | Açık kaynak | Kapalı |

### Looker — LookML ile Semantic Layer

```lookml
# LookML: SQL'i soyutlayan modelleme dili
view: siparisler {
  sql_table_name: dbo.siparisler ;;

  dimension: siparis_id {
    primary_key: yes
    type: number
    sql: ${TABLE}.id ;;
  }

  dimension_group: siparis_tarihi {
    type: time
    timeframes: [date, week, month, year]
    sql: ${TABLE}.created_at ;;
  }

  measure: toplam_tutar {
    type: sum
    sql: ${TABLE}.tutar ;;
    value_format_name: usd
  }

  measure: ortalama_tutar {
    type: average
    sql: ${TABLE}.tutar ;;
  }
}
```

**Looker'ın güçlü yanı:** Merkezi metrik tanımları → herkes aynı "gelir" tanımını kullanır, çelişkili rakamlar olmaz.

### Metabase — Hızlı Self-Serve Analytics

```bash
# Docker ile kurulum (5 dakika)
docker run -d -p 3000:3000 \
  -e "MB_DB_TYPE=postgres" \
  -e "MB_DB_DBNAME=metabase" \
  -e "MB_DB_PORT=5432" \
  -e "MB_DB_USER=user" \
  -e "MB_DB_PASS=password" \
  -e "MB_DB_HOST=postgres" \
  --name metabase metabase/metabase
```

**Metabase'in güçlü yanı:** SQL bilmeyen iş analistleri için soru-cevap arayüzü. Grafik, tablo, dashboard — tümü GUI ile.

### Apache Superset — Açık Kaynak Güç

```bash
# Docker Compose ile kurulum
git clone https://github.com/apache/superset
cd superset
docker compose up
# localhost:8088 → admin / admin
```

**Superset Özellikler:**
- 40+ grafik türü
- SQL Lab: gelişmiş sorgu editörü
- Dashboard: interaktif filtreler, drill-down
- Row-level security: kullanıcıya göre veri görünürlüğü
- Scheduled reports: otomatik e-posta raporları

### dbt + BI Entegrasyonu (Modern Stack)

```
Ham Veri (S3 / Warehouse)
         │
         ▼ dbt (transform)
Mart Tabloları (temiz, aggregate)
         │
         ├── Looker / Superset (dashboard)
         └── Python (ad-hoc analiz)
```

### Hangi Araç Ne Zaman?

| Senaryo | Öneri |
|---|---|
| Startup, hızlı kurulum | **Metabase** |
| Büyük kurumsal, Google ekosistemi | **Looker** |
| Açık kaynak, SQL odaklı ekip | **Superset** |
| Microsoft 365 ekosistemi | **Power BI** |
| Veri bilimci ekibi, esnek | **Superset** veya custom |

---

## 💡 Bağlantılar
- [[BI - Giriş ve Temel Kavramlar]]
- [[BI - Power BI vs Tableau Karşılaştırması]]
- [[DE - dbt (data build tool) ile Veri Modelleme]]
- [[DE - Veri Ambarı ve Modern Veri Yığını (BigQuery, Snowflake)]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- [Looker LookML Docs](https://cloud.google.com/looker/docs/lookml-intro)
- [Metabase Docs](https://www.metabase.com/docs/)
- [Apache Superset Docs](https://superset.apache.org/)
