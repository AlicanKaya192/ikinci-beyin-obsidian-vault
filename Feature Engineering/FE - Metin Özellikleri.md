---
tarih: 2025-01-01
konu: Metin Özellikleri, TF-IDF, Bag of Words, Word Embeddings, NLP
etiket: [feature-engineering, metin, NLP, TF-IDF, word2vec, embedding, bag-of-words]
kaynak:
zorluk: ⭐⭐⭐
---

## 📌 Özet

Ham metin sayısal vektöre dönüştürülmelidir. Basit yöntemlerden (BoW, TF-IDF) modern derin öğrenme embedding'lerine giden geniş bir araç seti mevcuttur.

---

## 🧠 Detay

### Temel Metin İstatistikleri

```python
df['metin'] = df['metin'].astype(str)

# Uzunluk özellikleri
df['karakter_sayisi']   = df['metin'].str.len()
df['kelime_sayisi']     = df['metin'].str.split().str.len()
df['cumle_sayisi']      = df['metin'].str.count(r'[.!?]+')
df['benzersiz_kelime']  = df['metin'].apply(lambda x: len(set(x.split())))

# Oran özellikleri
df['ort_kelime_uzunlugu'] = df['metin'].apply(
    lambda x: np.mean([len(w) for w in x.split()]) if x.split() else 0
)
df['leksik_zenginlik'] = df['benzersiz_kelime'] / df['kelime_sayisi'].clip(lower=1)

# Noktalama ve büyük harf
df['noktalama_sayisi']  = df['metin'].str.count(r'[.,!?;:]')
df['buyuk_harf_sayisi'] = df['metin'].str.count(r'[A-ZÇŞĞÜÖİ]')
df['rakam_sayisi']      = df['metin'].str.count(r'\d')

# Özel içerik
df['url_var_mi']    = df['metin'].str.contains(r'http|www').astype(int)
df['email_var_mi']  = df['metin'].str.contains(r'@').astype(int)
df['emoji_sayisi']  = df['metin'].str.count(r'[^\x00-\x7F]')
```

---

### Metin Ön İşleme

```python
import re
import string
from nltk.corpus import stopwords
from nltk.stem import PorterStemmer, WordNetLemmatizer
import nltk

nltk.download(['stopwords', 'wordnet', 'punkt'])

stop_words = set(stopwords.words('turkish'))  # veya 'english'
stemmer = PorterStemmer()
lemmatizer = WordNetLemmatizer()

def preprocess(text):
    # Küçük harf
    text = text.lower()
    # URL kaldır
    text = re.sub(r'http\S+|www\S+', '', text)
    # Noktalama kaldır
    text = text.translate(str.maketrans('', '', string.punctuation))
    # Rakam kaldır
    text = re.sub(r'\d+', '', text)
    # Boşlukları düzelt
    text = re.sub(r'\s+', ' ', text).strip()
    # Stopwords kaldır
    tokens = text.split()
    tokens = [t for t in tokens if t not in stop_words]
    # Lemmatize
    tokens = [lemmatizer.lemmatize(t) for t in tokens]
    return ' '.join(tokens)

df['temiz_metin'] = df['metin'].apply(preprocess)
```

---

### 1. Bag of Words (BoW)

```python
from sklearn.feature_extraction.text import CountVectorizer

cv = CountVectorizer(
    max_features=5000,      # En sık 5000 kelime
    min_df=2,               # En az 2 belgede geçsin
    max_df=0.95,            # En fazla %95 belgede geçsin
    ngram_range=(1, 2),     # Unigram + Bigram
    analyzer='word'
)

X_train_bow = cv.fit_transform(train['temiz_metin'])
X_test_bow  = cv.transform(test['temiz_metin'])

# Sözlük
print(cv.vocabulary_)
print(cv.get_feature_names_out()[:20])
```

---

### 2. TF-IDF ⭐

$$TF\text{-}IDF(t,d) = TF(t,d) \times \log\frac{N}{df(t)}$$

