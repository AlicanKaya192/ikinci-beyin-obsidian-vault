# 🤝 Katkıda Bulunma Rehberi — The Tech Cortex

Bu vault'a katkıda bulunmak istediğin için teşekkürler! Birlikte daha güçlü bir kaynak oluşturalım.

---

## 📁 Klasör Yapısı

Her konu kendi klasöründe yaşar. Yeni not eklerken doğru klasörü seç:

```
the-tech-cortex/
├── Python Programlama/
├── Makine Öğrenmesi/
├── Derin Öğrenme/
├── PostgreSQL/
├── MLOps & CI-CD/
├── Kariyer & Portfolio/
└── ... (diğer konular)
```

## 📝 Not Formatı

Her not bu şablonla başlar:

```markdown
---
tarih: YYYY-MM-DD
konu: [Klasör Adı]
etiket: [etiket1, etiket2, etiket3]
kaynak: Kaynak adı / URL
zorluk: başlangıç | orta | ileri
---

## 📌 Özet
Konunun 2-3 cümlelik özeti.

---

## 🧠 Detay
Ana içerik buraya gelir.

---

## 💡 Bağlantılar
- [[İlgili Not Adı]]

## ❓ Sorular / Anlamadıklarım

## 🔗 Kaynaklar
- [Kaynak Adı](URL)
```

## 🏷️ Etiket (Tag) Sistemi

Etiketleri küçük harfle, Türkçe veya İngilizce teknik terimler olarak yaz:

| Kategori | Örnekler |
|----------|----------|
| **Seviye** | `başlangıç`, `orta`, `ileri` |
| **Teknoloji** | `python`, `postgresql`, `docker`, `pytorch` |
| **Konu** | `ml`, `nlp`, `mlops`, `güvenlik` |
| **Tip** | `cheatsheet`, `proje`, `mülakat`, `karşılaştırma` |

## 📌 Dosya İsimlendirme

```
[Kısaltma] - [Konu Başlığı (Türkçe)].md

Örnekler:
✓ PG - İleri SQL Window Functions.md
✓ ML - Gradient Boosting Algoritmaları.md
✓ Kariyer - GitHub Profil Optimizasyonu.md

Her klasörde bir giriş notu:
✓ 00 - [Konu] Giriş ve Yol Haritası.md
```

## ✅ Kalite Kontrol Listesi

Notunu göndermeden önce:

- [ ] Frontmatter eksiksiz mi? (tarih, konu, etiket, kaynak, zorluk)
- [ ] Özet bölümü var mı?
- [ ] Kod blokları dil etiketli mi? (```python, ```sql, ```bash)
- [ ] En az 1 wikilink var mı? ([[İlgili Not]])
- [ ] Kaynaklar bölümü dolu mu?
- [ ] Mermaid diyagramı eklendi mi? (akış şeması varsa)

## 🔗 Wikilink Kullanımı

İlgili notlara `[[Not Adı]]` sözdizimi ile bağlan. Bu Obsidian'ın Graph View'unda görsel bağlantı oluşturur:

```markdown
## 💡 Bağlantılar
- [[PG - İndeksleme Stratejileri]]
- [[ML - Feature Engineering Temelleri]]
- [[Docker - Container Temelleri]]
```

## 🚀 Pull Request Süreci

1. Repo'yu fork'la
2. Yeni branch aç: `git checkout -b konu/yeni-not-adi`
3. Notu ekle, formata uy
4. PR aç — başlığa konu ve not adını yaz
5. PR açıklamasında notun ne kattığını kısaca belirt

---

*Sorular için GitHub Issues kullan.*
