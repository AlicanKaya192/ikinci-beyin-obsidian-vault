---
tarih: 2026-08-17
konu: Kariyer & Portfolio
etiket: [kariyer, cv, linkedin, iş-arama, data-science]
kaynak: Glassdoor, LinkedIn Career Advice
zorluk: başlangıç
---

## 📌 Özet

DS/ML CV'si teknik pozisyon için optimize edilmeli: ATS (Applicant Tracking System) taramasını geçmeli, teknik becerileri net göstermeli ve iş etkisini sayısal olarak ifade etmeli. LinkedIn ise ağ kurma ve keşfedilirlik için.

---

## 🧠 Detay

### CV Yapısı — DS/ML Pozisyonu İçin

```
1. İletişim Bilgisi
   Ad Soyad | E-posta | LinkedIn | GitHub | Şehir

2. Özet (3-4 satır)
   "X yıllık Python ve ML deneyimiyle [etki/başarı].
    [Domain] odaklı, production ML geliştirme konusunda
    [somut başarı]."

3. Teknik Beceriler
   Programlama: Python, SQL, R
   ML: scikit-learn, XGBoost, PyTorch, Hugging Face
   MLOps: Docker, MLflow, FastAPI, Git, DVC
   Cloud: AWS (S3, EC2, SageMaker), GCP
   BI: Power BI, Tableau, Metabase

4. Deneyim (Ters kronoloji)
   [Şirket] | [Pozisyon] | [Tarih]
   • RAKAMLA başla: "Churn modeliyle aylık ₺500K müşteri kaybı önlendi"
   • Araç + Sonuç formatı: "XGBoost ile fraud tespiti — F1 %89"
   • Aktif fiil: Geliştirdim, kurdum, azalttım, artırdım

5. Eğitim

6. Projeler (deneyim yoksa üstte olabilir)

7. Sertifikalar (opsiyonel)
```

### ATS Geçme Stratejisi

ATS, CV'yi taramadan önce anahtar kelimeler arar:

```python
# İş ilanından anahtar kelimeleri çıkar
ilan_anahtar_kelimeler = [
    "machine learning", "deep learning", "Python",
    "SQL", "model deployment", "scikit-learn"
]

# CV'nde bu kelimelerin geçtiğinden emin ol
# Özellikle 'Teknik Beceriler' bölümünde
```

**ATS kuralları:**
- Tablo, grafik, sütun kullanma → ATS okuyamaz
- Standart bölüm başlıkları: "Experience", "Skills", "Education"
- PDF yerine Word öner (bazı ATS'ler PDF okuyamaz)
- Tek sütunlu, düz metin öncelikli

### Deneyim Bölümü — STAR Formatı

```
Durum: "Müşteri churn oranı aylık %8'e yükselmişti"
Görev: "Proaktif müdahale modeli geliştirmem istendi"
Aksiyon: "LightGBM ile 7 gün önceden churn tahmini kurdum,
          SMOTE ile class imbalance çözdüm"
Sonuç: "Churn oranı %8 → %5.2'ye düştü, aylık ₺400K tasarruf"

CV formatı (tek satır):
"LightGBM churn modeli geliştirerek aylık müşteri kaybını
 %35 azalttım (₺400K tasarruf) — SMOTE + feature engineering."
```

### LinkedIn Profil Optimizasyonu

#### Başlık (Headline) — En Kritik Alan

```
Kötü:  "Data Scientist at Acme Corp"
İyi:   "ML Engineer | Python · PyTorch · MLOps | Üretim ML sistemleri geliştiriyorum"
İyi:   "Data Scientist | Churn Tahmini · NLP · LLM | Açık kaynak katkıcı"
```

#### About Bölümü

```markdown
Ben [Ad] — [X yıl] boyunca [sektör] alanında ML çözümleri geliştiriyorum.

Uzmanlık alanlarım:
→ Üretimde model geliştirme (FastAPI + Docker + MLflow)
→ NLP & LLM uygulamaları (RAG, fine-tuning)
→ Veri pipeline tasarımı (dbt + Airflow)

En son: [Proje veya başarı]

Bağlantı kurmaktan memnuniyet duyarım — [email]
```

#### Featured Bölümü

LinkedIn'de "Featured" alanına şunları ekle:
- GitHub profil linki
- En iyi Kaggle notebook
- Medium/blog yazısı (varsa)
- Portfolio web sitesi

#### LinkedIn Algoritmasyonu

```
Daha fazla görünürlük için:
  ✓ Haftada 2-3 gönderi (proje güncellemeleri, öğrendiklerin)
  ✓ Başkalarının gönderilerine yorum yap (salt beğeni değil)
  ✓ "Open to Work" banner'ı aç (işe alım uzmanları filtreler)
  ✓ Skills bölümüne ekle → endorse al
  ✓ 500+ bağlantı → "500+ connections" rozetini aç
```

### Sertifikalar — Hangileri Değerli?

| Sertifika | Platform | Ağırlık |
|-----------|----------|---------|
| AWS Certified ML Specialty | AWS | ★★★★★ |
| Google Professional ML Engineer | GCP | ★★★★★ |
| TensorFlow Developer | Google | ★★★★☆ |
| Deep Learning Specialization | Coursera (Andrew Ng) | ★★★★☆ |
| MLOps Specialization | Coursera | ★★★☆☆ |
| Kaggle Certifications | Kaggle | ★★★☆☆ |
| DataCamp certifications | DataCamp | ★★☆☆☆ |

> **Not:** Sertifikalar deneyimin yoksa yardımcı olur, deneyimin varsa önemsizdir. Proje önce gelir.

### Cover Letter — DS Pozisyonları İçin

```
Paragraf 1: Neden bu şirketi seçtim? (Spesifik araştırma yap)
Paragraf 2: Bir proje örneği — tam STAR formatı
Paragraf 3: Bu pozisyon + geçmiş deneyim bağlantısı
Paragraf 4: Harekete geçirici — "görüşmek isterim"
```

---

## 💡 Bağlantılar
- [[Kariyer - GitHub Profil Optimizasyonu]]
- [[Kariyer - Mülakat Süreci ve Hazırlık Stratejisi]]
- [[00 - ML Mülakat Hazırlık Rehberi]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- [Jobscan - ATS CV Checker](https://www.jobscan.co/)
- [Resume Worded](https://resumeworded.com/)
- [LinkedIn Learning - Job Search](https://learning.linkedin.com/)
