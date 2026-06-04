# MSSQL - Güvenlik, Audit ve Row-Level Security

## 📌 Özet
SQL Server'da veri güvenliği, sadece yetkilendirme ile sınırlı kalmayıp, satır bazlı erişim kontrolü (RLS) ve hassas verilerin gizlenmesi (Data Masking) gibi ileri düzey teknikleri içerir. Row-Level Security, uygulama katmanında karmaşık filtrelemeler yapmak yerine, veritabanı motoru seviyesinde kullanıcı bazlı satır kısıtlaması sağlar. SQL Audit ise sistemdeki tüm kritik işlemleri kayıt altına alarak yasal uyumluluk ve güvenlik denetimi imkanı tanır. Bu rehber, hassas verilerin korunması ve izlenmesi için gerekli yapılandırmaları kapsamaktadır.

## 🧠 Detay

```mermaid
graph TD
    "Uygulama Sorgusu" --> "Security Predicate (RLS)"
    "Security Predicate (RLS)" -- "Erişim Var" --> "Veri Satırı"
    "Security Predicate (RLS)" -- "Erişim Yok" --> "Filtrelenmiş Sonuç"
    
    subgraph "Güvenlik Katmanları"
        "Authentication" --> "Authorization"
        "Authorization" --> "RLS (Satır Seviyesi)"
        "RLS (Satır Seviyesi)" --> "Data Masking (Sütun Seviyesi)"
    end
    
    "İşlem Kaydı" --> "SQL Server Audit"
    "SQL Server Audit" --> "Audit Log Dosyası"
```

### Row-Level Security (RLS) Mekanizması

RLS, bir fonksiyon (Predicate Function) ve bu fonksiyonu tabloya bağlayan bir politika (Security Policy) ile çalışır.
- **Filter Predicate**: Okuma işlemlerinde satırları gizler.
- **Block Predicate**: Yazma (Insert, Update, Delete) işlemlerinde yetkisiz veri girişini engeller.

### Dynamic Data Masking (DDM)

DDM, veriyi diskte değiştirmez; ancak yetkisiz kullanıcılara veriyi maskeli bir şekilde sunar (Örn: Kredi kartı numarasının son 4 hanesini göstermek).

### Kod Örnekleri

Row-Level Security Uygulaması:

```sql
-- 1. Predicate Function Oluşturma
CREATE FUNCTION Security.fn_securitypredicate(@SalesRep AS sysname)
    RETURNS TABLE
WITH SCHEMABINDING
AS
    RETURN SELECT 1 AS fn_securitypredicate_result
    WHERE @SalesRep = USER_NAME() OR USER_NAME() = 'Manager';

-- 2. Security Policy Oluşturma
CREATE SECURITY POLICY SalesFilter
ADD FILTER PREDICATE Security.fn_securitypredicate(SalesRep)
ON Sales.Orders
WITH (STATE = ON);
```

Dynamic Data Masking Tanımlama:

```sql
-- Sütun bazlı maskeleme ekleme
ALTER TABLE Employees.Staff
ALTER COLUMN Email ADD MASKED WITH (FUNCTION = 'email()');

ALTER TABLE Employees.Staff
ALTER COLUMN Phone ADD MASKED WITH (FUNCTION = 'partial(2, "XXX", 2)');
```

## 🔗 İlgili Notlar
- [[MSSQL - İleri Transaction İzolasyon Seviyeleri]]
- [[Cloud - Bulut Güvenliği ve IAM]]
