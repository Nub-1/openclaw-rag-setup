# 🚀 RAG System Setup Guide (ภาษาไทย)

> คู่มือการตั้งค่าระบบ RAG (Retrieval-Augmented Generation) ด้วย Qdrant + OpenAI สำหรับ OpenClaw

![License](https://img.shields.io/badge/license-MIT-green)
![Version](https://img.shields.io/badge/version-1.0-blue)
![Status](https://img.shields.io/badge/status-active-success)

---

## 📱 ภาพรวมระบบ

```
┌─────────────────────────────────────────────────────────────────────┐
│                        RAG System Architecture                       │
├─────────────────────────────────────────────────────────────────────┤
│   User (Telegram) ──▶ OpenClaw (AI Agent) ──▶ OpenAI API      │
│                            │                                         │
│                            ▼                                         │
│                     Python Scripts ──▶ Qdrant (Vector DB)          │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✨ ฟีเจอร์

- ✅ **OCR เอกสาร** ด้วย OpenAI Vision API
- ✅ **Vector Database** ด้วย Qdrant
- ✅ **Semantic Search** รองรับภาษาไทย
- ✅ **Automated Pipeline** ด้วย Cron Jobs

---

## 📖 เนื้อหา (คลิกเพื่อไปยังคู่มือ)

| # | หัวข้อ | ไฟล์ |
|---|--------|------|
| 1 | [ภาพรวมระบบ](./docs/RAG-Setup-Guide-TH.md#1-ภาพรวมระบบ) | docs |
| 2 | [การติดตั้ง Qdrant](./docs/RAG-Setup-Guide-TH.md#2-การติดตั้ง-qdrant-บน-unraid) | docs |
| 3 | [การตั้งค่า OpenAI API](./docs/RAG-Setup-Guide-TH.md#3-การตั้งค่า-openai-api) | docs |
| 4 | [Python Dependencies](./docs/RAG-Setup-Guide-TH.md#4-การติดตั้ง-python-และ-dependencies) | docs |
| 5 | [OCR ด้วย OpenAI Vision](./docs/RAG-Setup-Guide-TH.md#5-การทำ-ocr-ด้วย-openai-vision) | docs |
| 6 | [Embedding + Qdrant](./docs/RAG-Setup-Guide-TH.md#6-การทำ-embedding-และเก็บใน-qdrant) | docs |
| 7 | [การค้นหา](./docs/RAG-Setup-Guide-TH.md#7-การค้นหาข้อมูล) | docs |
| 8 | [แก้ไขปัญหา](./docs/RAG-Setup-Guide-TH.md#8-การแก้ไขปัญหาที่พบบ่อย) | docs |
| 9 | [Use Cases](./docs/RAG-Setup-Guide-TH.md#9-use-cases-และตัวอย่าง) | docs |

---

## ⚡ เริ่มต้นง่ายๆ

### 1. ติดตั้ง Qdrant

```bash
# ด้วย Docker
docker run -d -p 6333:6333 -p 6334:6334 \
  -v qdrant_storage:/qdrant/storage \
  qdrant/qdrant:latest
```

### 2. สร้าง Collection

```bash
curl -X PUT "http://localhost:6333/collections/YOUR-KNOWLEDGE" \
  -H "Content-Type: application/json" \
  -d '{"vectors": {"size": 1536, "distance": "Cosine"}}'
```

### 3. รัน OCR + Embedding

```bash
python3 ocr_bot.py --input your_document.pdf
```

### 4. ค้นหา

```python
results = search("คำถามของคุณ")
```

---

## 📦 Scripts ที่มีให้ใช้

| Script | หน้าที่ |
|--------|---------|
| `ocr_bot.py` | OCR PDF ด้วย OpenAI Vision |
| `rag_agent.py` | เพิ่มเอกสารเข้า Qdrant |
| `ocr_resume.py` | ทำต่อจากที่ค้างไว้ |

---

## 🛠️ Requirements

- Python 3.11+
- OpenAI API Key
- Qdrant (Local หรือ Cloud)
- Docker (แนะนำ)

---

## 💰 ค่าใช้จ่าย

| บริการ | ราคา |
|--------|------|
| OpenAI GPT-4o-mini (OCR) | $0.001-0.005/หน้า |
| OpenAI Embedding | $0.02/1M tokens |
| Qdrant (Self-host) | ฟรี |

---

## 📖 เอกสารเพิ่มเติม

| เอกสาร | รายละเอียด |
|--------|-------------|
| [คู่มือภาษาไทย](./docs/RAG-Setup-Guide-TH.md) | คู่มือการติดตั้งแบบละเอียด |
| [Scripts](./scripts/) | Python Scripts สำหรับ OCR และ RAG |

---

## 📞 ติดต่อ

- **GitHub Issues:** https://github.com/Nub-1/openclaw-rag-setup/issues

---

## 📝 License

MIT License - สามารถนำไปใช้ได้อิสระ

---

## 👨‍💻 เครดิต

| ชื่อ | บทบาท |
|------|--------|
| **โจ้ พัฒนากร** | ผู้ออกแบบระบบ |
| **น้องมิว (Mew)** | AI Assistant |

---

*สร้างด้วย ❤️ โดย โจ้ พัฒนากร และ น้องมิว AI Assistant*
