---
tarih: 2026-06-04
konu: Sharding ve Veri Göçü
etiket: [sharding, partitioning, data-migration, zero-downtime, distributed]
kaynak: ""
zorluk: İleri
---
## 📌 Özet
Tek bir veritabanı sunucusunun disk, CPU veya RAM sınırlarına (dikey ölçekleme - vertical scaling) ulaşıldığında, verinin yatay olarak (horizontal scaling) birden fazla bağımsız sunucuya bölünmesi işlemine Sharding (Parçalama) adı verilir. Veri, mantıksal bir Şartlama Anahtarı (Shard Key) kullanılarak parçalara ayrılır ve yük, kümedeki düğümler arasına dağıtılarak sistemin toplam işlem kapasitesi (throughput) artırılır. Ancak sharding, transaction yönetimini (cross-shard transactions) karmaşıklaştırır ve Join operasyonlarını neredeyse imkansız hale getirerek mimari karmaşıklığı ciddi boyutta artırır. Sistemin büyüyüp yeni shard'ların eklenmesi veya mevcut monolitik yapının parçalanmış mimariye geçirilmesi durumunda, Veri Göçü (Data Migration) kritik bir operasyon halini alır. Bu geçiş sürecini, müşteri deneyimini etkilemeden Sıfır Kesinti (Zero-Downtime) ile tamamlayabilmek için Çift Yazma (Dual-Write) veya CDC (Change Data Capture) gibi ileri seviye stratejilerin titizlikle orkestre edilmesi gerekmektedir.

## ⚙️ Teknik Detaylar

### Parçalama (Sharding) Stratejileri
Verinin nasıl bölüneceği (Shard Key seçimi) sistemin geleceğini belirler. Yanlış bir anahtar, verinin bir düğümde birikmesine (Hotspot) yol açar.

1. **Range-Based (Aralık Tabanlı) Sharding:**
   - Veriler belirli değer aralıklarına göre bölünür (Örn: A-H arası kullanıcılar Shard1'e, I-Z arası Shard2'ye; ya da Tarihlere göre aylık shardlama).
   - *Avantaj:* Aralık sorguları (Range Query) çok hızlıdır.
   - *Dezavantaj:* Yeni veriler hep son shard'a yazılıyorsa o düğüm darboğaz (hotspot) olur.
2. **Hash-Based (Hash Tabanlı) Sharding:**
   - Shard anahtarının (örn. UserID) hash değeri alınarak `Hash(Key) % N` formülüyle verinin gideceği shard belirlenir (Redis, Cassandra kullanır).
   - *Avantaj:* Veri dağılımı rastgele ve eşittir (Hotspot engellenir).
   - *Dezavantaj:* Düğüm sayısı (N) değiştiğinde tüm verilerin yer değiştirmesi (Rehashing) gerekir. Bu sorunu çözmek için **Consistent Hashing** kullanılır.
3. **Directory/Routing-Based Sharding:**
   - Hangi verinin hangi shard'da olduğunu tutan merkezi bir "Lookup Table" (Yönlendirme Tablosu) kullanılır. Esnektir ama yönlendirici tek nokta hatası (SPOF) olabilir.

```mermaid
graph TD
    App["Application Layer"] --> Router["Shard Router / API Gateway"]
    Router -->|Hash(UID)=1| Shard1["Shard 1 (DB Node)"]
    Router -->|Hash(UID)=2| Shard2["Shard 2 (DB Node)"]
    Router -->|Hash(UID)=3| Shard3["Shard 3 (DB Node)"]
```

### Sharding'in Yarattığı Sorunlar
- **Cross-Shard Joins:** Farklı düğümlerdeki verileri birleştirmek ağ maliyeti doğurur. Çözüm olarak veri denormalizasyonu yapılmalı veya sık okunan tablolar tüm shard'lara kopyalanmalıdır (Global Tables).
- **Dağıtık Transactionlar:** Bir işlem birden fazla shard'ı etkiliyorsa Two-Phase Commit (2PC) veya Saga Pattern kullanılmalıdır, bu da performansı düşürür.

### Sıfır Kesintiyle Veri Göçü (Zero-Downtime Migration)
Mevcut büyük bir veritabanını sharded yapıya taşımak veya altyapı değiştirmek için kesinti olmadan geçiş stratejileri:

#### 1. Dual-Write (Çift Yazma) Stratejisi
1. **Hazırlık:** Yeni (Sharded) veritabanı altyapısı kurulur.
2. **Kod Güncellemesi:** Uygulama katmanına, gelen her yazma (Insert/Update/Delete) işlemini eş zamanlı olarak hem Eski hem Yeni veritabanına yazacak kod eklenir. Okumalar sadece Eski DB'den yapılır.
3. **Historical Sync:** Çift yazma devredeyken, arkada bir script (Backfill) ile Eski DB'deki geçmiş veriler Yeni DB'ye aktarılır. Yeni gelen çift yazma komutları aktarılan geçmiş verinin üstüne yazıldığı için ezilme olmaz.
4. **Doğrulama:** Yeni DB'deki veri bütünlüğü kontrol edilir.
5. **Switch-over:** Okumalar da Yeni DB'ye yönlendirilir. Çift yazma kapatılır.

#### 2. CDC (Change Data Capture) ile Göç
Çift yazma stratejisi kod tarafında karmaşıklık yaratıyorsa, veritabanı seviyesinde CDC araçları (Örn: Debezium) kullanılır.
1. CDC aracı Eski DB'nin log dosyasını (WAL/Binlog) okuyarak anlık değişiklikleri bir Kafka kuyruğuna atar.
2. Consumer uygulamalar bu kuyruktaki veriyi asenkron olarak Yeni DB'ye sürekli yazar.
3. Geçmiş veri ve anlık loglar senkronize olduğunda kısa bir okuma-yazma kilitlenmesiyle geçiş (Switch) yapılır.