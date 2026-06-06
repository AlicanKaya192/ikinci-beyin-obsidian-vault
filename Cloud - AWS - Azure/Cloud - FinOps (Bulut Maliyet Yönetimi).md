---
tarih: 2026-06-06
konu: FinOps
etiket: [cloud, finance, operations, optimization]
kaynak: Gemini CLI
zorluk: Orta
---

## 📌 Ozet
FinOps, bulut harcamalarını optimize etmek için finans, teknoloji ve iş birimlerini bir araya getiren bir kültürel disiplindir. Amaç sadece "ucuz" olmak değil, buluta harcanan her kuruşun iş değerine (business value) dönüşmesini sağlamaktır.

## 🧠 Detay

```mermaid
graph LR
    Inform["Inform (Görünürlük)"] --> Optimize["Optimize (Tasarruf)"]
    Optimize --> Operate["Operate (Süreklilik)"]
    Operate --> Inform
```

### 1. FinOps Yaşam Döngüsü
- **Inform:** Kim, ne kadar harcıyor? (Tagging, Dashboards).
- **Optimize:** Gereksiz kaynakları silme, "Reserved Instances" veya "Spot Instances" kullanımı.
- **Operate:** Maliyet bilincini şirket kültürüne yerleştirme.

### 2. Temel Kavramlar
- **Right-sizing:** İhtiyaçtan fazlasını kullanmamak.
- **Unused Resources:** Çalışmayan ama para yazan diskler ve IP leri temizlemek.

## 💡 Baglantilar
- [[Cloud - Bulut Bilişime Giriş]]
- [[K8s - HPA ve VPA (Autoscaling)]]
