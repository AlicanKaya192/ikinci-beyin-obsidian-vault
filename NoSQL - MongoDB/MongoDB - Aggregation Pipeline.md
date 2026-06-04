---
tarih: 2026-06-04
konu: MongoDB Aggregation Pipeline
etiket: [mongodb, aggregation, pipeline, analytics]
kaynak: MongoDB University
zorluk: İleri
---

## 📌 Özet
Aggregation Pipeline, MongoDB'de verileri işlemek, dönüştürmek ve karmaşık analizler yapmak için kullanılan çok aşamalı bir veri işleme çerçevesidir. SQL'deki `GROUP BY` ve `JOIN` işlemlerinin çok daha gelişmiş bir versiyonu olarak düşünülebilir. Veriler bir aşamadan geçip (filter, group, sort) sonucu bir sonraki aşamaya aktarılarak nihai rapor oluşturulur. `$match`, `$group`, `$project` ve `$lookup` en sık kullanılan operatörlerdir.

## 🧠 Detay

```mermaid
graph LR
    A["Koleksiyon"] --> B["$match (Süzme)"]
    B --> C["$group (Özetleme)"]
    C --> D["$sort (Sıralama)"]
    D --> E["$limit (Kısıtlama)"]
    E --> F["Final Çıktı"]
```

### 1. Temel Aşamalar
- **$match:** Sorgu kriterlerine göre dökümanları seçer. İndeks kullanması için en başta olmalıdır.
- **$group:** Belirli bir anahtara göre gruplama yapar ve toplam/ortalama hesaplar.
- **$project:** Hangi alanların sonuçta yer alacağını belirler.

### 2. $lookup (Join)
Başka bir koleksiyondan veri çekerek dizi olarak ekler.
```javascript
{ $lookup: { from: "orders", localField: "_id", foreignField: "user_id", as: "orders" } }
```

## 💡 Bağlantılar
- [[MongoDB - Sorgulama ve Operatörler]]
- [[MongoDB - İndeksler ve Performans]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- MongoDB Aggregation Stages Reference
- Practical MongoDB Aggregation (Book)
