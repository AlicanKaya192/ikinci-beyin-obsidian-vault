---
tarih: 2026-06-08
konu: ML Interview Prep
etiket: [mülakat, interview, data-science, machine-learning, kariyer]
kaynak: Glassdoor, LeetCode, NeetCode
zorluk: başlangıç
---

## 📌 Özet

Bu klasör, **Veri Bilimi, Makine Öğrenmesi ve MLOps** pozisyonları için mülakat hazırlık notlarını içerir. Soru bankalarından teknik kavramlara, pratik kodlamadan vaka çalışmalarına kadar kapsamlı bir kaynak.

---

## 🧠 Detay

### Şirket Türüne Göre Mülakat Formatları

| Şirket Tipi | Odak | Ağırlık |
|---|---|---|
| **FAANG/MAANG** | Algoritma + ML theory + System Design | %40 kod / %40 ML / %20 SD |
| **Startup** | Hızlı prototip + pratik ML | %60 pratik / %40 teori |
| **Danışmanlık** | Vaka çalışması + iletişim | %50 case / %50 teknik |
| **Fintech/Sigorta** | İstatistik + risk modelleme | %60 istatistik |

### Hazırlık Yol Haritası (8 Hafta)

```
Hafta 1-2: Temeller
  ├── İstatistik & Olasılık → [[Interview - İstatistik ve Olasılık Soruları]]
  └── Python & SQL          → [[Interview - Python ve Kodlama Soruları]]

Hafta 3-4: ML Teori
  └── Klasik ML algorithms  → [[Interview - Makine Öğrenmesi Temel Sorular]]

Hafta 5-6: Derin Öğrenme & LLM
  └── DL + Transformers     → [[Interview - Derin Öğrenme ve LLM Soruları]]

Hafta 7: MLOps & System Design
  └── Production ML         → [[Interview - MLOps ve Sistem Tasarımı Soruları]]

Hafta 8: Vaka Çalışmaları
  └── End-to-end problems   → [[Interview - Data Science Vaka Çalışmaları]]
```

### En Sık Sorulan Konular (2024-2025)

1. **Bias-Variance tradeoff** — her mülakatın klasiği
2. **Transformer / Attention mekanizması** — GenAI çağında zorunlu
3. **A/B Test tasarımı** — data scientist standartı
4. **Feature importance & SHAP** — model yorumlanabilirlik
5. **Imbalanced dataset** — neredeyse her projede karşılaşılır
6. **RAG mimarisi** — 2024-2025 trendi
7. **SQL pencere fonksiyonları** — veri mühendisliği kritik
8. **System design: öneri sistemi** — sık sorulan case

### Davranışsal Sorular (STAR Yöntemi)

```
S (Situation): Hangi bağlamdasındı?
T (Task):      Görevin neydi?
A (Action):    Ne yaptın?
R (Result):    Sonuç ne oldu? (sayısal)

Örnek: "Bir ML modelini production'a aldığında ne oldu?"
→ "Fraud detection projesinde (S) modeli FastAPI ile servis etmem gerekiyordu (T).
   Docker + Kubernetes deploy ettim, Prometheus ile izledim (A).
   Gecikme %40 azaldı, yanlış alarm %15 düştü (R)."
```

---

## 💡 Bağlantılar
- [[Interview - Makine Öğrenmesi Temel Sorular]]
- [[Interview - Derin Öğrenme ve LLM Soruları]]
- [[Interview - İstatistik ve Olasılık Soruları]]
- [[Interview - Python ve Kodlama Soruları]]
- [[Interview - MLOps ve Sistem Tasarımı Soruları]]
- [[Interview - Data Science Vaka Çalışmaları]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- [ML Interview Guide - Chip Huyen](https://huyenchip.com/ml-interviews-book/)
- [Glassdoor DS Interviews](https://www.glassdoor.com)
