# n8n on DigitalOcean App Platform

Deploy the powerful workflow automation platform [n8n](https://n8n.io) to DigitalOcean App Platform.

## Choose Your Deployment Tier

### 🚀 Simple Mode (Recommended Start)

**Best for:** Personal use, testing, < 100 workflows/day

- ✅ One-click deployment
- ✅ Single instance (UI + API + execution)
- ✅ PostgreSQL database
- ✅ SSL/TLS included
- 💰 **$27/month**

**Prerequisites: (⚠️ MUST DO)**
- Generate n8n encryption key: `openssl rand -base64 32`. Replace `N8N_ENCRYPTION_KEY` env variable in template(doctl) or app(UI)

[![Deploy to DO](https://www.deploytodo.com/do-btn-blue.svg)](https://cloud.digitalocean.com/apps/new?repo=https://github.com/AppPlatform-Templates/n8n-appplatform/tree/main)

[📖 Simple Mode Guide](docs/SIMPLE-MODE.md) | [📄 Spec](.do/examples/starter.yaml)

---

### ⚡ Queue Mode (Production Ready)

**Best for:** 100-1000 workflows/day, teams, scalability

- 🔄 Main + Worker architecture
- 🔴 Redis job queue
- 📈 Horizontal scaling (add workers)
- 💪 PostgreSQL + Redis databases
- 💰 **$54/month base** (+$12 per worker)

**Prerequisites: (⚠️ MUST DO)**
- Generate n8n encryption key: `openssl rand -base64 32`. Replace `N8N_ENCRYPTION_KEY` env variable in template(doctl) or app(UI)
- Create PostgreSQL: `doctl databases create n8n-postgres --engine pg --version 17 --region <region> --size db-s-1vcpu-1gb`
- Create Redis: `doctl databases create n8n-redis --engine valkey --version 8 --region <region> --size db-s-1vcpu-1gb`

[📖 Deploy Queue Mode](docs/QUEUE-MODE.md) | [📄 Spec](.do/examples/queue-mode.yaml)

---

### 🔒 With Task Runners

**Best for:** JavaScript/Python code execution, security

- 🏃 Code execution in sandbox
- 🛡️ Secure isolation for Code nodes
- ⚙️ Single instance + runners
- 💰 **$39/month base**

**Prerequisites: (⚠️ MUST DO)**
- Generate n8n encryption key: `openssl rand -base64 32`. Replace `N8N_ENCRYPTION_KEY` env variable in template(doctl) or app(UI)
- Generate n8n runner token: `openssl rand -base64 32`. Replace `N8N_RUNNERS_AUTH_TOKEN` env variable in template(doctl) or app(UI)
- Create PostgreSQL: `doctl databases create n8n-postgres --engine pg --version 17 --region <region> --size db-s-1vcpu-1gb`

[📖 Deploy With Runners](docs/WITH-RUNNERS.md) | [📄 Spec](.do/examples/with-runners.yaml)

---

### 🏢 Production (Enterprise Scale)

**Best for:** 1000+ workflows/day, code-heavy, HA

- 🎯 Queue + Workers + Runners
- 📊 Auto-scaling capable
- 🔄 High availability
- 💪 Full production stack
- 💰 **$66/month base** (scales with load)

**Prerequisites: (⚠️ MUST DO)**
- Generate n8n encryption key: `openssl rand -base64 32`. Replace `N8N_ENCRYPTION_KEY` env variable in template(doctl) or app(UI)
- Generate n8n runner token: `openssl rand -base64 32`. Replace `N8N_RUNNERS_AUTH_TOKEN` env variable in template(doctl) or app(UI)
- Create PostgreSQL: `doctl databases create n8n-postgres --engine pg --version 17 --region <region> --size db-s-1vcpu-1gb`
- Create Redis: `doctl databases create n8n-redis --engine valkey --version 8 --region <region> --size db-s-1vcpu-1gb`

[📖 Deploy Production](docs/PRODUCTION-SETUP.md) | [📄 Spec](.do/examples/production.yaml)

---

## Deployment Method

### Simple Mode (Deploy Button)
1. Click "Deploy to DO" button above
2. Generate encryption key: `openssl rand -base64 32`
3. Replace `N8N_ENCRYPTION_KEY` in app env variables
4. Click "Create App", wait for app to deploy
5. Access at your app URL

### Advanced Modes (CLI)
```bash
# Queue Mode
doctl apps create --spec .do/examples/queue-mode.yaml

# With Runners
doctl apps create --spec .do/examples/with-runners.yaml

# Production
doctl apps create --spec .do/examples/production.yaml
```

## Quick Comparison

| Feature | Simple | Queue | Runners | Production |
|---------|--------|-------|---------|------------|
| Workflows/day | < 100 | 100-1000 | < 500 | 1000+ |
| Deploy method | Button | CLI | CLI | CLI |
| Horizontal scale | ❌ | ✅ | ❌ | ✅ |
| Code sandboxing | ❌ | ❌ | ✅ | ✅ |
| Redis queue | ❌ | ✅ | ❌ | ✅ |
| Cost/month | $27 | $54+ | $39+ | $66+ |

**Need help deciding?** See [SCALING.md](SCALING.md)

---

### ⚠️ Production Storage Consideration

**Important for Production Deployments:**

App Platform uses **ephemeral storage** - files are lost on container restart. For production use:

- **✅ Database** stores: workflows, credentials, execution history (persistent)
- **❌ Container** stores: binary files, custom nodes, uploads (ephemeral)

**Recommendation**: Configure [DigitalOcean Spaces](PRODUCTION.md#-persistent-storage-critical-for-production) ($5/month) for persistent file storage if:
- Workflows handle file uploads/downloads
- Using custom community nodes
- Processing binary data (images, PDFs, etc.)

**Quick Setup** (5 minutes):
1. Create a Space via [DigitalOcean Control Panel](https://cloud.digitalocean.com/spaces)
2. Generate access keys: `doctl spaces keys create n8n-storage-key`
3. Add credentials to your app spec - see PRODUCTION.md for full config

📖 **Full guide**: [Persistent Storage Setup](PRODUCTION.md#-persistent-storage-critical-for-production)

---

## What is n8n?

n8n is a **fair-code licensed workflow automation tool** - an open-source alternative to Zapier that you can self-host.

- 🔌 400+ integrations (Google, Slack, GitHub, etc.)
- 🎨 Visual workflow builder
- 🤖 AI capabilities with LangChain
- 🔐 Self-hosted - you own your data
- 💰 Free for personal use

## Documentation

### Getting Started
- **[Deployment Guide](DEPLOY_TO_DO.md)** - Step-by-step deployment
- **[How to Use n8n](HOW_TO_USE_N8N.md)** - Build your first workflow
- **[Environment Variables](ENV_TEMPLATE.md)** - Configuration reference

### Scaling & Production
- **[Scaling Guide](SCALING.md)** - When and how to scale
- **[Production Setup](PRODUCTION.md)** - Security & best practices
- **[Version Info](VERSION.md)** - Updates & versioning

### Mode-Specific Guides
- **[Simple Mode](docs/SIMPLE-MODE.md)** - Default deployment
- **[Queue Mode](docs/QUEUE-MODE.md)** - Scalable architecture
- **[Production Setup](docs/PRODUCTION-SETUP.md)** - Enterprise deployment

## What's Included

- n8n v1.122.2 (latest stable)
- PostgreSQL 17 database (persistent storage)
- SSL/TLS encryption
- Automated backups
- Health monitoring
- Optional: Spaces for persistent file storage ($5/month)

## Example Use Cases

- Automate repetitive tasks
- Sync data between apps
- Build custom API integrations
- Schedule automated reports
- Create webhook listeners
- Build AI-powered workflows

## Support

- **n8n Docs**: [docs.n8n.io](https://docs.n8n.io)
- **Community**: [community.n8n.io](https://community.n8n.io)
- **Templates**: [n8n.io/workflows](https://n8n.io/workflows)
- **Issues**: [GitHub](https://github.com/AppPlatform-Templates/n8n-appplatform/issues)

---

**Ready to automate?** Choose your tier above and start building workflows! 🚀
