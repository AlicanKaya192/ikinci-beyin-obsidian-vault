---
tarih: 2026-06-08
konu: ML Interview Prep
etiket: [mülakat, interview, case-study, vaka, data-science, problem-solving]
kaynak: Glassdoor, McKinsey Cases
zorluk: orta-ileri
---

## 📌 Özet

Büyük şirketlerde DS mülakatlarının önemli bir kısmı **açık uçlu vaka çalışmalarından** oluşur. Amaç düşünce sürecini, varsayımları ve trade-off'ları nasıl yönettiğini görmek. Bu notlar en sık sorulan case kategorilerini ve çözüm çerçevelerini içerir.

---

## 🧠 Detay

### Vaka Çalışması Çözüm Çerçevesi

```
1. PROBLEMİ NET ANLA
   ├── "Başarı nasıl ölçülüyor?"
   ├── "Kısıtlar neler? (zaman, veri, compute)"
   └── "Hangi karar alınacak bu sonuca göre?"

2. VERİYE BAK (var olduğunu varsay)
   ├── Hangi veriyi kullanırsın?
   └── Veri kalitesi sorunları neler olabilir?

3. YAKLAŞIM ÖNER
   ├── Basit baseline'dan başla
   ├── Karmaşıklık gerekçesini açıkla
   └── Alternatif yaklaşımları say

4. RİSKLERİ ADLANDIR
   └── Overfitting, data leakage, bias

5. BAŞARIYA KARAR VER
   ├── Offline metrik (AUC, RMSE...)
   └── Online metrik (CTR, revenue...)
```

---

### Case 1: Churn Tahmini

**Senaryo:** "Mobil uygulamamızda kullanıcı kaybı artıyor. Ne yaparsın?"

> **Problem tanımı:**
> - Churn tanımı: 30 gün boyunca uygulamayı açmayan kullanıcı
> - Hedef: 7 gün önceden churn'ü tahmin et (müdahale için süre tanı)
>
> **Özellikler (feature ideas):**
> - Son 7/14/30 gündeki oturum sayısı, süresi
> - Uygulama içi işlem geçmişi
> - Push notification açma oranı
> - Son özellik kullanım tarihi
> - Demografik: platform (iOS/Android), kayıt tarihi
>
> **Model yaklaşımı:**
> 1. Baseline: son 30 gün hiç giriş yapmamış → churn (kural tabanlı)
> 2. Logistic Regression → yorumlanabilirlik için
> 3. LightGBM → performans için
>
> **Başarı metrikleri:**
> - Precision/Recall tradeoff: Recall öncelikli (kaçırmamak önemli)
> - Business: Proaktif müdahale sonrası churn oranı azalması
>
> **Tuzaklar:**
> - Data leakage: "churn oldu" özelliğini feature olarak kullanmak
> - Zamansallık: futurist özellikler kullanmamak için proper time-split

---

### Case 2: Fraud Tespiti

**Senaryo:** "Ödeme sistemimizde dolandırıcılığı tespit et."

> **Zorluklar:**
> - Ciddi class imbalance: %0.1 fraud, %99.9 normal
> - Gecikme kritik: <100ms karar
> - Etiketler gecikmeli: chargeback'ler haftalarca sonra gelir
>
> **İki aşamalı yaklaşım:**
> ```
> Kural Motoru (hızlı red)
>         │ şüpheli işlemler
>         ▼
> ML Modeli (skor)
>         │ yüksek riskli
>         ▼
> Manuel İnceleme
> ```
>
> **Model:** Random Forest veya Isolation Forest (anomali tespiti için)
> **Metrik:** Precision @ yüksek recall (örn. recall=90%'da precision nedir?)

---

### Case 3: Fiyat Optimizasyonu (Pricing)

**Senaryo:** "Otel fiyatlarını dinamik olarak nasıl belirlersin?"

> **Price Elasticity:** Fiyat %1 artınca talep ne kadar düşer?
>
> **Yaklaşım:**
> 1. **Geçmiş veri analizi:** Price-demand ilişkisi histogramları
> 2. **Causal model:** Fiyat-talep nedenselliği (OLS, IV regresyon)
> 3. **Bandit algoritması:** Epsilon-greedy veya UCB ile fiyat testi
> 4. **Kısıtlar:** Min fiyat (maliyet altında gitme), rakip fiyatları
>
> **A/B testi zorluğu:** Farklı kullanıcılara farklı fiyat göstermek etik mi?
> → Segment bazlı (lokasyon, cihaz) → zamana göre (sabah/akşam)

---

### Case 4: Arama Sıralaması (Search Ranking)

**Senaryo:** "E-ticaret arama sonuçlarını nasıl sıralarım?"

> **Learning to Rank yaklaşımı:**
> - **Pointwise:** Her sonuç için skor tahmin et → basit regresyon
> - **Pairwise:** A > B mi? → binary classification
> - **Listwise:** Tüm sıralamayı optimize et → LambdaMART
>
> **Özellikler:**
> - Sorgu-ürün benzerliği (TF-IDF, BERT embedding)
> - Ürün özellikleri: rating, stok, fiyat
> - Kullanıcı geçmişi: önceki aramalar, satın almalar
>
> **Metrik:** NDCG@K (Normalized Discounted Cumulative Gain)

---

### Case 5: Duygu Analizi (Sentiment)

**Senaryo:** "Müşteri yorumlarından memnuniyeti ölç."

> **Veri:** Amazon/Trendyol yorumları, yıldız puanları
>
> **Yaklaşım tırmanması:**
> 1. Rule-based: VADER, TextBlob (hızlı baseline)
> 2. ML: TF-IDF + Logistic Regression
> 3. DL: BERT fine-tune → en yüksek doğruluk
>
> **Etiket sorunu:** Yıldız sayısı ≠ yorum tonu ("5 yıldız ama kargo yavaştı")
> → Aspect-based sentiment: ürün kalitesi, kargo, müşteri hizmetleri ayrı ayrı

---

## 💡 Bağlantılar
- [[Interview - MLOps ve Sistem Tasarımı Soruları]]
- [[ML - Anomali Tespiti]]
- [[ML - Öneri Sistemleri]]
- [[STAT - A-B Testi Tasarımı ve Analizi]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- [Emma Ding - Data Science Interviews](https://www.youtube.com/@DataInterviewPro)
- [Ace the Data Science Interview](https://www.acethedatascienceinterview.com/)
