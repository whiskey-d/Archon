# Deploying Archon on Dokploy

This fork includes modifications to make Archon work seamlessly with Dokploy deployments.

## Prerequisites

- Dokploy instance running
- Supabase instance deployed (can be in a separate Dokploy compose project)
- Anthropic API key

## Environment Variables

**CRITICAL**: You MUST configure environment variables in the Dokploy UI for the deployment to work.

### Required Environment Variables

Go to your Dokploy project → Environment Variables and add:

```bash
# Supabase Configuration
SUPABASE_URL=http://YOUR_SUPABASE_IP:8000
SUPABASE_SERVICE_KEY=your-supabase-service-role-jwt

# AI API Keys (REQUIRED)
ANTHROPIC_API_KEY=your-anthropic-api-key

# Optional
OPENAI_API_KEY=
LOGFIRE_TOKEN=
GITHUB_PAT_TOKEN=

# Service Ports (use defaults)
ARCHON_SERVER_PORT=8181
ARCHON_MCP_PORT=8051
ARCHON_AGENTS_PORT=8052
AGENT_WORK_ORDERS_PORT=8053
ARCHON_UI_PORT=3737

# Host Configuration
HOST=0.0.0.0
ARCHON_HOST=localhost

# Feature Flags
AGENTS_ENABLED=false
ENABLE_AGENT_WORK_ORDERS=true
PROD=false
VITE_SHOW_DEVTOOLS=false

# Logging
LOG_LEVEL=INFO

# Service Discovery
SERVICE_DISCOVERY_MODE=docker_compose
STATE_STORAGE_TYPE=supabase
TRANSPORT=sse

# Docker Environment
DOCKER_ENV=true
```

## Changes from Original Archon

This fork includes the following modifications for Dokploy compatibility:

### 1. HTTP Support for IP Addresses (`python/src/server/config/config.py`)

Modified the `validate_supabase_url()` function to allow HTTP connections to any valid IP address (not just private IPs). This is safe because:
- Docker network connections are isolated
- Same-host IP connections don't traverse the internet
- Required for Dokploy's containerized architecture

```python
# Allows HTTP for all valid IP addresses in Docker environments
if not ip.is_unspecified:
    return True
```

### 2. Dokploy Network Support (`docker-compose.yml`)

Added `dokploy-network` to all services to enable communication between separate Dokploy compose projects (e.g., between Archon and Supabase).

```yaml
networks:
  app-network:
    driver: bridge
  dokploy-network:
    external: true
```

## Deployment Steps

1. **Create Compose Project in Dokploy**
   - Go to Dokploy → New Compose Project
   - Repository: `https://github.com/whiskey-d/Archon`
   - Branch: `main`
   - Compose File: `docker-compose.yml`

2. **Configure Environment Variables**
   - Add all required variables listed above in Dokploy UI
   - Especially critical: `SUPABASE_URL`, `SUPABASE_SERVICE_KEY`, `ANTHROPIC_API_KEY`

3. **Deploy**
   - Click Deploy
   - Wait for build to complete
   - Check logs for "Supabase connection successful"

## Troubleshooting

### Container Fails with "HTTPS required" Error

This means you're using an older version. Pull latest changes:
- Commit af897d8 includes the IP address HTTP fix

### Container Fails with "Unknown Supabase key role 'None'"

Environment variables are not set. Check Dokploy UI → Environment Variables.

### Container Fails with "Temporary failure in name resolution"

Supabase is not reachable. Verify:
1. Supabase Kong container is running
2. Both projects are on `dokploy-network`
3. Try using IP address instead of container name for `SUPABASE_URL`

## Support

For issues specific to this Dokploy fork:
- GitHub: https://github.com/whiskey-d/Archon/issues

For general Archon questions:
- Original repo: https://github.com/coleam00/Archon

## License

Same as original Archon project.
