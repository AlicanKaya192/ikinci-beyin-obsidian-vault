---
tarih: 2026-06-04
konu: MongoDB Şema Tasarımı ve Veri Modelleme
etiket: [mongodb, schema-design, data-modeling, embedding]
kaynak: MongoDB Patterns
zorluk: İleri
---

## 📌 Özet
MongoDB'de şema tasarımı, "uygulama veriyi nasıl okuyacak?" sorusu etrafında şekillenir. Normalize edilmiş tablolar yerine, verilerin iç içe gömülmesi (Embedding) veya ID ile bağlanması (Referencing) arasında seçim yapılır. Esnek şema, her dökümanın farklı alanlara sahip olmasına izin verir. Tasarımın temel amacı, okuma hızını maksimize etmek ve JOIN maliyetini düşürmektir.

## 🧠 Detay

```mermaid
graph TD
    A["Tasarım Kararı"] --> B["Embedding (Gömme)"]
    A --> C["Referencing (Referans)"]
    
    B -- "Artı" --> B1["Hızlı Okuma (Tek Sorgu)"]
    B -- "Eksi" --> B2["16MB Limit / Veri Tekrarı"]
    
    C -- "Artı" --> C1["Esneklik / Normalize"]
    C -- "Eksi" --> C2["$lookup Gerekir (Yavaş)"]
```

### 1. Temel İlke
"Veri beraber okunuyorsa, beraber saklanmalıdır."

### 2. İlişki Yönetimi
- **1:1**: Genellikle Embedding.
- **1:N**: Az veri varsa Embedding, çok veri (binlerce) varsa Referencing.

## 💡 Bağlantılar
- [[MongoDB - Data Modeling Design Patterns]]
- [[MongoDB - Giriş ve Temel Kavramlar]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- Data Modeling Introduction
- MongoDB Schema Design Best Practices
