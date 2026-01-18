# Mirza Agent - Production Deployment Guide

## Overview
This guide provides step-by-step instructions for deploying Mirza Agent to production.

## Prerequisites

### Required Software
- **Python**: 3.12 or higher
- **Node.js**: 18.x or higher
- **pnpm**: 8.x or higher
- **uv**: Latest version
- **Docker** (optional): For containerized deployment

### System Requirements
- **Memory**: Minimum 4GB RAM (8GB recommended)
- **Storage**: Minimum 10GB free space
- **OS**: Linux, macOS, or Windows with WSL2

---

## Environment Variables Checklist

Create a `.env` file in the project root with the following variables:

### Required Variables

```bash
# LLM Provider Configuration (Choose one)

## Option 1: OpenAI
OPENAI_API_KEY=your-openai-api-key-here
AGENT_SETTING_CONFIG="settings.openai.toml"

## Option 2: IBM WatsonX
# WATSONX_API_KEY=your-watsonx-api-key
# WATSONX_PROJECT_ID=your-project-id
# WATSONX_URL=https://us-south.ml.cloud.ibm.com
# AGENT_SETTING_CONFIG="settings.watsonx.toml"

## Option 3: Azure OpenAI
# AZURE_OPENAI_API_KEY=your-azure-key
# AZURE_OPENAI_ENDPOINT=your-azure-endpoint
# OPENAI_API_VERSION=2024-08-01-preview
# AGENT_SETTING_CONFIG="settings.azure.toml"

## Option 4: OpenRouter
# OPENROUTER_API_KEY=your-openrouter-key
# AGENT_SETTING_CONFIG="settings.openrouter.toml"

# Application Configuration
PORT=7860
HOST=0.0.0.0
```

### Optional Variables

```bash
# Model Overrides
MODEL_NAME=gpt-4o  # Override default model

# Logging
LOG_LEVEL=INFO

# Feature Flags
ENABLE_MEMORY=false
ENABLE_SANDBOX=false
```

---

## Asset Replacement Checklist

### Logo Files Required

Place your brand assets in the following locations:

1. **Main Logo** (100x100px, PNG/SVG):
   - Path: `src/frontend_workspaces/agentic_chat/public/logo.png`
   - Used in: Navigation header, welcome screen

2. **Favicon** (32x32px, PNG/ICO):
   - Path: `src/frontend_workspaces/frontend/static/favicon.ico`
   - Used in: Browser tab

3. **Open Graph Image** (1200x630px, PNG/JPG):
   - Path: `src/frontend_workspaces/frontend/static/og-image.png`
   - Used in: Social media previews

### Update Logo References

If you've added local logo files, update the `BRAND_CONFIG` in:
`src/frontend_workspaces/agentic_chat/src/constants.ts`

```typescript
export const BRAND_CONFIG = {
  name: "Mirza Agent",
  tagline: "The Configurable Generalist Agent - Built for Enterprise",
  primaryColor: "#667eea",
  secondaryColor: "#764ba2",
  logoUrl: "/logo.png",  // Update to your local path
};
```

---

## Build Instructions

### 1. Install Dependencies

```bash
# Backend dependencies
cd /path/to/MirxaAgent
uv venv --python=3.12 && source .venv/bin/activate
uv sync

# Frontend dependencies
cd src/frontend_workspaces
pnpm install
```

### 2. Build Frontend

```bash
cd src/frontend_workspaces/frontend
pnpm run build
```

This will create optimized production files in `dist/` directory.

### 3. Verify Build

Check for common issues:

```bash
# Check for TypeScript errors
cd src/frontend_workspaces/agentic_chat
pnpm run build

# Check bundle size
ls -lh src/frontend_workspaces/frontend/dist/
```

---

## Deployment Options

### Option 1: Docker Deployment (Recommended)

#### Build Docker Image

```bash
cd /path/to/MirxaAgent
docker build -t mirza-agent:latest .
```

#### Run Container

```bash
docker run -d \
  --name mirza-agent \
  -p 7860:7860 \
  -e OPENAI_API_KEY=your-key \
  -e AGENT_SETTING_CONFIG=settings.openai.toml \
  -v $(pwd)/.env:/app/.env \
  mirza-agent:latest
```

#### Access Application

Navigate to: `http://localhost:7860`

---

### Option 2: Vercel Deployment

#### Prerequisites
- Vercel account
- Vercel CLI installed: `npm install -g vercel`

