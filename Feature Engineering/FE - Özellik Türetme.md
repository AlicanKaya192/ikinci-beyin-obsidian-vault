---
tarih: 2025-01-01
konu: Özellik Türetme, Etkileşim Özellikleri, Domain Özellikleri, Polinom
etiket: [feature-engineering, özellik-türetme, interaction, polinom, domain]
kaynak:
zorluk: ⭐⭐⭐
---

## 📌 Özet

Mevcut özelliklerden yeni, daha anlamlı özellikler üretmek. Model doğruluğunu artırmanın en etkili yollarından biri — domain bilgisi burada kritik rol oynar.

---

## 🧠 Detay

### Matematiksel Türetme

#### Temel İşlemler

```python
# Toplama / Çıkarma
df['boy_kilo_fark'] = df['boy'] - df['kilo']

# Çarpma / Oran
df['bmi'] = df['kilo'] / (df['boy'] / 100) ** 2
df['gelir_borclanma_orani'] = df['borc'] / df['gelir']

# Yüzde değişim
df['fiyat_degisim'] = (df['bugun_fiyat'] - df['dun_fiyat']) / df['dun_fiyat']

# Kümülatif
df['kumulatif_satis'] = df['satis'].cumsum()
```

#### Etkileşim Özellikleri (Interaction Terms)

```python
# Manuel
df['yas_x_gelir'] = df['yas'] * df['gelir']

# sklearn — tüm ikililer
from sklearn.preprocessing import PolynomialFeatures

poly = PolynomialFeatures(degree=2, interaction_only=True, include_bias=False)
X_interact = poly.fit_transform(X[['yas', 'gelir', 'egitim_yil']])
feature_names = poly.get_feature_names_out(['yas', 'gelir', 'egitim_yil'])
# → yas, gelir, egitim_yil, yas*gelir, yas*egitim_yil, gelir*egitim_yil
```

#### Polinom Özellikler

```python
poly = PolynomialFeatures(degree=2, include_bias=False)
X_poly = poly.fit_transform(X[['yas']])
# → yas, yas²

# Dikkat: degree=3 ile bile çok fazla özellik üretebilir
# n özellik, degree=2 → n(n+1)/2 + n özellik
```

---

### İstatistiksel Türetme

```python
# Gruba göre istatistikler (Group Aggregation)
agg = df.groupby('sehir')['gelir'].agg(['mean', 'std', 'median', 'max'])
agg.columns = ['sehir_gelir_mean', 'sehir_gelir_std',
               'sehir_gelir_median', 'sehir_gelir_max']
df = df.merge(agg, on='sehir', how='left')

# Kişinin grubuna göre sapması
df['gelir_sehir_sapma'] = df['gelir'] - df['sehir_gelir_mean']
df['gelir_sehir_zscore'] = (df['gelir'] - df['sehir_gelir_mean']) / df['sehir_gelir_std']
```

---

### Tarih/Zaman Özellikleri

```python
df['tarih'] = pd.to_datetime(df['tarih'])

# Bileşenler
df['yil']         = df['tarih'].dt.year
df['ay']          = df['tarih'].dt.month
df['gun']         = df['tarih'].dt.day
df['haftanin_gunu'] = df['tarih'].dt.dayofweek   # 0=Pazartesi
df['yilin_gunu']  = df['tarih'].dt.dayofyear
df['hafta_no']    = df['tarih'].dt.isocalendar().week
df['ceyrek']      = df['tarih'].dt.quarter
df['saat']        = df['tarih'].dt.hour
df['dakika']      = df['tarih'].dt.minute

# İkili (Binary) özellikler
df['hafta_sonu']  = df['haftanin_gunu'].isin([5, 6]).astype(int)
df['is_gunu']     = (~df['tarih'].dt.dayofweek.isin([5, 6])).astype(int)
df['sabah_mi']    = (df['saat'].between(6, 12)).astype(int)

# Döngüsel encoding (ay, saat dairesel!)
import numpy as np
df['ay_sin'] = np.sin(2 * np.pi * df['ay'] / 12)
df['ay_cos'] = np.cos(2 * np.pi * df['ay'] / 12)
df['saat_sin'] = np.sin(2 * np.pi * df['saat'] / 24)
df['saat_cos'] = np.cos(2 * np.pi * df['saat'] / 24)

# İki tarih arası fark
df['kac_gun_musteriyiz'] = (df['bugun'] - df['uyelik_tarihi']).dt.days

# Belirli tarihten uzaklık
referans = pd.Timestamp('2020-01-01')
df['covid_sonrasi_gun'] = (df['tarih'] - referans).dt.days.clip(lower=0)
```

