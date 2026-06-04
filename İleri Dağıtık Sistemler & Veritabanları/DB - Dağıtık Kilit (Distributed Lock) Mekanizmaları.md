---
tarih: 2026-06-04
konu: Dağıtık Kilit (Distributed Lock)
etiket: [distributed-systems, lock, redis, zookeeper, concurrency]
kaynak: ""
zorluk: İleri
---
## 📌 Özet
Dağıtık sistemlerde, aynı anda birden fazla servisin veya uygulamanın ortak bir kaynağa (veritabanı, dosya sistemi, dış API) eşzamanlı erişimini (concurrency) düzenlemek için kullanılan senkronizasyon araçlarına Dağıtık Kilit (Distributed Lock) denir. Tek sunuculu (monolith) mimarilerde thread veya process seviyesinde kullanılan lokal kilitler (mutex, semaphore), yatay olarak ölçeklenen mikroservis ortamlarında etkisiz kalır. Bu tür ağ tabanlı kilit sistemleri, karşılıklı dışlamayı (mutual exclusion) garanti etmeli, aynı zamanda ağ bölünmeleri (network partition) veya süreç çökmeleri (process crash) durumlarında deadlock (kilitlenme) oluşumunu engellemek için TTL (Time-To-Live) gibi zaman aşımı mekanizmaları içermelidir. Redis (Redlock algoritması), Apache ZooKeeper veya etcd gibi yüksek erişilebilir, güçlü tutarlılık (strong consistency) sağlayan sistemler genellikle bu kilitlerin merkezi yöneticisi olarak kullanılır. Dağıtık kilitlerin yanlış implementasyonu sistemin performansını darboğaza sokabileceği gibi, kaynakların birden fazla düğüm tarafından aynı anda değiştirilmesi sonucu telafisi zor veri bozulmalarına da (data corruption) yol açabilir.

## ⚙️ Teknik Detaylar

### Neden Dağıtık Kilit Kullanırız?
1. **Verimlilik (Efficiency):** Ağır veya maliyetli bir işlemin (örneğin gece çalışan ağır bir raporlama işi, veya 3. parti paralı bir API çağrısı) birden fazla node tarafından aynı anda yapılmasını engelleyerek kaynak tasarrufu sağlamak.
2. **Doğruluk (Correctness):** Ortak bir verinin aynı anda değiştirilerek (Race Condition) veri bütünlüğünün (Data Integrity) bozulmasını önlemek. Örn: Bir siparişin stok miktarını aynı anda düşürmeye çalışan iki farklı instance.

### Dağıtık Kilidin Taşıması Gereken Özellikler
- **Mutual Exclusion (Karşılıklı Dışlama):** Herhangi bir anda sadece tek bir client kilide sahip olmalıdır.
- **Deadlock Free (Kilitlenme Koruması):** Kilidi alan client çökse veya ağdan kopsa bile kilit sonsuza dek kilitli kalmamalıdır (TTL / Lease).
- **Fault Tolerance (Hata Toleransı):** Kilit yöneticisi sunuculardan biri çökse bile sistem kilitleri yönetmeye devam etmelidir.

```mermaid
graph TD
    App1["Service Instance 1"] -->|1. Try Lock| DL["Distributed Lock Manager (Redis/ZooKeeper)"]
    App2["Service Instance 2"] -->|2. Try Lock (Blocked)| DL
    App3["Service Instance 3"] -->|3. Try Lock (Blocked)| DL
    DL -.->|Lock Granted| App1
    App1 -->|Access Resource| DB[("Shared Database/Resource")]
    App1 -.->|Release Lock| DL
```

### Redis Üzerinde Dağıtık Kilit (Redlock Algoritması)
Redis, hızlı ve bellek içi olduğu için kilit mekanizmalarında sıkça tercih edilir.

- **Tek Node Redis Kilit Mantığı:** `SET resource_name my_random_value NX PX 30000`
  - `NX`: Yalnızca anahtar yoksa oluştur (Mutex garantisi).
  - `PX`: 30 saniye sonra otomatik sil (Deadlock koruması).
- **Redlock Algoritması:** Tek Redis sunucusu çökerse kilitler kaybolur (SPOF). Redlock, kilit işlemini (N adet bağımsız) Redis cluster'ına gönderir. Eğer çoğunluktan (Quorum, `(N/2)+1`) belirtilen süre zarfında başarılı yanıt alırsa kilit elde edilmiş sayılır.
- **Kilit Serbest Bırakma (Unlock):** Sadece kilidi oluşturan sahip silebilir. Bu işlem için anahtarın değeri (UUID) kontrol edilerek silinir (Atomik işlem için Lua Script kullanılır).

### Apache ZooKeeper ve etcd ile Kilit
Redis eventual consistent olduğu için aşırı kritik (örn. finansal) işlemlerde %100 güvenlik sağlamayabilir. Bu durumlarda CP tabanlı sistemler kullanılır.

- **ZooKeeper Yaklaşımı:** ZK, hiyerarşik dosya sistemi gibi çalışır. `Ephemeral Sequential` node'lar kullanılarak kilit oluşturulur.
  1. Client, `/lock` dizini altında geçici ve ardışık bir dosya yaratır (örn. `node_001`).
  2. Dizin altındaki en küçük numaralı dosya kendisiyse, kilit onundur.
  3. Değilse, bir önceki dosyanın silinmesini bekler (Watch mechanism).
  4. Client kopsa bile (session timeout), ZK geçici dosyayı otomatik silerek kilidi açar.

### Kilit Kırılma Senaryoları (Fencing Tokens)
Dağıtık sistemlerde "Pause" (GC Pause veya Thread dondurma) durumları tehlikelidir.
1. Client kilidi alır, ardından 10 saniyelik bir GC (Garbage Collection) pause yaşar.
2. TTL dolar, kilit otomatik açılır. Başka bir client kilidi alır ve veritabanına yazar.
3. İlk client uyanır, kilit hala kendisinde zannedip veritabanına yazar (Veri ezildi).
**Çözüm:** *Fencing Token*. Lock mekanizması her başarılı kilit işleminde artan bir token (örn. 33) verir. Veritabanı sadece mevcut tokendan daha büyük bir token (örn. 34) gelirse işlemi kabul edecek şekilde tasarlanmalıdır.