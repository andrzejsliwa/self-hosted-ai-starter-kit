# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Self-hosted AI Starter Kit - A Docker Compose template for initializing a comprehensive local AI and low-code development environment. It combines n8n workflow automation with AI components like Ollama (LLM), Qdrant (vector DB), and supporting services.

## Common Commands

### Initial Setup
```bash
# Copy environment variables
cp .env.sample .env
# Edit .env with your configuration

# For NVIDIA GPU users
docker compose --profile gpu-nvidia up

# For AMD GPU users (Linux only)
docker compose --profile gpu-amd up

# For CPU-only or Mac users
docker compose --profile cpu up
# Or for Mac with local Ollama
docker compose up
```

### Service Management
```bash
# Stop all services
docker compose down

# View logs
docker compose logs -f [service-name]

# Restart specific service
docker compose restart [service-name]

# Upgrade services (NVIDIA GPU example)
docker compose --profile gpu-nvidia pull
docker compose create && docker compose --profile gpu-nvidia up
```

### n8n Backup/Restore Operations
```bash
# Export workflows and credentials
docker compose run --rm n8n-backup

# Import workflows and credentials
docker compose run --rm n8n-import
```

## Architecture

### Service Endpoints
- n8n: http://n8n.localhost:5678
- Qdrant: http://qdrant.localhost:6333
- NocoDB: http://nocodb.localhost:8081
- Traefik Dashboard: http://traefik.localhost:8080
- ngrok Inspector: http://localhost:4040 (when enabled)
- Ollama API: http://localhost:11434 (internal service)

### Docker Profiles
- `cpu`: CPU-only Ollama setup
- `gpu-nvidia`: NVIDIA GPU support
- `gpu-amd`: AMD GPU support (Linux only)
- No profile: For Mac/Apple Silicon users

### Key Directories
- `./n8n/backup/`: Pre-configured workflows and credentials
  - `credentials/`: Ollama API credentials for local LLM access
  - `workflows/`: Demo workflow showcasing Ollama integration
- `./shared/`: Mounted at `/data/shared` in n8n container for file access
- `./redis/redis.conf`: Redis configuration with security and memory settings
- `./assets/`: Documentation assets (n8n-demo.gif)

### Environment Variables
Required in `.env`:
- `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`: Database credentials
- `N8N_ENCRYPTION_KEY`: n8n data encryption
- `N8N_USER_MANAGEMENT_JWT_SECRET`: JWT secret
- `N8N_LICENSE_ACTIVATION_KEY`: n8n license (optional for pro features)
- `REDIS_PASSWORD`: Redis authentication
- `NGROK_AUTHTOKEN`, `NGROK_DOMAIN`: For external webhook access (optional)

### Service Dependencies
- n8n depends on PostgreSQL being healthy
- n8n depends on n8n-import completing successfully
- Ollama init containers wait for Ollama service before pulling models
- NocoDB depends on PostgreSQL being healthy

## Development Notes

### Local File Access in n8n
Files in `./shared/` are accessible in n8n at `/data/shared/`. Use this path in nodes that interact with the filesystem (Read/Write Files, Local File Trigger, Execute Command).

### Mac Users with Local Ollama
If running Ollama locally on Mac (not in Docker), modify the n8n service environment:
```yaml
environment:
  - OLLAMA_HOST=host.docker.internal:11434
```
And update the Ollama credential in n8n UI to use `http://host.docker.internal:11434/`

### Model Management
The starter kit automatically pulls `llama3.2` model on first start. Additional models can be pulled using:
```bash
docker exec ollama ollama pull [model-name]
```

### Troubleshooting

#### Common Issues
- **Port conflicts**: Ensure ports 5678, 6333, 8080, 8081, 11434 are not in use
- **GPU not detected**: Verify Docker GPU support with `docker run --rm --gpus all nvidia-smi` (NVIDIA) or check ROCm installation (AMD)
- **Services not accessible**: Check Traefik is running and `.localhost` domains resolve correctly
- **n8n import fails**: Ensure PostgreSQL is healthy before n8n starts

#### Useful Commands
```bash
# Check service health
docker compose ps

# View specific service logs
docker compose logs -f [service-name]

# Execute commands in containers
docker exec -it [container-name] /bin/sh

# Reset everything (WARNING: deletes all data)
docker compose down -v
```

## Project Structure

### Core Services
- **n8n**: Workflow automation with 400+ integrations and AI capabilities
- **Ollama**: Local LLM inference supporting multiple models
- **Qdrant**: Vector database for embeddings and semantic search
- **PostgreSQL**: Primary database for n8n and NocoDB
- **Redis**: Caching layer with password authentication
- **NocoDB**: No-code database platform
- **Traefik**: Reverse proxy for clean URL routing
- **ngrok**: Optional tunnel for external webhook access

### Configuration Files
- `docker-compose.yml`: Main orchestration file with service definitions
- `.env.sample`: Template for required environment variables
- `redis/redis.conf`: Redis configuration
- `README.md`: User documentation
- `.gitignore`: Excludes sensitive files (.env, CLAUDE.md)