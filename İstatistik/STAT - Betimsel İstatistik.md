---
tarih: 2025-01-01
konu: Betimsel İstatistik, Merkezi Eğilim ve Yayılım Ölçüleri
etiket: [istatistik, betimsel, ortalama, medyan, mod, standart-sapma]
kaynak:
zorluk: ⭐⭐
---

## 📌 Özet

Betimsel istatistik, veri setini özetleyen ve açıklayan ölçütlerin hesaplanmasıdır. Merkezi eğilim ölçüleri (ortalama, medyan, mod) ve yayılım ölçüleri (varyans, standart sapma, IQR) ana araçlardır.

---

## 🧠 Detay

### Merkezi Eğilim Ölçüleri

#### Aritmetik Ortalama (Mean)

$$\bar{x} = \frac{\sum_{i=1}^{n} x_i}{n} = \frac{x_1 + x_2 + \cdots + x_n}{n}$$

- Anakütle ortalaması: $\mu$ (mu)
- Örneklem ortalaması: $\bar{x}$ (x-bar)
- **Dezavantaj**: Aykırı değerlere (outlier) duyarlıdır

#### Ağırlıklı Ortalama

$$\bar{x}_w = \frac{\sum w_i x_i}{\sum w_i}$$

#### Medyan (Ortanca)

Verileri sıraladıktan sonra ortadaki değer.

- **n tek ise**: Medyan = $\frac{n+1}{2}$. sıradaki değer
- **n çift ise**: Medyan = $\frac{n/2 + (n/2+1)}{2}$. sıradakilerin ortalaması

> Aykırı değerlere karşı **dayanıklıdır (robust)**.

#### Mod (Tepe değer)

En sık tekrar eden değer. Birden fazla mod olabilir (bimodal, multimodal).

#### Geometrik Ortalama

$$G = \sqrt[n]{x_1 \cdot x_2 \cdots x_n} = \left(\prod_{i=1}^{n} x_i\right)^{1/n}$$

Büyüme oranları, finansal getiriler için kullanılır.

#### Harmonik Ortalama

$$H = \frac{n}{\sum_{i=1}^{n} \frac{1}{x_i}}$$

Hız, verimlilik gibi oran verilerinde kullanılır.

**Sıralama**: $H \leq G \leq \bar{x}$ (eşitlik ancak tüm değerler eşitse)

---

### Yayılım (Dağılım) Ölçüleri

#### Ranj (Değişim Aralığı)

$$R = x_{max} - x_{min}$$

#### Varyans

Anakütle varyansı:
$$\sigma^2 = \frac{\sum_{i=1}^{N}(x_i - \mu)^2}{N}$$

Örneklem varyansı (Bessel düzeltmesi):
$$s^2 = \frac{\sum_{i=1}^{n}(x_i - \bar{x})^2}{n-1}$$

> **Neden n-1?** Örneklem ortalaması kullanıldığında bir serbestlik derecesi kaybedilir → yansız tahmin için n-1.

#### Standart Sapma

$$\sigma = \sqrt{\sigma^2} \quad \text{veya} \quad s = \sqrt{s^2}$$

Ortalama ile aynı birimde olduğu için yorumlaması kolaydır.

#### Değişim Katsayısı (CV)

$$CV = \frac{s}{\bar{x}} \times 100\%$$

Farklı ölçekteki değişkenlerin yayılımını karşılaştırmak için kullanılır.

#### Çeyrekler ve IQR

- **Q1** (1. Çeyrek): Verinin %25'i bu değerin altındadır
- **Q2** (2. Çeyrek): Medyan
- **Q3** (3. Çeyrek): Verinin %75'i bu değerin altındadır
- **IQR** = Q3 - Q1

**Aykırı değer tespiti (IQR yöntemi)**:
- Alt sınır: $Q1 - 1.5 \times IQR$
- Üst sınır: $Q3 + 1.5 \times IQR$

---

### Şekil Ölçüleri

#### Çarpıklık (Skewness)

$$\text{Skewness} = \frac{1}{n}\sum\left(\frac{x_i - \bar{x}}{s}\right)^3$$

| Değer | Anlam |
|---|---|
| Skewness = 0 | Simetrik dağılım |
| Skewness > 0 | Sağa çarpık (pozitif) |
| Skewness < 0 | Sola çarpık (negatif) |

Sağa çarpık: Ortalama > Medyan > Mod

#### Basıklık (Kurtosis)

$$\text{Kurtosis} = \frac{1}{n}\sum\left(\frac{x_i - \bar{x}}{s}\right)^4 - 3$$

| Değer | Tip | Anlam |
|---|---|---|
| = 0 | Mesokurtik | Normal dağılım gibi |
| > 0 | Leptokurtik | Sivri, kalın kuyruklu |
| < 0 | Platykurtik | Basık, ince kuyruklu |

---

### Beş Sayı Özeti (Five Number Summary)

$$\{x_{min},\ Q1,\ Medyan,\ Q3,\ x_{max}\}$$

**Kutu-bıyık grafiği (Box plot)** bu beş sayıyı görselleştirir.

---

## 💡 Bağlantılar
- [[STAT - Temel Kavramlar ve Tanımlar]]
- [[STAT - Frekans Dağılımları ve Grafikler]]
- [[STAT - Normal Dağılım]]
- [[DS - Betimsel İstatistik]]
- [[DS - Aykırı Değer Analizi]]

## ❓ Sorular / Anlamadıklarım


## 🔗 Kaynaklar
- OpenStax Statistics - Chapter 2-3
- Khan Academy - Descriptive Statistics