#### Create `vercel.json`

```json
{
  "version": 2,
  "buildCommand": "cd src/frontend_workspaces/frontend && pnpm install && pnpm run build",
  "outputDirectory": "src/frontend_workspaces/frontend/dist",
  "framework": null,
  "env": {
    "OPENAI_API_KEY": "@openai-api-key"
  },
  "functions": {
    "api/**/*.py": {
      "runtime": "python3.12"
    }
  }
}
```

#### Deploy

```bash
# Login to Vercel
vercel login

# Deploy
vercel --prod

# Set environment variables
vercel env add OPENAI_API_KEY production
```

---

### Option 3: Traditional Server Deployment

#### 1. Prepare Server

```bash
# Install system dependencies
sudo apt-get update
sudo apt-get install -y python3.12 python3-pip nodejs npm

# Install pnpm
npm install -g pnpm

# Install uv
curl -LsSf https://astral.sh/uv/install.sh | sh
```

#### 2. Clone and Build

```bash
git clone https://github.com/Mirxa27/MirxaAgent.git
cd MirxaAgent

# Setup Python environment
uv venv --python=3.12 && source .venv/bin/activate
uv sync

# Build frontend
cd src/frontend_workspaces/frontend
pnpm install && pnpm run build
```

#### 3. Run with Process Manager (PM2)

```bash
# Install PM2
npm install -g pm2

# Start application
cd /path/to/MirxaAgent
pm2 start "cuga start demo" --name mirza-agent

# Enable auto-restart on server reboot
pm2 startup
pm2 save
```

#### 4. Setup Nginx Reverse Proxy

```nginx
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://localhost:7860;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

---

## Production Checklist

### Pre-Deployment

- [ ] All environment variables configured
- [ ] Logo and asset files in place
- [ ] Build completes without errors
- [ ] TypeScript types validated
- [ ] No console.log statements in production code
- [ ] robots.txt configured
- [ ] SSL certificate ready (for HTTPS)

### Post-Deployment

- [ ] Application accessible at production URL
- [ ] All API endpoints responding correctly
- [ ] Logo and branding display properly
- [ ] Mobile responsive design verified
- [ ] Performance testing completed
- [ ] Error monitoring configured (e.g., Sentry)
- [ ] Backup strategy in place

---

## Monitoring & Maintenance

### Health Checks

```bash
# Check application status
curl http://localhost:7860/health

# Check logs
pm2 logs mirza-agent

# Or with Docker
docker logs mirza-agent
```

### Performance Monitoring

Consider integrating:
- **Application Performance**: New Relic, DataDog
- **Error Tracking**: Sentry
- **Uptime Monitoring**: UptimeRobot, Pingdom

### Updates

```bash
# Pull latest changes
git pull origin main

# Update dependencies
uv sync
cd src/frontend_workspaces/frontend && pnpm install

# Rebuild and restart
pnpm run build
pm2 restart mirza-agent
```

---

## SEO Configuration

### Sitemap Generation

Create `src/frontend_workspaces/frontend/static/sitemap.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://your-domain.com/</loc>
    <lastmod>2024-01-18</lastmod>
    <changefreq>weekly</changefreq>
    <priority>1.0</priority>
  </url>
</urlset>
```

### Submit to Search Engines

- Google Search Console: https://search.google.com/search-console
- Bing Webmaster Tools: https://www.bing.com/webmasters

---

## Troubleshooting

### Common Issues

#### Build Fails
```bash
# Clear cache and rebuild
rm -rf node_modules dist
pnpm install
pnpm run build
```

#### Port Already in Use
```bash
# Find and kill process on port 7860
lsof -ti:7860 | xargs kill -9
```

#### Environment Variables Not Loading
```bash
# Verify .env file exists
ls -la .env

# Check file permissions
chmod 600 .env
```

---

## Support

- **Documentation**: https://docs.github.com/Mirxa27
- **GitHub Issues**: https://github.com/Mirxa27/MirxaAgent/issues
- **Discord**: https://discord.gg/aH6rAEEW

---

## Security Notes

1. **Never commit** `.env` files to version control
2. Use **secrets management** for production (AWS Secrets Manager, HashiCorp Vault)
3. Enable **HTTPS** for all production deployments
4. Implement **rate limiting** on API endpoints
5. Regular **security audits** and dependency updates

---

**Last Updated**: January 2024  
**Version**: 0.2.6
