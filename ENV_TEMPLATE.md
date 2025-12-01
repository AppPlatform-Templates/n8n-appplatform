# n8n Environment Variables Guide

This document describes all environment variables required to run n8n on DigitalOcean App Platform.

## Required Environment Variables

### Database Configuration

| Variable | Description | Example | Required |
|----------|-------------|---------|----------|
| `DB_TYPE` | Database type | `postgresdb` | ✅ Yes |
| `DB_POSTGRESDB_HOST` | PostgreSQL host | `${n8n-postgres.HOSTNAME}` | ✅ Yes |
| `DB_POSTGRESDB_PORT` | PostgreSQL port | `${n8n-postgres.PORT}` | ✅ Yes |
| `DB_POSTGRESDB_DATABASE` | Database name | `${n8n-postgres.DATABASE}` | ✅ Yes |
| `DB_POSTGRESDB_USER` | Database user | `${n8n-postgres.USERNAME}` | ✅ Yes |
| `DB_POSTGRESDB_PASSWORD` | Database password | `${n8n-postgres.PASSWORD}` | ✅ Yes (SECRET) |
| `DB_POSTGRESDB_SSL_ENABLED` | Enable SSL for database | `true` | ✅ Yes (DO managed DB) |
| `DB_POSTGRESDB_SSL_REJECT_UNAUTHORIZED` | SSL certificate validation | `false` | ✅ Yes (DO managed DB) |

**Note:** DigitalOcean managed databases require SSL connections.

### n8n Core Configuration

| Variable | Description | Example | Required |
|----------|-------------|---------|----------|
| `WEBHOOK_URL` | Webhook base URL (overrides N8N_HOST + N8N_PROTOCOL) | `${APP_URL}/` | ✅ Yes |
| `N8N_PROXY_HOPS` | Number of reverse proxies | `1` | ✅ Yes (App Platform) |
| `N8N_ENCRYPTION_KEY` | Encryption key for credentials | Random 32+ char string | ✅ Yes (SECRET) |

**IMPORTANT**: Generate a strong encryption key:
```bash
openssl rand -base64 32
```

**Note:** `WEBHOOK_URL` uses `${APP_URL}/` which App Platform automatically replaces with your app's public URL.

### Application Settings

| Variable | Description | Example | Required |
|----------|-------------|---------|----------|
| `GENERIC_TIMEZONE` | Workflow timezone | `Europe/Amsterdam` | No |
| `N8N_DIAGNOSTICS_ENABLED` | Send usage data | `false` | No |
| `N8N_METRICS` | Enable Prometheus metrics | `false` | No |
| `N8N_LOG_LEVEL` | Logging level | `info` | No |

## Optional Environment Variables

### File Storage (DigitalOcean Spaces)

If you need to store uploaded files persistently:

| Variable | Description | Example |
|----------|-------------|---------|
| `N8N_DEFAULT_BINARY_DATA_MODE` | Storage mode | `filesystem` |
| `AWS_ACCESS_KEY_ID` | Spaces access key | `DO00XXX...` |
| `AWS_SECRET_ACCESS_KEY` | Spaces secret key | `xxx...` |
| `AWS_S3_BUCKET` | Bucket name | `n8n-storage-ams3` |
| `AWS_S3_ENDPOINT` | Spaces endpoint | `https://ams3.digitaloceanspaces.com` |

### User Management

| Variable | Description | Example |
|----------|-------------|---------|
| `N8N_BASIC_AUTH_ACTIVE` | Enable basic auth | `true` |
| `N8N_BASIC_AUTH_USER` | Auth username | `admin` |
| `N8N_BASIC_AUTH_PASSWORD` | Auth password | `secure-password` |

Or use the built-in user management (recommended):
- First user created becomes the owner
- Additional users can be invited via the UI

### SMTP (Email Notifications)

