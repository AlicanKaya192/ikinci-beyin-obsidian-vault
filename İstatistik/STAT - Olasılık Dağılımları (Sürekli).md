---
tarih: 2025-01-01
konu: Sürekli Olasılık Dağılımları, Normal, Uniform, Üstel, t, Chi-kare, F
etiket: [istatistik, olasılık, normal-dağılım, t-dağılımı, chi-kare, F-dağılımı]
kaynak:
zorluk: ⭐⭐⭐
---

## 📌 Özet

Sürekli rastlantı değişkenleri aralıkta sonsuz değer alabilir. Olasılık yoğunluk fonksiyonu (PDF) ile tanımlanır. Normal dağılım en temel ve yaygın sürekli dağılımdır.

---

## 🧠 Detay

### Temel Kavramlar

**Olasılık Yoğunluk Fonksiyonu (PDF)**:
$$P(a \leq X \leq b) = \int_a^b f(x)\, dx, \quad \int_{-\infty}^{\infty} f(x)\, dx = 1$$

**Kümülatif Dağılım Fonksiyonu (CDF)**:
$$F(x) = P(X \leq x) = \int_{-\infty}^{x} f(t)\, dt$$

**Beklenti ve Varyans**:
$$E[X] = \int_{-\infty}^{\infty} x \cdot f(x)\, dx$$
$$Var(X) = \int_{-\infty}^{\infty} (x-\mu)^2 f(x)\, dx$$

---

### Düzgün (Uniform) Dağılım $U(a, b)$

Her değer eşit olasılıklı.

$$f(x) = \frac{1}{b-a}, \quad a \leq x \leq b$$

$$E[X] = \frac{a+b}{2}, \quad Var(X) = \frac{(b-a)^2}{12}$$

---

### Normal (Gauss) Dağılım $N(\mu, \sigma^2)$ ⭐

İstatistiğin en önemli dağılımı.

$$f(x) = \frac{1}{\sigma\sqrt{2\pi}} e^{-\frac{(x-\mu)^2}{2\sigma^2}}, \quad -\infty < x < \infty$$

$$E[X] = \mu, \quad Var(X) = \sigma^2$$

**Standart Normal $Z \sim N(0, 1)$**:
$$Z = \frac{X - \mu}{\sigma}$$

**Ampirik Kural (68-95-99.7)**:
- $P(\mu - \sigma < X < \mu + \sigma) \approx 0.6827$
- $P(\mu - 2\sigma < X < \mu + 2\sigma) \approx 0.9545$
- $P(\mu - 3\sigma < X < \mu + 3\sigma) \approx 0.9973$

**Normal Dağılım Özellikleri**:
- Simetrik, çan şekli
- Ortalama = Medyan = Mod
- Çarpıklık = 0, Basıklık = 0

---

### Üstel (Exponential) Dağılım $Exp(\lambda)$

Olaylar arasındaki bekleme süreleri.

$$f(x) = \lambda e^{-\lambda x}, \quad x \geq 0$$

$$E[X] = \frac{1}{\lambda}, \quad Var(X) = \frac{1}{\lambda^2}$$

**Hafızasızlık**: $P(X > s+t | X > s) = P(X > t)$

**Poisson ile ilişki**: Olaylar $Pois(\lambda)$ ise, bekleme süresi $Exp(\lambda)$.

---

### Gamma Dağılımı $\Gamma(\alpha, \beta)$

Üstel dağılımın genelleştirmesi. $k$ olay için bekleme süresi.

$$f(x) = \frac{x^{\alpha-1}e^{-x/\beta}}{\Gamma(\alpha)\beta^\alpha}$$

$$E[X] = \alpha\beta, \quad Var(X) = \alpha\beta^2$$

Özel durumlar:
- $\alpha=1$: Üstel dağılım
- $\alpha=n/2, \beta=2$: Ki-kare dağılımı

---

### Beta Dağılımı $Beta(\alpha, \beta)$

$[0,1]$ aralığında tanımlı. Olasılıkları modellemek için.

$$f(x) = \frac{x^{\alpha-1}(1-x)^{\beta-1}}{B(\alpha,\beta)}, \quad 0 < x < 1$$

$$E[X] = \frac{\alpha}{\alpha+\beta}$$

Bayesci istatistikte Binom için **konjuge önsel** dağılım.

---

### Student t-Dağılımı $t_\nu$

Örneklem boyutu küçükken, $\sigma$ bilinmiyorken ortalama tahmini için.

$$f(t) = \frac{\Gamma\left(\frac{\nu+1}{2}\right)}{\sqrt{\nu\pi}\,\Gamma\left(\frac{\nu}{2}\right)} \left(1+\frac{t^2}{\nu}\right)^{-\frac{\nu+1}{2}}$$

- $\nu$ = serbestlik derecesi = $n - 1$
- Normal dağılımdan **daha kalın kuyruklu**
- $\nu \to \infty$ iken $t \to Z \sim N(0,1)$

**Ne zaman kullanılır?**
- Küçük örneklem ($n < 30$)
- $\sigma$ bilinmiyor

---

### Ki-Kare Dağılımı $\chi^2_\nu$

$\nu$ adet bağımsız standart normal değişkenin karelerinin toplamı.

$$\chi^2_\nu = Z_1^2 + Z_2^2 + \cdots + Z_\nu^2$$

$$E[X] = \nu, \quad Var(X) = 2\nu$$

**Kullanım alanları**:
- İyilik-uyum testi
- Bağımsızlık testi
- Varyans tahmini

---

### F-Dağılımı $F_{d_1, d_2}$

İki ki-kare dağılımının oranı.

$$F = \frac{\chi^2_{d_1}/d_1}{\chi^2_{d_2}/d_2}$$

**Kullanım**: ANOVA, iki varyansın karşılaştırması, regresyon anlamlılık testi

---

### Dağılımlar Arası İlişki

```
Normal N(0,1)
    ↓ kare
Chi-kare χ²(1)
    ↓ n adet topla
Chi-kare χ²(n)
    ↓ oran
F Dağılımı
    ↓ özel durum
t Dağılımı (F = t²)
```

---

## 💡 Bağlantılar
- [[STAT - Normal Dağılım]]
- [[STAT - Merkezi Limit Teoremi]]
- [[STAT - Güven Aralıkları]]
- [[STAT - Hipotez Testleri - Temel Kavramlar]]

## ❓ Sorular / Anlamadıklarım


## 🔗 Kaynaklar
- Introduction to Probability (Blitzstein & Hwang) - Ch. 5-6
- scipy.stats Documentation
