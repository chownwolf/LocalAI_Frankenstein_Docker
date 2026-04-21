# LocalAI

A local AI stack running Ollama + Open WebUI with GPU acceleration, Traefik reverse proxy, and on-demand model setup via Docker Compose profiles.

## Services

| Service | Port | Description |
|---|---|---|
| open-webui | 8080 | Web UI for chatting with models |
| ollama | 11434 | LLM inference backend |
| traefik | 8881 (http), 8882 (dashboard) | Reverse proxy |

## Requirements

- Docker + Docker Compose
- NVIDIA GPU with drivers installed (configured for RTX 5000, 16GB VRAM)
- NVIDIA Container Toolkit

## Quick Start

```bash
# Start core services
docker compose up -d

# Access the UI
open http://localhost:8080
```

## Model Profiles

Model setup services run on-demand via profiles. They download GGUFs, create Modelfiles, and register models with Ollama. Files persist in the `models` volume.

```bash
docker compose --profile <profile> up --force-recreate
```

### Individual Models

| Profile | Model | Size | Notes |
|---|---|---|---|
| `qwen25-14b` | Qwen 2.5 14B | ~9GB | Baseline coding/JSON |
| `qwen25-coder` | Qwen 2.5 Coder 14B | ~9GB | Legacy dedicated coder |
| `qwen3-coder` | Qwen3 Coder 30B-A3B | ~8GB | MOE, fast (3B active) |
| `qwen3-coder-abliterated` | Qwen3 Coder 30B-A3B Abliterated | ~16GB | Uncensored MOE coder |
| `qwen3-coder-unsloth` | Qwen3 Coder 30B-A3B (Unsloth) | ~15GB | SOTA, native tool calling |
| `watt-tool-8b` | Watt-Tool 8B | ~6GB | Purpose-built tool calling |
| `groq-tool-8b` | Llama-3-Groq-8B-Tool-Use | ~6GB | Groq fine-tune for tools |

### Group Profiles

| Profile | Includes |
|---|---|
| `baseline` | All Q1 baseline models |
| `coder` | All coder models |
| `tool` | All tool-calling models |
| `specialized` | All Q3 specialized models |
| `all` | Everything (~100GB+) |

```bash
# List all available profiles
docker compose config --profiles
```

## Ollama Configuration

The Ollama service is tuned for high-VRAM local use:

| Variable | Value | Effect |
|---|---|---|
| `OLLAMA_FLASH_ATTENTION` | 1 | Faster attention |
| `OLLAMA_KEEP_ALIVE` | 24h | Models stay loaded |
| `OLLAMA_MAX_LOADED_MODELS` | 2 | Parallel model slots |
| `OLLAMA_NUM_PARALLEL` | 4 | Concurrent requests |

## Volumes

| Volume | Purpose |
|---|---|
| `models` | Downloaded GGUFs and Modelfiles |
| `ollama` | Ollama model registry |
| `data` | Open WebUI data |
| `open-webui` | Open WebUI backend data |

## Notes

- **Secret key**: `WEBUI_SECRET_KEY` is a demo value — generate a real one for any non-local deployment: `openssl rand -base64 32`
- **Watchtower**: commented out but included for auto-update support
- The external network `localai_ollama-net` must exist before starting
