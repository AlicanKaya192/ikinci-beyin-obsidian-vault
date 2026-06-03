---
tarih: 2026-05-28
konu: MLOps
etiket: ["mlops", "wandb", "weights-biases", "deney-takibi", "sweep"]
kaynak: W&B Dokümantasyon
zorluk: orta
---

## 📌 Özet
Weights & Biases (W&B), deney takibi, model görselleştirme ve hiperparametre optimizasyonu için gelişmiş bir platform. MLflow'a güçlü bir alternatif, özellikle derin öğrenme projelerinde yaygın.

## 🧠 Detay

### Kurulum
```bash
pip install wandb
wandb login  # API key gir
```

### Temel Kullanım
```python
import wandb
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score, f1_score

# Run başlat
wandb.init(
    project="kredi-onay-modeli",
    name="rf-deney-1",
    config={
        "n_estimators": 100,
        "max_depth": 5,
        "min_samples_leaf": 10,
        "random_state": 42
    },
    tags=["random-forest", "baseline"]
)

config = wandb.config

# Eğitim
model = RandomForestClassifier(
    n_estimators=config.n_estimators,
    max_depth=config.max_depth,
    random_state=config.random_state
)
model.fit(X_train, y_train)

# Metrik logla
for epoch in range(10):
    wandb.log({
        "train_accuracy": accuracy_score(y_train, model.predict(X_train)),
        "val_accuracy": accuracy_score(y_val, model.predict(X_val)),
        "epoch": epoch
    })

# Final metrikler
wandb.log({
    "test_accuracy": accuracy_score(y_test, model.predict(X_test)),
    "test_f1": f1_score(y_test, model.predict(X_test))
})

# Model kaydet
wandb.sklearn.plot_classifier(model, X_train, X_test, y_train, y_test,
    model.predict(X_test), model.predict_proba(X_test),
    labels=["Ret", "Onay"])

wandb.finish()
```

### Grafik ve Artifact Loglama
```python
import matplotlib.pyplot as plt
import numpy as np

# Confusion matrix
wandb.log({"confusion_matrix": wandb.sklearn.plot_confusion_matrix(
    y_test, model.predict(X_test), ["Ret", "Onay"]
)})

# Özellik önemi grafiği
fig, ax = plt.subplots(figsize=(10, 6))
onem = pd.Series(model.feature_importances_, index=X.columns)
onem.sort_values().plot(kind="barh", ax=ax)
wandb.log({"ozellik_onemi": wandb.Image(fig)})
plt.close()

# Model artifact
artifact = wandb.Artifact("kredi-modeli", type="model")
artifact.add_file("model/rf_model.pkl")
wandb.log_artifact(artifact)
```

### W&B Sweep (Hiperparametre Optimizasyonu)
```python
# sweep_config.yaml
sweep_config = {
    "method": "bayes",   # random, grid, bayes
    "metric": {"name": "val_accuracy", "goal": "maximize"},
    "parameters": {
        "n_estimators": {"values": [50, 100, 200, 300]},
        "max_depth": {"min": 3, "max": 15},
        "min_samples_leaf": {"min": 1, "max": 20},
        "learning_rate": {
            "distribution": "log_uniform_values",
            "min": 0.001, "max": 0.1
        }
    }
}

def egitim():
    with wandb.init() as run:
        config = run.config

        model = RandomForestClassifier(
            n_estimators=config.n_estimators,
            max_depth=config.max_depth,
            min_samples_leaf=config.min_samples_leaf
        )
        model.fit(X_train, y_train)

        wandb.log({
            "val_accuracy": accuracy_score(y_val, model.predict(X_val)),
            "val_f1": f1_score(y_val, model.predict(X_val))
        })

# Sweep başlat
sweep_id = wandb.sweep(sweep_config, project="kredi-onay-modeli")
wandb.agent(sweep_id, function=egitim, count=50)  # 50 deneme
```

### Model Registry
```python
# Artifact kaydet
artifact = wandb.Artifact(
    "kredi-onay-modeli",
    type="model",
    metadata={"accuracy": 0.92, "f1": 0.89}
)
artifact.add_file("model/rf_model.pkl")
wandb.log_artifact(artifact)

# Production'a taşı
artifact.link("model-registry/kredi-onay-modeli", aliases=["production"])

# Kullan
api = wandb.Api()
artifact = api.artifact("kullanici/proje/kredi-onay-modeli:production")
artifact.download("model/")
```

### MLflow vs W&B
| Özellik | MLflow | W&B |
|---------|--------|-----|
| Self-hosted | ✅ Kolay | ✅ Zor |
| UI | Temel | Gelişmiş |
| Sweep | ❌ Yok | ✅ Güçlü |
| Deep learning | Orta | Mükemmel |
| Fiyat | Ücretsiz | Freemium |

## 💡 Bağlantılar
- [[MLOps - MLflow ile Deney Takibi]]
- [[MLOps - Giriş ve Temel Kavramlar]]
- [[ML - Hiperparametre Optimizasyonu]]

## ❓ Sorular / Anlamadıklarım
- MLflow ile W&B aynı anda kullanılabilir mi?
- Sweep'te Bayesian optimizasyon neden random'dan daha iyi?

## 🔗 Kaynaklar
- https://docs.wandb.ai/