| Variable | Description | Example |
|----------|-------------|---------|
| `N8N_EMAIL_MODE` | Email mode | `smtp` |
| `N8N_SMTP_HOST` | SMTP server | `smtp.gmail.com` |
| `N8N_SMTP_PORT` | SMTP port | `587` |
| `N8N_SMTP_USER` | SMTP username | `your-email@gmail.com` |
| `N8N_SMTP_PASS` | SMTP password | `your-app-password` |
| `N8N_SMTP_SENDER` | From email | `n8n@yourdomain.com` |

### Advanced Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `EXECUTIONS_DATA_SAVE_ON_ERROR` | Save failed executions | `all` |
| `EXECUTIONS_DATA_SAVE_ON_SUCCESS` | Save successful executions | `all` |
| `EXECUTIONS_DATA_SAVE_MANUAL_EXECUTIONS` | Save manual runs | `true` |
| `N8N_METRICS` | Enable Prometheus metrics | `false` |
| `N8N_LOG_LEVEL` | Logging level | `info` |
| `N8N_LOG_OUTPUT` | Log output format | `console` |

## Security Best Practices

### 1. Encryption Key
- **Never reuse** encryption keys between environments
- **Never commit** encryption keys to git
- **Generate** a new random key for each deployment:
  ```bash
  openssl rand -hex 32
  ```

### 2. Database Credentials
- Use **strong passwords** (20+ characters)
- Enable **trusted sources** in database firewall after testing
- Consider using **VPC** for additional network isolation

### 3. Basic Auth
- If using basic auth, use **strong passwords**
- Consider using **n8n's built-in user management** instead
- Enable **2FA** when available

## Environment-Specific Settings

### Development
```bash
N8N_DIAGNOSTICS_ENABLED=true
N8N_LOG_LEVEL=debug
```

### Production
```bash
N8N_DIAGNOSTICS_ENABLED=false
N8N_LOG_LEVEL=info
EXECUTIONS_DATA_SAVE_ON_SUCCESS=all
N8N_METRICS=true
```

## Setting Environment Variables in App Platform

### Via Control Panel
1. Go to your app in the DigitalOcean control panel
2. Click on the n8n service
3. Go to "Environment Variables"
4. Click "Edit" and add your variables
5. Mark sensitive values (passwords, keys) as "Encrypted"

### Via Deploy Button
When using the Deploy-to-DO button:
1. Click the button
2. Fill in the required environment variables
3. Replace `CHANGE_THIS_TO_RANDOM_STRING` with actual values
4. Deploy

### Via doctl CLI
```bash
doctl apps update YOUR_APP_ID --spec .do/app.yaml
```

## Troubleshooting

### Database Connection Issues
- Verify `DB_POSTGRESDB_*` variables are correct
- Check database firewall allows App Platform
- Ensure database is in the same region or has VPC access

### Webhook Issues
- Ensure `WEBHOOK_URL` matches your app's public URL
- Must use `https://` (not `http://`)
- Include trailing slash: `https://your-app.ondigitalocean.app/`

### File Upload Issues
- Configure Spaces environment variables
- Ensure Spaces bucket exists
- Verify Spaces credentials are correct

## References

