# 💻 AI-Driven Coding - IDE ve Terminal Entegrasyonları

## 🧠 Geliştirici Ortamının (DevEnv) Yeniden Keşfi
AI-Augmented Engineering'in pratikteki uygulama alanı, geliştiricilerin günlük yaşam alanı olan IDE'ler (Entegre Geliştirme Ortamları) ve Terminal/CLI entegrasyonlarıdır. AI destekli kodlama, klasik "Copilot" tarzı satır içi (inline) kod tamamlamalarından, doğrudan proje genelinde değişiklik yapabilen "Agentic IDE" yapısına (Cursor, Windsurf vb.) evrilmiştir. Bu entegrasyonlar, LLM'lerin (Large Language Models) yapay zekasını geliştiricinin yerel makine gücüyle (Dosya Sistemi, Linter, Compiler) birleştirir.

## 🛠️ Yeni Nesil IDE'ler: Cursor ve Windsurf
Geleneksel VS Code üzerine eklenti (extension) kurma mantığı yerini, çekirdeğinde AI barındıran ("AI-Native") IDE fork'larına bırakmıştır.

1.  **Cursor IDE:** VS Code fork'u olan Cursor, şu anki en popüler AI IDE'sidir. En büyük gücü "Composer" ve "Cmd+K" mimarisidir. Composer aracı, çoklu dosyaları (multi-file edit) aynı anda düzenleme, projenin bağlamını anlama ve doğrudan terminal komutlarını izleyerek (örneğin Linter hatalarını otomatik okuyup düzeltme) kendi kendine tamir (self-healing) yeteneklerine sahiptir.
2.  **Windsurf:** Codeium tarafından geliştirilen bu IDE, "Flow" (Akış) kavramına odaklanır. Geliştiricinin niyetini (intent) arka planda analiz eder, sadece yazılan kodu değil, kullanıcının fare hareketlerini, dosya geçişlerini ve imleç (cursor) pozisyonunu da bağlama katarak daha derin ve kesintisiz bir "eşli programlama" (pair programming) deneyimi sunar.

Bu IDE'ler, arka planda Claude 3.7 Sonnet veya GPT-4o gibi güçlü modelleri kullanarak, proje genelindeki bağımlılıkları (Dependencies) mükemmel şekilde yönetirler.

## ⚙️ Terminal ve Komut Satırı Ajanları (CLI) Entegrasyonu
IDE'ler mükemmel bir görsel deneyim ve mikro-müdahaleler sunsa da, gerçek "Otonom İş Akışı" CLI tabanlı ajanlarla sağlanır. Aider, Gemini CLI ve OpenHands gibi araçlar, doğrudan kabuk (shell) ortamında çalışır.

-   **Özellikleri:** Git entegrasyonu (Değişiklikleri otomatik diff'leme ve commit'leme), test suite'lerini çalıştırma (örn: `pytest`, `npm run test`) ve "Test Driven Development" (TDD) döngüsünü otonom yönetme. Ajan terminalde kodu yazar, testleri koşturur, hata çıkarsa konsol logunu okur, kodu düzeltir ve testler yeşile dönene kadar (pass) bu döngüyü ısrarla sürdürür.

## 📊 IDE ve CLI Entegrasyon Akışı (Mermaid)

```mermaid
sequenceDiagram
    actor Dev as Geliştirici
    participant IDE as Cursor / Windsurf IDE
    participant CLI as CLI Agent (Aider / Gemini CLI)
    participant FS as Local File System
    participant Compiler as Compiler/Linter (Node/Python)

    Dev->>IDE: "Auth sistemini JWT'den OAuth2'ye geçir" (Composer)
    activate IDE
    IDE->>FS: Tüm Auth dosyalarını oku & haritala
    IDE-->>Dev: Yapılacak değişikliklerin planı ve Diff
    Dev->>IDE: Diff'leri onayla (Accept All)
    IDE->>FS: Dosyaları kaydet
    deactivate IDE

    Dev->>CLI: "Testleri çalıştır, hata varsa otonom düzelt."
    activate CLI
    CLI->>Compiler: npm run test
    activate Compiler
    Compiler-->>CLI: 3 Test Başarısız (Error Logs)
    deactivate Compiler
    
    CLI->>CLI: Claude 3.7 / Gemini ile hataları analiz et
    CLI->>FS: Test ve Kaynak dosyalarını onar (Self-Healing)
    CLI->>Compiler: npm run test (Tekrar)
    activate Compiler
    Compiler-->>CLI: Tüm Testler Başarılı (Yeşil)
    deactivate Compiler
    CLI->>FS: `git commit -m "Fix OAuth2 Auth Test Issues"`
    CLI-->>Dev: Görev başarıyla tamamlandı ve commit edildi.
    deactivate CLI
```

## 💡 Kurumsal Çözümler İçin Best Practices
-   **Kural Dosyaları (Rule Files):** IDE ve CLI ajanlarını projenin kodlama standartlarına (Naming conventions, mimari kalıplar) uymaya zorlamak için proje kök dizininde muhakkak detaylı `.cursorrules`, `GEMINI.md` veya `.windsurfrules` dosyaları barındırılmalıdır.
-   **Context Limitasyonları:** İster IDE ister CLI kullanılsın, modeli gereksiz verilerle boğmamak (ve maliyeti kontrol altında tutmak) adına gereksiz klasörler (`node_modules`, `build`, `dist`) daima ignore edilmelidir.
-   **Hibrit Çalışma Model:** Tasarım, UI revizyonları ve anlık mimari kurgular IDE'ler üzerinden; veri tabanı migrasyonları, büyük refactoring'ler ve otonom test onarımları ise CLI ajanları üzerinden koordine edildiğinde maksimum mühendislik verimine ulaşılır. 🔗
