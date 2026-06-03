---
tarih: 2025-01-01
konu: Tahmin Teorisi, Yansızlık, Etkinlik, Tutarlılık, MLE, MOM
etiket: [istatistik, tahmin, MLE, yansızlık, tutarlılık, Cramér-Rao]
kaynak:
zorluk: ⭐⭐⭐⭐
---

## 📌 Özet

Tahmin teorisi, bilinmeyen parametreleri veriden nasıl tahmin etmemiz gerektiğini inceler. İyi bir tahmincinin özellikleri: yansızlık, tutarlılık, etkinlik ve yeterlilik.

---

## 🧠 Detay

### Tahminci Özellikleri

#### 1. Yansızlık (Unbiasedness)

$$E[\hat{\theta}] = \theta$$

**Önyargı (Bias):**
$$Bias(\hat{\theta}) = E[\hat{\theta}] - \theta$$

**Örnekler:**
- $\bar{X}$: $\mu$ için yansız tahminci ✅
- $s^2 = \frac{\sum(x_i-\bar{x})^2}{n-1}$: $\sigma^2$ için yansız ✅
- $\frac{\sum(x_i-\bar{x})^2}{n}$: $\sigma^2$ için yanlı ❌ (küçük örneklemde küçük tahmin)

#### 2. Tutarlılık (Consistency)

$$\hat{\theta}_n \xrightarrow{p} \theta \quad (n \to \infty)$$

Yeterli koşul: Yansız + $Var(\hat{\theta}_n) \to 0$

#### 3. Etkinlik (Efficiency)

Belirli bir örneklem büyüklüğü için en küçük varyanslı yansız tahminci → **UMVUE** (Uniformly Minimum Variance Unbiased Estimator).

**Ortalama Karesel Hata (MSE):**
$$MSE(\hat{\theta}) = Var(\hat{\theta}) + [Bias(\hat{\theta})]^2$$

Önyargı-Varyans Dengesi (Bias-Variance Tradeoff)!

#### 4. Yeterlilik (Sufficiency)

$T(X)$ yeterli tahminci ↔ $T(X)$ verilen $X$'in dağılımı $\theta$'ya bağlı değil.

**Fisher-Neyman Çarpanlama Teoremi**:
$$f(\mathbf{x}|\theta) = g(T(\mathbf{x})|\theta) \cdot h(\mathbf{x})$$

---

### Cramér-Rao Alt Sınırı

Yansız tahmincinin varyansı ne kadar küçük olabilir?

**Fisher Bilgisi:**
$$\mathcal{I}(\theta) = E\!\left[\left(\frac{\partial \ln f(X|\theta)}{\partial \theta}\right)^2\right] = -E\!\left[\frac{\partial^2 \ln f(X|\theta)}{\partial \theta^2}\right]$$

**Cramér-Rao Eşitsizliği:**
$$Var(\hat{\theta}) \geq \frac{1}{n \cdot \mathcal{I}(\theta)}$$

Eşitlik sağlanıyorsa → **Etkin tahminci** (UMVUE).

---

### Momentler Yöntemi (MOM)

**Fikir:** Teorik momentleri örneklem momentlerine eşitle, çöz.

**k. örneklem momenti:** $m_k = \frac{1}{n}\sum x_i^k$

**k. teorik moment:** $\mu_k' = E[X^k]$ ($\theta$'nun fonksiyonu)

$\mu_k' = m_k$ denklemlerini çöz → MOM tahminleri.

**Örnek — Gamma $(\alpha, \beta)$:**
$$E[X] = \alpha\beta = \bar{x}$$
$$E[X^2] = \alpha(\alpha+1)\beta^2 = m_2$$

Bu iki denklemden $\hat{\alpha}$ ve $\hat{\beta}$ bulunur.

---

### Maksimum Olabilirlik Tahmini (MLE) ⭐

**Fikir:** Gözlenen veriyi en olası kılacak parametre değerini bul.

**Olabilirlik Fonksiyonu:**
$$L(\theta|\mathbf{x}) = \prod_{i=1}^n f(x_i|\theta)$$

**Log-Olabilirlik (daha kolay):**
$$\ell(\theta) = \ln L(\theta) = \sum_{i=1}^n \ln f(x_i|\theta)$$

**MLE:** $\hat{\theta}_{MLE} = \arg\max_\theta \ell(\theta)$

**Birinci Sıra Koşul (Skor Denklemi):**
$$\frac{\partial \ell(\theta)}{\partial \theta} = 0$$

#### MLE Örnekleri

**Normal $N(\mu, \sigma^2)$:**
$$\hat{\mu}_{MLE} = \bar{x}, \quad \hat{\sigma}^2_{MLE} = \frac{\sum(x_i-\bar{x})^2}{n}$$

> $\hat{\sigma}^2_{MLE}$ **yanlı**: $n$ ile böler, $n-1$ değil. Yine de MLE.

**Binom $B(n,p)$:**
$$\hat{p}_{MLE} = \frac{k}{n}$$

**Üstel $Exp(\lambda)$:**
$$\hat{\lambda}_{MLE} = \frac{1}{\bar{x}}$$

**Poisson $Pois(\lambda)$:**
$$\hat{\lambda}_{MLE} = \bar{x}$$

#### MLE'nin Özellikleri

1. **Tutarlılık**: $\hat{\theta}_{MLE} \xrightarrow{p} \theta$
2. **Asimptotik Normallik**: $\sqrt{n}(\hat{\theta}_{MLE} - \theta) \xrightarrow{d} N(0, 1/\mathcal{I}(\theta))$
3. **Asimptotik Etkinlik**: Cramér-Rao sınırına ulaşır (büyük $n$'de)
4. **Değişmezlik**: $g(\theta)$'yı tahmin etmek için $g(\hat{\theta}_{MLE})$ kullan

---

### Delta Metodu

$g(\hat{\theta})$'nun asimptotik dağılımı:

$$\sqrt{n}(g(\hat{\theta}) - g(\theta)) \xrightarrow{d} N\!\left(0, [g'(\theta)]^2 \cdot \frac{1}{\mathcal{I}(\theta)}\right)$$

**Kullanım:** MLE'nin dönüşümü için standart hata.

---

### Bayes Tahmini vs MLE

| Özellik | MLE | MAP | Bayes Posterior Ortalaması |
|---|---|---|---|
| Prior | Kullanmaz | Kullanır (mod) | Kullanır (ortalama) |
| Belirsizlik | Nokta tahmin | Nokta tahmin | Tam dağılım |
| Küçük n | Zayıf | İyi | İyi |
| Büyük n | MLE → MAP → Bayes | Yakınsar | Yakınsar |

---

## 💡 Bağlantılar
- [[STAT - Olasılık Temelleri]]
- [[STAT - Bayes İstatistiği]]
- [[STAT - Güven Aralıkları]]
- [[STAT - Merkezi Limit Teoremi ve Örnekleme]]

## ❓ Sorular / Anlamadıklarım


## 🔗 Kaynaklar
- All of Statistics (Wasserman) - Ch. 9-10
- Statistical Inference (Casella & Berger) - Ch. 7
