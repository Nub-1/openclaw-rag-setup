# คู่มือการตั้งค่า RAG System ด้วย Qdrant + OpenAI

**เวอร์ชัน:** 1.0  
**วันที่:** 19 มีนาคม 2569  
**ผู้จัดทำ:** โจ้ พัฒนากร และ น้องมิว (Mew) - AI Assistant  
**GitHub:** https://github.com/joe-cdd/openclaw-rag-setup

---

## 📋 สารบัญ

1. [ภาพรวมระบบ](#1ภาพรวมระบบ)
2. [การติดตั้ง Qdrant บน Unraid](#2การติดตั้ง-qdrant-บน-unraid)
3. [การตั้งค่า OpenAI API](#3การตั้งค่า-openai-api)
4. [การติดตั้ง Python และ Dependencies](#4การติดตั้ง-python-และ-dependencies)
5. [การทำ OCR ด้วย OpenAI Vision](#5การทำ-ocr-ด้วย-openai-vision)
6. [การทำ Embedding และเก็บใน Qdrant](#6การทำ-embedding-และเก็บใน-qdrant)
7. [การค้นหาข้อมูล](#7การค้นหาข้อมูล)
8. [การแก้ไขปัญหาที่พบบ่อย](#8การแก้ไขปัญหาที่พบบ่อย)
9. [Use Cases และตัวอย่าง](#9use-cases-และตัวอย่าง)
10. [เครดิต](#10เครดิต)

---

## 1. ภาพรวมระบบ

### 1.1 สถาปัตยกรรมระบบ

```
┌─────────────────────────────────────────────────────────────────────┐
│                        RAG System Architecture                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌──────────┐     ┌──────────────┐     ┌───────────────────────┐  │
│   │  User    │────▶│  OpenClaw    │────▶│  OpenAI API          │  │
│   │ (Telegram)│     │  (Agent)     │     │  (Vision + Embedding)│  │
│   └──────────┘     └──────────────┘     └───────────────────────┘  │
│                              │                     │                │
│                              ▼                     ▼                │
│                      ┌──────────────┐     ┌───────────────────────┐  │
│                      │  Python      │     │  Qdrant Vector DB    │  │
│                      │  Scripts     │────▶│  (YOUR-QDRANT-IP)    │  │
│                      └──────────────┘     └───────────────────────┘  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 1.2 สิ่งที่ต้องเตรียม

| รายการ | รายละเอียด |
|---------|-------------|
| **Qdrant** | Vector Database สำหรับเก็บ Embeddings |
| **OpenAI API Key** | สำหรับ Vision OCR และ Embedding |
| **Python 3.11+** | สำหรับรัน Scripts |
| **Unraid Server** | (Optional) สำหรับรัน Qdrant แบบ Docker |

---

## 2. การติดตั้ง Qdrant บน Unraid

### 2.1 ผ่าน Docker Template

1. เปิด Unraid Web UI (https://YOUR-QDRANT-IP:9876)
2. ไปที่ **Docker** > **Add Container**
3. ค้นหา `qdrant/qdrant:latest`
4. ตั้งค่าดังนี้:

| การตั้งค่า | ค่า |
|------------|------|
| **Container Name** | `Qdrant-RAG` |
| **Network** | `bridge` |
| **Port 6333** | `6333` (REST API) |
| **Port 6334** | `6334` (gRPC) |
| **Volume 1** | `/mnt/user/appdata/qdrant:/qdrant/storage` |

5. กด **Create** รอจน Container รัน

### 2.2 ตรวจสอบ Qdrant

```bash
# ตรวจสอบว่า Qdrant ทำงาน
curl http://YOUR-QDRANT-IP:6333/
```

ผลลัพธ์ควรเป็น:
```json
{"status":"ok","version":"1.x.x"}
```

### 2.3 สร้าง Collection

```bash
curl -X PUT "http://YOUR-QDRANT-IP:6333/collections/MYW-Knowledge" \
  -H "Content-Type: application/json" \
  -d '{
    "vectors": {
      "size": 1536,
      "distance": "Cosine"
    }
  }'
```

**หมายเหตุ:**
- `size: 1536` สำหรับ OpenAI text-embedding-3-small
- `size: 384` สำหรับ sentence-transformers (paraphrase-multilingual-MiniLM-L12-v2)

---

## 3. การตั้งค่า OpenAI API

### 3.1 สมัคร OpenAI Account

1. ไปที่ https://platform.openai.com/
2. สร้าง Account หรือ Login
3. ไปที่ **API Keys** > **Create new secret key**

### 3.2 เพิ่ม Billing (สำคัญ!)

> ⚠️ **สำคัญ:** ต้องเติมเงินใน Account ก่อนถึงจะใช้งานได้

1. ไปที่ https://platform.openai.com/account/billing
2. เติมเงินขั้นต่ำ $5
3. ราคาที่ใช้:
   - **gpt-4o-mini (OCR):** ประมาณ $0.001-0.005 ต่อหน้า PDF
   - **text-embedding-3-small:** $0.02 / 1M tokens

### 3.3 ตั้งค่า Environment Variable

```bash
# ตั้งค่า API Key
export OPENAI_API_KEY="sk-..."
```

---

## 4. การติดตั้ง Python และ Dependencies

### 4.1 ติดตั้ง Python

```bash
# ติดตั้ง pip (ถ้ายังไม่มี)
apt-get update && apt-get install -y python3-pip
```

### 4.2 ติดตั้ง Libraries

```bash
pip3 install --break-system-packages \
    openai \
    sentence-transformers \
    torch \
    requests \
    pdf2image \
    PyPDF2
```

### 4.3 ติดตั้ง Poppler (สำหรับ PDF to Image)

```bash
apt-get install -y poppler-utils
```

---

## 5. การทำ OCR ด้วย OpenAI Vision

### 5.1 OCR Script สำหรับ PDF

สร้างไฟล์ `ocr_bot.py`:

```python
#!/usr/bin/env python3
"""
OCR Bot - ใช้ OpenAI Vision API สำหรับ OCR PDF
"""

import os
import base64
import requests
from openai import OpenAI
from pdf2image import convert_from_path

# Config
PDF_PATH = "/path/to/your/document.pdf"
OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")

client = OpenAI(api_key=OPENAI_API_KEY)

def encode_image(image_path):
    """แปลง image เป็น base64"""
    with open(image_path, "rb") as f:
        return base64.b64encode(f.read()).decode('utf-8')

def ocr_page(image_path, page_num):
    """OCR หน้าเดียวด้วย OpenAI"""
    print(f"  🔍 OCR page {page_num}...")
    
    base64_image = encode_image(image_path)
    
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{
            "role": "user",
            "content": [
                {
                    "type": "text",
                    "text": "Extract ALL text from this page. Return in Thai language."
                },
                {
                    "type": "image_url",
                    "image_url": {
                        "url": f"data:image/png;base64,{base64_image}"
                    }
                }
            ]
        }],
        max_tokens=4000
    )
    
    return response.choices[0].message.content

def process_pdf(start_page=1, end_page=10):
    """ประมวลผล PDF ทีละช่วง"""
    images = convert_from_path(PDF_PATH, first_page=start_page, last_page=end_page)
    
    for i, img in enumerate(images):
        page_num = start_page + i
        img_path = f"/tmp/page_{page_num}.png"
        img.save(img_path, "PNG")
        
        text = ocr_page(img_path, page_num)
        print(f"  ✅ Page {page_num}: {len(text)} chars")
        
        # บันทึก text หรือส่งไป Qdrant
        with open(f"/tmp/ocr_{page_num}.txt", "w", encoding="utf-8") as f:
            f.write(text)
        
        os.remove(img_path)

if __name__ == "__main__":
    process_pdf(1, 10)  # ประมวลผลหน้า 1-10
```

### 5.2 วิธีใช้

```bash
# รัน OCR
python3 ocr_bot.py
```

### 5.3 การแก้ไข Rate Limit

ถ้าเจอ error 429 (Rate limit):

```python
import time

def ocr_page_with_retry(image_path, page_num, max_retries=3):
    """OCR พร้อม retry เมื่อ rate limit"""
    for attempt in range(max_retries):
        try:
            return ocr_page(image_path, page_num)
        except Exception as e:
            if "429" in str(e):
                wait_time = (attempt + 1) * 10  # รอ 10, 20, 30 วินาที
                print(f"  ⚠️ Rate limit, waiting {wait_time}s...")
                time.sleep(wait_time)
            else:
                raise
    return None
```

---

## 6. การทำ Embedding และเก็บใน Qdrant

### 6.1 Embedding with Sentence-Transformers

```python
#!/usr/bin/env python3
"""
RAG Agent - เพิ่มเอกสารเข้า Qdrant พร้อม Embedding
"""

import os
import requests
from sentence_transformers import SentenceTransformer

# Config
QDRANT_URL = "http://YOUR-QDRANT-IP:6333"
QDRANT_API_KEY = "your-api-key-here"
COLLECTION = "MYW-Knowledge"

# โหลด model
model = SentenceTransformer('paraphrase-multilingual-MiniLM-L12-v2')

def add_document(text, metadata=None):
    """เพิ่มเอกสารเข้า Qdrant"""
    # สร้าง embedding
    embedding = model.encode(text).tolist()
    
    payload = {
        "ids": [int(os.urandom(4).hex(), 16)],
        "points": [{
            "id": int(os.urandom(4).hex(), 16),
            "vector": embedding,
            "payload": {
                "text": text,
                **(metadata or {})
            }
        }]
    }
    
    # ส่งไป Qdrant (ใช้ PUT)
    response = requests.put(
        f"{QDRANT_URL}/collections/{COLLECTION}/points",
        headers={"Authorization": f"Bearer {QDRANT_API_KEY}"},
        json=payload
    )
    
    return response.json()

def search_document(query, limit=5):
    """ค้นหาเอกสาร"""
    embedding = model.encode(query).tolist()
    
    payload = {
        "vector": embedding,
        "limit": limit,
        "with_payload": True
    }
    
    response = requests.post(
        f"{QDRANT_URL}/collections/{COLLECTION}/points/search",
        headers={"Authorization": f"Bearer {QDRANT_API_KEY}"},
        json=payload
    )
    
    return response.json()

# ตัวอย่างการใช้
if __name__ == "__main__":
    # เพิ่มเอกสาร
    result = add_document(
        "คู่มือดำเนินงาน โครงการแก้ไขปัญหาความยากจน กข.คจ.",
        {"source": "manual", "page": 1}
    )
    print(result)
    
    # ค้นหา
    results = search_document("ขั้นตอนการปล่อยกู้")
    print(results)
```

### 6.2 Embedding with OpenAI

```python
#!/usr/bin/env python3
"""
RAG Agent - ใช้ OpenAI Embedding
"""

import os
import requests
from openai import OpenAI

# Config
QDRANT_URL = "http://YOUR-QDRANT-IP:6333"
QDRANT_API_KEY = "your-api-key-here"
COLLECTION = "MYW-Knowledge"

client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

def get_embedding(text):
    """สร้าง embedding ด้วย OpenAI"""
    response = client.embeddings.create(
        model="text-embedding-3-small",
        input=text
    )
    return response.data[0].embedding

def add_document(text, metadata=None):
    """เพิ่มเอกสาร"""
    embedding = get_embedding(text)
    
    import time
    payload = {
        "ids": [int(time.time() * 1000)],
        "points": [{
            "id": int(time.time() * 1000),
            "vector": embedding,
            "payload": {
                "text": text,
                **(metadata or {})
            }
        }]
    }
    
    response = requests.put(
        f"{QDRANT_URL}/collections/{COLLECTION}/points",
        headers={"Authorization": f"Bearer {QDRANT_API_KEY}"},
        json=payload
    )
    return response.json()

def search_document(query, limit=5):
    """ค้นหาเอกสาร"""
    embedding = get_embedding(query)
    
    payload = {
        "vector": embedding,
        "limit": limit,
        "with_payload": True
    }
    
    response = requests.post(
        f"{QDRANT_URL}/collections/{COLLECTION}/points/search",
        headers={"Authorization": f"Bearer {QDRANT_API_KEY}"},
        json=payload
    )
    return response.json()
```

---

## 7. การค้นหาข้อมูล

### 7.1 ค้นหาด้วย RAG Agent

```python
#!/usr/bin/env python3
"""
RAG Search - ค้นหาข้อมูลจาก Knowledge Base
"""

import os
import requests
from sentence_transformers import SentenceTransformer

QDRANT_URL = "http://YOUR-QDRANT-IP:6333"
QDRANT_API_KEY = "your-api-key-here"
COLLECTION = "MYW-Knowledge"

model = SentenceTransformer('paraphrase-multilingual-MiniLM-L12-v2')

def search(query, limit=5):
    """ค้นหาเอกสารที่เกี่ยวข้อง"""
    embedding = model.encode(query).tolist()
    
    payload = {
        "vector": embedding,
        "limit": limit,
        "with_payload": True
    }
    
    response = requests.post(
        f"{QDRANT_URL}/collections/{COLLECTION}/points/search",
        headers={"Authorization": f"Bearer {QDRANT_API_KEY}"},
        json=payload
    )
    
    results = response.json().get("result", [])
    
    print(f"\n🔍 ผลการค้นหา: '{query}'")
    print("=" * 50)
    
    for i, r in enumerate(results, 1):
        print(f"\n{i}. Score: {r.get('score', 0):.4f}")
        print(f"   Text: {r['payload']['text'][:200]}...")
        if 'page' in r['payload']:
            print(f"   Page: {r['payload']['page']}")
    
    return results

if __name__ == "__main__":
    # ทดสอบค้นหา
    search("ขั้นตอนการปล่อยกู้ กข.คจ")
```

### 7.2 วิธีใช้ใน OpenClaw

```python
# เรียกใช้จาก OpenClaw Agent
result = subprocess.run(
    ["python3", "/path/to/rag_search.py", "คำถาม"],
    capture_output=True,
    text=True
)
print(result.stdout)
```

---

## 8. การแก้ไขปัญหาที่พบบ่อย

### 8.1 Rate Limit Error (429)

**ปัญหา:** 
```
Rate limit reached for gpt-4o-mini... Limit 200000, Used 200000
```

**วิธีแก้:**
1. เพิ่ม delay ระหว่าง request:
```python
import time
time.sleep(2)  # รอ 2 วินาที
```

2. ลดขนาด batch:
```python
BATCH_SIZE = 5  # แทนที่ 10
```

3. อัปเกรด OpenAI plan (ถ้าใช้บ่อย)

### 8.2 Billing Not Active

**ปัญหา:**
```
Your account is not active, please check your billing details
```

**วิธีแก้:**
1. ไปที่ https://platform.openai.com/account/billing
2. เติมเงินขั้นต่ำ $5
3. รอ 5-10 นาทีให้ระบบอัปเดต

### 8.3 Qdrant Connection Error

**ปัญหา:**
```
Connection refused to YOUR-QDRANT-IP:6333
```

**วิธีแก้:**
1. ตรวจสอบว่า Qdrant container รันอยู่:
```bash
docker ps | grep qdrant
```

2. ตรวจสอบ firewall:
```bash
curl http://localhost:6333/
```

3. ตรวจสอบ port mapping ใน Docker

### 8.4 Dimension Mismatch

**ปัญหา:**
```
Vector dimension error: expected dim: 1536, got 384
```

**วิธีแก้:**
ตรวจสอบว่า vector size ตรงกับ model:
- **OpenAI text-embedding-3-small:** 1536 dimensions
- **sentence-transformers (multilingual):** 384 dimensions

```bash
# สร้าง collection ใหม่ด้วยขนาดที่ถูกต้อง
curl -X PUT "http://YOUR-QDRANT-IP:6333/collections/YOUR-COLLECTION" \
  -H "Content-Type: application/json" \
  -d '{
    "vectors": {
      "size": 384,  # หรือ 1536 ขึ้นอยู่กับ model
      "distance": "Cosine"
    }
  }'
```

### 8.5 Timeout Error

**ปัญหา:**
```
Request timed out before a response was generated
```

**วิธีแก้:**
1. เพิ่ม timeout ใน config:
```json
{
  "agents": {
    "defaults": {
      "timeoutSeconds": 600
    }
  }
}
```

2. หรือรันแบบ background:
```bash
nohup python3 ocr_bot.py > output.log 2>&1 &
```

---

## 9. Use Cases และตัวอย่าง

### 9.1 Use Case 1: OCR เอกสารคู่มือราชการ

**สถานการณ์:** ต้องการแปลง PDF คู่มือราชการ 128 หน้าให้เป็นข้อความที่ค้นหาได้

**ขั้นตอน:**

1. ดาวน์โหลด PDF มาเก็บในเครื่อง
2. รัน OCR Bot:
```bash
python3 ocr_bot.py --input manual.pdf --start 1 --end 128
```

3. ตรวจสอบผลลัพธ์:
```bash
ls /tmp/ocr_*.txt
```

### 9.2 Use Case 2: สร้าง Knowledge Base สำหรับองค์กร

**สถานการณ์:** ต้องการสร้างระบบ Q&A อัตโนัติจากเอกสารองค์กร

**ขั้นตอน:**

1. รวบรวมเอกสารทั้งหมด (PDF, Word, Text)
2. OCR ทุกเอกสาร
3. แบ่งเป็น chunks (500-1000 ตัวอักษร)
4. สร้าง embeddings และเก็บใน Qdrant
5. สร้าง Chatbot ที่ค้นจาก Qdrant

**ตัวอย่าง Code:**

```python
def process_document_to_knowledgebase(file_path, collection_name):
    """Process เอกสาร -> Knowledge Base"""
    
    # 1. OCR ถ้าเป็น PDF
    if file_path.endswith('.pdf'):
        text = ocr_pdf(file_path)
    else:
        with open(file_path, 'r', encoding='utf-8') as f:
            text = f.read()
    
    # 2. แบ่งเป็น chunks
    chunks = split_into_chunks(text, chunk_size=500)
    
    # 3. เพิ่มแต่ละ chunk เข้า Qdrant
    for i, chunk in enumerate(chunks):
        add_document(
            chunk,
            {
                "source": file_path,
                "chunk_id": i,
                "type": "knowledge"
            }
        )
    
    print(f"✅ เพิ่ม {len(chunks)} chunks เข้า {collection_name}")
```

### 9.3 Use Case 3: Chat with Documents

**สถานการณ์:** ถาม-ตอบกับเอกสาร

**ขั้นตอน:**

1. รับคำถามจาก user
2. ค้นหาจาก Qdrant
3. ส่ง context ให้ LLM ตอบ

```python
def chat_with_document(question):
    # 1. ค้นหา relevant documents
    results = search(question, limit=3)
    
    # 2. รวบรวม context
    context = "\n\n".join([r['payload']['text'] for r in results])
    
    # 3. ส่งให้ LLM ตอบ
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {
                "role": "system", 
                "content": "คุณเป็นผู้ช่วยที่ตอบคำถามจากเอกสารที่ให้มา"
            },
            {
                "role": "user",
                "content": f"Context:\n{context}\n\nQuestion: {question}"
            }
        ]
    )
    
    return response.choices[0].message.content
```

---

## 10. เครดิต

### ผู้พัฒนา

| ชื่อ | บทบาท |
|------|--------|
| **โจ้ พัฒนากร** | ผู้ออกแบบระบบ, ผู้ใช้งานหลัก |
| **น้องมิว (Mew)** | AI Assistant, ผู้พัฒนา Scripts |

### ขอบคุณ

- **OpenClaw Team** - สำหรับ AI Agent Platform
- **Qdrant Team** - สำหรับ Vector Database
- **OpenAI** - สำหรับ Vision และ Embedding APIs

---

## 📞 ติดต่อ

- **GitHub:** https://github.com/joe-cdd/openclaw-rag-setup
- **Issues:** https://github.com/joe-cdd/openclaw-rag-setup/issues

---

**หมายเหตุ:** คู่มือนี้จัดทำขึ้นเพื่อเป็นแนวทางสำหรับผู้ที่ต้องการตั้งค่าระบบ RAG ด้วยตนเอง หากพบปัญหาหรือมีข้อเสนอแนะ สามารถเปิด Issue ได้ที่ GitHub

---

*สร้างด้วย ❤️ โดย โจ้ พัฒนากร และ น้องมิว AI Assistant*
