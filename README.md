<div align="center">

# Silo

**A fully local AI coding assistant for VS Code — works like Claude Code, runs 100% on your machine.**

</div>

---

## What it does

Silo brings AI-assisted coding — chat, inline completions, refactoring, and code analysis — to VS
Code without sending your code anywhere. No API keys, no data leaving your computer, no
subscriptions. It's powered by **Qwen2.5-Coder-32B** running on [Ollama](https://ollama.com) using
your own GPU.

## Features

- **Chat with full project context** — access to all your open files, streaming responses, session memory
- **Inline code completion** — AI suggestions as you type, Tab to accept, learns from your codebase patterns
- **Refactoring & analysis** — select code + an instruction and Silo rewrites it; detects bugs, performance issues, and code smells in a file
- **Code explanation** — understand complex code on demand
- **VS Code-native** — right-click menu, command palette, zero setup beyond Ollama

## Installation / Usage

1. Install [Ollama](https://ollama.com/download) and pull a model: `ollama pull qwen2.5-coder:32b`
2. Start the backend: `cd silo && start-backend.bat` (Windows) — see [docs/USAGE.md](docs/USAGE.md) for Mac/Linux
3. Install the extension — search **"Silo"** in VS Code Extensions, or [get it from the Marketplace](https://marketplace.visualstudio.com/items?itemName=danielmadridg.silo-local-ai)
4. Open the command palette (`Ctrl+Shift+P`) → **"Silo: Open Chat"**

Full requirements, configuration, architecture, troubleshooting, and performance benchmarks per model: **[docs/USAGE.md](docs/USAGE.md)**.

## Architecture

- **Backend (Python + FastAPI)** — Ollama integration, context management, prompt building, SSE streaming
- **Extension (TypeScript)** — chat webview, inline completion provider, command handlers, context collector
- **Model** — Qwen2.5-Coder-32B-Instruct, running locally via Ollama

## Tech stack

Python, FastAPI, uvicorn, sse-starlette, TypeScript, VS Code Extension API, [Ollama](https://ollama.com), [Qwen2.5-Coder](https://github.com/QwenLM/Qwen)

## License

See [LICENSE](LICENSE).
