# 🚀 RAG System Setup Guide

> คู่มือการตั้งค่าระบบ RAG (Retrieval-Augmented Generation) ด้วย Qdrant + OpenAI สำหรับ OpenClaw

[![OpenClaw](https://img.shields.io/badge/OpenClaw-2026.4.1-blue)](https://github.com/openclaw/openclaw)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)
[![Author](https://img.shields.io/badge/Author-Mew-purple)](https://github.com/openclaw)

---

## ⚠️ สิ่งสำคัญที่ต้องจำ!

**ใช้ OpenAI `text-embedding-3-small` เสมอ - สร้าง 1536 dimensions**

```python
# ❌ ผิด - model เก่าสร้างแค่ 384 dimensions
model = SentenceTransformer('paraphrase-multilingual-MiniLM-L12-v2')

# ✅ ถูกต้อง - ใช้ OpenAI text-embedding-3-small (1536 dimensions)
response = requests.post(
    "https://api.openai.com/v1/embeddings",
    headers={"Authorization": f"Bearer {OPENAI_API_KEY}"},
    json={"model": "text-embedding-3-small", "input": text}
)
```

> **ห้ามใช้ sentence-transformers กับ collection 1536d เด็ดขาด!**

---

## 📋 สารบัญ

| เอกสาร | คำอธิบาย |
|---------|-----------|
| [คู่มือติดตั้ง RAG](docs/RAG-Setup-Guide-TH.md) | คู่มือฉบับเต็มสำหรับตั้งค่าระบบ RAG |

---

## 🎯 Features

| Feature | Status | คำอธิบาย |
|---------|--------|-----------|
| **OCR เอกสาร** | ✅ | ด้วย OpenAI Vision API |
| **Vector Database** | ✅ | ด้วย Qdrant (1536 dimensions) |
| **Semantic Search** | ✅ | รองรับภาษาไทย |
| **Automated Pipeline** | ✅ | ด้วย Cron Jobs |

---

## 📱 ภาพรวมระบบ

```
┌─────────────────────────────────────────────────────────────────────┐
│                        RAG System Architecture                       │
├─────────────────────────────────────────────────────────────────────┤
│   User (Telegram) ──▶ OpenClaw (AI Agent) ──▶ OpenAI API (1536d)   │
│                            │                                         │
│                            ▼                                         │
│                     Python Scripts ──▶ Qdrant (Vector DB)          │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ⚡ เริ่มต้นใช้งาน

👉 **[อ่านคู่มือฉบับเต็ม](docs/RAG-Setup-Guide-TH.md)**

---

## 📚 โครงสร้างไฟล์

```
openclaw-rag-setup/
├── README.md                    ← สารบัญ (ไฟล์นี้)
├── LICENSE                      ← MIT License
├── .gitignore                  ← Git ignore
├── docs/
│   └── RAG-Setup-Guide-TH.md   ← คู่มือฉบับเต็ม
└── scripts/                    ← Scripts folder (empty)
```

---

## 🛠️ ข้อมูลทั่วไป

| รายการ | รายละเอียด |
|---------|-------------|
| **เวอร์ชัน** | 1.0 |
| **ผู้จัดทำ** | โจ้ พัฒนากร |
| **ผู้เขียน** | มิว (Mew AI) |
| **สัญญาอนุญาต** | MIT License |

---

## 🔗 ลิงก์ที่เกี่ยวข้อง

- [OpenClaw Documentation](https://docs.openclaw.ai)
- [Qdrant](https://qdrant.tech/)
- [OpenAI API](https://platform.openai.com/)
- [OpenClaw Community](https://discord.com/invite/clawd)

---

*คู่มือนี้เป็นส่วนหนึ่งของ OpenClaw RAG Setup — พัฒนาโดย โจ้ พัฒนากร | ผู้เขียน: มิว (Mew AI)* 💖
