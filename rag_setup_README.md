# 🚀 RAG System Setup Guide (ภาษาไทย)

> คู่มือการตั้งค่าระบบ RAG (Retrieval-Augmented Generation) ด้วย Qdrant + OpenAI สำหรับ OpenClaw

![License](https://img.shields.io/badge/license-MIT-green)
![Version](https://img.shields.io/badge/version-1.0-blue)
![Status](https://img.shields.io/badge/status-active-success)

---

## ⚠️ สิ่งสำคัญที่ต้องจำ! (อ่านก่อน!)

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
embedding = response.json()["data"][0]["embedding"]  # 1536 dimensions
```

**ห้ามใช้ sentence-transformers กับ collection 1536d เด็ดขาด!**

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

## ✨ ฟีเจอร์

- ✅ **OCR เอกสาร** ด้วย OpenAI Vision API
- ✅ **Vector Database** ด้วย Qdrant (1536 dimensions)
- ✅ **Semantic Search** รองรับภาษาไทย
- ✅ **Automated Pipeline** ด้วย Cron Jobs

---

## 📖 เนื้อหา

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

### 2. สร้าง Collection (ใช้ 1536 dimensions!)

```bash
curl -X PUT "http://localhost:6333/collections/YOUR-KNOWLEDGE" \
  -H "Content-Type: application/json" \
  -d '{"vectors": {"size": 1536, "distance": "Cosine"}}'
```

### 3. ใช้งาน RAG

```bash
# เพิ่มเอกสาร
python3 rag_openai.py add <collection> "<text>"

# ค้นหา
python3 rag_openai.py search <collection> "<query>"
```

---

## 📦 Scripts ที่มีให้ใช้

| Script | หน้าที่ | Dimensions |
|--------|---------|------------|
| `rag_openai.py` | เพิ่ม/ค้นหาเอกสารด้วย OpenAI | **1536d** ✅ |
| `ocr_bot.py` | OCR PDF ด้วย OpenAI Vision | - |
| `rag_agent.py` | เพิ่มเอกสารเข้า Qdrant (เก่า - 384d) | 384d ❌ |

---

## 🛠️ Requirements

- Python 3.11+
- **OpenAI API Key** (สำคัญ!)
- Qdrant (Local หรือ Cloud)
- Docker (แนะนำ)

---

## 💰 ค่าใช้จ่าย

| บริการ | ราคา |
|--------|------|
| OpenAI GPT-4o-mini (OCR) | $0.001-0.005/หน้า |
| **OpenAI text-embedding-3-small** | **$0.01/1M tokens** |
| Qdrant (Self-host) | ฟรี |

---

## 📖 เอกสารเพิ่มเติม

| เอกสาร | รายละเอียด |
|--------|-------------|
| [คู่มือภาษาไทย](./docs/RAG-Setup-Guide-TH.md) | คู่มือการติดตั้งแบบละเอียด |
| [Scripts](./scripts/) | Python Scripts สำหรับ OCR และ RAG |

---

## ❌ ข้อผิดพลาดที่พบบ่อย

### Q: ใช้งานไม่ได้ ขึ้น error?

**ตรวจสอบ:**
1. ใช้ model ให้ถูกต้องหรือไม่?
   - ✅ ต้องเป็น `text-embedding-3-small` (1536d)
   - ❌ ห้ามใช้ `paraphrase-multilingual-MiniLM-L12-v2` (384d)

2. Collection dimensions ตรงกับ model หรือไม่?
   - ถ้าใช้ 1536d → ต้องสร้าง collection ใหม่รองรับ 1536d

---

## 📞 ติดต่อ

- **GitHub Issues:** https://github.com/Nub-1/openclaw-rag-setup/issues

---

## 📝 License

MIT License - สามารถนำไปใช้ได้อิสระ

---

## 👨💻 เครดิต

| ชื่อ | บทบาท |
|------|--------|
| **โจ้ พัฒนากร** | ผู้ออกแบบระบบ |
| **น้องมิว (Mew)** | AI Assistant |

---

*สร้างด้วย ❤️ โดย โจ้ พัฒนากร และ น้องมิว AI Assistant*
