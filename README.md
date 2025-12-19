# Twin in Disguise

Run Gemini 3 Pro models in Claude Code.

This proxy translates between Anthropic and Gemini APIs, letting you use Google's Gemini 3 models (including extended thinking capabilities) with Claude Code and other Claude-compatible tools.

**Note:** Designed specifically for Gemini 3 models with extended thinking support. May work with other Gemini models, but primary use case is Gemini 3 Pro.

## Quickstart

### Why HTTPS is Required

Claude Code and most Claude-compatible tools require HTTPS connections for security. Since the proxy runs locally (on your machine), you need an HTTPS tunnel to expose it securely. You can use any HTTPS tunneling service—ngrok is just one popular option. Other alternatives include Cloudflare Tunnel, localtunnel, or serveo.

### Quick Setup with Docker

1. Get the Docker image running:
   ```bash
   docker run --rm -it -p 8080:8080 -e "GEMINI_API_KEY=${GEMINI_API_KEY}" savaki/twin-in-disguise
   ```

2. Set up HTTPS tunnel (example using ngrok, but any tunnel service works):
   ```bash
   # Install ngrok (or use your preferred tunneling service)
   # Download from https://ngrok.com/download

   # In another terminal, create HTTPS tunnel to your local proxy
   ngrok http 8080
   ```

3. In another terminal, enter these environment variables in your shell:
   ```bash
   # Use the HTTPS URL from your tunnel service
   export ANTHROPIC_BASE_URL=https://your-tunnel-url  # e.g., https://abc123.ngrok.io
   export ANTHROPIC_AUTH_TOKEN=test
   export ANTHROPIC_MODEL="gemini-3-pro-preview"
   export ANTHROPIC_DEFAULT_OPUS_MODEL="gemini-3-pro-preview"
   export ANTHROPIC_DEFAULT_SONNET_MODEL="gemini-3-pro-preview"
   export ANTHROPIC_DEFAULT_HAIKU_MODEL="gemini-2.0-flash"
   export CLAUDE_CODE_SUBAGENT_MODEL="gemini-3-pro-preview"
   ```

You're now ready to use Gemini models with Claude Code in that terminal!

## Installation

### Prerequisites

