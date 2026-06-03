---
tarih: 2025-01-01
konu: ANOVA, Tek Yönlü, İki Yönlü, Post-Hoc Testler
etiket: [istatistik, ANOVA, varyans-analizi, post-hoc, Tukey]
kaynak:
zorluk: ⭐⭐⭐
---

## 📌 Özet

ANOVA (Analysis of Variance), ikiden fazla grup ortalamasını karşılaştırmak için kullanılır. Çoklu t-testi yerine ANOVA kullanmak Tip I hata birikimini önler.

---

## 🧠 Detay

### Neden ANOVA?

3 grup için 3 t-testi yapılırsa:
$$P(\text{en az 1 Tip I Hata}) = 1 - (1-0.05)^3 = 0.143 \approx \%14$$

ANOVA tüm grupları tek bir testte karşılaştırır → $\alpha = 0.05$ korunur.

---

### Tek Yönlü ANOVA (One-Way ANOVA)

**Soru**: $k$ bağımsız grubun ortalaması birbirinden farklı mı?

$$H_0: \mu_1 = \mu_2 = \cdots = \mu_k$$
$$H_1: \text{En az bir } \mu_i \text{ farklıdır}$$

#### Varsayımlar
1. Normallik (her grupta)
2. Varyans homojenliği (Levene testi)
3. Bağımsız gözlemler

#### Varyans Ayrıştırması

$$SS_{Total} = SS_{Between} + SS_{Within}$$

| Kaynak | SS | df | MS | F |
|---|---|---|---|---|
| Gruplar arası | $\sum n_i(\bar{x}_i - \bar{x})^2$ | $k-1$ | $SS_B/(k-1)$ | $MS_B/MS_W$ |
| Gruplar içi | $\sum \sum (x_{ij} - \bar{x}_i)^2$ | $N-k$ | $SS_W/(N-k)$ | — |
| Toplam | $\sum \sum (x_{ij} - \bar{x})^2$ | $N-1$ | — | — |

$$F = \frac{MS_{Between}}{MS_{Within}} \sim F_{k-1,\ N-k}$$

#### Etki Büyüklüğü: Eta-kare ($\eta^2$)

$$\eta^2 = \frac{SS_{Between}}{SS_{Total}}$$

| $\eta^2$ | Etki |
|---|---|
| 0.01 | Küçük |
| 0.06 | Orta |
| 0.14 | Büyük |

---

### Post-Hoc Testler

ANOVA anlamlıysa → **hangi çiftler farklı?** sorusunu yanıtlar.

| Test | Kullanım | Özellik |
|---|---|---|
| **Tukey HSD** | Eşit örneklem, konservatif | En yaygın |
| **Bonferroni** | Her durum, çok konservatif | Karşılaştırma sayısını böler |
| **Scheffe** | Doğrusal kombinasyonlar | En konservatif |
| **Games-Howell** | Eşit olmayan varyans | Welch uyarlaması |
| **LSD** | Liberal | Tip I hataya yatkın |

---

### İki Yönlü ANOVA (Two-Way ANOVA)

**Soru**: İki bağımsız faktörün etkisi ve birleşik (interaction) etkisi var mı?

$$Y_{ijk} = \mu + \alpha_i + \beta_j + (\alpha\beta)_{ij} + \varepsilon_{ijk}$$

| Kaynak | Test Eder |
|---|---|
| Faktör A | A'nın ana etkisi |
| Faktör B | B'nin ana etkisi |
| A × B | Etkileşim (interaction) |

**Etkileşim anlamlıysa**: Ana etkileri yorumlamak yanıltıcı olabilir.

---

### Tekrarlı Ölçümler ANOVA (Repeated Measures)

Aynı denekler birden fazla koşulda ölçülüyorsa.

**Avantaj**: Bireysel farklılıklar kontrol altında → Daha güçlü test
**Ek Varsayım**: **Sfersellik** (Mauchly testi ile kontrol)

Sfersellik bozulursa: Greenhouse-Geisser veya Huynh-Feldt düzeltmesi

---

### ANCOVA (Kovaryans Analizi)

ANOVA + sürekli kovaryat kontrolü:
$$Y = \mu + \alpha_i + \beta X + \varepsilon$$

**Amaç**: Kovaryatı istatistiksel olarak kontrol ederek gruplar arası farkı incele.

---

### Parametrik Olmayan Alternatifler

| ANOVA Türü | Alternatif |
|---|---|
| Tek yönlü | Kruskal-Wallis H testi |
| Tekrarlı ölçümler | Friedman testi |

---

### Python ile ANOVA

```python
from scipy import stats
import statsmodels.api as sm
from statsmodels.formula.api import ols
import pingouin as pg

# Tek yönlü ANOVA (scipy)
f_stat, p_val = stats.f_oneway(group1, group2, group3)

# Tek yönlü ANOVA (statsmodels - tam tablo)
model = ols('score ~ C(group)', data=df).fit()
anova_table = sm.stats.anova_lm(model, typ=1)

# İki yönlü ANOVA
model = ols('score ~ C(A) + C(B) + C(A):C(B)', data=df).fit()
anova_table = sm.stats.anova_lm(model, typ=2)

# Post-hoc (Tukey)
from statsmodels.stats.multicomp import pairwise_tukeyhsd
tukey = pairwise_tukeyhsd(df['score'], df['group'])
print(tukey)

# Tekrarlı ölçümler (pingouin)
aov = pg.rm_anova(data=df, dv='score', within='time', subject='id')
```

---

## 💡 Bağlantılar
- [[STAT - Hipotez Testleri - Temel Kavramlar]]
- [[STAT - Hipotez Testleri - t-testi]]
- [[STAT - Parametrik Olmayan Testler]]
- [[ML - Model Değerlendirme Metrikleri]]

## ❓ Sorular / Anlamadıklarım


## 🔗 Kaynaklar
- Field, A. - Discovering Statistics Using IBM SPSS Statistics
- pingouin Documentation