---

### Metin Tabanlı Türetme

```python
# Uzunluk özellikleri
df['yorum_uzunluk']     = df['yorum'].str.len()
df['kelime_sayisi']     = df['yorum'].str.split().str.len()
df['buyuk_harf_orani']  = df['yorum'].str.count(r'[A-Z]') / df['yorum_uzunluk']
df['noktalama_sayisi']  = df['yorum'].str.count(r'[.,!?;:]')
df['soru_isaretli_mi']  = df['yorum'].str.contains(r'\?').astype(int)

# İçerik kontrolü
df['ucretsiz_kelimesi_var'] = df['yorum'].str.lower().str.contains('ücretsiz').astype(int)
```

---

### Coğrafi Özellikler

```python
from math import radians, sin, cos, sqrt, atan2

def haversine(lat1, lon1, lat2, lon2):
    """İki koordinat arası mesafe (km)"""
    R = 6371
    lat1, lon1, lat2, lon2 = map(radians, [lat1, lon1, lat2, lon2])
    dlat = lat2 - lat1
    dlon = lon2 - lon1
    a = sin(dlat/2)**2 + cos(lat1)*cos(lat2)*sin(dlon/2)**2
    return R * 2 * atan2(sqrt(a), sqrt(1-a))

df['merkeze_mesafe'] = df.apply(
    lambda r: haversine(r['lat'], r['lon'], 41.01, 28.97), axis=1
)  # İstanbul'a mesafe

# Clustering ile bölge özellikleri
from sklearn.cluster import KMeans
kmeans = KMeans(n_clusters=10, random_state=42)
df['cografi_cluster'] = kmeans.fit_predict(df[['lat', 'lon']])
```

---

### Domain Spesifik Örnekler

```python
# E-Ticaret
df['sepet_baskina_urun'] = df['urun_sayisi'] / df['sepet_sayisi']
df['ortalama_siparis_degeri'] = df['toplam_gelir'] / df['siparis_sayisi']
df['iade_orani'] = df['iade_sayisi'] / df['siparis_sayisi']

# Finans
df['borc_gelir_orani'] = df['aylik_borc'] / df['aylik_gelir']
df['likiditie_orani'] = df['donen_varliklar'] / df['kisa_vadeli_borc']

# Sağlık
df['bmi'] = df['kilo_kg'] / (df['boy_m'] ** 2)
df['tansiyon_fark'] = df['sistolik'] - df['diastolik']

# İnsan kaynakları
df['sirket_yasinda'] = (pd.Timestamp.now() - df['ise_baslama']).dt.days / 365
df['pozisyon_basi_maas'] = df['maas'] / df['pozisyon_katsayi']
```

---

### Özellik Türetmede İpuçları

1. **EDA yap önce** — değişkenler arası ilişkileri gör
2. **Domain uzmanına danış** — hangi kombinasyonlar anlamlı?
3. **Her türetilen özelliği doğrula** — model skoru arttı mı?
4. **Data leakage'a dikkat** — hedef bilgisi kullanma!
5. **Çok özellik = overfitting** — seçimle dengele

---

## 💡 Bağlantılar
- [[FE - Tarih ve Zaman Özellikleri]]
- [[FE - Metin Özellikleri]]
- [[FE - Özellik Seçimi Yöntemleri]]
- [[FE - Encoding Yöntemleri]]

## ❓ Sorular / Anlamadıklarım


## 🔗 Kaynaklar
- Feature Engineering for Machine Learning (Zheng & Casari) - Ch. 6
- Kaggle Notebooks - Feature Engineering