- [n8n Environment Variables Documentation](https://docs.n8n.io/hosting/configuration/environment-variables/)
- [DigitalOcean App Platform Environment Variables](https://docs.digitalocean.com/products/app-platform/how-to/use-environment-variables/)
- [n8n Configuration](https://docs.n8n.io/hosting/configuration/)

## Queue Mode Configuration (Advanced)

For scalable deployments with worker pools. See [SCALING.md](SCALING.md) and [docs/QUEUE-MODE.md](docs/QUEUE-MODE.md).

### Redis Configuration

| Variable | Description | Example | Required |
|----------|-------------|---------|----------|
| `EXECUTIONS_MODE` | Execution mode | `queue` | ✅ Yes |
| `QUEUE_BULL_REDIS_HOST` | Redis hostname | `${n8n-redis.HOSTNAME}` | ✅ Yes |
| `QUEUE_BULL_REDIS_PORT` | Redis port | `${n8n-redis.PORT}` | ✅ Yes |
| `QUEUE_BULL_REDIS_PASSWORD` | Redis password | `${n8n-redis.PASSWORD}` | ✅ Yes (SECRET) |
| `QUEUE_BULL_REDIS_TLS` | Enable TLS for Redis | `true` | ✅ Yes (DO managed DB) |
| `QUEUE_HEALTH_CHECK_ACTIVE` | Enable queue health checks | `true` | No |

**Note:** DigitalOcean managed Redis/Valkey requires TLS connections.

### Worker Configuration

| Variable | Description | Default | Notes |
|----------|-------------|---------|-------|
| `N8N_CONCURRENCY_PRODUCTION_LIMIT` | Max concurrent workflows per worker | `10` | Adjust based on workflow complexity |

**Example Queue Mode Setup:**
```bash
# Main Service
EXECUTIONS_MODE=queue
QUEUE_BULL_REDIS_HOST=${n8n-redis.HOSTNAME}
QUEUE_BULL_REDIS_PORT=${n8n-redis.PORT}
QUEUE_BULL_REDIS_PASSWORD=${n8n-redis.PASSWORD}

# Worker
EXECUTIONS_MODE=queue
QUEUE_BULL_REDIS_HOST=${n8n-redis.HOSTNAME}
N8N_CONCURRENCY_PRODUCTION_LIMIT=10
```

## Task Runner Configuration (Advanced)

For secure JavaScript/Python code execution. See [docs/WITH-RUNNERS.md](docs/WITH-RUNNERS.md).

### Runner Variables (Main Service)

| Variable | Description | Example |
|----------|-------------|---------|
| `N8N_RUNNERS_ENABLED` | Enable task runners | `true` |
| `N8N_RUNNERS_MODE` | Runner mode | `external` |
| `N8N_RUNNERS_BROKER_LISTEN_ADDRESS` | Broker address | `0.0.0.0` |
| `N8N_RUNNERS_AUTH_TOKEN` | Runner authentication | Random 32+ char string (SECRET) |

**Generate runner token:**
```bash
openssl rand -base64 32
```

**CRITICAL:** Main service must expose internal port 5679 in app spec:
```yaml
services:
  - name: n8n
    internal_ports:
      - 5679  # Task Runner broker port
```

### Runner Variables (Runner Component)

| Variable | Description | Example |
|----------|-------------|---------|
| `N8N_RUNNERS_TASK_BROKER_URI` | Main service broker | `http://n8n:5679` or `http://n8n-main:5679` |
| `N8N_RUNNERS_AUTH_TOKEN` | Same token as main | (SECRET) |

**Example Runner Setup:**
```yaml
# Main Service
services:
  - name: n8n
    internal_ports:
      - 5679
    envs:
      - key: N8N_RUNNERS_ENABLED
        value: "true"
      - key: N8N_RUNNERS_MODE
        value: external
      - key: N8N_RUNNERS_BROKER_LISTEN_ADDRESS
        value: 0.0.0.0
      - key: N8N_RUNNERS_AUTH_TOKEN
        value: your-generated-token
        scope: RUN_TIME
        type: SECRET

# Runner Component
workers:
  - name: n8n-runner
    envs:
      - key: N8N_RUNNERS_TASK_BROKER_URI
        value: http://n8n:5679
      - key: N8N_RUNNERS_AUTH_TOKEN
        value: same-token-as-main
        scope: RUN_TIME
        type: SECRET
```

## Deployment Tier Environment Variables

### Simple Mode
Required: Database + Core n8n variables (see above)

### Queue Mode  
Required: Database + Core + Redis + Queue variables

### With Runners
Required: Database + Core + Runner variables

### Production
Required: All of the above (Database + Core + Redis + Queue + Runner)

