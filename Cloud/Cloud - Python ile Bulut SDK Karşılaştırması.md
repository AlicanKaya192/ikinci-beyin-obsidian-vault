---
tarih: 2026-05-28
konu: Cloud
etiket: ["cloud", "boto3", "azure-sdk", "python", "sdk", "karşılaştırma"]
kaynak: 
zorluk: başlangıç
---

## 📌 Özet
AWS ve Azure, Python SDK'ları üzerinden programatik olarak kontrol edilir. Boto3 (AWS) ve azure-sdk (Azure) en temel araçlardır. Ortak iş akışları için hızlı başvuru kaynağı.

## 🧠 Detay

### Kurulum
```bash
# AWS
pip install boto3

# Azure
pip install azure-storage-blob azure-identity azure-ai-ml azure-keyvault-secrets
```

### Kimlik Doğrulama

```python
# AWS - ortam değişkenleri (önerilen)
# AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_DEFAULT_REGION
import boto3
s3 = boto3.client("s3")

# AWS - profil
session = boto3.Session(profile_name="ml-project")
s3 = session.client("s3")

# Azure - DefaultAzureCredential (sırayla dener)
from azure.identity import DefaultAzureCredential
# 1. Environment variables
# 2. Managed Identity
# 3. Azure CLI
# 4. VS Code
credential = DefaultAzureCredential()
```

### Depolama Karşılaştırması

```python
import boto3
import pandas as pd
from io import BytesIO, StringIO

# ===== AWS S3 =====
s3 = boto3.client("s3")

# Yükle
s3.upload_file("yerel.csv", "my-bucket", "data/yerel.csv")

# İndir
obj = s3.get_object(Bucket="my-bucket", Key="data/egitim.parquet")
df_aws = pd.read_parquet(BytesIO(obj["Body"].read()))

# Listele
objects = s3.list_objects_v2(Bucket="my-bucket", Prefix="data/")["Contents"]

# ===== Azure Blob =====
from azure.storage.blob import BlobServiceClient
blob_client = BlobServiceClient.from_connection_string(CONN_STR)

# Yükle
container = blob_client.get_container_client("my-container")
with open("yerel.csv", "rb") as f:
    container.upload_blob("data/yerel.csv", f, overwrite=True)

# İndir
blob = blob_client.get_blob_client("my-container", "data/egitim.parquet")
df_azure = pd.read_parquet(BytesIO(blob.download_blob().readall()))

# Listele
blobs = container.list_blobs(name_starts_with="data/")
```

### Veritabanı Karşılaştırması
```python
import pandas as pd
from sqlalchemy import create_engine

# ===== AWS RDS PostgreSQL =====
engine_aws = create_engine(
    "postgresql://user:pass@rds-endpoint:5432/mldb"
)
df = pd.read_sql("SELECT * FROM tahmin_loglar", engine_aws)

# ===== Azure SQL =====
import pyodbc, urllib
params = urllib.parse.quote_plus(
    "DRIVER={ODBC Driver 18 for SQL Server};"
    "SERVER=server.database.windows.net;DATABASE=mldb;"
    "UID=admin;PWD=Sifre123!;Encrypt=yes;"
)
engine_azure = create_engine(f"mssql+pyodbc:///?odbc_connect={params}")
df = pd.read_sql("SELECT * FROM TahminLoglar", engine_azure)
```

### Secret Yönetimi
```python
# ===== AWS Secrets Manager =====
import boto3, json
secrets = boto3.client("secretsmanager")
secret = json.loads(secrets.get_secret_value(SecretId="prod/db")["SecretString"])
db_password = secret["password"]

# ===== Azure Key Vault =====
from azure.keyvault.secrets import SecretClient
from azure.identity import DefaultAzureCredential
kv = SecretClient("https://my-vault.vault.azure.net/", DefaultAzureCredential())
db_password = kv.get_secret("db-password").value
```

### ML Platform Karşılaştırması
```python
# ===== AWS SageMaker =====
import sagemaker
from sagemaker.sklearn import SKLearn

estimator = SKLearn("train.py", role=role, instance_type="ml.m5.xlarge")
estimator.fit({"train": "s3://bucket/data/"})
predictor = estimator.deploy(1, "ml.t2.medium")

# ===== Azure ML =====
from azure.ai.ml import MLClient, command
from azure.identity import DefaultAzureCredential

ml_client = MLClient(DefaultAzureCredential(), subscription_id, rg, workspace)
job = command(code="./src", command="python train.py", compute="cpu-cluster")
returned_job = ml_client.jobs.create_or_update(job)
```

### Hızlı Başvuru Tablosu
| İşlem | AWS (boto3) | Azure (SDK) |
|-------|-------------|-------------|
| Dosya yükle | `s3.upload_file()` | `container.upload_blob()` |
| Dosya indir | `s3.get_object()` | `blob.download_blob()` |
| Secret al | `secrets.get_secret_value()` | `kv.get_secret()` |
| Model eğit | `SKLearn.fit()` | `ml_client.jobs.create_or_update()` |
| Deploy | `estimator.deploy()` | `ml_client.online_endpoints.*` |

### .env ile Konfigürasyon
```python
# .env
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG
AWS_DEFAULT_REGION=eu-west-1
AZURE_STORAGE_CONN_STR=DefaultEndpointsProtocol=https;...
AZURE_KEY_VAULT_URL=https://my-vault.vault.azure.net/

# Python
from dotenv import load_dotenv
import os
load_dotenv()

aws_region = os.getenv("AWS_DEFAULT_REGION")
azure_conn = os.getenv("AZURE_STORAGE_CONN_STR")
```

## 💡 Bağlantılar
- [[Cloud - Bulut Bilişime Giriş]]
- [[AWS - S3 ile Veri Depolama]]
- [[Azure - Blob Storage ve Veri Gölü]]
- [[Cloud - Bulut Güvenliği ve IAM]]

## ❓ Sorular / Anlamadıklarım
- Multi-cloud projede kimlik yönetimi nasıl yapılır?
- SDK hata yönetimi nasıl yapılmalı?

## 🔗 Kaynaklar
- https://boto3.amazonaws.com/v1/documentation/api/latest/
- https://learn.microsoft.com/tr-tr/azure/developer/python/
