---
tarih: 2026-06-08
konu: BI — İş Zekası (Power BI & Tableau)
etiket: [bi, dashboard, kpi, görselleştirme, tasarım, data-storytelling]
kaynak: Storytelling with Data, Few's Dashboard Design
zorluk: başlangıç
---

## 📌 Özet

İyi bir KPI dashboard'u teknik yeterlilik değil **iletişim** meselesidir. Doğru metriği seçmek, doğru grafikle sunmak ve gereksiz gürültüyü temizlemek — bunların tümü veri hikayeciliğinin temelidir.

---

## 🧠 Detay

### Dashboard Hiyerarşisi

```
Seviye 1: Yönetici Dashboard
  → 3-5 KPI, trafik ışığı renkleri, trendi
  → 30 saniyede okunabilir

Seviye 2: Operasyonel Dashboard
  → Departman metrikleri, günlük/haftalık trend
  → 2-5 dakikada incelenebilir

Seviye 3: Analitik Dashboard
  → Detaylı drilldown, segmentasyon, filtreler
  → 15-30 dakika analiz için
```

### KPI Seçim Çerçevesi (SMART Metrikler)

| Kriter | Açıklama | Örnek |
|---|---|---|
| **S**pecific | Belirsiz değil, net | "Aktif kullanıcı" değil "Son 7 gün ≥1 oturum" |
| **M**easurable | Sayısal olarak ölçülebilir | Conversion rate % |
| **A**ctionable | Harekete geçirilebilir | Sepet terk oranı → UX aksiyonu |
| **R**elevant | İş hedefiyle bağlantılı | CAC < LTV |
| **T**ime-bound | Dönem tanımlı | Aylık MRR büyümesi |

### Grafik Seçim Rehberi

| Göstermek İstediğin | Grafik Türü |
|---|---|
| Zaman trendi | Çizgi grafik |
| Karşılaştırma | Yatay çubuk grafik |
| Bileşim (parça/bütün) | Yığılmış çubuk (pasta değil!) |
| Dağılım | Histogram veya Box plot |
| Korelasyon | Scatter plot |
| Coğrafi | Choropleth harita |
| Tek KPI | Scorecard / Big number |

> ⚠ **Pasta grafikten kaçın:** İnsan gözü açıları iyi karşılaştıramaz. Yatay çubuk kullan.

### Gereksiz Gürültü (Chartjunk) Temizle

```
KALDIR:
  ✗ 3D efektler
  ✗ Gölgeler
  ✗ Gereksiz ızgara çizgileri
  ✗ Onlarca renk
  ✗ Veriyi desteklemeyen görseller
  ✗ İkincil eksen (genellikle kafa karıştırır)

TUT:
  ✓ Veri etiketleri (gerekiyorsa)
  ✓ Referans çizgisi (hedef, ortalama)
  ✓ Açıklayıcı başlık (ne gösteriyor değil, ne anlatıyor)
  ✓ Renk tutarlılığı (kırmızı=kötü, yeşil=iyi)
```

### Power BI — DAX ile KPI Göstergesi

```dax
-- Aylık gelir hedef oranı
Hedef Gerçekleşme % = 
DIVIDE(
    [Aylık Toplam Gelir],
    [Aylık Gelir Hedefi],
    0
)

-- Traffic light conditional formatting
KPI Renk = 
SWITCH(
    TRUE(),
    [Hedef Gerçekleşme %] >= 1.0, "Yeşil",
    [Hedef Gerçekleşme %] >= 0.8, "Sarı",
    "Kırmızı"
)
```

### Etkili Dashboard Başlığı

```
Kötü başlık:  "Satış Verileri Q4 2024"
              (Ne gösteriyor? Ne anlamamı istiyorsun?)

İyi başlık:   "Q4 2024 Satışları Hedefin %12 Üzerinde — Mobil Kanalın Katkısı"
              (Insight zaten başlıkta → kullanıcı detaya bakıp doğrular)
```

### Dashboard Kalite Kontrol Listesi

```
☐ Her KPI'ın hedefi/benchmarki var mı?
☐ Zaman filtresi kullanıcı tarafından değiştirilebilir mi?
☐ "Neden?" sorusuna cevap verilebiliyor mu? (drill-down)
☐ Mobilde okunabilir mi?
☐ Yüklenme süresi <3 saniye mi?
☐ Veri tazeliği belirtilmiş mi? ("Son güncelleme: ...")
☐ Kaynak ve metodoloji notu var mı?
```

---

## 💡 Bağlantılar
- [[BI - Giriş ve Temel Kavramlar]]
- [[DS - Veri Görselleştirme İlkeleri]]
- [[Power BI - Veri Modelleme (DAX)]]
- [[BI - İleri DAX ve Performans Optimizasyonu]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- [Storytelling with Data - Cole Nussbaumer](https://www.storytellingwithdata.com/)
- [The Big Book of Dashboards](https://www.bigbookofdashboards.com/)
