---
tarih: 2026-06-08
konu: ML Interview Prep
etiket: [mülakat, interview, mlops, sistem-tasarımı, deployment, production]
kaynak: Chip Huyen Designing ML Systems, System Design Interview
zorluk: orta-ileri
---

## 📌 Özet

Kıdemli DS/MLE pozisyonlarında sıklıkla sorulan MLOps ve sistem tasarımı soruları. Cevaplar her zaman **trade-off'ları** açıklamayı içermelidir.

---

## 🧠 Detay

### ML Sistem Tasarımı Çerçevesi

```
1. PROBLEMİ ANLA
   ├── Girdi/çıktı ne? (metin, görüntü, sayısal)
   ├── Gecikme gereksinimleri? (real-time <100ms / batch / near-real-time)
   └── Ölçek? (günde kaç tahmin)

2. VERİ
   ├── Kaynaklar ve kalite
   ├── Etiketleme stratejisi
   └── Saklama / erişim

3. MODELLEMEu
   ├── Baseline (kural tabanlı, basit ML)
   ├── Metrik seçimi (business + teknik)
   └── Offline değerlendirme

4. SERVİS MİMARİSİ
   ├── Batch vs real-time inference
   ├── Feature store ihtiyacı?
   └── Model versiyonlama

5. İZLEME VE BAKIM
   ├── Veri drifti
   ├── Model drifti
   └── Yeniden eğitim tetikleyicileri
```

---

### Klasik Case: Öneri Sistemi Tasarla

**S: Büyük bir e-ticaret için ürün öneri sistemi tasarla.**

> **Adım 1 — Gereksinimler netleştir:**
> - Kaç kullanıcı? (100M+)
> - Gecikme? (<150ms için homepage)
> - Soğuk başlangıç sorunu var mı? (yeni kullanıcı/ürün)
>
> **Adım 2 — Veri:**
> - Tıklama, satın alma, arama, oturum logları
> - Kullanıcı profili, ürün metadatası
>
> **Adım 3 — Mimari:**
> ```
> Candidate Generation (ANN)
>         │ ~1000 aday
>         ▼
> Re-ranking Model (LightGBM/DNN)
>         │ ~50 öğe
>         ▼
> Business Rules (stok, kar marjı)
>         │ ~20 öğe
>         ▼
> Kullanıcıya Göster
> ```
>
> **Adım 4 — Metrikler:**
> - Offline: NDCG, MRR, Hit Rate@K
> - Online: CTR, GMV (brüt ticaret hacmi), dwell time

---

### Sık Sorulan MLOps Soruları

**S: Model drift nedir, nasıl tespit edilir?**
> **Data drift:** Girdi özelliklerinin dağılımı değişti
> **Concept drift:** Girdi-çıktı ilişkisi değişti (model eskidi)
> **Label drift:** Hedef değişkenin dağılımı değişti
>
> Tespit:
> - **PSI (Population Stability Index)** → özellik dağılım değişimi
> - **KS testi, Jensen-Shannon divergence** → dağılım karşılaştırma
> - **Performans izleme:** F1, AUC gerçek etiketle izle (label delay varsa)

```python
from evidently.report import Report
from evidently.metric_preset import DataDriftPreset

report = Report(metrics=[DataDriftPreset()])
report.run(reference_data=train_df, current_data=production_df)
report.save_html("drift_raporu.html")
```

---

**S: A/B testi vs shadow mode vs canary deploy farkı?**
> - **A/B testi:** Kullanıcı trafiği bölünerek iş metriği ölçülür
> - **Shadow mode:** Yeni model tahmin yapar ama kullanıcıya gösterilmez; loglar karşılaştırılır
> - **Canary deploy:** Trafiğin %5-10'u yeni modele yönlendirilir, stabil ise artırılır

---

**S: Feature store neden gerekli?**
> Aynı özelliklerin (örn. "son 7 gün kullanıcı alışverişi") farklı ekipler tarafından farklı şekillerde hesaplanmasını önler.
> - **Training-serving skew** önler: eğitimde ve production'da aynı hesaplama
> - **Reuse:** Bir ekip hesapladı, tümü kullandı
> - **Point-in-time correctness:** Data leakage'ı engeller

---

**S: Batch vs real-time inference ne zaman?**

| Kriter | Batch | Real-time |
|---|---|---|
| Gecikme | Saatler/günler | Ms/saniye |
| Maliyet | Düşük | Yüksek |
| Kullanım | Sabah raporu, email | Anlık öneri, fraud |
| Altyapı | Spark, Airflow | FastAPI, Kafka |

---

**S: Production'da model versiyonlama nasıl yapılır?**
> ```
> MLflow Model Registry:
>   Staging → Validation → Production → Archived
>
> Deployment stratejisi:
>   v1.0 → canary (5%) → v1.1 → blue-green → v2.0
>
> Rollback: önceki versiyona anında geçiş için model binary sakla
> ```

---

### Online Learning vs Batch Learning

```
Batch Learning:
  + Kararlı, tekrarlanabilir
  - Yeni veriyle güncelleme için yeniden eğitim

Online Learning (Incremental):
  + Anlık güncelleme
  - Concept drift'i hızla benimser (hem avantaj hem risk)
  
Kullanım: Reklam tıklama tahmini → online (her tıklama yeni bilgi)
          Dolandırıcılık tespiti → batch + periyodik güncelleme
```

---

## 💡 Bağlantılar
- [[MLOps - Model Monitoring ve Drift Tespiti]]
- [[MLOps - AB Test ve Shadow Mode]]
- [[MLOps - Feature Store]]
- [[System Design & Mimari]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- [Chip Huyen - Designing ML Systems](https://www.oreilly.com/library/view/designing-machine-learning/9781098107956/)
- [ML System Design Interview](https://www.educative.io/courses/machine-learning-system-design)
