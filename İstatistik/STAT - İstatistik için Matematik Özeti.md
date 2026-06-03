---
tarih: 2025-01-01
konu: Matematik Özeti, Integral, Türev, Logaritma, İstatistik için Analiz
etiket: [matematik, kalkülüs, integral, türev, logaritma, istatistik-matematiği]
kaynak:
zorluk: ⭐⭐
---

## 📌 Özet

İstatistik ve olasılık teorisi için gerekli temel matematik: türev, integral, logaritma, limit ve optimizasyon. Hızlı başvuru notu.

---

## 🧠 Detay

### Logaritma ve Üstel Fonksiyon

$$\ln(ab) = \ln a + \ln b$$
$$\ln(a/b) = \ln a - \ln b$$
$$\ln(a^r) = r\ln a$$
$$\ln(e^x) = x, \quad e^{\ln x} = x$$
$$\frac{d}{dx}\ln(x) = \frac{1}{x}, \quad \frac{d}{dx}e^x = e^x$$

---

### Temel Türevler

| f(x) | f'(x) |
|---|---|
| $x^n$ | $nx^{n-1}$ |
| $e^x$ | $e^x$ |
| $\ln x$ | $1/x$ |
| $\sin x$ | $\cos x$ |
| $\cos x$ | $-\sin x$ |
| $\sigma(x) = \frac{1}{1+e^{-x}}$ | $\sigma(x)(1-\sigma(x))$ |

**Zincir Kuralı**: $(f \circ g)'(x) = f'(g(x)) \cdot g'(x)$

**Çarpım Kuralı**: $(fg)' = f'g + fg'$

---

### Temel İntegraller

| f(x) | $\int f(x)\,dx$ |
|---|---|
| $x^n$ ($n \neq -1$) | $\frac{x^{n+1}}{n+1} + C$ |
| $1/x$ | $\ln|x| + C$ |
| $e^x$ | $e^x + C$ |
| $e^{ax}$ | $\frac{1}{a}e^{ax} + C$ |

**Parçalı İntegral**: $\int u\,dv = uv - \int v\,du$

**Gauss İntegrali** (çok önemli!):
$$\int_{-\infty}^{\infty} e^{-x^2}\,dx = \sqrt{\pi}$$

$$\int_{-\infty}^{\infty} e^{-x^2/2}\,dx = \sqrt{2\pi}$$

---

### Gamma Fonksiyonu

$$\Gamma(n) = \int_0^\infty x^{n-1}e^{-x}\,dx$$

- $\Gamma(n) = (n-1)!$ (pozitif tam sayı için)
- $\Gamma(1/2) = \sqrt{\pi}$
- $\Gamma(n+1) = n\Gamma(n)$
- $\Gamma(1) = 1$

**Beta Fonksiyonu:**
$$B(a,b) = \int_0^1 x^{a-1}(1-x)^{b-1}\,dx = \frac{\Gamma(a)\Gamma(b)}{\Gamma(a+b)}$$

---

### Optimizasyon

**Birinci Sıra Koşul (FOC):** $f'(x) = 0$

**İkinci Sıra Koşul:**
- $f''(x) > 0$: Minimum
- $f''(x) < 0$: Maksimum

**Lagrange Çarpanları** (kısıtlı optimizasyon):
$$\mathcal{L}(x, \lambda) = f(x) - \lambda g(x)$$

$\nabla f = \lambda \nabla g$ ve $g(x) = 0$

---

### Çok Değişkenli Türev

**Kısmi Türev**: $\frac{\partial f}{\partial x_i}$

**Gradyan**:
$$\nabla f = \left(\frac{\partial f}{\partial x_1}, \frac{\partial f}{\partial x_2}, \ldots, \frac{\partial f}{\partial x_n}\right)$$

**Hessian Matrisi**:
$$H_{ij} = \frac{\partial^2 f}{\partial x_i \partial x_j}$$

- $H$ pozitif tanımlı → Minimum
- $H$ negatif tanımlı → Maksimum

**Gradient Descent:**
$$\boldsymbol{\theta}_{t+1} = \boldsymbol{\theta}_t - \eta \nabla f(\boldsymbol{\theta}_t)$$

---

### Taylor Serisi

$$f(x) = f(a) + f'(a)(x-a) + \frac{f''(a)}{2!}(x-a)^2 + \cdots$$

$$e^x = 1 + x + \frac{x^2}{2!} + \frac{x^3}{3!} + \cdots$$

$$\ln(1+x) = x - \frac{x^2}{2} + \frac{x^3}{3} - \cdots \quad |x| \leq 1$$

**Kullanım:** MGF türevleri, Delta yöntemi, Newton-Raphson.

---

### Limit ve Süreklilik

$$\lim_{n\to\infty}\left(1+\frac{x}{n}\right)^n = e^x$$

$$\lim_{x\to 0}\frac{\sin x}{x} = 1$$

$$\lim_{n\to\infty} n^{1/n} = 1$$

**L'Hôpital Kuralı** (0/0 veya ∞/∞ belirsizliği):
$$\lim_{x\to a}\frac{f(x)}{g(x)} = \lim_{x\to a}\frac{f'(x)}{g'(x)}$$

---

### Kombinatorik

$$n! = n \times (n-1) \times \cdots \times 2 \times 1, \quad 0! = 1$$

$$\binom{n}{k} = \frac{n!}{k!(n-k)!}$$

**Binom Teoremi:**
$$(x+y)^n = \sum_{k=0}^n \binom{n}{k} x^k y^{n-k}$$

**Stirling Yaklaşımı:**
$$n! \approx \sqrt{2\pi n}\left(\frac{n}{e}\right)^n$$

---

### İstatistikte Sık Kullanılan Kimlikler

$$\sum_{i=1}^n (x_i - \bar{x}) = 0$$

$$\sum_{i=1}^n (x_i - \bar{x})^2 = \sum_{i=1}^n x_i^2 - n\bar{x}^2$$

$$Var(X) = E[X^2] - (E[X])^2$$

$$E[(X-\mu)^2] = Var(X)$$

---

## 💡 Bağlantılar
- [[STAT - Olasılık Kuramı Matematiği]]
- [[STAT - Olasılık Dağılımları (Sürekli)]]
- [[STAT - Doğrusal Cebir ve İstatistik]]
- [[STAT - Tahmin Teorisi]]

## ❓ Sorular / Anlamadıklarım


## 🔗 Kaynaklar
- Mathematics for Machine Learning (Deisenroth et al.) - Free PDF
- Khan Academy Kalkülüs
