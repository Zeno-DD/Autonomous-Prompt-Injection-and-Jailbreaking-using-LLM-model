## 1. Chuẩn Bị Môi Trường

### 1.1 Kali VM — kiểm tra Python

```bash
python3 --version   # cần >= 3.10
pip3 --version
```

Nếu chưa có pip:

```bash
sudo apt update && sudo apt install python3-pip -y
```

### 1.2 Windows — cài VSCode SSH extension

Trên Windows, mở VSCode → Extensions → cài **Remote - SSH** → kết nối vào Kali:

```
ssh kali@<IP_KALI_VM>
```

> Từ đây toàn bộ code được viết trên Windows VSCode nhưng chạy trực tiếp trên Kali.
> 

---

## 2. Cài Đặt Ollama + Llama-3

Chạy trên **Kali VM**:

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

Khởi động service:

```bash
ollama serve &
```

Pull model dolphin-llama3:

```bash
ollama pull dolphin-llama3
```

Kiểm tra:

```bash
ollama list
# NAME                      ID              SIZE    MODIFIED
# dolphin-llama3:latest   ...             4.7 GB  ...
```

Test nhanh:

```bash
ollama run dolphin-llama3 "Hello, who are you?"
```

> **Lưu ý:** Nếu Kali chạy root mặc định:
`OLLAMA_HOST=0.0.0.0 ollama serve`
> 

---

## 3. Cài Đặt LiteLLM & Python Dependencies

```bash
pip install litellm groq google-generativeai \
            flask python-dotenv pandas \
            jupyter matplotlib seaborn \
            --break-system-packages
```

Tạo file `requirements.txt` để nhóm dùng chung:

```
litellm>=1.35.0
groq>=0.9.0
google-generativeai>=0.5.0
flask>=3.0.0
python-dotenv>=1.0.0
pandas>=2.0.0
jupyter>=1.0.0
matplotlib>=3.8.0
seaborn>=0.13.0
```

---

## 4. Cấu Hình API Keys

### 4.1 Lấy API key

| Service | Lấy key tại | Chi phí |
| --- | --- | --- |
| **Groq** | https://console.groq.com | Free tier |
| **Gemini** | https://aistudio.google.com | Free tier |

### 4.2 Tạo file `.env`

```bash
# .env
GROQ_API_KEY=gsk_xxxxxxxxxxxxxxxxxxxxxxxx
GEMINI_API_KEY=AIzaxxxxxxxxxxxxxxxxxxxxxxxx

# Ollama chạy local, không cần key
OLLAMA_BASE_URL=http://localhost:11434
```

> ⚠️ Thêm `.env` vào `.gitignore` — không commit key lên GitHub.
> 

```bash
echo ".env" >> .gitignore
```

---

## 5. Cấu Trúc Thư Mục Dự Án

```
red-teaming-llm/
│
├── .env                        # API keys (không commit)
├── .gitignore
├── requirements.txt
│
├── data/
│   ├── harmful_behaviors.csv   # Seed dataset — đã tải sẵn từ HuggingFace
│   ├── harmful_strings.csv     # Harmful strings — đã tải sẵn từ HuggingFace
│   └── results/                # Kết quả thực nghiệm (JSON)
│
├── pipeline/
│   ├── __init__.py
│   ├── dataset.py              # Tải & xử lý dataset từ HuggingFace
│   ├── attacker.py             # Automation Engine (sinh biến thể tấn công)
│   ├── target.py               # LiteLLM wrapper gọi target models
│   ├── evaluator.py            # LLM-as-Judge + Keyword matching, tính ASR
│   └── runner.py               # Orchestrator chạy toàn bộ pipeline
│
├── demo_app/
│   └── app.py                  # Flask app demo Prompt Injection thực tế
│
└── analysis/
    └── asr_analysis.ipynb      # Jupyter notebook phân tích kết quả
```
