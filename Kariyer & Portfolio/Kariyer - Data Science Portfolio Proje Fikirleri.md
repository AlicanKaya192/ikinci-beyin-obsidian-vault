---
tarih: 2026-08-17
konu: Kariyer & Portfolio
etiket: [kariyer, portfolio, proje, data-science, github]
kaynak: Towards Data Science, Ken Jee
zorluk: başlangıç
---

## 📌 Özet

İşe alım uzmanları bir DS portfolio'sunda ne arar? Teknik karmaşıklıktan çok **iş sorusu netliği**, **hikaye anlatımı** ve **deployment**. Bu not, etki yaratan proje fikirlerini ve nasıl sunulacağını açıklar.

---

## 🧠 Detay

### Harika Bir Portfolio Projesi = 3 Unsur

```
1. GERÇEK BİR SORUN
   → "Makine öğrenmesi uyguladım" değil
   → "X problemini Y% daha iyi çözdüm"

2. UÇTAN UCA
   → Veri toplama → EDA → Model → Deployment → Monitoring

3. HİKAYE
   → README neden bu problemi seçtin?
   → Ne öğrendin? Ne işe yaramadı?
```

### Proje Fikirleri — Seviyeye Göre

#### 🟢 Başlangıç Projeleri (ilk 2-3 ay)

| Proje | Teknikler | Veri Kaynağı |
|-------|-----------|--------------|
| Ev Fiyat Tahmini | Linear/Ridge Regression, EDA | Kaggle Housing |
| Müşteri Churn Tahmini | Logistic Reg., Random Forest | Kaggle Telco |
| Film/Kitap Öneri Sistemi | Collaborative Filtering | MovieLens |
| COVID-19 Trend Analizi | Zaman serisi, görselleştirme | Our World in Data |
| Türkiye Hava Durumu EDA | pandas, matplotlib, seaborn | MGM / Kaggle |

#### 🟡 Orta Seviye Projeler (3-6 ay)

| Proje | Teknikler | Neden İyi? |
|-------|-----------|------------|
| Haber Kategorisi Sınıflandırıcı | NLP, TF-IDF, BERT | NLP + Turkish text |
| Borsa Fiyat Tahmini | LSTM, Prophet, XGBoost | Zaman serisi + finans |
| E-ticaret Ürün Öneri API'si | Matrix Factorization + FastAPI | Deploy dahil |
| Sosyal Medya Duygu Analizi | Transformers, Hugging Face | LLM dünyasına giriş |
| Anomali Tespiti Sistemi | Isolation Forest, Autoencoder | Sektörel kullanım |

#### 🔴 İleri Projeler (6+ ay)

| Proje | Teknikler | Fark Yaratan |
|-------|-----------|--------------|
| Türkçe RAG Sistemi | LangChain, pgvector, Claude API | LLM + Türkçe |
| Gerçek Zamanlı Fraud Detection | Kafka, Redis, ML serving | Stream processing |
| End-to-end MLOps Pipeline | MLflow + Docker + K8s + CI/CD | Production-grade |
| Multimodal Görsel+Metin Arama | CLIP, Qdrant | Modern AI |
| Federated Learning Demo | Flower framework | Privacy ML |

---

### Proje Sunumu — README Şablonu

```markdown
# Proje Adı — Kısa ve Etkileyici

## 🎯 Problem
Neden bu problemi seçtim? Kim için faydalı?

## 💡 Çözüm
Hangi yaklaşımı seçtim ve neden?

## 📊 Sonuçlar
- Model A: %85 doğruluk (baseline)
- Model B: %92 doğruluk (final)
- Üretimde X ms gecikme

## 🛠️ Kullanılan Teknolojiler
Python | pandas | scikit-learn | FastAPI | Docker

## 🚀 Kurulum ve Çalıştırma
\`\`\`bash
git clone ...
pip install -r requirements.txt
python app.py
\`\`\`

## 📁 Proje Yapısı
## 🔄 Gelecek Adımlar
## 📬 İletişim
```

### Portfolio Web Sitesi vs GitHub

```
Sadece GitHub: ✓ Teknik kitleye yeterli
              ✗ İşe alım uzmanları için zor gezinme

Portfolio site: ✓ Profesyonel görünüm
               ✓ CV gibi bağlantı verilebilir
               → GitHub Pages (ücretsiz) veya Streamlit Share

En iyi kombinasyon:
  GitHub (kodlar) + Streamlit (demo) + LinkedIn (tanıtım)
```

### Hangi Projeyi İlk Yap?

```python
if hedef == "ilk iş":
    önce = "Churn veya Fiyat Tahmini — bilinen problem, ölçülebilir başarı"
elif hedef == "yükseltme":
    önce = "End-to-end MLOps — production deneyimi göster"
elif hedef == "startup":
    önce = "Gerçek veriyle LLM uygulaması — güncel teknoloji"
```

### İşe Alım Uzmanlarının Aradığı

- **Temiz kod** → docstring, type hints, testler
- **Açıklama** → "neden bu algoritma?" yorumları
- **Gerçek veri** → toy dataset değil, Kaggle/API/scraping
- **Deployment** → Hugging Face Spaces, Streamlit Cloud, render.com

---

## 💡 Bağlantılar
- [[Kariyer - GitHub Profil Optimizasyonu]]
- [[Kariyer - CV ve LinkedIn Hazırlama]]
- [[DATA SCIENCE & ML ÖĞRENME YOL HARİTASI]]
- [[FastAPI - Giriş ve Proje Yapısı]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- [Ken Jee - Portfolio Review](https://www.youtube.com/watch?v=agBNOFGHHf0)
- [Towards Data Science - Portfolio Guide](https://towardsdatascience.com/)
- [Streamlit Cloud](https://streamlit.io/cloud)
