# Moon Bridge

Moon Bridge is a protocol conversion and model routing proxy written in Go. It exposes the **OpenAI Responses API** (`/v1/responses`) externally, while supporting multiple upstream protocols internally, including **Anthropic Messages**, **Google Gemini (GenAI)**, and **OpenAI Chat Completions**. When a client specifies different model aliases, Moon Bridge automatically routes requests to the corresponding upstream provider and converts between protocols.

> 🍳 **New here? Start with** [CookBook.md](CookBook.md): a goal-oriented recipe book that gets your first conversation running in 5 minutes.
> Official QQ group: 1103798316

---

## Quick Start

```bash
# Copy and edit the configuration
cp config.example.yml config.yml
# Update api_key in config.yml

# Start
go run ./cmd/moonbridge -config config.yml

# See CookBook.md for detailed usage scenarios
```

Requires Go 1.25+.

## Core Features

- **Protocol conversion**: OpenAI Responses → Anthropic Messages / Google Gemini / OpenAI Chat, adapting across four upstream protocols
- **Model routing**: Map model aliases to upstream model names from different providers via `routes`
- **Plugin extensibility**: `CorePluginHooks` interface for request preprocessing, response postprocessing, and stream interception
- **Request tracing**: Full-chain records with traceable conversion steps
- **Usage statistics**: Aggregate tokens and cost by session
- **Management API**: Runtime hot-reload configuration when persistence is enabled
- **Web Search injection**: Auto/injection modes with Tavily and Firecrawl support
- **Prompt caching**: Explicit, automatic, and hybrid modes

## Operating Modes

| Mode | Behavior |
|------|----------|
| `Transform` (default) | Receive OpenAI Responses requests → convert protocol → forward → convert back and return |
| `CaptureAnthropic` | Receive Anthropic Messages requests → transparently forward to an Anthropic upstream |
| `CaptureResponse` | Receive OpenAI Responses requests → transparently forward to an OpenAI upstream |

## Configuration

Moon Bridge uses YAML configuration with three core sections: `models`, `providers`, and `routes`. See [CONFIGURATION.md](docs/CONFIGURATION.md) for the full configuration reference.

## Using with Codex CLI

Set the Moon Bridge address as Codex's OpenAI API Base URL:

```toml
[openai]
base_url = "http://127.0.0.1:38440/v1"
api_key = "[REDACTED:api-key]"
```

Then define routes in the Moon Bridge configuration using the same names as Codex models.

## Using with Claude Code

```bash
claude --model your-alias --api-url http://127.0.0.1:38440 --api-key any-value
```

## Docker Deployment

```bash
docker build -t moonbridge .
docker run -p 38440:38440 -v $(pwd)/config.yml:/config/config.yml moonbridge
```

## CLI Options

| Option | Default | Description |
|--------|---------|-------------|
| `-config` | `${XDG_CONFIG_HOME}/moonbridge/config.yml` | Configuration file path |
| `-addr` | From configuration file | Override listen address |
| `-mode` | From configuration file | Override operating mode (Transform/CaptureAnthropic/CaptureResponse) |
| `-print-addr` | — | Print the configured listen address and exit |
| `-print-mode` | — | Print the configured operating mode and exit |
| `-print-default-model` | — | Print the default model alias and exit |
| `-print-codex-model` | — | Print the Codex model and exit |
| `-print-codex-config <model>` | — | Generate Codex config.toml for the specified model and exit |
| `-dump-config-schema` | — | Generate config.schema.json and exit |

## HTTP API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/v1/responses` | POST | Main OpenAI Responses API entry point |
| `/responses` | POST | Same as above, without the `/v1` prefix |
| `/v1/models` | GET | List available models |
| `/models` | GET | Same as above |
| `/api/v1/` | — | Management API (requires persistence to be enabled) |
| `/health` | GET | Health check |

See [API.md](docs/api.md) for detailed API documentation.

## Request Tracing

Enable request tracing with `trace.enabled` in the configuration or through specific operating modes. Moon Bridge records the full request/response chain to files. Trace files are organized as `session/model-name/category/sequence.json` and support Chat, Response, and Anthropic categories.

## License

[GPL v3](LICENSE)
