---
tarih: 2026-05-28
konu: Cloud
etiket: ["cloud", "güvenlik", "iam", "role", "policy", "azure-ad"]
kaynak: AWS / Azure Dokümantasyon
zorluk: orta
---

## 📌 Özet
Kimlik ve Erişim Yönetimi (IAM), bulut bilişim ortamındaki kaynakların güvenliğini sağlamak için kullanılan en kritik katmandır ve "kimin, hangi kaynağa, hangi koşullar altında erişebileceğini" merkezi olarak tanımlar. "En Az Ayrıcalık Prensibi" (Least Privilege) çerçevesinde, kullanıcılara ve otonom servislere yalnızca işlerini yapmaları için gereken minimum yetkiler verilerek sistemin saldırı yüzeyi önemli ölçüde daraltılır. Özellikle veri bilimi projelerinde, hassas veri göllerine, veritabanlarına ve ML modellerine erişim; roller (Roles), politikalar (Policies) ve Managed Identity gibi modern mekanizmalarla yönetilerek veri güvenliği ve uyumluluk standartları korunur. Bulut güvenliği, altyapı sağlayıcısı ile kullanıcı arasında paylaşılan bir sorumluluk modeli üzerine kuruludur ve bu modelin doğru anlaşılması güvenli bir mimari için vazgeçilmezdir.

## 🧠 Detay

### 🛡️ Paylaşımlı Sorumluluk Modeli

```mermaid
graph TD
    subgraph "Müşteri Sorumluluğu (Buluttaki Güvenlik)"
        A["Veri Güvenliği"]
        B["Uygulama Güvenliği"]
        C["Kimlik ve Erişim Yönetimi (IAM)"]
        D["İşletim Sistemi (IaaS)"]
    end
    
    subgraph "Sağlayıcı Sorumluluğu (Bulutun Güvenliği)"
        E["Fiziksel Sunucular"]
        F["Ağ Altyapısı"]
        G["Veri Merkezi Güvenliği"]
        H["Sanallaştırma Katmanı"]
    end
    
    A & B & C & D --- Sağ Çizgisi["Güvenlik Sınırı"]
    Sağ Çizgisi --- E & F & G & H
```

### AWS IAM

#### Temel Kavramlar
```
User     → İnsan kullanıcı
Group    → Kullanıcı grubu
Role     → Servis/uygulama kimliği (EC2, Lambda)
Policy   → İzin belgesi (JSON)
MFA      → Çok faktörlü doğrulama
```

#### ML Servisi için IAM Role
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::ml-veri-golum/*"
    },
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::ml-veri-golum"
    },
    {
      "Effect": "Allow",
      "Action": [
        "sagemaker:CreateTrainingJob",
        "sagemaker:DescribeTrainingJob",
        "sagemaker:CreateModel",
        "sagemaker:CreateEndpoint"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    }
  ]
}
```

#### Boto3 ile IAM
```python
import boto3

iam = boto3.client("iam")

# Role oluştur
role_policy = {
    "Version": "2012-10-17",
    "Statement": [{
        "Effect": "Allow",
        "Principal": {"Service": "lambda.amazonaws.com"},
        "Action": "sts:AssumeRole"
    }]
}

role = iam.create_role(
    RoleName="ml-lambda-role",
    AssumeRolePolicyDocument=json.dumps(role_policy),
    Description="ML Lambda için IAM Role"
)

# Policy ekle
iam.attach_role_policy(
    RoleName="ml-lambda-role",
    PolicyArn="arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"
)

# Inline policy ekle
iam.put_role_policy(
    RoleName="ml-lambda-role",
    PolicyName="s3-write-policy",
    PolicyDocument=json.dumps(my_policy)
)
```

#### Secrets Manager
```python
import boto3
import json

secrets_client = boto3.client("secretsmanager", region_name="eu-west-1")

# Secret oluştur
secrets_client.create_secret(
    Name="prod/ml-api/credentials",
    SecretString=json.dumps({
        "api_key": "gizli-anahtar",
        "db_password": "db-sifre"
    })
)

# Secret oku
def secret_al(secret_name: str) -> dict:
    response = secrets_client.get_secret_value(SecretId=secret_name)
    return json.loads(response["SecretString"])

creds = secret_al("prod/ml-api/credentials")
```

### Azure IAM (RBAC)

#### Temel Kavramlar
```
Azure AD  → Kimlik sağlayıcı
RBAC      → Role-Based Access Control
Managed Identity → Servis kimliği (parola yok)
Service Principal → Uygulama kimliği
Key Vault → Secret yönetimi
```

#### Managed Identity ile Güvenli Erişim
```python
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient
from azure.storage.blob import BlobServiceClient

# Managed Identity (kimlik bilgisi gerekmez!)
credential = DefaultAzureCredential()

# Key Vault'tan secret al
kv_url = "https://ml-keyvault.vault.azure.net/"
kv_client = SecretClient(vault_url=kv_url, credential=credential)

api_key = kv_client.get_secret("api-key").value
db_password = kv_client.get_secret("db-password").value

# Blob Storage'a eriş
blob_client = BlobServiceClient(
    account_url="https://storageaccount.blob.core.windows.net",
    credential=credential
)
```

#### Azure Key Vault
```python
from azure.keyvault.secrets import SecretClient
from azure.identity import ClientSecretCredential

# Service Principal ile
credential = ClientSecretCredential(
    tenant_id="TENANT_ID",
    client_id="CLIENT_ID",
    client_secret="CLIENT_SECRET"
)

client = SecretClient(
    vault_url="https://ml-keyvault.vault.azure.net/",
    credential=credential
)

# Secret oluştur
client.set_secret("model-api-key", "gizli-anahtar-1234")

# Secret al
secret = client.get_secret("model-api-key")
print(secret.value)

# Secret sil
client.begin_delete_secret("eski-secret")
```

### En İyi Pratikler
```
✅ Her zaman minimum yetki (least privilege)
✅ Root hesabı asla kullanma
✅ MFA aktif et
✅ Key/secret'ları kod içine yazma
✅ Managed Identity kullan (parola yok)
✅ Secret rotation aktif et
✅ CloudTrail/Activity Log ile izle
✅ Resource tag'leme yap
❌ Wildcard (*) izin verme
❌ AccessKey'i git'e commit etme
```

## 💡 Bağlantılar
- [[Cloud - Bulut Bilişime Giriş]]
- [[AWS - S3 ile Veri Depolama]]
- [[Azure - Blob Storage ve Veri Gölü]]
- [[Cloud - Terraform ile Altyapı Yönetimi]]

## ❓ Sorular / Anlamadıklarım
- IAM Permission Boundary ne zaman kullanılır?
- Azure AD Conditional Access nasıl çalışır?

## 🔗 Kaynaklar
- https://docs.aws.amazon.com/iam/
- https://learn.microsoft.com/tr-tr/azure/active-directory/
