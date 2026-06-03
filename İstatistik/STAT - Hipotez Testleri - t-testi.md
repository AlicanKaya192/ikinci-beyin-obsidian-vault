---
tarih: 2025-01-01
konu: t-testi, Tek Örneklem, İki Örneklem, Eşleştirilmiş t-testi
etiket: [istatistik, t-testi, hipotez, karşılaştırma]
kaynak:
zorluk: ⭐⭐
---

## 📌 Özet

t-testi, ortalama(lar) hakkında hipotez sınar. σ bilinmediğinde z-testi yerine t-dağılımı kullanılır. Üç ana türü: tek örneklem, bağımsız iki örneklem, eşleştirilmiş.

---

## 🧠 Detay

### Varsayımlar

- Bağımlı değişken sürekli
- Veriler yaklaşık normal dağılımlı (veya $n \geq 30$)
- Gözlemler bağımsız (eşleştirilmiş hariç)
- İki örneklem için: varyanslar eşit (Levene testi ile kontrol)

---

### 1. Tek Örneklem t-testi

**Amaç**: Örneklem ortalamasını belirli bir değerle karşılaştır.

$$H_0: \mu = \mu_0 \quad vs \quad H_1: \mu \neq \mu_0$$

**Test İstatistiği:**
$$t = \frac{\bar{x} - \mu_0}{s/\sqrt{n}} \sim t_{n-1}$$

**Örnek**: Bir ilaç şirketi yeni ilacın kan basıncını ortalama 10 mmHg düşürdüğünü iddia ediyor. Örneklem: $n=25$, $\bar{x}=8.5$, $s=4$. Test et.

$$t = \frac{8.5 - 10}{4/\sqrt{25}} = \frac{-1.5}{0.8} = -1.875, \quad df=24$$

$t_{0.025, 24} = 2.064$ → $|t| < t_{krit}$ → $H_0$ reddedilmez.

---

### 2. Bağımsız İki Örneklem t-testi

**Amaç**: İki bağımsız grubun ortalamalarını karşılaştır.

$$H_0: \mu_1 = \mu_2 \quad vs \quad H_1: \mu_1 \neq \mu_2$$

#### a) Eşit Varyans Varsayımı (Student's t)

**Havuzlanmış standart sapma:**
$$s_p = \sqrt{\frac{(n_1-1)s_1^2 + (n_2-1)s_2^2}{n_1+n_2-2}}$$

**Test istatistiği:**
$$t = \frac{\bar{x}_1 - \bar{x}_2}{s_p\sqrt{1/n_1 + 1/n_2}} \sim t_{n_1+n_2-2}$$

#### b) Eşit Olmayan Varyans (Welch's t)

$$t = \frac{\bar{x}_1 - \bar{x}_2}{\sqrt{s_1^2/n_1 + s_2^2/n_2}}$$

**Welch-Satterthwaite serbestlik derecesi:**
$$df = \frac{(s_1^2/n_1 + s_2^2/n_2)^2}{\frac{(s_1^2/n_1)^2}{n_1-1} + \frac{(s_2^2/n_2)^2}{n_2-1}}$$

> 💡 **Tavsiye**: Her zaman Welch's t kullan, daha güvenilir.

#### Varyans Eşitliği: Levene Testi

$H_0: \sigma_1^2 = \sigma_2^2$ → $p > 0.05$ ise Student's t, değilse Welch's t

---

### 3. Eşleştirilmiş (Paired) t-testi

**Amaç**: Aynı birimlerin iki durumunu karşılaştır (önce-sonra, sol-sağ).

$$d_i = x_{1i} - x_{2i}$$

$$H_0: \mu_d = 0 \quad vs \quad H_1: \mu_d \neq 0$$

$$t = \frac{\bar{d}}{s_d/\sqrt{n}} \sim t_{n-1}$$

**Ne zaman eşleştirilmiş?**
- Aynı denek, iki farklı ölçüm (pretest-posttest)
- Eşleştirilmiş çiftler (ikiz çalışmaları)
- Tekrarlı ölçümler

---

### Python ile t-testi

```python
from scipy import stats
import numpy as np

# 1. Tek örneklem
t_stat, p_val = stats.ttest_1samp(data, popmean=mu0)

# 2. Bağımsız iki örneklem
# equal_var=False → Welch's t (önerilen)
t_stat, p_val = stats.ttest_ind(group1, group2, equal_var=False)

# 3. Eşleştirilmiş
t_stat, p_val = stats.ttest_rel(before, after)

# Levene testi (varyans eşitliği)
stat, p = stats.levene(group1, group2)
```

---

### Etki Büyüklüğü: Cohen's d

**Tek örneklem:**
$$d = \frac{\bar{x} - \mu_0}{s}$$

**İki örneklem:**
$$d = \frac{\bar{x}_1 - \bar{x}_2}{s_p}$$

| d | Etki |
|---|---|
| 0.2 | Küçük |
| 0.5 | Orta |
| 0.8 | Büyük |

---

### Parametrik Olmayan Alternatifleri

| t-testi | Alternatif |
|---|---|
| Tek örneklem t | Wilcoxon işaretli sıra testi |
| Bağımsız iki örneklem | Mann-Whitney U testi |
| Eşleştirilmiş t | Wilcoxon işaretli sıra testi |

---

## 💡 Bağlantılar
- [[STAT - Hipotez Testleri - Temel Kavramlar]]
- [[STAT - ANOVA]]
- [[STAT - Parametrik Olmayan Testler]]
- [[STAT - Normal Dağılım]]

## ❓ Sorular / Anlamadıklarım


## 🔗 Kaynaklar
- scipy.stats.ttest_ind Documentation
- OpenStax Statistics - Ch. 10
