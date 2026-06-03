---
tarih: 2025-01-01
konu: Hipotez Testi, p-değeri, Tip I Hata, Tip II Hata, Güç
etiket: [istatistik, hipotez, p-değeri, tip1-hata, tip2-hata, güç]
kaynak:
zorluk: ⭐⭐⭐
---

## 📌 Özet

Hipotez testi, örneklem verisiyle anakütle hakkında bir iddiayı (hipotezi) sınamaktır. Sıfır hipotezini reddetmek ya da reddetmemek için istatistiksel kanıt değerlendirme yöntemidir.

---

## 🧠 Detay

### Hipotez Türleri

| Hipotez | Simge | Anlam |
|---|---|---|
| **Sıfır (Null)** | $H_0$ | Fark yok / Etkisiz / Status quo |
| **Alternatif** | $H_1$ veya $H_a$ | Fark var / Etki var |

**Örnekler:**
- $H_0: \mu = 5$ vs $H_1: \mu \neq 5$ (iki yönlü)
- $H_0: \mu \leq 5$ vs $H_1: \mu > 5$ (sağ yönlü)
- $H_0: \mu \geq 5$ vs $H_1: \mu < 5$ (sol yönlü)

### Test Türleri

| Tür | $H_1$ | Kritik Bölge |
|---|---|---|
| **İki Yönlü** | $\mu \neq \mu_0$ | Her iki kuyruk |
| **Sağ Yönlü** | $\mu > \mu_0$ | Sağ kuyruk |
| **Sol Yönlü** | $\mu < \mu_0$ | Sol kuyruk |

### Hata Türleri

|  | $H_0$ Doğru | $H_0$ Yanlış |
|---|---|---|
| **$H_0$ Reddedildi** | **Tip I Hata** ($\alpha$) | ✅ Doğru Karar |
| **$H_0$ Reddedilmedi** | ✅ Doğru Karar | **Tip II Hata** ($\beta$) |

- **Tip I Hata (False Positive)**: $P(\text{ret} | H_0 \text{ doğru}) = \alpha$ (anlamlılık düzeyi)
- **Tip II Hata (False Negative)**: $P(\text{ret yok} | H_0 \text{ yanlış}) = \beta$
- **Test Gücü (Power)**: $1 - \beta = P(\text{ret} | H_0 \text{ yanlış})$

### p-Değeri ⭐

> "Eğer $H_0$ doğru olsaydı, gözlemlediğimiz ya da daha aşırı bir test istatistiği elde etme olasılığı"

$$p\text{-değeri} = P(\text{test istatistiği} \geq t_{gözlem} \mid H_0)$$

| p-değeri | Karar |
|---|---|
| $p \leq \alpha$ | $H_0$ reddet (istatistiksel olarak anlamlı) |
| $p > \alpha$ | $H_0$ reddetme (yeterli kanıt yok) |

**⚠️ Yaygın Yanlış Anlama:**
- p-değeri $H_0$'ın doğru olma olasılığı DEĞİLDİR
- p-değeri pratik önemi göstermez (etki büyüklüğüne bakılmalı)
- $p > 0.05$ $H_0$'ın doğru olduğu anlamına gelmez

### Hipotez Testi Adımları

1. $H_0$ ve $H_1$ belirle
2. Anlamlılık düzeyini seç ($\alpha = 0.05$)
3. Test istatistiğini hesapla
4. p-değerini ya da kritik değeri bul
5. Karar ver ve yorumla

### Test İstatistikleri

**Z-testi** ($\sigma$ biliniyorsa):
$$Z = \frac{\bar{x} - \mu_0}{\sigma/\sqrt{n}} \sim N(0,1)$$

**t-testi** ($\sigma$ bilinmiyorsa):
$$T = \frac{\bar{x} - \mu_0}{s/\sqrt{n}} \sim t_{n-1}$$

**Oran testi**:
$$Z = \frac{\hat{p} - p_0}{\sqrt{p_0(1-p_0)/n}} \sim N(0,1)$$

### Güç Analizi

Test gücünü etkileyen faktörler:

| Faktör | Güç Üzerindeki Etki |
|---|---|
| $\alpha$ ↑ | Güç ↑ |
| n ↑ | Güç ↑ |
| Etki büyüklüğü ↑ | Güç ↑ |
| $\sigma$ ↑ | Güç ↓ |

**Etki Büyüklüğü (Cohen's d)**:
$$d = \frac{\mu_1 - \mu_0}{\sigma}$$

| d | Yorum |
|---|---|
| 0.2 | Küçük etki |
| 0.5 | Orta etki |
| 0.8 | Büyük etki |

### İstatistiksel vs Pratik Anlamlılık

- Büyük n ile küçük farklar istatistiksel anlamlı olabilir
- Küçük n ile büyük farklar anlamlı çıkmayabilir
- Her zaman **etki büyüklüğünü ve güven aralığını** raporla!

### Çoklu Karşılaştırma Problemi

5 bağımsız test, $\alpha = 0.05$ ise:
$$P(\text{en az 1 Tip I Hata}) = 1 - (1-0.05)^5 \approx 0.226$$

**Düzeltmeler:**
- **Bonferroni**: $\alpha^* = \alpha / m$ (m = test sayısı)
- **FDR (Benjamini-Hochberg)**: False Discovery Rate kontrolü

---

## 💡 Bağlantılar
- [[STAT - Hipotez Testleri - t-testi]]
- [[STAT - Hipotez Testleri - Ki-kare ve F Testi]]
- [[STAT - Güven Aralıkları]]
- [[STAT - ANOVA]]

## ❓ Sorular / Anlamadıklarım


## 🔗 Kaynaklar
- OpenStax Statistics - Ch. 9-10
- Statistics Done Wrong (Reinhart) - Free online
