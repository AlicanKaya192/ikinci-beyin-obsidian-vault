---
tarih: 2026-05-28
konu: Data Science
etiket: ["data-science", "pandas", "dataframe", "sıralama", "gruplama"]
kaynak: Pandas Resmi Dokümantasyon
zorluk: orta
---

## 📌 Özet
DataFrame üzerinde sıralama, gruplama, pivot, apply gibi ileri düzey işlemler veri analizinin çekirdeğini oluşturur.

## 🧠 Detay

### Sıralama
```python
# Değere göre sırala
df.sort_values("yas")
df.sort_values("yas", ascending=False)
df.sort_values(["sehir","yas"])

# Index'e göre
df.sort_index()
```

### apply() Fonksiyonu
```python
# Sütuna fonksiyon uygula
df["yas_kategori"] = df["yas"].apply(
    lambda x: "genç" if x < 30 else "orta"
)

# Kendi fonksiyonu
def kategorize(yas):
    if yas < 25: return "genç"
    elif yas < 40: return "orta"
    else: return "yaşlı"

df["kategori"] = df["yas"].apply(kategorize)
```

### groupby()
```python
# Şehre göre grupla, yaş ortalaması
df.groupby("sehir")["yas"].mean()

# Birden fazla agregasyon
df.groupby("sehir").agg({
    "yas": ["mean", "min", "max"],
    "isim": "count"
})

# Çok sütunla grupla
df.groupby(["sehir", "kategori"])["yas"].mean()
```

### value_counts() ve nunique()
```python
print(df["sehir"].value_counts())     # her şehirden kaç kişi
print(df["sehir"].nunique())          # kaç farklı şehir
print(df["sehir"].unique())           # benzersiz değerler
```

### pivot_table()
```python
pd.pivot_table(df,
    values="yas",
    index="sehir",
    columns="kategori",
    aggfunc="mean"
)
```

### rename() ve drop()
```python
df.rename(columns={"isim": "ad", "yas": "yil"}, inplace=True)
df.drop(columns=["gereksiz_sutun"], inplace=True)
df.drop(index=[0, 2], inplace=True)
```

## 💡 Bağlantılar
- [[DS - Pandas Temel Kullanım]]
- [[DS - Pandas Gruplama ve Agregasyon]]
- [[DS - Pandas Merge ve Join]]

## ❓ Sorular / Anlamadıklarım
- `inplace=True` ne zaman kullanmalıyım, ne zaman kaçınmalıyım?
- `apply` yerine vektörel işlemler ne zaman daha iyi?

## 🔗 Kaynaklar
- https://pandas.pydata.org/docs/user_guide/groupby.html