```python
from sklearn.feature_extraction.text import TfidfVectorizer

tfidf = TfidfVectorizer(
    max_features=10000,
    ngram_range=(1, 2),
    min_df=2,
    max_df=0.95,
    sublinear_tf=True       # log(TF+1) — büyük frekansları bastır
)

X_train_tfidf = tfidf.fit_transform(train['temiz_metin'])
X_test_tfidf  = tfidf.transform(test['temiz_metin'])

# En önemli kelimeler
feature_names = tfidf.get_feature_names_out()
tfidf_means = X_train_tfidf.mean(axis=0).A1
top_words = pd.Series(tfidf_means, index=feature_names).nlargest(20)
```

---

### 3. Word2Vec Embeddings

```python
from gensim.models import Word2Vec

# Model eğit
sentences = [text.split() for text in df['temiz_metin']]
w2v = Word2Vec(sentences, vector_size=100, window=5,
               min_count=2, workers=4, epochs=10)

# Kelime vektörü
w2v.wv['bilgisayar']

# Belge vektörü (kelime vektörlerinin ortalaması)
def doc_vector(text, model):
    words = [w for w in text.split() if w in model.wv]
    if not words:
        return np.zeros(model.vector_size)
    return np.mean([model.wv[w] for w in words], axis=0)

df['w2v_vec'] = df['temiz_metin'].apply(lambda x: doc_vector(x, w2v))
X_w2v = np.vstack(df['w2v_vec'].values)
```

---

### 4. Pretrained Embeddings (Sentence Transformers)

```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('paraphrase-multilingual-MiniLM-L12-v2')
# Türkçe dahil çok dilli destek

embeddings = model.encode(
    df['metin'].tolist(),
    batch_size=64,
    show_progress_bar=True
)
# Shape: (n_samples, 384)

# DataFrame'e ekle
embed_df = pd.DataFrame(embeddings, columns=[f'emb_{i}' for i in range(384)])
df = pd.concat([df, embed_df], axis=1)
```

---

### 5. N-gram Özellikleri

```python
# Karakter n-gram (yazım hataları için dayanıklı)
tfidf_char = TfidfVectorizer(
    analyzer='char_wb',
    ngram_range=(3, 5),
    max_features=5000
)
X_char = tfidf_char.fit_transform(df['metin'])

# Kelime n-gram (bigram, trigram)
tfidf_ngram = TfidfVectorizer(
    analyzer='word',
    ngram_range=(2, 3),
    max_features=5000
)
```

---

### 6. Duygu Analizi Skoru

```python
from textblob import TextBlob

df['duygu_skoru'] = df['metin'].apply(
    lambda x: TextBlob(x).sentiment.polarity
)
df['oznel_mi'] = df['metin'].apply(
    lambda x: TextBlob(x).sentiment.subjectivity
)

# Türkçe için
from transformers import pipeline
sentiment = pipeline('sentiment-analysis',
                     model='savasy/bert-base-turkish-sentiment-cased')
df['sentiment'] = df['metin'].apply(
    lambda x: sentiment(x[:512])[0]['label']
)
```

---

### Boyut İndirgeme (Seyrek Matris Sonrası)

```python
from sklearn.decomposition import TruncatedSVD  # LSA

# TF-IDF → LSA → 100 boyut
svd = TruncatedSVD(n_components=100, random_state=42)
X_lsa = svd.fit_transform(X_train_tfidf)

# PCA (dense için)
# NMF (non-negative matrix factorization)
from sklearn.decomposition import NMF
nmf = NMF(n_components=50, random_state=42)
X_nmf = nmf.fit_transform(X_train_tfidf)
```

---

### Hangi Yöntem Ne Zaman?

| Durum | Yöntem |
|---|---|
| Küçük veri, yorumlanabilirlik | BoW + TF-IDF |
| Kelime ilişkileri önemli | Word2Vec / GloVe |
| Anlam-tabanlı benzerlik | Sentence Transformers |
| Yazım hataları var | Karakter n-gram |
| Çok dilli | Multilingual BERT |
| Duygu analizi | TextBlob / Transformer |

---

## 💡 Bağlantılar
- [[FE - Özellik Türetme]]
- [[FE - Özellik Seçimi Yöntemleri]]
- [[ML - Doğal Dil İşleme Temelleri]]

## ❓ Sorular / Anlamadıklarım


## 🔗 Kaynaklar
- sklearn.feature_extraction.text Documentation
- Sentence-Transformers Documentation
- Gensim Word2Vec Tutorial
