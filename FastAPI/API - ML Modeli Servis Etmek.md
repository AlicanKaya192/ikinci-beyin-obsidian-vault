---
tarih: 2026-05-28
konu: FastAPI
etiket: ["fastapi", "ml", "model-serving", "scikit-learn", "pickle"]
kaynak: FastAPI Dokümantasyon
zorluk: orta
---

## 📌 Özet
Eğitilmiş ML modellerini FastAPI ile servis etmek, modeli production'a almanın en yaygın yoludur. Model yükleme, tahmin ve batch işlem adımlarını kapsar.

## 🧠 Detay

### Model Kaydetme ve Yükleme
```python
import pickle
import joblib
from sklearn.ensemble import RandomForestClassifier

# Model eğit ve kaydet
model = RandomForestClassifier()
model.fit(X_train, y_train)

# joblib (büyük modeller için daha hızlı)
joblib.dump(model, "model/rf_model.pkl")
joblib.dump(scaler, "model/scaler.pkl")

# pickle
with open("model/model.pkl", "wb") as f:
    pickle.dump(model, f)
```

### Startup'ta Model Yükleme
```python
from fastapi import FastAPI
from contextlib import asynccontextmanager
import joblib
import numpy as np
from pydantic import BaseModel
from typing import List

# Global model nesnesi
ml_modeller = {}

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Uygulama başlarken yükle
    ml_modeller["model"] = joblib.load("model/rf_model.pkl")
    ml_modeller["scaler"] = joblib.load("model/scaler.pkl")
    ml_modeller["label_encoder"] = joblib.load("model/label_encoder.pkl")
    print("✅ Model yüklendi")
    yield
    # Uygulama kapanırken temizle
    ml_modeller.clear()
    print("Model temizlendi")

app = FastAPI(lifespan=lifespan)
```

### Tek Tahmin Endpoint'i
```python
class TahminGiris(BaseModel):
    yas: int
    gelir: float
    kredi_skoru: int
    egitim_yili: int
    is_suresi: float

class TahminCikis(BaseModel):
    tahmin: int
    karar: str
    olasilik: float
    model_versiyonu: str = "1.0.0"

@app.post("/tahmin", response_model=TahminCikis)
def tek_tahmin(giris: TahminGiris):
    # Veriyi array'e çevir
    veri = np.array([[
        giris.yas,
        giris.gelir,
        giris.kredi_skoru,
        giris.egitim_yili,
        giris.is_suresi
    ]])

    # Ölçekle
    veri_scaled = ml_modeller["scaler"].transform(veri)

    # Tahmin
    tahmin = ml_modeller["model"].predict(veri_scaled)[0]
    olasilik = ml_modeller["model"].predict_proba(veri_scaled)[0].max()

    return TahminCikis(
        tahmin=int(tahmin),
        karar="Onaylandı" if tahmin == 1 else "Reddedildi",
        olasilik=round(float(olasilik), 4)
    )
```

### Batch Tahmin Endpoint'i
```python
class BatchGiris(BaseModel):
    kayitlar: List[TahminGiris]

class BatchCikis(BaseModel):
    tahminler: List[TahminCikis]
    toplam: int
    sure_ms: float

import time

@app.post("/tahmin/batch", response_model=BatchCikis)
def batch_tahmin(giris: BatchGiris):
    baslangic = time.time()

    veri = np.array([[
        k.yas, k.gelir, k.kredi_skoru,
        k.egitim_yili, k.is_suresi
    ] for k in giris.kayitlar])

    veri_scaled = ml_modeller["scaler"].transform(veri)
    tahminler = ml_modeller["model"].predict(veri_scaled)
    olasiliklar = ml_modeller["model"].predict_proba(veri_scaled).max(axis=1)

    sonuclar = [
        TahminCikis(
            tahmin=int(t),
            karar="Onaylandı" if t == 1 else "Reddedildi",
            olasilik=round(float(o), 4)
        )
        for t, o in zip(tahminler, olasiliklar)
    ]

    sure = (time.time() - baslangic) * 1000

    return BatchCikis(
        tahminler=sonuclar,
        toplam=len(sonuclar),
        sure_ms=round(sure, 2)
    )
```

### Model Bilgi Endpoint'i
```python
@app.get("/model/bilgi")
def model_bilgi():
    model = ml_modeller["model"]
    return {
        "model_tipi": type(model).__name__,
        "n_estimators": getattr(model, "n_estimators", None),
        "ozellikler": ["yas", "gelir", "kredi_skoru", "egitim_yili", "is_suresi"],
        "siniflar": model.classes_.tolist(),
        "versiyon": "1.0.0"
    }
```

## 💡 Bağlantılar
- [[API - FastAPI Giriş]]
- [[API - Pydantic ile Veri Doğrulama]]
- [[API - FastAPI Hata Yönetimi]]
- [[API - Docker ile Deploy]]
- [[ML - Random Forest]]

## ❓ Sorular / Anlamadıklarım
- Büyük modeller (>500MB) için daha iyi yükleme stratejisi?
- Birden fazla model versiyonunu aynı anda nasıl servis ederim?

## 🔗 Kaynaklar
- https://fastapi.tiangolo.com/advanced/events/
