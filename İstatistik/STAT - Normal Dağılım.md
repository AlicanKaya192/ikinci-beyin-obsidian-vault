---
tarih: 2025-01-01
konu: Normal Dağılım, Standartlaştırma, Z-Skoru, Normallik Testleri
etiket: [istatistik, normal-dağılım, z-skoru, standartlaştırma, normallik]
kaynak:
zorluk: ⭐⭐
---

## 📌 Özet

Normal dağılım, istatistiğin en temel dağılımıdır. Simetrik, çan şeklinde yapısıyla merkezi limit teoremine dayanan çoğu istatistiksel yöntemin temelidir.

---

## 🧠 Detay

### Normal Dağılımın Özellikleri

$$X \sim N(\mu, \sigma^2)$$

$$f(x) = \frac{1}{\sigma\sqrt{2\pi}} \exp\!\left(-\frac{(x-\mu)^2}{2\sigma^2}\right)$$

**Temel özellikler:**
- Ortalama = Medyan = Mod = $\mu$
- Simetri ekseni $x = \mu$
- İki bükülme noktası: $x = \mu \pm \sigma$
- Çarpıklık = 0, Basıklık = 0 (normal için referans)
- Asimptotik: kuyruklar x-eksenine hiç değmez

### Ampirik Kural (68-95-99.7)

```
       |←   68%   →|
       |←    95%     →|
       |←     99.7%     →|
   ────────────────────────
   μ-3σ μ-2σ μ-σ  μ  μ+σ μ+2σ μ+3σ
```

| Aralık | Olasılık |
|---|---|
| $(\mu \pm \sigma)$ | 68.27% |
| $(\mu \pm 2\sigma)$ | 95.45% |
| $(\mu \pm 3\sigma)$ | 99.73% |

### Standart Normal Dönüşümü (Z-Skoru)

$$Z = \frac{X - \mu}{\sigma} \sim N(0, 1)$$

**Z-skorunun anlamı**: Değerin ortalamadan kaç standart sapma uzakta olduğu.

| Z | Anlamı |
|---|---|
| Z = 0 | Ortalamada |
| Z = 1 | Ortalamanın 1σ üstünde → Üst %84.13 |
| Z = -1 | Ortalamanın 1σ altında → Alt %15.87 |
| Z = 1.96 | %95 güven aralığı sınırı |
| Z = 2.576 | %99 güven aralığı sınırı |

### Z Tablosu Okuma

$P(Z < z)$ değerleri tablo ile bulunur.

**Örnek**: $X \sim N(100, 15^2)$, $P(X < 120) = ?$

$$Z = \frac{120 - 100}{15} = 1.33$$

$$P(Z < 1.33) = 0.9082 \quad \text{(tablodan)}$$

**Yararlı değerler:**

| z | P(Z < z) |
|---|---|
| 1.28 | 0.90 |
| 1.645 | 0.95 |
| 1.96 | 0.975 |
| 2.33 | 0.99 |
| 2.576 | 0.995 |

### Normal Dağılımın Doğrusal Dönüşümü

$$Y = aX + b \Rightarrow Y \sim N(a\mu + b,\ a^2\sigma^2)$$

$$X_1 + X_2 \sim N(\mu_1+\mu_2,\ \sigma_1^2+\sigma_2^2) \quad \text{(bağımsız ise)}$$

### Normallik Testleri

#### Görsel Yöntemler
- **Histogram**: Çan şekli mi?
- **Q-Q Plot**: Noktalar düz çizgi üzerinde mi?
- **Kutu grafiği**: Simetrik mi? Aykırı değer var mı?

#### İstatistiksel Testler

| Test | Küçük Örneklem | Büyük Örneklem |
|---|---|---|
| Shapiro-Wilk | ✅ En güçlü | ❌ |
| Kolmogorov-Smirnov | ✅ | ✅ |
| Anderson-Darling | ✅ | ✅ |
| Lilliefors | ✅ | ✅ |
| Jarque-Bera | ❌ | ✅ |

$H_0$: Veri normal dağılımlıdır
$p < 0.05$ ise $H_0$ reddedilir → Normal değil

### Normal Dağılım Gerektiren Testler

- t-testi
- ANOVA
- Pearson korelasyonu
- Doğrusal regresyon (artıklar için)

**Normallik sağlanamıyorsa**:
- Veri dönüşümü: $\log(X)$, $\sqrt{X}$, $1/X$
- Parametrik olmayan testler kullan

### Lognormal Dağılım

$Y = \ln(X)$ normal dağılımlıysa $X$ **lognormal** dağılımlıdır.

$$X \sim LN(\mu, \sigma^2)$$

Gelir, fiyatlar, bekleme süreleri genelde lognormal dağılır.

---

## 💡 Bağlantılar
- [[STAT - Olasılık Dağılımları (Sürekli)]]
- [[STAT - Merkezi Limit Teoremi]]
- [[STAT - Güven Aralıkları]]
- [[STAT - Hipotez Testleri - t-testi]]

## ❓ Sorular / Anlamadıklarım


## 🔗 Kaynaklar
- OpenStax Statistics - Ch. 6
- scipy.stats.norm Documentation
