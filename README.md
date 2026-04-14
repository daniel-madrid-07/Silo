# Silo — Local AI Coding Assistant for VS Code

A powerful, fully local AI coding assistant for VS Code that works like Claude Code but runs 100% on your machine. No API keys, no data leaving your computer, no subscriptions.

Powered by **Qwen2.5-Coder-32B** running on [Ollama](https://ollama.com) with your GPU.

## 🎯 Features

### Chat & Conversation
- **Full project context** — Chat with access to all your open files
- **Streaming responses** — Real-time AI answers
- **Session memory** — Maintains conversation history within a session
- **Code-aware** — Understands code structure and intent

### Code Completion
- **Inline completions** — AI suggestions as you type
- **Tab to accept** — Smooth integration with VS Code editing
- **Context-aware** — Learns from your codebase patterns

### Code Analysis & Refactoring
- **File analysis** — Detect bugs, performance issues, and code smells
- **Refactor selection** — Select code + give instruction = automatic refactoring
- **Code explanation** — Understand complex code with AI analysis
- **Quality suggestions** — Get professional refactoring advice

### Developer Experience
- **Right-click menu** — Quick access to refactor/explain
- **Command palette** — All features accessible via Cmd/Ctrl+Shift+P
- **Configurable backend** — Choose your model and AI settings
- **Zero setup needed** — Works with any Ollama-compatible model

## 📋 Requirements

### Hardware
- **GPU with 8GB+ VRAM** (NVIDIA, AMD, or Metal on Mac)
  - 32B model: 12GB+ recommended
  - 14B model: 8GB+ sufficient
- **CPU fallback** — Works on CPU but slower (~10-20s per response)
- **8GB+ RAM** recommended

### Software
- [Ollama](https://ollama.com/download) installed and running
- VS Code 1.90+
- Python 3.8+ (for backend)

## 🚀 Quick Start

### 1. Install Ollama
Download from [ollama.com/download](https://ollama.com/download)

### 2. Download the Model
Choose based on your VRAM:

```bash
# 32B (recommended, 12GB+ VRAM)
ollama pull qwen2.5-coder:32b

# 14B (8GB+ VRAM)
ollama pull qwen2.5-coder:14b

# Test with smaller model (6GB VRAM)
ollama pull qwen2.5-coder:7b
```

### 3. Start the Silo Backend

**Option A: Windows (easiest)**
```bash
cd silo
start-backend.bat
```

**Option B: Mac/Linux**
```bash
cd silo/backend

# Create virtual environment
python3 -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install dependencies
pip install fastapi "uvicorn[standard]" httpx pydantic sse-starlette aiofiles

# Start server
uvicorn main:app --host 127.0.0.1 --port 8942
```

### 4. Install VS Code Extension
Search for **"Silo"** in VS Code Extensions or [install from Marketplace](https://marketplace.visualstudio.com/items?itemName=danielmadridg.silo-local-ai)

### 5. Start Coding!
- Press `Ctrl+Shift+P` (or `Cmd+Shift+P`) → "Silo: Open Chat"
- Select code and press right-click → Refactor/Explain
- Type code and watch inline completions appear

## 🎮 Usage Examples

### Chat with Your Code
```
You: "Why is this function slow?"
You: "Refactor it to use a set instead of a list"
You: "Explain what this algorithm does"
```

### Refactoring
```
1. Select code in editor
2. Right-click → "Silo: Refactor Selection"
3. Type your instruction: "Convert to async"
4. Changes apply instantly
```

### Code Explanation
```
1. Select any code snippet
2. Right-click → "Silo: Explain Selection"
3. Silo explains it in the chat panel
```

### File Analysis
```
Press Cmd/Ctrl+Shift+P
Type "Silo: Analyze Current File"
Get instant bug detection and suggestions
```

## ⚙️ Configuration

### VS Code Settings (`settings.json`)

```json
{
  "silo.backendUrl": "http://127.0.0.1:8942",
  "silo.contextFiles": 5
}
```

### Backend Configuration (`backend/config.py`)

```python
# Change model
MODEL_NAME = "qwen2.5-coder:14b"  # or qwen2.5-coder:32b

# Adjust token limits
MAX_TOKENS_CHAT = 2048
MAX_TOKENS_COMPLETE = 256

# Change backend port
PORT = 8942
```

## 🏗️ Architecture

### Backend (Python + FastAPI)
- **Model runner** — Ollama API integration
- **Context management** — Collects file context from VS Code
- **Prompt builder** — Crafts optimized prompts for the LLM
- **Streaming** — Real-time token streaming via SSE
- **Router system** — Chat, completions, and analysis endpoints

### Extension (TypeScript)
- **Chat webview** — Interactive chat panel in VS Code
- **Inline provider** — Completion suggestions while typing
- **Command handlers** — Refactor, explain, analyze operations
- **Context collector** — Gathers open files and selection for context
- **API client** — Communicates with backend via HTTP

### Model
- **Qwen2.5-Coder-32B-Instruct** — SOTA open-source coding LLM
- **Optimized for** — Code completion, explanation, refactoring, debugging
- **Supported languages** — Python, JavaScript, TypeScript, Go, Rust, C++, Java, etc.

## 📊 Performance Notes

| Model | VRAM | Speed | Quality |
|-------|------|-------|---------|
| qwen2.5-coder:7b | 6GB | ~2-3s | Good |
| qwen2.5-coder:14b | 8GB | ~3-5s | Excellent |
| qwen2.5-coder:32b | 12GB | ~5-10s | Best |

**Note:** All times are per response. Depends on GPU, prompt length, and output tokens.

## 🔧 Troubleshooting

### Ollama not connecting
```bash
# Check if Ollama is running
curl http://127.0.0.1:11434/api/tags

# Restart Ollama
# Windows: Quit from taskbar, reopen
# Mac: brew services restart ollama
# Linux: systemctl restart ollama
```

### Backend won't start
```bash
# Check if port 8942 is in use
netstat -tlnp | grep 8942  # Linux/Mac
netstat -ano | findstr :8942  # Windows

# Change port in config.py
PORT = 8943
```

### Slow responses
- Switch to a smaller model: `ollama pull qwen2.5-coder:14b`
- Check GPU usage: `nvidia-smi` (should show high utilization)
- Reduce context files: `"silo.contextFiles": 3`

### Model won't download
```bash
# Download manually
ollama pull qwen2.5-coder:32b

# Check storage
# Models stored at: ~/.ollama/models (Mac/Linux) or C:\Users\<user>\.ollama\models (Windows)
```

## 🛠️ Development

### Building the extension
```bash
cd extension
npm install
npm run compile
npm run package  # Creates .vsix file for distribution
```

### Running backend in dev mode
```bash
cd backend
source .venv/bin/activate
uvicorn main:app --reload --host 127.0.0.1 --port 8942
```

### Testing the API
```bash
# Chat endpoint
curl -X POST http://127.0.0.1:8942/chat \
  -H "Content-Type: application/json" \
  -d '{"message": "Write a hello world in Python", "history": [], "file_context": ""}'

# Health check
curl http://127.0.0.1:8942/health
```

## 🚀 Advanced Configuration

### Use custom Ollama models
Edit `backend/config.py`:
```python
# Use Llama 2
MODEL_NAME = "llama2"

# Use Code Llama
MODEL_NAME = "codellama"

# Use your own Ollama-compatible server
OLLAMA_BASE_URL = "http://192.168.1.100:11434"
```

### Multi-user setup
Run backend on a server and point multiple VS Code instances:
```json
{
  "silo.backendUrl": "http://your-server:8942"
}
```

### Disable inline completions
```json
{
  "silo.enableInlineCompletions": false
}
```

## 📚 Model Recommendations

- **Best all-around** — `qwen2.5-coder:32b` (if you have 12GB+ VRAM)
- **Balanced** — `qwen2.5-coder:14b` (8-12GB VRAM)
- **Budget/Laptop** — `qwen2.5-coder:7b` (6GB+ VRAM)
- **Alternative** — `codellama:34b` (similar performance, more code-focused)

## 🤝 Contributing

Contributions welcome! Areas of interest:
- New analysis backends
- Additional language support
- Performance optimizations
- UI/UX improvements
- Documentation

## 📄 License

MIT License — Use freely, modify, distribute.

## 🔗 Links

- [GitHub](https://github.com/danielmadridg/silo)
- [Ollama](https://ollama.com)
- [Qwen Models](https://github.com/QwenLM/Qwen)
- [VS Code Extension API](https://code.visualstudio.com/api)

---

**No cloud. No subscriptions. Pure local AI power.** ⚡
