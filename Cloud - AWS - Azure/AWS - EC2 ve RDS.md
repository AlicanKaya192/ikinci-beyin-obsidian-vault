---
tarih: 2026-05-28
konu: Cloud
etiket: ["aws", "ec2", "rds", "sanal-makine", "veritabanı"]
kaynak: AWS Dokümantasyon
zorluk: başlangıç
---

## 📌 Özet
Amazon EC2 (Elastic Compute Cloud), AWS'nin sunduğu temel Altyapı Servisi (IaaS) olup, kullanıcılara tam işletim sistemi kontrolü ve özel yazılım yığınları kurma imkanı tanıyan ölçeklenebilir sanal sunucular sağlar. RDS (Relational Database Service) ise yedekleme, güvenlik yamaları ve donanım ölçeklendirme gibi karmaşık yönetimsel görevleri otomatikleştirerek geliştiricilerin altyapı yerine uygulama mantığına odaklanmasına olanak tanıyan tam yönetilen bir ilişkisel veritabanı servisidir. Bu iki servis, EC2'nin işlem ve uygulama mantığını, RDS'nin ise kalıcı ve güvenilir veri depolamayı üstlendiği modern bulut mimarilerinin omurgasını oluşturur. Özellikle yüksek performans, yüksek erişilebilirlik ve ölçeklenebilirlik gerektiren web uygulamaları, makine öğrenmesi API'leri ve veri yoğunluklu servisler için bu servislerin birlikte kullanımı endüstri standardı olarak kabul edilir.

## 🧠 Detay

```mermaid
graph LR
    A["Kullanıcı / İstemci"] --> B["EC2 Instance (Uygulama Sunucusu)"]
    subgraph "AWS VPC (Sanal Özel Bulut)"
    B --> C["Security Group (Güvenlik Duvarı)"]
    C --> D["RDS Instance (Veritabanı)"]
    B -- "Boto3 / SDK" --> E["Secrets Manager (Şifre Yönetimi)"]
    end
    D -- "Sorgu Sonuçları" --> B
    style B fill:#f9f,stroke:#333,stroke-width:2px
    style D fill:#bbf,stroke:#333,stroke-width:2px
```

### EC2 Instance Türleri
```
Genel Amaçlı:
  t3.micro    → 2 vCPU, 1 GB RAM (Free Tier)
  t3.medium   → 2 vCPU, 4 GB RAM
  m5.xlarge   → 4 vCPU, 16 GB RAM

Bellek Optimized:
  r5.xlarge   → 4 vCPU, 32 GB RAM (büyük veri)
  r5.2xlarge  → 8 vCPU, 64 GB RAM

GPU (ML Eğitim):
  p3.2xlarge  → 1x V100, 61 GB RAM
  p4d.24xlarge → 8x A100

Hesaplama Optimized:
  c5.2xlarge  → 8 vCPU, 16 GB RAM
```

### Boto3 ile EC2 Yönetimi
```python
import boto3

ec2 = boto3.resource("ec2", region_name="eu-west-1")
ec2_client = boto3.client("ec2", region_name="eu-west-1")

# Instance başlat
instance = ec2.create_instances(
    ImageId="ami-0c55b159cbfafe1f0",    # Amazon Linux 2 AMI
    MinCount=1,
    MaxCount=1,
    InstanceType="t3.medium",
    KeyName="anahtar-cifti",
    SecurityGroupIds=["sg-xxxx"],
    SubnetId="subnet-xxxx",
    IamInstanceProfile={"Name": "ml-ec2-role"},
    UserData="""#!/bin/bash
        yum update -y
        pip3 install scikit-learn pandas fastapi uvicorn
        cd /home/ec2-user
        aws s3 cp s3://ml-veri-golum/app/ . --recursive
        uvicorn main:app --host 0.0.0.0 --port 8000 &
    """,
    TagSpecifications=[{
        "ResourceType": "instance",
        "Tags": [{"Key": "Name", "Value": "ml-api-server"}]
    }]
)[0]

print(f"Instance ID: {instance.id}")
instance.wait_until_running()
instance.reload()
print(f"Public IP: {instance.public_ip_address}")

# Durdur
instance.stop()

# Başlat
instance.start()

# Sonlandır
instance.terminate()
```

### Spot Instance (Ucuz Eğitim)
```python
# %70-90 indirimli ama kesintiye uğrayabilir
response = ec2_client.request_spot_instances(
    InstanceCount=1,
    SpotPrice="0.05",           # Max saat fiyatı $0.05
    LaunchSpecification={
        "ImageId": "ami-xxxx",
        "InstanceType": "p3.2xlarge",   # GPU instance
        "KeyName": "anahtar-cifti",
        "UserData": user_data_b64
    }
)
```

### RDS Veritabanı

#### Bağlantı (Python)
```python
import psycopg2   # PostgreSQL
import pymysql    # MySQL

# RDS PostgreSQL
conn = psycopg2.connect(
    host="database.cluster-xxxx.eu-west-1.rds.amazonaws.com",
    port=5432,
    database="mldb",
    user="admin",
    password="Sifre123!",
    sslmode="require"
)

# Pandas ile
import pandas as pd
from sqlalchemy import create_engine

engine = create_engine(
    "postgresql://admin:Sifre123!@rds-endpoint:5432/mldb"
)
df = pd.read_sql("SELECT * FROM tahmin_loglar LIMIT 1000", engine)
df.to_sql("model_metrikleri", engine, if_exists="append", index=False)
```

#### RDS Secrets Manager ile Güvenli Bağlantı
```python
import boto3
import json

def rds_baglanti():
    """Şifreyi Secrets Manager'dan al"""
    client = boto3.client("secretsmanager", region_name="eu-west-1")
    secret = client.get_secret_value(SecretId="prod/mldb/credentials")
    creds = json.loads(secret["SecretString"])

    engine = create_engine(
        f"postgresql://{creds['username']}:{creds['password']}"
        f"@{creds['host']}:{creds['port']}/{creds['dbname']}"
    )
    return engine

engine = rds_baglanti()
```

### Boto3 ile RDS Yönetimi
```python
rds = boto3.client("rds", region_name="eu-west-1")

# Snapshot al
rds.create_db_snapshot(
    DBSnapshotIdentifier="mldb-snapshot-20260528",
    DBInstanceIdentifier="ml-database"
)

# DB boyutlandırma
rds.modify_db_instance(
    DBInstanceIdentifier="ml-database",
    DBInstanceClass="db.t3.medium",
    ApplyImmediately=True
)
```

### EC2 vs Lambda vs SageMaker
```
EC2:
  ✅ Tam kontrol
  ✅ Uzun süren işlemler
  ✅ Özel kurulum gerektiren ML
  ❌ Her zaman çalışıyor (masraf)

Lambda:
  ✅ Kısa işlemler (<15 dk)
  ✅ Düşük trafik
  ✅ Pay-per-use
  ❌ Cold start, sınırlı kaynak

SageMaker:
  ✅ Yönetilen ML pipeline
  ✅ Otomatik ölçekleme
  ❌ Yüksek maliyet
```

## 💡 Bağlantılar
- [[Cloud - Bulut Bilişime Giriş]]
- [[AWS - S3 ile Veri Depolama]]
- [[AWS - Lambda ile Serverless]]
- [[MSSQL - Python ile MSSQL Bağlantısı]]

## ❓ Sorular / Anlamadıklarım
- Spot instance eğitim yarıda kesilirse ne olur?
- RDS Multi-AZ ne zaman gerekli?

## 🔗 Kaynaklar
- https://docs.aws.amazon.com/ec2/
- https://docs.aws.amazon.com/rds/
