---
tarih: 2026-05-28
konu: Data Science
etiket: ["data-science", "api", "web-scraping", "veri-toplama"]
kaynak: 
zorluk: orta
---

## 📌 Özet
Veri bilimciler sıklıkla dış kaynaklardan veri toplar. API'lar yapılandırılmış veri sunarken, web scraping ham HTML'den veri çeker.

## 🧠 Detay

### requests ile API Kullanımı
```python
import requests
import pandas as pd

# GET isteği
yanit = requests.get(
    "https://api.example.com/veri",
    params={"limit": 100, "offset": 0},
    headers={"Authorization": "Bearer TOKEN"}
)

print(yanit.status_code)   # 200 → başarılı
veri = yanit.json()
df = pd.DataFrame(veri["results"])
```

### Sayfalama (Pagination)
```python
tum_veri = []
sayfa = 1

while True:
    yanit = requests.get(
        "https://api.example.com/veri",
        params={"page": sayfa, "per_page": 100}
    )
    veri = yanit.json()

    if not veri["data"]:
        break

    tum_veri.extend(veri["data"])
    sayfa += 1

df = pd.DataFrame(tum_veri)
```

### BeautifulSoup ile Web Scraping
```python
from bs4 import BeautifulSoup
import requests

yanit = requests.get("https://example.com")
soup = BeautifulSoup(yanit.content, "html.parser")

# Element bul
baslik = soup.find("h1").text
tablolar = soup.find_all("table")

# Tablo → DataFrame
df = pd.read_html(str(tablolar[0]))[0]
```

### Selenium ile Dinamik Sayfalar
```python
from selenium import webdriver
from selenium.webdriver.common.by import By

driver = webdriver.Chrome()
driver.get("https://example.com")

eleman = driver.find_element(By.ID, "buton")
eleman.click()

icerik = driver.page_source
driver.quit()
```

### Rate Limiting ve Etik Kullanım
```python
import time

for url in url_listesi:
    yanit = requests.get(url)
    # işle
    time.sleep(1)  # 1 saniye bekle, sunucuyu zorlaма
```

## 💡 Bağlantılar
- [[Python - JSON İşlemleri]]
- [[DS - Pandas Veri Okuma ve Yazma]]

## ❓ Sorular / Anlamadıklarım
- API rate limit aşılırsa ne yapmalıyım?
- robots.txt nedir, neden önemli?

## 🔗 Kaynaklar
- https://docs.python-requests.org
- https://www.crummy.com/software/BeautifulSoup/bs4/doc/