**For Docker:**
- Docker and Docker Compose installed
- A Google Gemini API key (get one at https://ai.google.dev/)
- For remote access: ngrok or similar tunneling service (see [HTTPS Setup](#https-setup) below)

**For building from source:**
- Go 1.24 or later
- A Google Gemini API key (get one at https://ai.google.dev/)
- For remote access: ngrok or similar tunneling service (see [HTTPS Setup](#https-setup) below)

### Option 1: Docker (Recommended)

**Using the pre-built image:**

```bash
# Pull and run the public image
docker run -p 8080:8080 -e GEMINI_API_KEY="your-api-key-here" savaki/twin-in-disguise
```

**Building from source:**

```bash
# Clone the repository
git clone https://github.com/savaki/twin-in-disguise.git
cd twin-in-disguise

# Set your Gemini API key
export GEMINI_API_KEY="your-api-key-here"

# Run with docker-compose
docker-compose up -d

# Or build and run manually for Intel/AMD64
docker build --platform linux/amd64 -t twin-in-disguise .
docker run -p 8080:8080 -e GEMINI_API_KEY="your-api-key-here" twin-in-disguise

# Build for your native platform (auto-detect)
docker build -t twin-in-disguise .
docker run -p 8080:8080 -e GEMINI_API_KEY="your-api-key-here" twin-in-disguise
```

The Docker image can be built for Intel/AMD64 or your native architecture (ARM64/Apple Silicon, etc.).

### Option 2: Build from Source

```bash
# Clone the repository
git clone https://github.com/savaki/twin-in-disguise.git
cd twin-in-disguise

# Build the binary
go build -o twin-in-disguise ./cmd/twin-in-disguise

# Run the server
export GEMINI_API_KEY="your-api-key-here"
./twin-in-disguise
```

### HTTPS Setup

Most Claude-compatible tools (including Claude Code) require HTTPS connections. Since the proxy runs on localhost, you'll need to tunnel it through HTTPS:

**With Docker:**

1. Install ngrok: https://ngrok.com/download
2. Start the proxy server with Docker:
   ```bash
   docker-compose up -d
   ```
3. In another terminal, start ngrok:
   ```bash
   ngrok http 8080
   ```
4. Copy the HTTPS URL from ngrok (e.g., `https://abc123.ngrok.io`)
5. Set this as your `ANTHROPIC_BASE_URL`:
   ```bash
   export ANTHROPIC_BASE_URL=https://abc123.ngrok.io
   ```

**Without Docker:**

1. Install ngrok: https://ngrok.com/download
2. Start the proxy server:
   ```bash
   ./twin-in-disguise --port 8080
   ```
3. In another terminal, start ngrok:
   ```bash
   ngrok http 8080
   ```
4. Copy the HTTPS URL from ngrok (e.g., `https://abc123.ngrok.io`)
5. Set this as your `ANTHROPIC_BASE_URL`:
   ```bash
   export ANTHROPIC_BASE_URL=https://abc123.ngrok.io
   ```

### Environment Variables Reference

All environment variables needed for Claude Code:

```bash
# Required: Gemini API key (for the proxy server)
export GEMINI_API_KEY="your-gemini-api-key"

# Required: Claude Code configuration
export ANTHROPIC_BASE_URL=https://your-tunnel-url  # HTTPS URL from ngrok, Cloudflare Tunnel, etc.
export ANTHROPIC_AUTH_TOKEN=test  # Any value works, required but not validated

# Model configuration
export ANTHROPIC_MODEL="gemini-3-pro-preview"
export ANTHROPIC_DEFAULT_OPUS_MODEL="gemini-3-pro-preview"
export ANTHROPIC_DEFAULT_SONNET_MODEL="gemini-3-pro-preview"
export ANTHROPIC_DEFAULT_HAIKU_MODEL="gemini-2.0-flash"
export CLAUDE_CODE_SUBAGENT_MODEL="gemini-3-pro-preview"
```

Set these in your terminal session before running Claude Code.

## Command-Line Flags

The `twin-in-disguise` command supports the following flags:

### `--port, -p` (default: 8080)
Specifies the HTTP port the server listens on.

**Example:**
```bash
./twin-in-disguise --port 3000
```

**Environment variable:** `PORT`

### `--verbose`
Enables verbose logging, showing more detailed information about requests and responses.

**Example:**
```bash
./twin-in-disguise --verbose
```

**Environment variable:** `VERBOSE=true`

### `--debug`
Enables debug logging, including detailed Gemini API calls and responses. Useful for troubleshooting translation issues.

**Example:**
```bash
./twin-in-disguise --debug
```

**Environment variable:** `DEBUG=true`

### Combining Flags

You can combine multiple flags:
```bash
./twin-in-disguise --port 3000 --debug
```

### Using Flags with Docker

With Docker, you can pass flags by setting environment variables in `docker-compose.yml` or using the `-e` flag:

```bash
# Using docker-compose (edit docker-compose.yml)
environment:
  - PORT=3000
  - DEBUG=true
  - VERBOSE=true

# Or using docker run
docker run -p 3000:3000 -e GEMINI_API_KEY="your-key" -e PORT=3000 -e DEBUG=true twin-in-disguise
```

## How It Works

The proxy server implements a translation layer that handles the differences between Anthropic's and Gemini's APIs:

### Request Translation

When a request comes in to `/v1/messages`:
- Parses the Anthropic-formatted request (messages, system prompts, tools)
- Converts role mappings (`assistant` → `model`)
- Translates content blocks (text, images, tool calls, tool results)
- Maps Anthropic tool schemas to Gemini function declarations
- Cleans schemas (removes unsupported fields like `$schema`, `additionalProperties`)

### Thought Signature Management

For function calling (tool use):
- Caches thought signatures returned by Gemini when tools are called
- Injects cached signatures into subsequent requests referencing those tools
- Maintains signature cache in memory for conversation continuity

This is necessary for multi-turn tool use conversations.

### Response Translation

When Gemini responds:
- Converts Gemini's response format to Anthropic's format
- Maps content parts (text, function calls) to Anthropic content blocks
- Generates UUIDs for tool use blocks
- Preserves thought signatures for future requests
- Translates usage metadata (token counts)

## Supported Models

This proxy is designed for Gemini 3 models with thinking capabilities:

- `gemini-3-pro-preview` - Gemini 3 Pro model with extended thinking
- `gemini-2.0-flash` - Fast model for subagents and quick tasks

While other Gemini models may work, the primary design focus is on Gemini 3's unique capabilities.

## Outstanding Work

- Thought signature cache memory leak: The cache is never garbage collected and will grow indefinitely. Need to implement LRU cache with TTL or size limits.
- Limited error handling: Some edge cases in schema conversion and API errors could be handled more gracefully.
- Streaming support: Add support for streaming responses (currently only supports non-streaming)
- Additional endpoints: Support other Claude API endpoints like `/v1/complete`
- Model mapping configuration: Allow users to configure custom model name mappings
- Request/response logging: Optional logging to file for debugging
- Metrics and monitoring: Add Prometheus metrics for request rates, latency, errors
- Health check endpoint: Add `/health` endpoint for monitoring
- Integration test coverage: Add more integration tests for edge cases
- Performance benchmarks: Add benchmarks to track translation overhead
- API documentation: Generate OpenAPI/Swagger docs for the proxy API
- Troubleshooting guide: Document common issues and solutions
- systemd service: Add service file for running as daemon
- Homebrew formula: Create formula for easy installation on macOS
- Pre-built binaries: Provide pre-built binaries for major platforms
- Explicit caching: Currently relies on Gemini's implicit caching; add explicit cache control

## Docker Quick Reference

Useful Docker commands for managing the proxy:

```bash
# Run the pre-built public image
docker run -d -p 8080:8080 -e GEMINI_API_KEY="your-key" savaki/twin-in-disguise

# View logs (replace <container-id> with actual container ID from `docker ps`)
docker logs -f <container-id>

# Stop a running container
docker stop <container-id>

# Run with custom port
docker run -d -p 3000:3000 -e GEMINI_API_KEY="your-key" -e PORT=3000 savaki/twin-in-disguise

# Enable debug logging
docker run -d -p 8080:8080 -e GEMINI_API_KEY="your-key" -e DEBUG=true savaki/twin-in-disguise

# Using docker-compose (requires cloning the repo)
# Start the server in detached mode
docker-compose up -d

# View logs
docker-compose logs -f

# Stop the server
docker-compose down

# Rebuild the image after code changes
docker-compose build

# Restart the server
docker-compose restart

# Check container status
docker-compose ps

# Build for specific platform (Intel/AMD64)
docker build --platform linux/amd64 -t twin-in-disguise .

# Build for ARM64/Apple Silicon
docker build --platform linux/arm64 -t twin-in-disguise .
```

## Limitations

1. **No streaming support**: Currently only supports non-streaming responses
2. **In-memory state**: Thought signatures are stored in memory and lost on restart
3. **Single instance**: Not designed for horizontal scaling (due to in-memory cache)
4. **HTTPS requirement**: Most Claude tools require HTTPS, necessitating tunneling services
5. **No authentication**: The proxy doesn't validate ANTHROPIC_AUTH_TOKEN (it can be any value)
6. **Gemini 3 focused**: Designed primarily for Gemini 3 models with thinking capabilities

## License

Copyright 2025 Matt Ho

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

## Acknowledgments

Built using:
- [google/generative-ai-go](https://github.com/google/generative-ai-go) - Official Gemini SDK
- [urfave/cli](https://github.com/urfave/cli) - CLI framework



# Local SSL

To switch the listener to port **11443**, you just need to update the port mapping in your `docker-compose.yml` and ensure the internal Nginx configuration matches.

Here are the updated files and the final environment configuration:

### 1. Updated `docker-compose.yml`

Change the `ports` mapping for the Nginx service. We will map host port `11443` to container port `443`.

```yaml
services:
  twin-in-disguise:
    build:
      context: .
      dockerfile: Dockerfile
    environment:
      - GEMINI_API_KEY=${GEMINI_API_KEY}
      - PORT=8080
      - VERBOSE=true
      - DEBUG=true
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    ports:
      - "11443:443" # Changed host port to 11443
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl/server.crt:/etc/nginx/server.crt:ro
      - ./ssl/server.key:/etc/nginx/server.key:ro
    depends_on:
      - twin-in-disguise
    restart: unless-stopped

```

### 2. Verify `nginx.conf`

Ensure your `nginx.conf` is listening on `443 ssl` inside the container (as specified in your previous project's config):

```nginx
events {
    worker_connections 1024;
}

http {
    server {
        listen 443 ssl; # Container still listens on 443 internally
        server_name localhost;

        ssl_certificate /etc/nginx/server.crt;
        ssl_certificate_key /etc/nginx/server.key;

        location / {
            proxy_pass http://twin-in-disguise:8080;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            
            proxy_read_timeout 300;
        }
    }
}

```

### 3. Restart the Stack

```bash
docker compose down
docker compose up --build -d

```

### 4. Update Claude Code Environment Variables

Since you are now using a non-standard port, you must include it in the `ANTHROPIC_BASE_URL`.

```bash
# Point to the new port
export ANTHROPIC_BASE_URL=https://localhost:11443
export ANTHROPIC_AUTH_TOKEN=test

# Model Configs
export ANTHROPIC_MODEL="gemini-3-pro-preview"
export ANTHROPIC_DEFAULT_OPUS_MODEL="gemini-3-pro-preview"
export ANTHROPIC_DEFAULT_SONNET_MODEL="gemini-3-flash-preview"
export ANTHROPIC_DEFAULT_HAIKU_MODEL="gemini-2.5-flash-lite"
export CLAUDE_CODE_SUBAGENT_MODEL="gemini-3-pro-preview"

# Crucial: Tell Node/Claude Code to trust the local cert
export NODE_EXTRA_CA_CERTS="~/Code/twin-in-disguise/ssl/server.crt"

```

---

### Troubleshooting Note

If Claude Code complains about the certificate even with `NODE_EXTRA_CA_CERTS`, it may be because the certificate was generated for `localhost` but the tool is verifying the full URL. Since you're using `https://localhost:11443`, your `.NET dev-certs` should work perfectly as they are typically issued for `CN=localhost`.

**Would you like me to create a small bash script (e.g., `start-claude.sh`) that sets all these variables and launches the tool for you in one command?**
