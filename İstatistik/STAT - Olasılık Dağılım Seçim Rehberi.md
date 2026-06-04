---
tarih: 2025-01-01
konu: Olasılık Dağılımları, Dağılım Seçimi
etiket: [istatistik, olasılık, dağılım, normal, poisson, binom]
kaynak:
zorluk: ⭐⭐⭐
---

## 📌 Özet
Doğru olasılık dağılımını seçmek, verinin doğasını anlamak ve doğru istatistiksel modelleri kurmak için esastır. Verinin kesikli veya sürekli olması ve oluşma biçimi seçimi belirler.

---

## 🧠 Detay

### 🗺️ Olasılık Dağılım Seçim Karar Ağacı

```mermaid
graph TD
    Start[Veri Tipi?] --> Type{Kesikli mi?}
    
    Type -- Evet (Tam Sayı) --> D_Goal{Amaç?}
    D_Goal -- Başarı/Başarısızlık Sayısı --> Binom{Deney Sayısı?}
    Binom -- Sabit N Deney --> Binomial[Binom Dağılımı]
    Binom -- İlk Başarıya Kadar --> Geometric[Geometrik Dağılım]
    Binom -- k. Başarıya Kadar --> NegBin[Negatif Binom]
    
    D_Goal -- Zaman/Mekan Aralığında Olay --> Poisson[Poisson Dağılımı]
    
    Type -- Hayır (Sürekli) --> C_Goal{Dağılım Şekli?}
    C_Goal -- Çan Eğrisi / Simetrik --> Normal[Normal Dağılım]
    C_Goal -- Pozitif Çarpık / Bekleme Süresi --> Exponential[Üstel Dağılım]
    C_Goal -- Tüm Değerler Eşit Olası --> Uniform[Düzgün Dağılım]
    C_Goal -- Oranlar / 0-1 Arası --> Beta[Beta Dağılımı]
```

### Önemli Dağılımların Özeti

| Dağılım | Kullanım Alanı | Önemli Parametre |
|---|---|---|
| **Normal** | Boy, Kilo, Ölçüm Hataları | $\mu$ (Ortalama), $\sigma$ (Std) |
| **Binom** | Yazı-Tura, Geçti-Kaldı | $n$ (Deney), $p$ (Olasılık) |
| **Poisson** | Saatlik müşteri sayısı, radyoaktif bozunma | $\lambda$ (Ortalama oran) |
| **Üstel** | İki çağrı arası geçen süre | $\lambda$ (Hız) |
| **Log-Normal** | Gelir dağılımı, hisse senedi fiyatları | $\mu, \sigma$ (Logaritmik) |

---

## 💡 Bağlantılar
- [[STAT - Olasılık Dağılımları (Kesikli)]]
- [[STAT - Olasılık Dağılımları (Sürekli)]]
- [[STAT - Normal Dağılım]]

## ❓ Sorular / Anlamadıklarım
- Merkezi Limit Teoremi neden her şeyi Normal dağılıma yaklaştırır?

## 🔗 Kaynaklar
- Common Probability Distributions Guide (Towards Data Science)
