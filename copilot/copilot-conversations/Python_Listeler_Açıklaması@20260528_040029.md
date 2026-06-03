---
epoch: 1779930029964
modelKey: "gemini-2.5-flash|google"
topic: "Python Listeler Açıklaması"



tags:
  - copilot-conversation
---

**user**: Python listeler konusunu Türkçe açıkla. Oluşturma, indexleme, slice, append, remove ve list comprehension örnekleri ver.
[Context: Notes: 01 - Python/Python - Listeler.md]
[Timestamp: 2026/05/28 04:00:29]

**ai**: 
<errorChunk>Model request failed: API key is not provided for the model: google/gemini-3.5-flash|openrouterai.</errorChunk>
[Timestamp: 2026/05/28 04:00:30]

**user**: Python listeler konusunu Türkçe açıkla. Oluşturma, indexleme, slice, append, remove ve list comprehension örnekleri ver.
[Context: Notes: 01 - Python/Python - Listeler.md]
[Timestamp: 2026/05/28 04:06:32]

**ai**: Python'da listeler, farklı türdeki öğeleri (sayılar, metinler, hatta başka listeler) sıralı bir şekilde depolayan esnek ve değiştirilebilir veri yapılarıdır. Köşeli parantezler `[]` kullanılarak tanımlanırlar.

İşte Python listelerinin temel işlemleri ve örnekleri:

### 1. Liste Oluşturma

Boş bir liste veya başlangıç değerleri olan bir liste oluşturabilirsiniz.

```python
# Boş bir liste oluşturma
bos_liste = []
print(f"Boş liste: {bos_liste}")

# Elemanlarla bir liste oluşturma
sayilar = [1, 2, 3, 4, 5]
print(f"Sayılar listesi: {sayilar}")

meyveler = ["elma", "muz", "çilek"]
print(f"Meyveler listesi: {meyveler}")

# Farklı türde elemanlar içeren bir liste
karisik_liste = [1, "merhaba", 3.14, True]
print(f"Karışık liste: {karisik_liste}")
```

### 2. Indexleme (Indexing)

Listelerdeki elemanlara konumlarına (index) göre erişebilirsiniz. Indexler 0'dan başlar. Negatif indexler listenin sonundan başlar (-1 son eleman, -2 sondan ikinci eleman vb.).

```python
meyveler = ["elma", "muz", "çilek", "portakal"]

# İlk elemana erişim (index 0)
print(f"İlk meyve: {meyveler[0]}") # Çıktı: elma

# Üçüncü elemana erişim (index 2)
print(f"Üçüncü meyve: {meyveler[2]}") # Çıktı: çilek

# Son elemana erişim (negatif index -1)
print(f"Son meyve: {meyveler[-1]}") # Çıktı: portakal

# Sondan ikinci elemana erişim (negatif index -2)
print(f"Sondan ikinci meyve: {meyveler[-2]}") # Çıktı: çilek
```

### 3. Slice (Dilimleme)

Listenin bir bölümünü (alt listesini) almak için dilimleme kullanabilirsiniz. `[başlangıç:bitiş:adım]` formatı kullanılır. `bitiş` index'i dahil değildir.

```python
sayilar = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

# İlk 3 elemanı al
print(f"İlk 3 eleman: {sayilar[0:3]}") # Çıktı: [0, 1, 2]
print(f"İlk 3 eleman (kısa yol): {sayilar[:3]}") # Çıktı: [0, 1, 2]

# 3. indexten sonuna kadar al
print(f"3. indexten sonrası: {sayilar[3:]}") # Çıktı: [3, 4, 5, 6, 7, 8, 9]

# 2. indexten 7. indexe kadar al (7 dahil değil)
print(f"2'den 7'ye kadar: {sayilar[2:7]}") # Çıktı: [2, 3, 4, 5, 6]

# Tüm listeyi kopyala
print(f"Tüm liste kopyası: {sayilar[:]}") # Çıktı: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

# Her 2 elemanı bir al
print(f"Her 2 elemanı bir: {sayilar[::2]}") # Çıktı: [0, 2, 4, 6, 8]

# Listeyi ters çevir
print(f"Ters çevrilmiş liste: {sayilar[::-1]}") # Çıktı: [9, 8, 7, 6, 5, 4, 3, 2, 1, 0]
```

