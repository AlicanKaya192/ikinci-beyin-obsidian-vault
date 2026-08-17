---
tarih: 2026-06-08
konu: ML Interview Prep
etiket: [mülakat, interview, istatistik, olasılık, a-b-testi, hipotez]
kaynak: Statistics for Data Scientists, Glassdoor
zorluk: orta
---

## 📌 Özet

Data Scientist ve ML Engineer mülakatlarında istatistik soruları genellikle **A/B testi**, **hipotez testi**, **dağılımlar** ve **istatistiksel güç** konularında yoğunlaşır.

---

## 🧠 Detay

### A/B Testi

**S: A/B testi nasıl tasarlanır?**
> 1. **Hipotezi kur:** H₀ (fark yok) vs H₁ (B grubu daha iyi)
> 2. **Başarı metriği seç:** conversion rate, revenue per user vb.
> 3. **Örneklem büyüklüğü hesapla:** α=0.05, güç=0.80, minimum effect size
> 4. **Randomizasyon:** Kullanıcıları rastgele A/B'ye ata (cookie/user ID)
> 5. **Süresi belirle:** En az 1-2 hafta (haftalık döngüsellik için)
> 6. **Analiz:** t-test / z-test, p-value < 0.05 ise H₀ reddet

```python
from scipy import stats
import numpy as np

a_conversions = 120  # A grubunda dönüşüm
a_total = 1000       # A grubunda toplam
b_conversions = 145
b_total = 1000

# İki oranlı z-test
from statsmodels.stats.proportion import proportions_ztest
count = np.array([a_conversions, b_conversions])
nobs = np.array([a_total, b_total])
stat, p_value = proportions_ztest(count, nobs)
print(f"p-value: {p_value:.4f} → {'Anlamlı fark' if p_value < 0.05 else 'Fark yok'}")
```

---

**S: p-value nedir, ne anlama gelir?**
> H₀ doğru olsaydı, gözlemlediğimiz kadar veya daha aşırı bir sonuç elde etme olasılığı.
>
> **Dikkat:** p < 0.05 "etki önemli" demek değil, "istatistiksel anlamlı" demek.
> Pratik anlamlılık (effect size) için Cohen's d veya RR da hesaplanmalı.
>
> **Yaygın yanlış:** "p-value, H₀'ın doğru olma olasılığıdır" — YANLIŞ.

---

**S: Type I ve Type II hata nedir?**
> | | H₀ Doğru | H₀ Yanlış |
> |---|---|---|
> | **Reddet** | Type I Hata (α) | Doğru Karar ✅ |
> | **Reddetme** | Doğru Karar ✅ | Type II Hata (β) |
>
> - **Type I (α = 0.05):** Olmayan farkı "var" diyorsun → yanlış alarm
> - **Type II (β):** Gerçek farkı "yok" diyorsun → kaçırdık
> - **Güç = 1 - β:** Gerçek etkiyi yakalama olasılığı (genellikle %80 hedeflenir)

---

**S: Central Limit Theorem ne işe yarar?**
> Yeterince büyük örneklem için, herhangi dağılımdan çekilen örneklem ortalamalarının dağılımı normale yaklaşır. Bu sayede:
> - Normal dağılım varsaymayan veriler için z/t testi kullanabiliriz
> - Örneklem büyüklüğü genellikle n ≥ 30 yeterli kabul edilir

---

**S: Nedensellik vs korelasyon farkı? Örnek?**
> Korelasyon: iki değişken birlikte değişir. Nedensellik: biri diğerine neden olur.
>
> Örnek: Dondurma satışları ile boğulma ölümleri korelasyonlu. Neden? İkisi de sıcak havanın etkisi (confounding variable).
>
> Nedensellik tespiti için: A/B testi (randomizasyon) veya IV/DiD/RDD (gözlemsel veri)

---

### Dağılımlar

**S: Hangi dağılım hangi durumda?**

| Dağılım | Kullanım Alanı | Örnek |
|---|---|---|
| **Normal** | Sürekli, simetrik | Boy, ağırlık, IQ |
| **Binomial** | n denemede k başarı | Madeni para atışı |
| **Poisson** | Birim zamandaki olay sayısı | Saatte gelen müşteri |
| **Exponential** | Olaylar arası süre | Müşteri gelişleri arası bekleme |
| **Bernoulli** | Tek deneme, 0/1 | Tek tıklama |

---

**S: Bayes Teoremi basitçe ne diyor?**
> Yeni kanıt ışığında önceki inancımızı güncelleriz.
> `P(A|B) = P(B|A) × P(A) / P(B)`
>
> Örnek: Hastalık testi. Test %99 doğrulukta. Hastalık prevalansı %1.
> Pozitif test sonucu gerçekten hasta olma olasılığı? → Bayes ile ~%50 civarı!

```python
# Bayes: P(hasta | pozitif test)
p_hasta = 0.01         # prevalans (prior)
p_pozitif_hasta = 0.99 # sensitivite
p_pozitif_saglikli = 0.01  # 1 - özgüllük

p_pozitif = (p_pozitif_hasta * p_hasta + 
             p_pozitif_saglikli * (1 - p_hasta))

p_hasta_pozitif = (p_pozitif_hasta * p_hasta) / p_pozitif
print(f"Pozitif test → gerçek hasta olma: {p_hasta_pozitif:.2%}")
# → ~50%
```

---

**S: Örneklem büyüklüğü nasıl hesaplanır?**

```python
from statsmodels.stats.power import TTestIndPower

analysis = TTestIndPower()
n = analysis.solve_power(
    effect_size=0.2,    # Cohen's d (küçük etki)
    alpha=0.05,         # significance level
    power=0.80,         # istenen güç
    alternative="two-sided"
)
print(f"Her grup için gereken örneklem: {n:.0f}")
```

---

## 💡 Bağlantılar
- [[STAT - A-B Testi Tasarımı ve Analizi]]
- [[STAT - Hipotez Testleri - Temel Kavramlar]]
- [[STAT - Bayes İstatistiği]]
- [[Interview - Makine Öğrenmesi Temel Sorular]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- [Statistics for Data Scientists](https://www.oreilly.com/library/view/practical-statistics-for/9781492072935/)
