# Silo — detailed usage

## Requirements

### Hardware
- GPU with 8GB+ VRAM (NVIDIA, AMD, or Metal on Mac)
  - 32B model: 12GB+ recommended
  - 14B model: 8GB+ sufficient
- CPU fallback — works on CPU but slower (~10-20s per response)
- 8GB+ RAM recommended

### Software
- [Ollama](https://ollama.com/download) installed and running
- VS Code 1.90+
- Python 3.8+ (for backend)

## Installation

### 1. Install Ollama
Download from [ollama.com/download](https://ollama.com/download).

### 2. Download the model
Choose based on your VRAM:

```bash
# 32B (recommended, 12GB+ VRAM)
ollama pull qwen2.5-coder:32b

# 14B (8GB+ VRAM)
ollama pull qwen2.5-coder:14b

# Test with smaller model (6GB VRAM)
ollama pull qwen2.5-coder:7b
```

### 3. Start the Silo backend

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

### 4. Install the VS Code extension
Search for **"Silo"** in VS Code Extensions or
[install from Marketplace](https://marketplace.visualstudio.com/items?itemName=danielmadridg.silo-local-ai).

### 5. Start coding
- Press `Ctrl+Shift+P` (or `Cmd+Shift+P`) → "Silo: Open Chat"
- Select code and press right-click → Refactor/Explain
- Type code and watch inline completions appear

## Usage examples

**Chat with your code**
```
You: "Why is this function slow?"
You: "Refactor it to use a set instead of a list"
You: "Explain what this algorithm does"
```

**Refactoring**
```
1. Select code in editor
2. Right-click → "Silo: Refactor Selection"
3. Type your instruction: "Convert to async"
4. Changes apply instantly
```

**Code explanation**
```
1. Select any code snippet
2. Right-click → "Silo: Explain Selection"
3. Silo explains it in the chat panel
```

**File analysis**
```
Press Cmd/Ctrl+Shift+P
Type "Silo: Analyze Current File"
Get instant bug detection and suggestions
```

## Configuration

**VS Code settings (`settings.json`)**
```json
{
  "silo.backendUrl": "http://127.0.0.1:8942",
  "silo.contextFiles": 5
}
```

**Backend configuration (`backend/config.py`)**
```python
# Change model
MODEL_NAME = "qwen2.5-coder:14b"  # or qwen2.5-coder:32b

# Adjust token limits
MAX_TOKENS_CHAT = 2048
MAX_TOKENS_COMPLETE = 256

# Change backend port
PORT = 8942
```

### Advanced configuration

**Use custom Ollama models** — edit `backend/config.py`:
```python
# Use Llama 2
MODEL_NAME = "llama2"

# Use Code Llama
MODEL_NAME = "codellama"

# Use your own Ollama-compatible server
OLLAMA_BASE_URL = "http://192.168.1.100:11434"
```

**Multi-user setup** — run backend on a server and point multiple VS Code instances:
```json
{
  "silo.backendUrl": "http://your-server:8942"
}
```

**Disable inline completions**
```json
{
  "silo.enableInlineCompletions": false
}
```

## Development

**Building the extension**
```bash
cd extension
npm install
npm run compile
npm run package  # Creates .vsix file for distribution
```

**Running backend in dev mode**
```bash
cd backend
source .venv/bin/activate
uvicorn main:app --reload --host 127.0.0.1 --port 8942
```

**Testing the API**
```bash
# Chat endpoint
curl -X POST http://127.0.0.1:8942/chat \
  -H "Content-Type: application/json" \
  -d '{"message": "Write a hello world in Python", "history": [], "file_context": ""}'

# Health check
curl http://127.0.0.1:8942/health
```

## Troubleshooting

**Ollama not connecting**
```bash
# Check if Ollama is running
curl http://127.0.0.1:11434/api/tags

# Restart Ollama
# Windows: Quit from taskbar, reopen
# Mac: brew services restart ollama
# Linux: systemctl restart ollama
```

**Backend won't start**
```bash
# Check if port 8942 is in use
netstat -tlnp | grep 8942  # Linux/Mac
netstat -ano | findstr :8942  # Windows

# Change port in config.py
PORT = 8943
```

**Slow responses**
- Switch to a smaller model: `ollama pull qwen2.5-coder:14b`
- Check GPU usage: `nvidia-smi` (should show high utilization)
- Reduce context files: `"silo.contextFiles": 3`

**Model won't download**
```bash
# Download manually
ollama pull qwen2.5-coder:32b

# Check storage
# Models stored at: ~/.ollama/models (Mac/Linux) or C:\Users\<user>\.ollama\models (Windows)
```

## Performance notes

| Model | VRAM | Speed | Quality |
|-------|------|-------|---------|
| qwen2.5-coder:7b | 6GB | ~2-3s | Good |
| qwen2.5-coder:14b | 8GB | ~3-5s | Excellent |
| qwen2.5-coder:32b | 12GB | ~5-10s | Best |

All times are per response and depend on GPU, prompt length, and output tokens.

### Model recommendations

- Best all-around — `qwen2.5-coder:32b` (if you have 12GB+ VRAM)
- Balanced — `qwen2.5-coder:14b` (8-12GB VRAM)
- Budget/laptop — `qwen2.5-coder:7b` (6GB+ VRAM)
- Alternative — `codellama:34b` (similar performance, more code-focused)