### 4. `append()` Metodu

Listenin sonuna yeni bir eleman eklemek için `append()` metodunu kullanırız.

```python
meyveler = ["elma", "muz"]
print(f"Başlangıç meyveler: {meyveler}")

meyveler.append("çilek")
print(f"Çilek eklendi: {meyveler}") # Çıktı: ['elma', 'muz', 'çilek']

meyveler.append("kiraz")
print(f"Kiraz eklendi: {meyveler}") # Çıktı: ['elma', 'muz', 'çilek', 'kiraz']
```

### 5. `remove()` Metodu

Listeden belirli bir değeri kaldırmak için `remove()` metodunu kullanırız. Eğer değer listede birden fazla varsa, sadece ilk bulunanı kaldırır. Eğer değer listede yoksa `ValueError` hatası verir.

```python
urunler = ["kalem", "defter", "silgi", "kalem"]
print(f"Başlangıç ürünler: {urunler}")

urunler.remove("defter")
print(f"Defter kaldırıldı: {urunler}") # Çıktı: ['kalem', 'silgi', 'kalem']

urunler.remove("kalem") # İlk "kalem" kaldırılır
print(f"İlk kalem kaldırıldı: {urunler}") # Çıktı: ['silgi', 'kalem']

# urunler.remove("kitap") # Bu satır ValueError hatası verir çünkü "kitap" listede yok.
```

### 6. List Comprehension (Liste Anlayışı)

List comprehension, mevcut bir listeden veya başka bir yinelenebilir nesneden yeni bir liste oluşturmanın kısa ve okunabilir bir yoludur. Genellikle `for` döngüsü ve `if` koşulu içeren daha uzun kod bloklarını tek bir satıra indirger.

Genel yapısı: `[ifade for eleman in yinelenebilir_nesne if koşul]`

```python
# Örnek 1: Bir sayı listesindeki her sayının karesini al
sayilar = [1, 2, 3, 4, 5]

kareler = [sayi**2 for sayi in sayilar]
print(f"Kareler listesi: {kareler}") # Çıktı: [1, 4, 9, 16, 25]

# Örnek 2: Bir sayı listesindeki sadece çift sayıları al
sayilar = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

cift_sayilar = [sayi for sayi in sayilar if sayi % 2 == 0]
print(f"Çift sayılar listesi: {cift_sayilar}") # Çıktı: [2, 4, 6, 8, 10]

# Örnek 3: Bir metin listesindeki kelimeleri büyük harfe çevir
kelimeler = ["elma", "armut", "kiraz"]

buyuk_kelimeler = [kelime.upper() for kelime in kelimeler]
print(f"Büyük harfli kelimeler: {buyuk_kelimeler}") # Çıktı: ['ELMA', 'ARMUT', 'KİRAZ']

# Örnek 4: Hem koşul hem de dönüşüm
sayilar = [1, 2, 3, 4, 5, 6]
cift_sayilarin_kareleri = [sayi**2 for sayi in sayilar if sayi % 2 == 0]
print(f"Çift sayıların kareleri: {cift_sayilarin_kareleri}") # Çıktı: [4, 16, 36]
```

Bu örnekler, Python listeleriyle yapabileceğiniz temel işlemleri kapsamaktadır. Listeler, Python'da çok sık kullanılan ve oldukça güçlü veri yapılarıdır.

Daha fazla bilgi için [[Python - Listeler]] notuna bakabilirsiniz.
[Timestamp: 2026/05/28 04:06:43]

**user**: Python - Listeler notunu aç → Detay bölümünün altına yapıştır.
Sonra Özet bölümüne şunu yaz:
Python'da listeler, farklı türdeki öğeleri sıralı 
depolayan, değiştirilebilir veri yapılarıdır. [] 
ile tanımlanır.
Bağlantılar bölümüne de şunu ekle:
- [[Python - Döngüler]]
- [[Python - Veri Yapıları]]
Bu linkler henüz var olmayan notlara işaret ediyor ama Obsidian'da mor renkte görünecekler — ileride o notları oluşturduğunda otomatik bağlanacaklar. İşte 2. beynin gücü bu! 🕸️
[Context: Notes: 01 - Python/Python - Listeler.md]
[Timestamp: 2026/05/28 04:07:31]

