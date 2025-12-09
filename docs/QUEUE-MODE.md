# Queue Mode Deployment

Queue mode separates UI/API from workflow execution for scalability.

## Architecture

```
Main (UI/API) → Redis Queue → Workers (Execute)
```

## Components

1. **Main Service**: UI, API, webhooks
2. **Workers**: Pull jobs from Redis, execute workflows
3. **Redis**: Job queue coordination
4. **PostgreSQL**: Data storage

For detailed pricing information, visit the [DigitalOcean App Platform Pricing](https://www.digitalocean.com/pricing/app-platform) page.

## Deployment

```bash
doctl apps create --spec .do/examples/queue-mode.yaml
```

## Configuration

**Key Environment Variables:**
```
EXECUTIONS_MODE=queue
QUEUE_BULL_REDIS_HOST=${n8n-redis.HOSTNAME}
N8N_CONCURRENCY_PRODUCTION_LIMIT=10
```

## Scaling Workers

Edit spec and update:
```yaml
workers:
  - name: n8n-worker
    instance_count: 5  # Scale as needed
```

Then:
```bash
doctl apps update YOUR_APP_ID --spec <updated-spec>
```

## Monitoring

- Watch Redis queue depth
- Monitor worker CPU/memory
- Check execution times

## Troubleshooting

**Workers not picking up jobs:**
- Verify Redis connection
- Check EXECUTIONS_MODE=queue on all components
- Ensure same encryption key on main and workers

**Queue backing up:**
- Add more workers
- Check worker health
- Review workflow complexity

See [SCALING.md](../SCALING.md) for detailed scaling guide.
