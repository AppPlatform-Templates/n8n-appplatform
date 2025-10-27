# Production Setup

Full production deployment with queue mode, workers, and task runners.

## Architecture

```
Main → Redis → Workers → Runners
  ↓              ↓          ↓
 PostgreSQL   Execute   Code Execution
```

## Components

- Main: UI/API/Broker
- Workers: 2+ instances
- Runners: 2+ instances
- Redis: Job queue
- PostgreSQL: Data (HA enabled)

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
- [ ] Test workflow execution
- [ ] Test Code nodes

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
- Queue depth (Redis)
- Worker CPU/Memory
- Runner CPU/Memory
- Database connections
- Execution times

**App Platform Insights:**
- Set up CPU/Memory alerts
- Monitor request rates
- Track error rates

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
