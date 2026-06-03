---
tarih: 2025-01-01
konu: Kesikli Olasılık Dağılımları, Binom, Poisson, Geometrik
etiket: [istatistik, olasılık, dağılım, binom, poisson, kesikli]
kaynak:
zorluk: ⭐⭐⭐
---

## 📌 Özet

Kesikli rastlantı değişkenleri sayılabilir değerler alır. Binom, Poisson ve Geometrik dağılımlar en yaygın kesikli olasılık dağılımlarıdır.

---

## 🧠 Detay

### Rastlantı Değişkeni (Random Variable)

- **Kesikli (Discrete)**: Sayılabilir değerler {0, 1, 2, ...}
- **Sürekli (Continuous)**: Aralıkta sonsuz değer

**Olasılık Kütle Fonksiyonu (PMF)**:
$$P(X = x) = f(x), \quad \sum_x f(x) = 1$$

**Beklenen Değer (Expected Value)**:
$$E[X] = \mu = \sum_x x \cdot P(X=x)$$

**Varyans**:
$$Var(X) = \sigma^2 = E[X^2] - (E[X])^2 = \sum_x (x-\mu)^2 \cdot P(X=x)$$

---

### Bernoulli Dağılımı

Tek bir deney, iki sonuç: başarı (1) veya başarısızlık (0).

$$P(X=1) = p, \quad P(X=0) = 1-p = q$$

$$E[X] = p, \quad Var(X) = pq$$

---

### Binom Dağılımı $B(n, p)$

$n$ bağımsız Bernoulli denemesinde $k$ başarı sayısı.

**Koşullar**: n sabit, p sabit, bağımsız denemeler, 2 sonuç

$$P(X=k) = \binom{n}{k} p^k (1-p)^{n-k}, \quad k=0,1,\ldots,n$$

$$E[X] = np, \quad Var(X) = np(1-p)$$

**Örnek**: 10 yazı tura atışında kaç kez yazı gelir?
$$X \sim B(10, 0.5) \quad E[X]=5, \quad \sigma=\sqrt{2.5}\approx1.58$$

---

### Poisson Dağılımı $Pois(\lambda)$

Belirli bir zaman/alan biriminde nadir olayların sayısı.

**Koşullar**: Olaylar bağımsız, sabit oran $\lambda$, aynı anda iki olay olmaz

$$P(X=k) = \frac{e^{-\lambda} \lambda^k}{k!}, \quad k=0,1,2,\ldots$$

$$E[X] = \lambda, \quad Var(X) = \lambda$$

> Ortalama = Varyans! Bu Poisson'u tanımlayan özelliktir.

**Poisson Yaklaşımı**: $n$ büyük, $p$ küçük, $\lambda = np$ ise $B(n,p) \approx Pois(\lambda)$

**Örnek**: Bir web sitesine dakikada ortalama 3 istek gelirse, 5 istek gelme olasılığı:
$$P(X=5) = \frac{e^{-3} \cdot 3^5}{5!} = \frac{0.0498 \times 243}{120} \approx 0.101$$

---

### Geometrik Dağılım $Geom(p)$

İlk başarıya kadar kaç deneme gerekir?

$$P(X=k) = (1-p)^{k-1} p, \quad k=1,2,3,\ldots$$

$$E[X] = \frac{1}{p}, \quad Var(X) = \frac{1-p}{p^2}$$

**Hafızasızlık özelliği**: $P(X > m+n | X > m) = P(X > n)$

---

### Negatif Binom Dağılımı $NB(r, p)$

$r$'inci başarıya kadar gereken deneme sayısı.

$$P(X=k) = \binom{k-1}{r-1} p^r (1-p)^{k-r}$$

$$E[X] = \frac{r}{p}, \quad Var(X) = \frac{r(1-p)}{p^2}$$

---

### Hipergeometrik Dağılım

$N$ elemanlı anakütleden ($K$ başarılı) $n$ çekimde başarı sayısı. **Seçimler bağımsız değil** (geri koymadan).

$$P(X=k) = \frac{\binom{K}{k}\binom{N-K}{n-k}}{\binom{N}{n}}$$

$$E[X] = n\frac{K}{N}, \quad Var(X) = n\frac{K}{N}\frac{N-K}{N}\frac{N-n}{N-1}$$

**Fark**: Binom → geri koyarak; Hipergeometrik → geri koymadan

---

### Özet Tablosu

| Dağılım | Parametre | E[X] | Var(X) |
|---|---|---|---|
| Bernoulli | p | p | p(1-p) |
| Binom | n, p | np | np(1-p) |
| Poisson | λ | λ | λ |
| Geometrik | p | 1/p | (1-p)/p² |
| Negatif Binom | r, p | r/p | r(1-p)/p² |
| Hipergeometrik | N, K, n | nK/N | — |

---

## 💡 Bağlantılar
- [[STAT - Olasılık Temelleri]]
- [[STAT - Olasılık Dağılımları (Sürekli)]]
- [[STAT - Merkezi Limit Teoremi]]

## ❓ Sorular / Anlamadıklarım


## 🔗 Kaynaklar
- Introduction to Probability (Blitzstein & Hwang) - Ch. 3-4
- scipy.stats dokümantasyonu
