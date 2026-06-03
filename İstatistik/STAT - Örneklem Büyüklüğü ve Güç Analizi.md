---
tarih: 2025-01-01
konu: Güç Analizi, Örneklem Büyüklüğü, Etki Büyüklüğü
etiket: [istatistik, güç-analizi, örneklem-büyüklüğü, etki-büyüklüğü, power]
kaynak:
zorluk: ⭐⭐⭐
---

## 📌 Özet

Güç analizi, istatistiksel testin gerçek bir etkiyi ne olasılıkla tespit edebileceğini hesaplar. Araştırma tasarımında yeterli örneklem büyüklüğünü belirlemek için kullanılır.

---

## 🧠 Detay

### Dört Temel Parametre

| Parametre | Sembol | Tipik Değer |
|---|---|---|
| Anlamlılık düzeyi | $\alpha$ | 0.05 |
| Test gücü | $1 - \beta$ | 0.80 |
| Etki büyüklüğü | $d$, $f$, $w$ | Değişir |
| Örneklem büyüklüğü | $n$ | Hesaplanacak |

Bu dördünden herhangi üçü bilinirse dördüncüsü hesaplanır.

### Etki Büyüklüğü Ölçüleri

#### Cohen's d (t-testleri için)

$$d = \frac{\mu_1 - \mu_2}{\sigma}$$

| d | Etki |
|---|---|
| 0.2 | Küçük |
| 0.5 | Orta |
| 0.8 | Büyük |

#### Cohen's f (ANOVA için)

$$f = \frac{\sigma_{gruplar\ arası}}{\sigma_{gruplar\ içi}} = \sqrt{\frac{\eta^2}{1-\eta^2}}$$

| f | Etki |
|---|---|
| 0.10 | Küçük |
| 0.25 | Orta |
| 0.40 | Büyük |

#### Cohen's w (Ki-kare için)

$$w = \sqrt{\sum_i \frac{(P_{1i} - P_{0i})^2}{P_{0i}}}$$

| w | Etki |
|---|---|
| 0.1 | Küçük |
| 0.3 | Orta |
| 0.5 | Büyük |

#### Pearson r için

| r | Etki |
|---|---|
| 0.10 | Küçük |
| 0.30 | Orta |
| 0.50 | Büyük |

### Örneklem Büyüklüğü Formülleri

#### Bir Ortalama İçin

$$n = \left(\frac{z_{\alpha/2} + z_\beta}{d}\right)^2$$

$d$ = Cohen's d (etki büyüklüğü), veya:

$$n = \frac{(z_{\alpha/2} + z_\beta)^2 \sigma^2}{\Delta^2}$$

$\Delta = \mu_1 - \mu_0$ (tespit edilmek istenen fark)

#### İki Ortalama Karşılaştırma

$$n_1 = n_2 = \frac{2(z_{\alpha/2} + z_\beta)^2 \sigma^2}{\Delta^2}$$

#### Oran İçin

$$n = \frac{(z_{\alpha/2} + z_\beta)^2 [p_1(1-p_1) + p_2(1-p_2)]}{(p_1 - p_2)^2}$$

### Güç-n İlişkisi

| n | Güç (d=0.5, α=0.05) |
|---|---|
| 20 | ~0.41 |
| 50 | ~0.70 |
| 85 | ~0.85 |
| 130 | ~0.95 |

### A/B Testi Örneklem Hesabı

Dönüşüm oranı testi için minimum örneklem:

$$n = \frac{(z_{\alpha/2} + z_\beta)^2 \cdot 2\bar{p}(1-\bar{p})}{\delta^2}$$

$\bar{p} = (p_1 + p_2)/2$, $\delta = p_2 - p_1$ (minimum tespit edilebilir etki, MDE)

**Örnek**: $p_1 = 0.10$, MDE = 0.02, $\alpha=0.05$, güç=0.80

$$n \approx \frac{(1.96+0.84)^2 \times 2 \times 0.11 \times 0.89}{0.02^2} \approx 3843 \text{ (her grup)}$$

### Python ile Güç Analizi

```python
from statsmodels.stats.power import (
    TTestPower, TTestIndPower, NormalIndPower, GofChisquarePower
)
import matplotlib.pyplot as plt
import numpy as np

# 1. Tek örneklem t-testi: n hesapla
analysis = TTestPower()
n = analysis.solve_power(effect_size=0.5, power=0.8, alpha=0.05)
print(f"Gerekli n: {n:.0f}")

# 2. Bağımsız iki örneklem t-testi
analysis = TTestIndPower()
n = analysis.solve_power(effect_size=0.5, power=0.8, alpha=0.05, ratio=1.0)
print(f"Her grup için n: {n:.0f}")

# 3. Güç hesapla (n verilmişken)
power = analysis.solve_power(effect_size=0.5, nobs1=50, alpha=0.05)
print(f"Güç: {power:.3f}")

# 4. Güç eğrisi
effect_sizes = np.linspace(0.1, 1.0, 100)
powers = [analysis.solve_power(es, nobs1=50, alpha=0.05) 
          for es in effect_sizes]
plt.plot(effect_sizes, powers)
plt.xlabel('Etki Büyüklüğü')
plt.ylabel('Güç')
plt.axhline(0.8, color='r', linestyle='--')
```

### A priori vs Post hoc Analiz

- **A priori**: Çalışma öncesi n hesaplama → ✅ Doğru kullanım
- **Post hoc**: Elde edilen n ile güç hesaplama → ⚠️ Sorgulanabilir (p-değeriyle ilgili olduğu için döngüsel argüman riski)

---

## 💡 Bağlantılar
- [[STAT - Hipotez Testleri - Temel Kavramlar]]
- [[STAT - Hipotez Testleri - t-testi]]
- [[STAT - Güven Aralıkları]]
- [[STAT - ANOVA]]

## ❓ Sorular / Anlamadıklarım


## 🔗 Kaynaklar
- Cohen, J. - Statistical Power Analysis for the Behavioral Sciences
- G*Power (ücretsiz yazılım)
- statsmodels.stats.power Documentation
