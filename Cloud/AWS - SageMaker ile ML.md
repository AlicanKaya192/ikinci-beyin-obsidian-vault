---
tarih: 2026-05-28
konu: Cloud
etiket: ["aws", "sagemaker", "ml", "eğitim", "deploy", "endpoint"]
kaynak: AWS SageMaker Dokümantasyon
zorluk: orta
---

## 📌 Özet
Amazon SageMaker, AWS'nin tam yönetilen ML platformudur. Azure ML'e karşılık gelir. Eğitim, optimizasyon, deploy ve izleme adımlarını otomatikleştirir.

## 🧠 Detay

### Temel Kavramlar
```
Training Job    → Eğitim çalışması
Endpoint        → Gerçek zamanlı tahmin servisi
Batch Transform → Toplu tahmin
Model Registry  → Model versiyonlama
Pipeline        → ML iş akışı
Studio          → Entegre geliştirme ortamı
Experiments     → Deney takibi
```

### Kurulum
```bash
pip install sagemaker boto3
```

### Eğitim Job'u
```python
import sagemaker
from sagemaker.sklearn import SKLearn

# Session
session = sagemaker.Session()
role = "arn:aws:iam::ACCOUNT_ID:role/SageMakerRole"
bucket = "ml-veri-golum"

# train.py (eğitim script'i)
"""
import argparse, os, joblib
from sklearn.ensemble import RandomForestClassifier

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--n_estimators", type=int, default=100)
    parser.add_argument("--max_depth", type=int, default=5)
    parser.add_argument("--model_dir", type=str, default=os.environ["SM_MODEL_DIR"])
    parser.add_argument("--train", type=str, default=os.environ["SM_CHANNEL_TRAIN"])
    args = parser.parse_args()

    import pandas as pd
    df = pd.read_csv(f"{args.train}/egitim.csv")
    X, y = df.drop("hedef", axis=1), df["hedef"]

    model = RandomForestClassifier(n_estimators=args.n_estimators, max_depth=args.max_depth)
    model.fit(X, y)

    joblib.dump(model, f"{args.model_dir}/rf_model.pkl")
"""

# SageMaker estimator
sklearn_estimator = SKLearn(
    entry_point="train.py",
    role=role,
    instance_type="ml.m5.xlarge",
    instance_count=1,
    framework_version="1.2-1",
    hyperparameters={
        "n_estimators": 200,
        "max_depth": 10
    }
)

# Eğitimi başlat
sklearn_estimator.fit({
    "train": f"s3://{bucket}/data/egitim/",
    "test": f"s3://{bucket}/data/test/"
})
```

### Hyperparameter Tuning (HPO)
```python
from sagemaker.tuner import HyperparameterTuner, IntegerParameter, ContinuousParameter

tuner = HyperparameterTuner(
    estimator=sklearn_estimator,
    objective_metric_name="validation:accuracy",
    hyperparameter_ranges={
        "n_estimators": IntegerParameter(50, 300),
        "max_depth": IntegerParameter(3, 15),
        "min_samples_leaf": IntegerParameter(1, 20)
    },
    max_jobs=20,
    max_parallel_jobs=4,
    strategy="Bayesian"
)

tuner.fit({"train": f"s3://{bucket}/data/egitim/"})
print(f"En iyi parametreler: {tuner.best_estimator()}")
```

### Model Deploy (Endpoint)
```python
# Eğitimden deploy
predictor = sklearn_estimator.deploy(
    initial_instance_count=1,
    instance_type="ml.t2.medium",
    endpoint_name="kredi-onay-endpoint"
)

# Tahmin
import numpy as np
tahmin = predictor.predict(np.array([[35, 8000, 720, 16, 5.0]]))
print(tahmin)

# Endpoint sil (masraf önle!)
predictor.delete_endpoint()
```

### Batch Transform (Toplu Tahmin)
```python
transformer = sklearn_estimator.transformer(
    instance_count=1,
    instance_type="ml.m5.large",
    output_path=f"s3://{bucket}/tahminler/"
)

transformer.transform(
    data=f"s3://{bucket}/data/test/test.csv",
    content_type="text/csv",
    split_type="Line"
)
transformer.wait()
```

### SageMaker Experiments
```python
from sagemaker.experiments.run import Run

with Run(
    experiment_name="kredi-onay-deneyleri",
    run_name="rf-deney-v2",
    sagemaker_session=session
) as run:
    run.log_parameter("n_estimators", 200)
    run.log_parameter("max_depth", 10)

    # Eğitim kodu...
    run.log_metric("accuracy", 0.92)
    run.log_metric("f1_score", 0.89)
    run.log_artifact("model/rf_model.pkl", name="model")
```

### Model Registry
```python
from sagemaker.model import Model
from sagemaker import ModelPackage

# Modeli kaydet
model_package = sklearn_estimator.register(
    model_package_group_name="KrediOnayModelleri",
    approval_status="Approved",
    description="RF modeli - v2.0"
)

# Onaylı modeli deploy et
approved_model = ModelPackage(
    role=role,
    model_package_arn=model_package.model_package_arn,
    sagemaker_session=session
)
approved_model.deploy(
    initial_instance_count=1,
    instance_type="ml.t2.medium"
)
```

### SageMaker Pipeline
```python
from sagemaker.workflow.pipeline import Pipeline
from sagemaker.workflow.steps import TrainingStep, ProcessingStep
from sagemaker.workflow.parameters import ParameterInteger

n_estimators_param = ParameterInteger(name="NEstimators", default_value=100)

egitim_adimi = TrainingStep(
    name="ModelEgit",
    estimator=sklearn_estimator,
    inputs={"train": f"s3://{bucket}/data/egitim/"}
)

pipeline = Pipeline(
    name="KrediOnayPipeline",
    parameters=[n_estimators_param],
    steps=[egitim_adimi]
)

pipeline.upsert(role_arn=role)
execution = pipeline.start(parameters={"NEstimators": 200})
execution.wait()
```

## 💡 Bağlantılar
- [[Cloud - Bulut Bilişime Giriş]]
- [[AWS - S3 ile Veri Depolama]]
- [[AWS - Lambda ile Serverless]]
- [[MLOps - MLflow ile Deney Takibi]]

## ❓ Sorular / Anlamadıklarım
- SageMaker vs Azure ML ne zaman hangisi?
- Spot instance ile eğitim güvenli mi?

## 🔗 Kaynaklar
- https://docs.aws.amazon.com/sagemaker/
