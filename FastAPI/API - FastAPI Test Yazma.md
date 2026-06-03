---
tarih: 2026-05-28
konu: FastAPI
etiket: ["fastapi", "test", "pytest", "testclient", "unittest"]
kaynak: FastAPI Dokümantasyon
zorluk: orta
---

## 📌 Özet
FastAPI'da TestClient ile endpoint'leri pytest kullanarak kolayca test edebilirsiniz. Mock ile model bağımlılıklarını izole etmek production kalitesi için kritiktir.

## 🧠 Detay

### Kurulum
```bash
pip install pytest httpx pytest-asyncio
```

### Temel Test
```python
# tests/test_main.py
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_anasayfa():
    yanit = client.get("/")
    assert yanit.status_code == 200
    assert yanit.json()["durum"] == "aktif"

def test_saglik():
    yanit = client.get("/saglik")
    assert yanit.status_code == 200

def test_bulunamadi():
    yanit = client.get("/musteri/99999")
    assert yanit.status_code == 404
```

### ML Endpoint Testi
```python
import pytest
from unittest.mock import patch, MagicMock
import numpy as np

def test_tek_tahmin():
    yanit = client.post("/tahmin", json={
        "yas": 35,
        "gelir": 8000.0,
        "kredi_skoru": 720,
        "egitim_yili": 16,
        "is_suresi": 5.0
    })
    assert yanit.status_code == 200
    veri = yanit.json()
    assert "tahmin" in veri
    assert "olasilik" in veri
    assert veri["tahmin"] in [0, 1]
    assert 0 <= veri["olasilik"] <= 1

def test_gecersiz_giris():
    yanit = client.post("/tahmin", json={
        "yas": -5,      # geçersiz
        "gelir": 8000.0
    })
    assert yanit.status_code == 422

def test_eksik_alan():
    yanit = client.post("/tahmin", json={"yas": 35})
    assert yanit.status_code == 422
```

### Mock ile Test
```python
from unittest.mock import patch

@patch("main.ml_modeller")
def test_tahmin_mock(mock_modeller):
    # Modeli taklit et
    mock_model = MagicMock()
    mock_model.predict.return_value = np.array([1])
    mock_model.predict_proba.return_value = np.array([[0.15, 0.85]])

    mock_scaler = MagicMock()
    mock_scaler.transform.return_value = np.array([[0.5, 0.7, 0.8, 0.6, 0.4]])

    mock_modeller.__getitem__.side_effect = lambda k: {
        "model": mock_model,
        "scaler": mock_scaler
    }[k]

    yanit = client.post("/tahmin", json={
        "yas": 35, "gelir": 8000.0, "kredi_skoru": 720,
        "egitim_yili": 16, "is_suresi": 5.0
    })

    assert yanit.status_code == 200
    assert yanit.json()["tahmin"] == 1
    mock_model.predict.assert_called_once()
```

### Fixtures
```python
import pytest
from fastapi.testclient import TestClient

@pytest.fixture
def client():
    from main import app
    with TestClient(app) as c:
        yield c

@pytest.fixture
def gecerli_giris():
    return {
        "yas": 35, "gelir": 8000.0, "kredi_skoru": 720,
        "egitim_yili": 16, "is_suresi": 5.0
    }

def test_tahmin_with_fixtures(client, gecerli_giris):
    yanit = client.post("/tahmin", json=gecerli_giris)
    assert yanit.status_code == 200
```

### Batch Test
```python
def test_batch_tahmin():
    yanit = client.post("/tahmin/batch", json={
        "kayitlar": [
            {"yas": 25, "gelir": 5000.0, "kredi_skoru": 600,
             "egitim_yili": 12, "is_suresi": 2.0},
            {"yas": 45, "gelir": 15000.0, "kredi_skoru": 780,
             "egitim_yili": 20, "is_suresi": 15.0}
        ]
    })
    assert yanit.status_code == 200
    veri = yanit.json()
    assert veri["toplam"] == 2
    assert len(veri["tahminler"]) == 2
```

### Pytest Çalıştırma
```bash
pytest tests/ -v
pytest tests/ -v --cov=main --cov-report=html
pytest tests/test_main.py::test_tek_tahmin -v
```

## 💡 Bağlantılar
- [[API - FastAPI Giriş]]
- [[API - ML Modeli Servis Etmek]]
- [[API - Docker ile Deploy]]

## ❓ Sorular / Anlamadıklarım
- Async endpoint testleri nasıl yazılır?
- Integration test ile unit test ne zaman hangisi?

## 🔗 Kaynaklar
- https://fastapi.tiangolo.com/tutorial/testing/
