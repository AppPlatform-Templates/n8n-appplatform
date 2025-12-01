# Production Setup

Production deployment with queue mode and task runners for scalable workflow execution.

## Architecture

```
                    ┌─────────────┐
                    │   Main      │
                    │ UI/API/     │
                    │ Webhooks    │◄──── Runners (sandboxed code)
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │    Redis    │
                    │  Job Queue  │
                    └──────┬──────┐
                           │
                    ┌──────▼──────┐
                    │   Workers   │
                    │  (Execute   │◄──── Runners (sandboxed code)
                    │  workflows) │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │ PostgreSQL  │
                    └─────────────┘
```

## Components

- **Main**: UI/API/Webhooks with task runner broker and runners
- **Workers**: Execute queued workflows with task runner broker and runners (configured as services)
- **Runners**: Separate pools for main and workers (sandboxed code execution)
- **Redis**: Job queue coordination
- **PostgreSQL**: Data storage (HA enabled)

## Task Runner Architecture

Each component (main and workers) acts as a task runner broker with its own runner pool:
- ✅ **Main service**: Has runners for manual/webhook executions
- ✅ **Worker services**: Have their own runners for queued workflow executions
- ✅ **All Code nodes** execute in sandboxed runners (secure isolation)

This architecture provides:
- Full sandboxing for all code execution (manual and automated)
- Each worker has its own task runner pool (follows n8n's sidecar pattern)
- Horizontal scaling through worker services with dedicated runners

## Cost

- Base: $66/month
- Scale: +$24/month per worker+runner pair
- Auto-scale: +$22/month per instance (dedicated CPU)

## Deployment

```bash
doctl apps create --spec .do/examples/production.yaml
```

## Production Checklist

### Pre-Deployment
- [ ] Generate strong encryption key
- [ ] Generate runner auth token
- [ ] Plan instance sizes
- [ ] Configure database HA

### Deployment
- [ ] Deploy with production spec
- [ ] Verify all components healthy
- [ ] Test workflow execution (both manual and scheduled)
- [ ] Test Code nodes via UI (should use main's runners)
- [ ] Test Code nodes via scheduled trigger (should use worker's runners)

### Post-Deployment
- [ ] Enable database trusted sources
- [ ] Set up monitoring/alerts
- [ ] Configure backups
- [ ] Document configuration

## Auto-scaling

Requires dedicated CPU instances:

```yaml
workers:
  - name: n8n-worker
    instance_size_slug: apps-d-1vcpu-1gb
    autoscaling:
      min_instance_count: 2
      max_instance_count: 10
      metrics:
        cpu:
          percent: 80
```

## Monitoring

**Critical Metrics:**
- Queue depth (Redis) - ensure workers keep up
- Worker CPU/Memory - watch for code execution overhead
- Runner CPU/Memory - monitor sandboxed execution load
- Main service CPU/Memory - UI/API/webhook handling
- Database connections - prevent connection exhaustion
- Execution times - identify slow workflows

**App Platform Insights:**
- Set up CPU/Memory alerts (especially for workers)
- Monitor request rates on main service
- Track error rates across all components
- Watch for task timeout errors in logs

## Security

1. **Database**: Enable trusted sources after testing
2. **Encryption**: Use strong, unique keys
3. **Runners**: Secure auth tokens
4. **Access**: Configure user roles

## Backup Strategy

- Database: Automatic daily backups
- Workflows: Export regularly via API
- Configuration: Version control app specs

## Disaster Recovery

1. Maintain up-to-date app spec
2. Document environment variables
3. Export workflows monthly
4. Test restore procedures

See [PRODUCTION.md](../PRODUCTION.md) for complete production guide.
