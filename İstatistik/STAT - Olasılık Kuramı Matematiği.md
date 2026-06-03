---
tarih: 2025-01-01
konu: Olasılık Kuramı Matematiği, Moment Üreteci, Ortak Dağılımlar
etiket: [istatistik, matematik, moment, MGF, ortak-dağılım, beklenti]
kaynak:
zorluk: ⭐⭐⭐⭐
---

## 📌 Özet

İleri istatistiğin matematiksel temeli: moment üreten fonksiyonlar, beklenti operatörü özellikleri, ortak dağılımlar ve koşullu beklenti.

---

## 🧠 Detay

### Beklenti Operatörünün Özellikleri

$$E[aX + b] = aE[X] + b$$
$$E[X + Y] = E[X] + E[Y] \quad \text{(her zaman)}$$
$$E[XY] = E[X]E[Y] \quad \text{(sadece bağımsızsa)}$$
$$E[g(X)] = \sum_x g(x)P(X=x) \quad \text{(LOTUS)}$$

**LOTUS** (Law of the Unconscious Statistician):
$$E[g(X)] = \int_{-\infty}^{\infty} g(x) f_X(x)\, dx$$

### Varyans Özellikleri

$$Var(aX + b) = a^2 Var(X)$$
$$Var(X + Y) = Var(X) + Var(Y) + 2Cov(X,Y)$$
$$Var(X - Y) = Var(X) + Var(Y) - 2Cov(X,Y)$$

**Bağımsızsa**: $Var(X+Y) = Var(X) + Var(Y)$

### Momentler

**k. Ham Moment**:
$$\mu_k' = E[X^k]$$

**k. Merkezi Moment**:
$$\mu_k = E[(X-\mu)^k]$$

- $\mu_1' = \mu$ (ortalama)
- $\mu_2 = \sigma^2$ (varyans)
- $\mu_3 / \sigma^3$ = çarpıklık
- $\mu_4 / \sigma^4 - 3$ = aşırı basıklık

### Moment Üreten Fonksiyon (MGF)

$$M_X(t) = E[e^{tX}] = \sum_k \frac{t^k}{k!} E[X^k]$$

**Özellikler:**
- $M_X^{(k)}(0) = E[X^k]$ (k. türev, t=0'da = k. moment)
- $X \perp Y$ ise $M_{X+Y}(t) = M_X(t) \cdot M_Y(t)$
- MGF dağılımı **benzersiz şekilde** tanımlar

**Önemli MGF'ler:**

| Dağılım | MGF |
|---|---|
| Normal $N(\mu,\sigma^2)$ | $e^{\mu t + \sigma^2 t^2/2}$ |
| Poisson $(\lambda)$ | $e^{\lambda(e^t - 1)}$ |
| Binom $(n,p)$ | $(1-p+pe^t)^n$ |
| Üstel $(\lambda)$ | $\lambda/(\lambda-t)$, $t<\lambda$ |
| Gamma $(\alpha,\beta)$ | $(1-\beta t)^{-\alpha}$ |

---

### Ortak Dağılımlar

**Ortak PMF/PDF**: $f_{X,Y}(x,y)$

**Marjinal Dağılım**:
$$f_X(x) = \sum_y f_{X,Y}(x,y) \quad \text{veya} \quad \int_{-\infty}^{\infty} f_{X,Y}(x,y)\,dy$$

**Koşullu Dağılım**:
$$f_{X|Y}(x|y) = \frac{f_{X,Y}(x,y)}{f_Y(y)}$$

**Bağımsızlık**:
$$X \perp Y \iff f_{X,Y}(x,y) = f_X(x) \cdot f_Y(y)$$

---

### Kovaryans ve Korelasyonun Matematiği

$$Cov(X,Y) = E[XY] - E[X]E[Y]$$
$$Cov(aX+b, cY+d) = ac \cdot Cov(X,Y)$$
$$Cov(X+Y, Z) = Cov(X,Z) + Cov(Y,Z)$$

**Cauchy-Schwarz Eşitsizliği**:
$$|Cov(X,Y)|^2 \leq Var(X) \cdot Var(Y)$$
$$\therefore \quad -1 \leq \rho(X,Y) \leq 1$$

---

### Koşullu Beklenti

$$E[Y|X=x] = \int y \cdot f_{Y|X}(y|x)\, dy$$

**İçiçe Beklenti (Law of Total Expectation / Adam's Law)**:
$$E[Y] = E[E[Y|X]]$$

**Toplam Varyans Yasası (Eve's Law)**:
$$Var(Y) = E[Var(Y|X)] + Var(E[Y|X])$$

---

### Dönüşüm Yöntemi

$Y = g(X)$ için PDF bulma:

**Tek değişkenli**:
$$f_Y(y) = f_X(g^{-1}(y)) \cdot \left|\frac{dg^{-1}}{dy}\right|$$

**Örnek**: $X \sim Exp(\lambda)$, $Y = X^2$ için $f_Y(y)$:
$$f_Y(y) = f_X(\sqrt{y}) \cdot \frac{1}{2\sqrt{y}} = \frac{\lambda e^{-\lambda\sqrt{y}}}{2\sqrt{y}}, \quad y>0$$

**Çok değişkenli**: Jacobian determinantı kullanılır.

---

### Karakteristik Fonksiyon

MGF yakınsamayabilir, ama karakteristik fonksiyon her zaman var:

$$\varphi_X(t) = E[e^{itX}] = E[\cos(tX)] + iE[\sin(tX)]$$

**Tersine çevrilebilir** → dağılımı benzersiz belirler.
**MLT ispatında** kullanılır.

---

### Eşitsizlikler

**Markov Eşitsizliği** ($X \geq 0$):
$$P(X \geq a) \leq \frac{E[X]}{a}$$

**Chebyshev Eşitsizliği**:
$$P(|X - \mu| \geq k\sigma) \leq \frac{1}{k^2}$$

**Jensen Eşitsizliği** (konveks $g$ için):
$$g(E[X]) \leq E[g(X)]$$

(konveks: $g'' \geq 0$, ör. $x^2$, $e^x$, $-\ln x$)

---

## 💡 Bağlantılar
- [[STAT - Olasılık Temelleri]]
- [[STAT - Olasılık Dağılımları (Kesikli)]]
- [[STAT - Olasılık Dağılımları (Sürekli)]]
- [[STAT - Merkezi Limit Teoremi ve Örnekleme]]

## ❓ Sorular / Anlamadıklarım


## 🔗 Kaynaklar
- Introduction to Probability (Blitzstein & Hwang) - Ch. 4-7
- All of Statistics (Wasserman) - Ch. 2-3