**ai**: Harika bir fikir! [[Python - Listeler]] notunu güncelliyorum.

İşte güncellenmiş not içeriği:

```markdown
---
tarih: 2026-05-28
konu:
etiket: []
kaynak:
zorluk:
---

## 📌 Özet
Python'da listeler, farklı türdeki öğeleri sıralı 
depolayan, değiştirilebilir veri yapılarıdır. [] 
ile tanımlanır.

## 🧠 Detay
Python'da listeler, farklı türdeki öğeleri (sayılar, metinler, hatta başka listeler) sıralı bir şekilde depolayan esnek ve değiştirilebilir veri yapılarıdır. Köşeli parantezler `[]` kullanılarak tanımlanırlar.

İşte Python listelerinin temel işlemleri ve örnekleri:

### 1. Liste Oluşturma

Boş bir liste veya başlangıç değerleri olan bir liste oluşturabilirsiniz.

```python
# Boş bir liste oluşturma
bos_liste = []
print(f"Boş liste: {bos_liste}")

# Elemanlarla bir liste oluşturma
sayilar = [1, 2, 3, 4, 5]
print(f"Sayılar listesi: {sayilar}")

meyveler = ["elma", "muz", "çilek"]
print(f"Meyveler listesi: {meyveler}")

# Farklı türde elemanlar içeren bir liste
karisik_liste = [1, "merhaba", 3.14, True]
print(f"Karışık liste: {karisik_liste}")
```

### 2. Indexleme (Indexing)

Listelerdeki elemanlara konumlarına (index) göre erişebilirsiniz. Indexler 0'dan başlar. Negatif indexler listenin sonundan başlar (-1 son eleman, -2 sondan ikinci eleman vb.).

```python
meyveler = ["elma", "muz", "çilek", "portakal"]

# İlk elemana erişim (index 0)
print(f"İlk meyve: {meyveler[0]}") # Çıktı: elma

# Üçüncü elemana erişim (index 2)
print(f"Üçüncü meyve: {meyveler[2]}") # Çıktı: çilek

# Son elemana erişim (negatif index -1)
print(f"Son meyve: {meyveler[-1]}") # Çıktı: portakal

# Sondan ikinci elemana erişim (negatif index -2)
print(f"Sondan ikinci meyve: {meyveler[-2]}") # Çıktı: çilek
```

### 3. Slice (Dilimleme)

Listenin bir bölümünü (alt listesini) almak için dilimleme kullanabilirsiniz. `[başlangıç:bitiş:adım]` formatı kullanılır. `bitiş` index'i dahil değildir.

```python
sayilar = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

# İlk 3 elemanı al
print(f"İlk 3 eleman: {sayilar[0:3]}") # Çıktı: [0, 1, 2]
print(f"İlk 3 eleman (kısa yol): {sayilar[:3]}") # Çıktı: [0, 1, 2]

# 3. indexten sonuna kadar al
print(f"3. indexten sonrası: {sayilar[3:]}") # Çıktı: [3, 4, 5, 6, 7, 8, 9]

# 2. indexten 7. indexe kadar al (7 dahil değil)
print(f"2'den 7'ye kadar: {sayilar[2:7]}") # Çıktı: [2, 3, 4, 5, 6]

# Tüm listeyi kopyala
print(f"Tüm liste kopyası: {sayilar[:]}") # Çıktı: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

# Her 2 elemanı bir al
print(f"Her 2 elemanı bir: {sayilar[::2]}") # Çıktı: [0, 2, 4, 6, 8]

# Listeyi ters çevir
print(f"Ters çevrilmiş liste: {sayilar[::-1]}") # Çıktı: [9, 8, 7, 6, 5, 4, 3, 2, 1, 0]
```

### 4. `append()` Metodu

Listenin sonuna yeni bir eleman eklemek için `append()` metodunu kullanırız.

```python
meyveler = ["elma", "muz"]
print(f"Başlangıç meyveler: {meyveler}")

