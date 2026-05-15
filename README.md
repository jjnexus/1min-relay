# 1min-Relay

A lightweight relay server that translates the [1min.ai](https://1min.ai) API into an OpenAI-compatible format, allowing any client that supports a custom OpenAI endpoint to use 1min.ai models.

## Supported API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `GET /v1/models` | GET | List available models |
| `POST /v1/chat/completions` | POST | Chat completions with streaming support |
| `POST /v1/images/generations` | POST | Image generation |

## Supported Models

### Chat Models
- **OpenAI**: `gpt-5`, `gpt-5-mini`, `gpt-5-nano`, `gpt-4o`, `gpt-4o-mini`, `gpt-4-turbo`, `gpt-4`, `gpt-3.5-turbo`, `o1-preview`, `o1-mini`, `o3-mini`, `gpt-o1-pro`, `gpt-o4-mini`, `gpt-4.1-mini`, `gpt-4.1-nano`
- **Anthropic**: `claude-3-7-sonnet-20250219`, `claude-3-5-sonnet-20240620`, `claude-3-opus-20240229`, `claude-3-sonnet-20240229`, `claude-3-haiku-20240307`, `claude-2.1`, `claude-instant-1.2`
- **Google**: `gemini-1.5-pro`, `gemini-1.5-flash`, `gemini-1.0-pro`
- **Mistral**: `mistral-large-latest`, `mistral-small-latest`, `mistral-nemo`, `open-mistral-7b`
- **DeepSeek**: `deepseek-chat`, `deepseek-reasoner`
- **Meta (via Replicate)**: `meta/llama-2-70b-chat`, `meta/meta-llama-3-70b-instruct`, `meta/meta-llama-3.1-405b-instruct`
- **Cohere**: `command`

### Vision (Image Input) Models
Supports image URLs or base64-encoded images in the `messages` array:
- `gpt-4o`, `gpt-4o-mini`, `gpt-4-turbo`

### Image Generation Models
- **Stability AI**: `stable-image`, `stable-diffusion-xl-1024-v1-0`, `stable-diffusion-v1-6`, `esrgan-v1-x2plus`
- **Clipdrop**: `clipdrop`
- **Midjourney**: `midjourney`, `midjourney_6_1`
- **Leonardo AI**: `LEONARDO_PHOENIX`, `LEONARDO_LIGHTNING_XL`, `LEONARDO_ANIME_XL`, `LEONARDO_DIFFUSION_XL`, `LEONARDO_KINO_XL`, `LEONARDO_ALBEDO_BASE_XL`
- **Black Forest Labs**: `black-forest-labs/flux-schnell`

## Quick Deployment (Recommended: Docker Compose)

### Using the Pre-built GHCR Image

```bash
# 1. Download docker-compose.yml
curl -O https://raw.githubusercontent.com/jjnexus/1min-relay/main/docker-compose.yml

# 2. Start services
docker compose up -d

# 3. Verify
curl http://localhost:5001/v1/models
```

Once running, configure any OpenAI-compatible client with:
- **API Base URL**: `http://your-server-ip:5001/v1`
- **API Key**: Your 1min.ai API key (available at https://app.1min.ai/api)

### Environment Variables

Adjust in the `environment` section of `docker-compose.yml`:

| Variable | Default | Description |
|----------|---------|-------------|
| `SUBSET_OF_ONE_MIN_PERMITTED_MODELS` | `mistral-nemo,gpt-4o,deepseek-chat` | Comma-separated list of models to expose |
| `PERMIT_MODELS_FROM_SUBSET_ONLY` | `False` | Set to `True` to reject requests for models outside the list |

### Build from Source

```bash
git clone https://github.com/jjnexus/1min-relay.git
cd 1min-relay
docker compose up -d --build
```

## Local Development

Requires Python 3.10+ and [uv](https://github.com/astral-sh/uv).

```bash
git clone https://github.com/jjnexus/1min-relay.git
cd 1min-relay
uv venv
source .venv/bin/activate
uv pip install -r requirements.txt
python main.py
```

## CI/CD

Pushing to the `main` branch automatically triggers a GitHub Actions workflow that builds multi-platform images (`linux/amd64` and `linux/arm64`) and pushes them to GHCR:

```
ghcr.io/jjnexus/1min-relay:latest
```

To update a running VPS deployment:

```bash
docker compose pull && docker compose up -d
```

## Security

- **SSRF protection**: Vision image URLs are validated before fetching — only HTTPS is allowed, and private IP ranges, loopback addresses, and cloud metadata endpoints (e.g. `169.254.169.254`) are blocked.
- **API key validation**: Every request validates the 1min.ai API key before forwarding.

## License

Based on the upstream [kokofixcomputers/1min-relay](https://github.com/kokofixcomputers/1min-relay). See [LICENSE](LICENSE) for details.
