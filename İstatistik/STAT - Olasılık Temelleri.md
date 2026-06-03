---
tarih: 2025-01-01
konu: Olasılık Teorisi, Temel Kurallar, Koşullu Olasılık
etiket: [istatistik, olasılık, probability, Bayes, bağımsızlık]
kaynak:
zorluk: ⭐⭐⭐
---

## 📌 Özet

Olasılık, belirsiz olayların sayısal ifadesidir. 0 ile 1 arasında değer alır; 0 imkânsız, 1 kesin olayı temsil eder. İstatistiksel çıkarımın matematiksel temelidir.

---

## 🧠 Detay

### Temel Kavramlar

| Kavram | Tanım |
|---|---|
| **Deney (Experiment)** | Sonucu belirsiz süreç |
| **Örneklem Uzayı (Ω)** | Tüm olası sonuçlar kümesi |
| **Olay (Event)** | Örneklem uzayının alt kümesi |
| **Olasılık P(A)** | Olay A'nın gerçekleşme ihtimali |

### Olasılık Aksiyomları (Kolmogorov)

1. $P(A) \geq 0$ — Negatif olamaz
2. $P(\Omega) = 1$ — Örneklem uzayının olasılığı 1
3. Karşılıklı dışlayan olaylar için: $P(A \cup B) = P(A) + P(B)$

### Olasılık Yorumları

| Yorum | Açıklama |
|---|---|
| **Frekansçı** | Uzun vadede göreli frekans: $P(A) = \lim_{n \to \infty} \frac{n_A}{n}$ |
| **Klasik** | Eşit olası sonuçlar: $P(A) = \frac{\text{elverişli sonuç sayısı}}{\text{toplam sonuç sayısı}}$ |
| **Öznel (Bayesci)** | Kişisel inanç derecesi |

### Temel Olasılık Kuralları

#### Tümleyen (Complement)
$$P(A^c) = P(\bar{A}) = 1 - P(A)$$

#### Toplama Kuralı (Addition Rule)
$$P(A \cup B) = P(A) + P(B) - P(A \cap B)$$

Karşılıklı dışlayan ($A \cap B = \emptyset$) ise:
$$P(A \cup B) = P(A) + P(B)$$

#### Çarpım Kuralı (Multiplication Rule)
$$P(A \cap B) = P(A) \times P(B|A)$$

**Bağımsız olaylar** ($A \perp B$) için:
$$P(A \cap B) = P(A) \times P(B)$$

### Koşullu Olasılık

$$P(A|B) = \frac{P(A \cap B)}{P(B)}, \quad P(B) > 0$$

> "B olayı gerçekleşmişken A'nın olasılığı"

**Örnek**: Bir zarın 4 geldiği bilindiğinde çift sayı olasılığı:
$$P(\text{çift}|4) = \frac{P(4)}{P(4)} = 1$$

### Bağımsızlık

A ve B olayları bağımsızdır ↔
$$P(A|B) = P(A) \iff P(A \cap B) = P(A) \cdot P(B)$$

### Toplam Olasılık Teoremi

$B_1, B_2, \ldots, B_k$ karşılıklı dışlayan ve kapsamlı olaylar ise:
$$P(A) = \sum_{i=1}^{k} P(A|B_i) \cdot P(B_i)$$

### Bayes Teoremi ⭐

$$P(B_i|A) = \frac{P(A|B_i) \cdot P(B_i)}{\sum_{j=1}^{k} P(A|B_j) \cdot P(B_j)}$$

**Terminoloji:**
- $P(B_i)$: **Önsel (Prior)** olasılık
- $P(B_i|A)$: **Sonsal (Posterior)** olasılık
- $P(A|B_i)$: **Likelihood** (olabilirlik)

**Klasik Örnek — Hastalık Testi:**
- Hastalık yaygınlığı: $P(H) = 0.01$
- Test doğru pozitif: $P(+|H) = 0.99$
- Test yanlış pozitif: $P(+|H^c) = 0.05$

$$P(H|+) = \frac{0.99 \times 0.01}{0.99 \times 0.01 + 0.05 \times 0.99} = \frac{0.0099}{0.0594} \approx 0.167$$

Pozitif test rağmen gerçek hastalık olasılığı sadece ~%17!

### Sayma Teknikleri

| Yöntem | Formül | Sıra Önemli? | Tekrar? |
|---|---|---|---|
| Permütasyon | $P(n,r) = \frac{n!}{(n-r)!}$ | Evet | Hayır |
| Kombinasyon | $C(n,r) = \binom{n}{r} = \frac{n!}{r!(n-r)!}$ | Hayır | Hayır |
| Tekrarlı Permütasyon | $n^r$ | Evet | Evet |
| Tekrarlı Kombinasyon | $\binom{n+r-1}{r}$ | Hayır | Evet |

---

## 💡 Bağlantılar
- [[STAT - Olasılık Dağılımları (Kesikli)]]
- [[STAT - Olasılık Dağılımları (Sürekli)]]
- [[STAT - Bayes İstatistiği]]

## ❓ Sorular / Anlamadıklarım


## 🔗 Kaynaklar
- Introduction to Probability (Blitzstein & Hwang) - Free PDF
- Khan Academy Probability