meyveler.append("çilek")
print(f"Çilek eklendi: {meyveler}") # Çıktı: ['elma', 'muz', 'çilek']

meyveler.append("kiraz")
print(f"Kiraz eklendi: {meyveler}") # Çıktı: ['elma', 'muz', 'çilek', 'kiraz']
```

### 5. `remove()` Metodu

Listeden belirli bir değeri kaldırmak için `remove()` metodunu kullanırız. Eğer değer listede birden fazla varsa, sadece ilk bulunanı kaldırır. Eğer değer listede yoksa `ValueError` hatası verir.

```python
urunler = ["kalem", "defter", "silgi", "kalem"]
print(f"Başlangıç ürünler: {urunler}")

urunler.remove("defter")
print(f"Defter kaldırıldı: {urunler}") # Çıktı: ['kalem', 'silgi', 'kalem']

urunler.remove("kalem") # İlk "kalem" kaldırılır
print(f"İlk kalem kaldırıldı: {urunler}") # Çıktı: ['silgi', 'kalem']

# urunler.remove("kitap") # Bu satır ValueError hatası verir çünkü "kitap" listede yok.
```

### 6. List Comprehension (Liste Anlayışı)

List comprehension, mevcut bir listeden veya başka bir yinelenebilir nesneden yeni bir liste oluşturmanın kısa ve okunabilir bir yoludur. Genellikle `for` döngüsü ve `if` koşulu içeren daha uzun kod bloklarını tek bir satıra indirger.

Genel yapısı: `[ifade for eleman in yinelenebilir_nesne if koşul]`

```python
# Örnek 1: Bir sayı listesindeki her sayının karesini al
sayilar = [1, 2, 3, 4, 5]

kareler = [sayi**2 for sayi in sayilar]
print(f"Kareler listesi: {kareler}") # Çıktı: [1, 4, 9, 16, 25]

# Örnek 2: Bir sayı listesindeki sadece çift sayıları al
sayilar = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

cift_sayilar = [sayi for sayi in sayilar if sayi % 2 == 0]
print(f"Çift sayılar listesi: {cift_sayilar}") # Çıktı: [2, 4, 6, 8, 10]

# Örnek 3: Bir metin listesindeki kelimeleri büyük harfe çevir
kelimeler = ["elma", "armut", "kiraz"]

buyuk_kelimeler = [kelime.upper() for kelime in kelimeler]
print(f"Büyük harfli kelimeler: {buyuk_kelimeler}") # Çıktı: ['ELMA', 'ARMUT', 'KİRAZ']

# Örnek 4: Hem koşul hem de dönüşüm
sayilar = [1, 2, 3, 4, 5, 6]
cift_sayilarin_kareleri = [sayi**2 for sayi in sayilar if sayi % 2 == 0]
print(f"Çift sayıların kareleri: {cift_sayilarin_kareleri}") # Çıktı: [4, 16, 36]
```

## 💡 Bağlantılar
- [[Python - Döngüler]]
- [[Python - Veri Yapıları]]

## ❓ Sorular / Anlamadıklarım


## 🔗 Kaynaklar
```
[Timestamp: 2026/05/28 04:07:48]

**user**: sen direk ekleyemiyor musun
[Context: Notes: 01 - Python/Python - Listeler.md]
[Timestamp: 2026/05/28 04:07:58]

**ai**: Hayır, ben doğrudan Obsidian notlarınıza yazma veya onları düzenleme yeteneğine sahip değilim. Ben sadece size istediğiniz içeriği markdown formatında hazırlayabilirim. Bu içeriği kopyalayıp kendi [[Python - Listeler]] notunuzdaki ilgili bölümlere yapıştırmanız gerekmektedir.

Yukarıdaki yanıtımda size notun tamamının güncellenmiş halini verdim. Bu metni kopyalayıp notunuzun içeriğiyle değiştirebilirsiniz.
[Timestamp: 2026/05/28 04:08:00]