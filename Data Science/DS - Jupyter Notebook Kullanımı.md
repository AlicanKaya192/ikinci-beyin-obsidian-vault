---
tarih: 2026-05-28
konu: Data Science
etiket: ["data-science", "jupyter", "notebook", "araçlar"]
kaynak: Jupyter Dokümantasyon
zorluk: başlangıç
---

## 📌 Özet
Jupyter Notebook, veri bilimcilerin kod yazma, görselleştirme oluşturma ve zengin metin açıklamaları ekleme işlemlerini tek bir "yaşayan doküman" içerisinde birleştirmesine olanak tanıyan interaktif bir geliştirme ortamıdır. Projelerin sadece ham koddan ibaret kalmamasını, aynı zamanda bir hikaye anlatıcılığı ve reproducibility (tekrarlanabilirlik) sunmasını sağlar. Hücre bazlı çalışma yapısı sayesinde kod parçacıklarını bağımsız olarak test etmeye, hataları hızla ayıklamaya ve büyük veri setlerini her seferinde yeniden yüklemeden analiz etmeye imkan tanır. Veri biliminde prototipleme, eğitim materyalleri oluşturma ve karmaşık analizleri adım adım dökümante etme süreçlerinde endüstri standardı haline gelmiş vazgeçilmez bir araçtır.

## 🧠 Detay

### Jupyter Çalışma Mimarisi
```mermaid
graph LR
    A["Kullanıcı Arayüzü (Browser)"] -- "Kod Gönderir" --> B["Kernel (Python/R/Julia)"]
    B -- "İşlem Yapar" --> C["Sistem Kaynakları / Bellek"]
    C -- "Sonuç Döner" --> B
    B -- "Çıktı & Görsel Gönderir" --> A
    A -- "Durumu Kaydeder" --> D[".ipynb Dosyası (JSON)"]
```

### Kurulum ve Başlatma
```bash
pip install jupyter notebook
pip install jupyterlab   # modern arayüz

# Başlat
jupyter notebook
jupyter lab
```

### Temel Kısayollar
| Kısayol | Açıklama |
|---------|----------|
| `Shift+Enter` | Hücreyi çalıştır, sonrakine geç |
| `Ctrl+Enter` | Hücreyi çalıştır, kalmaya devam |
| `A` | Üste hücre ekle |
| `B` | Alta hücre ekle |
| `D D` | Hücre sil |
| `M` | Markdown moda geç |
| `Y` | Kod moda geç |
| `Ctrl+S` | Kaydet |

### Magic Komutlar
```python
# Zaman ölçümü
%time df.groupby("kategori").sum()
%timeit [x**2 for x in range(1000)]

# Dosya işlemleri
%ls
%pwd
%cd /path/to/dir

# Grafikleri notebook içinde göster
%matplotlib inline

# Yüklü değişkenler
%whos

# Harici script çalıştır
%run script.py
```

### Markdown Hücreleri
```markdown
# Başlık 1
## Başlık 2

**kalın**, *italik*, `kod`

- madde 1
- madde 2

| Sütun 1 | Sütun 2 |
|---------|---------|
| veri    | veri    |
```

### Notebook Paylaşma
```bash
# HTML'e dönüştür
jupyter nbconvert --to html notebook.ipynb

# PDF'e dönüştür
jupyter nbconvert --to pdf notebook.ipynb

# Tüm hücreleri çalıştırarak kaydet
jupyter nbconvert --to notebook --execute notebook.ipynb
```

### nbextensions
```bash
pip install jupyter_contrib_nbextensions
jupyter contrib nbextension install --user
# Table of Contents, Code Folding gibi eklentiler
```

## 💡 Bağlantılar
- [[DS - EDA - Keşifsel Veri Analizi]]
- [[Python - Virtual Environment]]
- [[DS - Matplotlib Temel Grafikler]]

## ❓ Sorular / Anlamadıklarım
- JupyterLab ile Jupyter Notebook arasındaki fark nedir?
- Büyük notebook'lar yavaşlarsa ne yapmalıyım?

## 🔗 Kaynaklar
- https://jupyter.org/documentation
