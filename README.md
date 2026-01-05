# Local Ollama Server

Dedicated LLM inference server for text generation on a local GPU.

Optimized for 16GB VRAM GPUs (RTX 4080, laptop 4090, etc.) to run large language models for wellness content generation.

## Requirements

- NVIDIA GPU with 16GB VRAM (RTX 4080, laptop 4090, etc.)
- Ubuntu Server 22.04+ (or any Linux with Docker)
- Docker & Docker Compose
- NVIDIA Driver 525+
- NVIDIA Container Toolkit

## Quick Start

### 1. Install NVIDIA Container Toolkit

```bash
# Add NVIDIA package repository
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

# Install toolkit
sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit

# Configure Docker runtime
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker

# Verify GPU is accessible
docker run --rm --gpus all nvidia/cuda:12.4.0-base-ubuntu22.04 nvidia-smi
```

### 2. Clone and Start

```bash
git clone https://github.com/profzeller/local-ollama-server.git
cd local-ollama-server
docker compose up -d
```

### 3. Pull a Model

**Recommended for 16GB VRAM (wellness content generation):**

```bash
# Qwen 2.5 14B - Best balance of quality and speed (~10GB VRAM)
docker exec ollama ollama pull qwen2.5:14b

# Alternative: Llama 3.1 8B - Faster, still good quality (~8GB VRAM)
docker exec ollama ollama pull llama3.1:8b

# Alternative: DeepSeek R1 14B - Good reasoning (~10GB VRAM)
docker exec ollama ollama pull deepseek-r1:14b
```

**Other useful models for 16GB:**

```bash
# For coding tasks
docker exec ollama ollama pull qwen2.5-coder:7b

# Smaller/faster options
docker exec ollama ollama pull llama3.2:3b
docker exec ollama ollama pull phi3:3.8b

# Larger (uses most of 16GB)
docker exec ollama ollama pull qwen2.5:14b-q8_0
```

## Model Recommendations

| Model | VRAM | Best For |
|-------|------|----------|
| `qwen2.5:14b` | ~10GB | **Recommended** - Content writing, creative tasks |
| `llama3.1:8b` | ~8GB | Fast general purpose, good quality |
| `deepseek-r1:14b` | ~10GB | Reasoning, analysis, research summaries |
| `qwen2.5-coder:7b` | ~6GB | Code generation, technical content |
| `llama3.2:3b` | ~3GB | Quick drafts, simple tasks |

## API Usage

### Generate Text

```bash
curl http://localhost:11434/api/generate -d '{
  "model": "qwen2.5:14b",
  "prompt": "Write a wellness tip about morning routines",
  "stream": false
}'
```

### Chat Completion

```bash
curl http://localhost:11434/api/chat -d '{
  "model": "qwen2.5:14b",
  "messages": [
    {"role": "system", "content": "You are a wellness content expert."},
    {"role": "user", "content": "Write 3 benefits of meditation"}
  ]
}'
```

### OpenAI-Compatible API

Ollama supports the OpenAI API format:

```bash
curl http://localhost:11434/v1/chat/completions -d '{
  "model": "qwen2.5:14b",
  "messages": [{"role": "user", "content": "Hello!"}]
}'
```

### Python Example

```python
import requests

response = requests.post("http://localhost:11434/api/generate", json={
    "model": "qwen2.5:14b",
    "prompt": "Write a short wellness tip about hydration",
    "stream": False
})
print(response.json()["response"])
```

## Network Configuration

By default, Ollama binds to all interfaces. Access from other machines:

```
http://<server-ip>:11434
```

### Firewall

```bash
sudo ufw allow 11434
```

## Management

```bash
# View logs
docker compose logs -f

# List models
docker exec ollama ollama list

# Remove a model
docker exec ollama ollama rm <model-name>

# Restart
docker compose restart

# Stop
docker compose down

# Update Ollama
docker compose pull
docker compose up -d
```

## Performance Tuning

### For 16GB VRAM

The default config is optimized for 16GB. Key settings in `docker-compose.yml`:

- `OLLAMA_NUM_PARALLEL=2` - Handle 2 concurrent requests
- `OLLAMA_MAX_LOADED_MODELS=1` - Keep 1 model in VRAM (important for 16GB)

### Memory Tips

- Stick to models under 14B parameters
- Use quantized versions when available (q4_K_M, q8_0)
- Only keep one model loaded at a time
- Restart if VRAM gets fragmented: `docker compose restart`

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
- Check VRAM usage: `nvidia-smi`
- Restart to clear VRAM: `docker compose restart`

### Slow inference

- Ensure model fits in VRAM (not swapping to RAM)
- Use quantized versions: `qwen2.5:32b-q4_K_M`

## License

MIT License - Use freely for personal and commercial projects.
