---
tarih: 2026-05-28
konu: Data Science
etiket: ["data-science", "görselleştirme", "ilkeler", "grafik-seçimi"]
kaynak: 
zorluk: başlangıç
---

## 📌 Özet
Etkili veri görselleştirme, doğru grafik türünü seçmek ve sade bir tasarımla mesajı net iletmektir. Yanlış grafik seçimi veriyi yanlış yorumlatabilir.

## 🧠 Detay

### Grafik Türü Seçim Rehberi

| Amaç | Grafik Türü |
|------|-------------|
| Dağılım göster | Histogram, KDE, Box Plot |
| Karşılaştır | Bar Chart, Box Plot |
| İlişki göster | Scatter Plot, Heatmap |
| Zaman trendi | Line Chart |
| Oran göster | Pie Chart (az kategori) |
| Çok değişken | Pair Plot, Heatmap |

### Renk Kullanımı
```python
import seaborn as sns
import matplotlib.pyplot as plt

# Sıralı veri → tek renkli palet
sns.color_palette("Blues", n_colors=5)

# Iraksak veri (pozitif/negatif) → çift uçlu palet
sns.color_palette("coolwarm")

# Kategorik → farklı renkler
sns.color_palette("tab10")

# Renk körlüğü dostu
sns.color_palette("colorblind")
```

### Grafik Başlığı ve Etiketler
```python
plt.figure(figsize=(10, 6))
plt.plot(x, y)

plt.title("Aylık Satış Trendi (2024)", fontsize=16, pad=20)
plt.xlabel("Ay", fontsize=13)
plt.ylabel("Satış (TL)", fontsize=13)
plt.tick_params(labelsize=11)

# Açıklayıcı not ekle
plt.text(0.05, 0.95, "* Veriler tahmini değildir",
    transform=plt.gca().transAxes, fontsize=9)
```

### Kaçınılacak Hatalar
```python
# ❌ 3D pasta grafik → yanlış algı
# ❌ Y ekseni sıfırdan başlamıyor → yanıltıcı
# ❌ Çok fazla kategori → okunaksız
# ❌ Birden fazla mesaj → kafa karışıklığı

# ✅ Basit ve odaklı grafik
# ✅ Y ekseni sıfırdan başlar (bar chart)
# ✅ Renk farkı anlamlı
# ✅ Kaynak ve tarih belirtilmiş
```

### Plotly ile İnteraktif Grafik
```python
import plotly.express as px

fig = px.scatter(df, x="yas", y="gelir",
    color="sehir", hover_name="isim",
    title="Yaş - Gelir İlişkisi")
fig.show()
```

## 💡 Bağlantılar
- [[DS - Matplotlib Temel Grafikler]]
- [[DS - Seaborn İstatistiksel Grafikler]]
- [[DS - EDA - Keşifsel Veri Analizi]]

## ❓ Sorular / Anlamadıklarım
- Hangi durumda interaktif grafik statik grafikten daha iyi?
- Dashboard için en iyi Python kütüphanesi hangisi?

## 🔗 Kaynaklar
- https://www.data-to-viz.com
- https://plotly.com/python/
