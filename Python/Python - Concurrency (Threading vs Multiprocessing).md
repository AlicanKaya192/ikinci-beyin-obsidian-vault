---
tarih: 2026-06-06
konu: Python Concurrency
etiket: [python, threading, multiprocessing, gil, concurrency]
kaynak: Gemini CLI
zorluk: Ileri
---

## 📌 Ozet
Python da eszamanlilik (concurrency) iki ana yontemle saglanir: Threading ve Multiprocessing.

## 🧠 Detay

```mermaid
graph TD
    Start["Gorev Tipi Nedir?"] --> IO["I/O Bound"]
    Start --> CPU["CPU Bound"]
    IO --> Threading["Threading / Asyncio"]
    CPU --> MP["Multiprocessing"]
    subgraph "Python Siniri"
    GIL["GIL - Global Interpreter Lock"] -.-> Threading
    end
```
