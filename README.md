# Local Ollama Server

> Part of the [P16 GPU Server](https://github.com/profzeller/p16-server-setup) ecosystem

Dedicated LLM inference server for text generation on a local GPU.

Optimized for 16GB VRAM GPUs (RTX 4080, laptop 4090, etc.) to run large language models.

## Quick Start

```bash
# Clone and configure
git clone https://github.com/profzeller/local-ollama-server.git
cd local-ollama-server
cp .env.example .env

# Edit .env to customize settings (optional)
nano .env

# Start the server
docker compose up -d

# Pull a model
docker exec ollama ollama pull mistral:7b
```

## Configuration

All settings are in `.env`:

```bash
# Default model to use
OLLAMA_DEFAULT_MODEL=mistral:7b

# Performance settings
OLLAMA_NUM_PARALLEL=2
OLLAMA_MAX_LOADED_MODELS=1
OLLAMA_KEEP_ALIVE=300

# Network
OLLAMA_HOST=0.0.0.0
OLLAMA_PORT=11434
```

## Recommended Models for 16GB VRAM

| Model | Size | Speed | Notes |
|-------|------|-------|-------|
| `mistral:7b` | 7B | Fast | Great all-around |
| `qwen2.5:7b` | 7B | Fast | Multilingual |
| `llama3.2:3b` | 3B | Very fast | Smaller but capable |
| `phi3:mini` | 3.8B | Very fast | Efficient |
| `gemma2:9b` | 9B | Medium | Good quality |

### For 24GB+ VRAM

| Model | Size | Notes |
|-------|------|-------|
| `qwen2.5:14b` | 14B | Excellent quality |
| `llama3.1:8b` | 8B | Good balance |
| `deepseek-r1:14b` | 14B | Great reasoning |

## Pulling Models

```bash
# Pull a model
docker exec ollama ollama pull mistral:7b

# List installed models
docker exec ollama ollama list

# Remove a model
docker exec ollama ollama rm <model-name>
```

## API Usage

### Chat Completion

```bash
curl http://localhost:11434/api/chat -d '{
  "model": "mistral:7b",
  "messages": [
    {"role": "user", "content": "Hello!"}
  ]
}'
```

### OpenAI-Compatible API

```bash
curl http://localhost:11434/v1/chat/completions -d '{
  "model": "mistral:7b",
  "messages": [{"role": "user", "content": "Hello!"}]
}'
```

### Python

```python
import requests

response = requests.post("http://localhost:11434/api/generate", json={
    "model": "mistral:7b",
    "prompt": "Write a wellness tip",
    "stream": False
})
print(response.json()["response"])
```

## Configuration Reference

| Variable | Description | Default |
|----------|-------------|---------|
| `OLLAMA_DEFAULT_MODEL` | Model to pull on setup | mistral:7b |
| `OLLAMA_NUM_PARALLEL` | Concurrent requests | 4 |
| `OLLAMA_MAX_LOADED_MODELS` | Models in memory | 1 |
| `OLLAMA_KEEP_ALIVE` | Keep model loaded | 5m |
| `OLLAMA_FLASH_ATTENTION` | Enable flash attention | 1 |
| `OLLAMA_HOST` | Bind address | 0.0.0.0 |
| `OLLAMA_PORT` | Port | 11434 |

## Performance Tuning

### Container Optimizations (Built-in)

The docker-compose.yml includes these optimizations:

- **`shm_size: 4g`** - Shared memory for large model tensors (prevents OOM on 13B+ models)
- **`ulimits.nofile: 65536`** - Higher file descriptor limits for concurrent connections
- **Flash Attention** - Faster inference on RTX 30/40 series GPUs

### Tuning for Your Hardware

**16GB VRAM (RTX 4080, laptop 4090):**
```bash
OLLAMA_NUM_PARALLEL=4
OLLAMA_MAX_LOADED_MODELS=1
```

**24GB+ VRAM (RTX 4090 desktop, A5000):**
```bash
OLLAMA_NUM_PARALLEL=8
OLLAMA_MAX_LOADED_MODELS=2
```

### Reducing Cold Start Latency

Models unload after `OLLAMA_KEEP_ALIVE`. To keep models warm:

```bash
# Keep loaded for 1 hour
OLLAMA_KEEP_ALIVE=1h

# Keep loaded indefinitely (until restart)
OLLAMA_KEEP_ALIVE=-1
```

### If You Experience OOM Errors

1. Reduce parallel requests: `OLLAMA_NUM_PARALLEL=2`
2. Use a smaller model (3B-7B instead of 9B+)
3. Disable flash attention: `OLLAMA_FLASH_ATTENTION=0`
4. Restart to clear VRAM: `docker compose restart`

## Management

```bash
# View logs
docker compose logs -f

# List models
docker exec ollama ollama list

# Restart
docker compose restart

# Stop
docker compose down

# Update Ollama
docker compose pull
docker compose up -d
```

## Troubleshooting

### GPU not detected

```bash
# Check NVIDIA driver
nvidia-smi

# Check Docker GPU access
docker run --rm --gpus all nvidia/cuda:12.4.0-base-ubuntu22.04 nvidia-smi
```

### Out of memory

- Use a smaller model or quantized version
- Check VRAM: `nvidia-smi`
- Restart to clear VRAM: `docker compose restart`

## Port

- `11434` - Ollama API

## Files

```
local-ollama-server/
├── docker-compose.yml    # Main configuration
├── .env.example          # Configuration template
├── .env                  # Your config (not in git)
└── README.md
```

## Requirements

- NVIDIA GPU with 16GB+ VRAM
- NVIDIA Driver 525+
- Docker with NVIDIA Container Toolkit

## License

MIT
