# Silo Backend Setup Guide

## Environment Configuration

Silo backend uses environment variables for configuration. **Never commit actual configuration to the repository.**

### 1. Create a `.env` file

Copy `.env.example` to create your `.env` file:

```bash
cp .env.example .env
```

### 2. Edit your configuration

Edit `.env` and customize as needed:

```
# Ollama connection
OLLAMA_BASE_URL=http://127.0.0.1:11434
MODEL_NAME=qwen2.5-coder:32b

# Server configuration
HOST=127.0.0.1
PORT=8942

# Token limits
MAX_TOKENS_CHAT=2048
MAX_TOKENS_COMPLETE=256
```

### Configuration Options

#### OLLAMA_BASE_URL
- **Default:** `http://127.0.0.1:11434`
- **Description:** URL of the Ollama service
- **Change when:** Running Ollama on a different machine
- **Example:** `http://192.168.1.100:11434` (remote server)

#### MODEL_NAME
- **Default:** `qwen2.5-coder:32b`
- **Description:** The Ollama model to use
- **Change when:** You want a different model size or type
- **Examples:**
  - `qwen2.5-coder:32b` (largest, best quality)
  - `qwen2.5-coder:14b` (balanced)
  - `qwen2.5-coder:7b` (smallest, fastest)
  - `codellama:34b` (alternative)

#### HOST
- **Default:** `127.0.0.1`
- **Description:** Server bind address
- **Change when:** You want to access from other machines
- **Examples:**
  - `127.0.0.1` (localhost only, safe)
  - `0.0.0.0` (all interfaces, use with caution)

#### PORT
- **Default:** `8942`
- **Description:** Server port
- **Change when:** Port 8942 is already in use
- **Example:** `8943`, `9000`, etc.

#### MAX_TOKENS_CHAT
- **Default:** `2048`
- **Description:** Maximum tokens for chat responses
- **Change when:** You want shorter/longer responses
- **Range:** 256-4096

#### MAX_TOKENS_COMPLETE
- **Default:** `256`
- **Description:** Maximum tokens for code completion
- **Change when:** You want shorter/longer completions
- **Range:** 64-512

## First-Time Setup

### 1. Ensure Ollama is Running

```bash
# Check if Ollama is running
curl http://127.0.0.1:11434/api/tags

# If it fails, start Ollama:
# - Windows: Click Ollama app
# - Mac: brew services start ollama
# - Linux: systemctl start ollama
```

### 2. Download Your Model

Choose a model based on your VRAM:

```bash
# 32B model (12GB+ VRAM recommended)
ollama pull qwen2.5-coder:32b

# 14B model (8-12GB VRAM)
ollama pull qwen2.5-coder:14b

# 7B model (6GB+ VRAM)
ollama pull qwen2.5-coder:7b
```

### 3. Create Python Virtual Environment

```bash
# Create venv
python3 -m venv .venv

# Activate
# Windows:
.venv\Scripts\activate
# Mac/Linux:
source .venv/bin/activate
```

### 4. Install Dependencies

```bash
pip install fastapi "uvicorn[standard]" httpx pydantic sse-starlette aiofiles
```

### 5. Create `.env` file

```bash
cp .env.example .env
# Edit .env if you want to customize values
```

### 6. Start the Server

```bash
uvicorn main:app --host 127.0.0.1 --port 8942
```

You should see:
```
INFO:     Uvicorn running on http://127.0.0.1:8942
```

## Testing the Backend

### Health Check

```bash
curl http://127.0.0.1:8942/health
```

Expected response:
```json
{"status": "ok", "model": "qwen2.5-coder:32b", "model_ready": true}
```

### Chat Endpoint

```bash
curl -X POST http://127.0.0.1:8942/chat \
  -H "Content-Type: application/json" \
  -d '{
    "message": "Write a hello world in Python",
    "history": [],
    "file_context": ""
  }'
```

### Completions Endpoint

```bash
curl -X POST http://127.0.0.1:8942/completions \
  -H "Content-Type: application/json" \
  -d '{
    "code_before": "def hello():",
    "code_after": "",
    "language": "python"
  }'
```

## Environment Variable Precedence

Silo checks for config in this order:
1. Environment variable (highest priority)
2. `.env` file in current directory
3. Default value in `config.py`

This means you can set variables in multiple ways:

### Method 1: .env file (Recommended)
```bash
# Create .env
OLLAMA_BASE_URL=http://127.0.0.1:11434
MODEL_NAME=qwen2.5-coder:32b
```

### Method 2: Global Environment Variables

**Windows PowerShell:**
```powershell
$env:OLLAMA_BASE_URL="http://192.168.1.100:11434"
$env:MODEL_NAME="qwen2.5-coder:14b"
python -m uvicorn main:app
```

**Mac/Linux:**
```bash
export OLLAMA_BASE_URL="http://192.168.1.100:11434"
export MODEL_NAME="qwen2.5-coder:14b"
uvicorn main:app --host 127.0.0.1 --port 8942
```

## Multi-Machine Setup

### Run Backend on Server
```bash
# On server machine (192.168.1.100)
OLLAMA_BASE_URL=http://127.0.0.1:11434 \
uvicorn main:app --host 0.0.0.0 --port 8942
```

### Point VS Code Extension to Server
```json
{
  "silo.backendUrl": "http://192.168.1.100:8942"
}
```

## Troubleshooting

### "Cannot connect to Ollama"
```bash
# Check if Ollama is running
curl http://127.0.0.1:11434/api/tags

# If failing, check:
# 1. Ollama is started (see First-Time Setup)
# 2. OLLAMA_BASE_URL is correct
# 3. Firewall isn't blocking port 11434
```

### "Model not found"
```bash
# List available models
ollama list

# Pull the model you configured
ollama pull qwen2.5-coder:32b

# Or change MODEL_NAME in .env to use a model you have
```

### Port 8942 already in use
```bash
# Find what's using port 8942
# Windows:
netstat -ano | findstr :8942

# Mac/Linux:
lsof -i :8942

# Kill the process or change PORT in .env
PORT=8943
uvicorn main:app --host 127.0.0.1 --port 8943
```

### Slow responses
1. Check GPU usage: `nvidia-smi` or equivalent
2. Use a smaller model: `MODEL_NAME=qwen2.5-coder:14b`
3. Reduce `MAX_TOKENS_CHAT` in `.env`
4. Check if CPU is at 100% (might be CPU-bound)

## Security Notes

- ✅ `.env` is in `.gitignore` — it will never be committed
- ✅ Localhost default (`127.0.0.1`) is safe
- ✅ No API keys needed — Ollama is local
- ✅ Only change to `0.0.0.0` if behind a firewall
- ✅ Always keep production `.env` files private

## Never Do This ❌

```python
# ❌ DON'T hardcode config
MODEL_NAME = "qwen2.5-coder:32b"
OLLAMA_BASE_URL = "http://192.168.1.100:11434"

# ❌ DON'T commit .env files
git add .env

# ❌ DON'T bind to 0.0.0.0 without firewall
HOST="0.0.0.0"  # Exposes to entire network
```

## Always Do This ✅

```python
# ✅ DO use environment variables
MODEL_NAME = os.getenv("MODEL_NAME", "qwen2.5-coder:32b")

# ✅ DO use .env.example as template
cp .env.example .env

# ✅ DO keep .env private
# It's already in .gitignore
```

---

**Questions?** Check the [README.md](../README.md) for more details about Silo features.
