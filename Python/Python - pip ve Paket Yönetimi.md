---
tarih: 2026-05-28
konu: Python
etiket: ["python", "pip", "paket", "araçlar"]
kaynak: Python Resmi Dokümantasyon
zorluk: başlangıç
---

## 📌 Özet
`pip`, Python'un paket yöneticisidir. PyPI (Python Package Index) üzerindeki binlerce kütüphaneyi yüklemek, güncellemek ve kaldırmak için kullanılır.

## 🧠 Detay

### Temel pip Komutları
```bash
# Paket kur
pip install numpy

# Belirli versiyon
pip install numpy==1.24.0
pip install numpy>=1.20

# Güncelle
pip install --upgrade numpy

# Kaldır
pip uninstall numpy

# Bilgi göster
pip show numpy

# Arama
pip search numpy   # (kısıtlı)
```

### requirements.txt
```bash
# Oluştur
pip freeze > requirements.txt

# Kur
pip install -r requirements.txt
```

```
# requirements.txt örneği
numpy==1.24.0
pandas==2.0.1
matplotlib==3.7.1
scikit-learn==1.3.0
```

### Data Science için Temel Paketler
```bash
pip install numpy pandas matplotlib seaborn scikit-learn jupyter
```

### pip Alternatifi: conda
```bash
conda install numpy
conda create -n myenv python=3.11
conda activate myenv
```

### Paket Versiyonlarını Kontrol Et
```python
import numpy
print(numpy.__version__)

import pkg_resources
print(pkg_resources.get_distribution("numpy").version)
```

## 💡 Bağlantılar
- [[Python - Virtual Environment]]
- [[Python - Modüller ve Paketler]]

## ❓ Sorular / Anlamadıklarım
- `pip` ile `pip3` arasındaki fark nedir?
- `conda` ne zaman `pip`'den daha iyi?

## 🔗 Kaynaklar
- https://pip.pypa.io/en/stable/
- https://pypi.org
