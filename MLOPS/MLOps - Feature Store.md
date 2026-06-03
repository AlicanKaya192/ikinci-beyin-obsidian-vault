---
tarih: 2026-05-28
konu: MLOps
etiket: ["mlops", "feature-store", "feast", "özellik-mühendisliği", "tekrar-kullanım"]
kaynak: Feast Dokümantasyon
zorluk: orta
---

## 📌 Özet
Feature Store, ML özelliklerini merkezi olarak depolayan, yöneten ve servis eden sistemdir. Eğitim ve tahmin sırasında aynı özelliklerin kullanılmasını garantiler (training-serving skew önler).

## 🧠 Detay

### Feature Store Neden Gerekli?
```
Problemler:
  - Her ekip aynı özellikleri yeniden hesaplıyor
  - Eğitim ve production'da farklı özellik hesabı
  - Özellik tutarsızlıkları → model hataları
  - Point-in-time correctness eksikliği

Feature Store Çözümleri:
  - Özellikler bir kez hesaplanır, tekrar kullanılır
  - Online (gerçek zamanlı) + Offline (batch) depolama
  - Point-in-time doğru özellik getirme
```

### Feast ile Feature Store
```bash
pip install feast
feast init ozellik-deposu
cd ozellik-deposu
```

### Özellik Tanımlama
```python
# feature_repo/ozellikler.py
from datetime import timedelta
from feast import Entity, Feature, FeatureView, FileSource, ValueType

# Entity (kimlik)
musteri = Entity(
    name="musteri_id",
    value_type=ValueType.INT64,
    description="Müşteri ID"
)

# Veri kaynağı
musteri_istatistik_kaynak = FileSource(
    path="data/musteri_istatistik.parquet",
    event_timestamp_column="event_timestamp"
)

# Feature View
musteri_istatistik = FeatureView(
    name="musteri_istatistik",
    entities=["musteri_id"],
    ttl=timedelta(days=30),
    features=[
        Feature(name="son_30_gun_siparis", dtype=ValueType.INT64),
        Feature(name="toplam_harcama", dtype=ValueType.FLOAT),
        Feature(name="ortalama_siparis_tutari", dtype=ValueType.FLOAT),
        Feature(name="aktif_gun_sayisi", dtype=ValueType.INT64),
    ],
    source=musteri_istatistik_kaynak
)
```

### Feature Store Kullanımı
```python
from feast import FeatureStore
import pandas as pd

store = FeatureStore(repo_path="feature_repo/")

# Materialize (özellikleri online store'a yükle)
store.materialize_incremental(end_date=datetime.now())

# Eğitim için özellik getir (offline)
entity_df = pd.DataFrame({
    "musteri_id": [1, 2, 3, 4, 5],
    "event_timestamp": [datetime.now()] * 5
})

egitim_df = store.get_historical_features(
    entity_df=entity_df,
    features=[
        "musteri_istatistik:son_30_gun_siparis",
        "musteri_istatistik:toplam_harcama",
        "musteri_istatistik:ortalama_siparis_tutari"
    ]
).to_df()

# Production için özellik getir (online - hızlı)
online_ozellikler = store.get_online_features(
    features=[
        "musteri_istatistik:son_30_gun_siparis",
        "musteri_istatistik:toplam_harcama"
    ],
    entity_rows=[{"musteri_id": 42}]
).to_dict()
```

### Training-Serving Skew Önleme
```python
# Hem eğitimde hem serviste aynı Feature Store kullan

# Eğitim
egitim_verileri = store.get_historical_features(
    entity_df=entity_df, features=OZELLIKLER
).to_df()
model.fit(egitim_verileri[OZELLIK_ISIMLERI], y)

# Production (FastAPI endpoint'i)
@app.post("/tahmin")
def tahmin_yap(musteri_id: int):
    ozellikler = store.get_online_features(
        features=OZELLIKLER,
        entity_rows=[{"musteri_id": musteri_id}]
    ).to_dict()

    X = pd.DataFrame(ozellikler)
    tahmin = model.predict(X)[0]
    return {"tahmin": int(tahmin)}
```

### Basit Feature Store (Pandas + Redis)
```python
import redis
import json
import pandas as pd

r = redis.Redis(host="localhost", port=6379)

def ozellikleri_kaydet(musteri_id: int, ozellikler: dict):
    key = f"musteri:{musteri_id}:ozellikler"
    r.setex(key, 3600, json.dumps(ozellikler))  # 1 saat TTL

def ozellikleri_getir(musteri_id: int) -> dict:
    key = f"musteri:{musteri_id}:ozellikler"
    veri = r.get(key)
    return json.loads(veri) if veri else None
```

## 💡 Bağlantılar
- [[MLOps - Giriş ve Temel Kavramlar]]
- [[MLOps - MLflow ile Deney Takibi]]
- [[DS - Veri Dönüşümleri]]
- [[ML - Veri Ön İşleme Pipeline]]

## ❓ Sorular / Anlamadıklarım
- Point-in-time correctness neden kritik?
- Feast yerine ne zaman Redis direkt kullanılır?

## 🔗 Kaynaklar
- https://docs.feast.dev/
