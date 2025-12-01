# Scaling n8n on DigitalOcean App Platform

This guide helps you choose the right deployment tier and upgrade as your needs grow.

## Quick Decision Matrix

| Your Needs | Recommended Tier | Cost | Deploy Method |
|------------|-----------------|------|---------------|
| < 100 workflows/day | Simple | $27/mo | Deploy Button |
| 100-1000 workflows/day | Queue Mode | $54/mo | doctl CLI |
| Heavy Code nodes | With Runners | $39/mo | doctl CLI |
| 1000+ workflows/day | Production | $66/mo+ | doctl CLI |
| Enterprise scale | Production + Autoscale | $100+/mo | doctl CLI |

## Deployment Tiers Explained

### Tier 1: Simple Mode (Default)

**Architecture:**
```
┌─────────────┐
│     n8n     │ ← All-in-one
│  Main + UI  │
└──────┬──────┘
       │
   ┌───▼───┐
   │  PG   │
   └───────┘
```

**What you get:**
- Single n8n instance
- PostgreSQL database
- UI, API, webhook handling
- Direct workflow execution

**Best for:**
- Personal use
- Testing and development
- < 100 workflows per day
- Small teams (1-5 users)
- Simple workflows without heavy computation

**Limitations:**
- No horizontal scaling
- Single point of execution
- All workflows run sequentially or with limited concurrency
- Code nodes run in main process (less secure)

**Cost:** $27/month

**Deploy:**
```bash
# Use the Deploy-to-DO button in README
```

---

### Tier 2: Queue Mode

**Architecture:**
```
┌──────────┐
│   Main   │ ← UI + API
└────┬─────┘
     │
┌────▼─────┐    ┌────────┐
│  Redis   │◄───┤ Worker │
└──────────┘    │ Pool   │
     ▲          └────────┘
     │              ▲
     └──────────────┘
        Pull jobs
```

**What you get:**
- Main instance (UI/API)
- Separate worker instances
- Redis for job coordination
- Horizontal scalability

**How it works:**
1. User triggers workflow via UI/webhook
2. Main adds job to Redis queue
3. Available worker picks up job
4. Worker executes workflow
5. Results stored in PostgreSQL

**Best for:**
- 100-1000 workflows/day
- Multiple concurrent executions
- Prod

uction deployments
- Growing teams (5-20 users)
- Need for reliability

**Scaling:**
- Start with 1 worker
- Add workers as load increases
- Each worker handles ~10 concurrent workflows
- Redis coordinates all workers automatically

**Cost:**
- Base: $54/month (Main + 1 Worker + PG + Redis)
- +$12/month per additional worker

**Deploy:**
```bash
doctl apps create --spec .do/examples/queue-mode.yaml
```

---

### Tier 3: With Runners

**Architecture:**
```
┌──────────┐
│   n8n    │ ← Main
└────┬─────┘
     │
     │  Code execution
     ▼
┌──────────┐
│  Runner  │ ← Sandboxed JS/Python
└──────────┘
```

**What you get:**
- Main n8n instance
- Task runner instances
- Code execution sandbox
- No queue mode (single instance)

**How it works:**
1. Workflow executes normally
2. When hitting Code node:
   - Code sent to runner
   - Runner executes in sandbox
   - Result returned to main
3. Continues workflow execution

**Best for:**
- Workflows with JavaScript/Python nodes
- Need code execution sandboxing
- Security-conscious deployments
- Don't need massive scale (< 500 workflows/day)

**Limitations:**
- Still single main instance
- No worker pool
- Can't scale horizontally

**Cost:** $39/month (Main + 1 Runner + PG)

**Deploy:**
```bash
doctl apps create --spec .do/examples/with-runners.yaml
```

---

### Tier 4: Production (Full Stack)

**Architecture:**
```
┌──────────┐
│   Main   │ ← UI + API
│          │◄──── Runners (sandboxed code execution)
└────┬─────┘
     │
┌────▼─────┐
│  Redis   │
│  Queue   │
└────┬─────┘
     │
┌────▼──────┐
│Worker Pool│ ← Execute workflows
│ (services)│◄──── Runners (sandboxed code execution)
└───────────┘
```

**What you get:**
- Main service with task runners
- Multiple worker services (configured as services, not workers)
- Each worker has its own task runner pool
- Redis coordination
- Full scalability with sandboxed execution

**How it works:**
1. Main receives requests → adds to queue
2. Workers (services) pull jobs → execute workflows
3. For **all executions** (manual and automated):
   - Code nodes execute in sandboxed runners
   - Main's runners handle manual executions
   - Worker's runners handle queued executions
4. Worker completes workflow → updates database

**Best for:**
- 1000+ workflows/day
- Enterprise deployments
- Heavy Code node usage
- High availability requirements
- Teams (20+ users)

**Scaling:**
- Workers: Scale 2-10+ services (each pulls from queue)
- Runners: Each worker service has its own runner pool (1-2 runners per worker)
- Redis: Consider larger instance for high queue depth
- PostgreSQL: Enable HA, add replicas for production

**Auto-scaling:**
- Requires dedicated CPU instances (`apps-d-*`)
- Configure in spec (see production.yaml)
- Scales based on CPU usage

**Cost:**
- Base: $66/month (Main + 2 Workers + 2 Runners + PG + Redis)
- Scale: +$24/month per worker+runner pair
- Dedicated CPU: +$22/month per instance vs basic

**Deploy:**
```bash
doctl apps create --spec .do/examples/production.yaml
```

## When to Upgrade

### From Simple → Queue Mode

**Indicators:**
- ✓ Executing > 100 workflows/day
- ✓ Workflows timing out due to queue
- ✓ UI becomes slow during heavy execution
- ✓ Need to scale beyond single instance
- ✓ Want better reliability

**Migration:**
1. Provision Redis database
2. Update app spec with queue mode config
3. Add worker component
4. Deploy updated spec
5. Test workflow execution
6. Monitor Redis and worker metrics

### From Simple → With Runners

**Indicators:**
- ✓ Using Code nodes extensively
- ✓ Need code execution sandboxing
- ✓ Security requirements
- ✓ Don't need massive scale yet

**Migration:**
1. Add runner component to spec
2. Configure runner broker in main
3. Deploy updated spec
4. Test Code nodes execute in runners

### From Queue Mode → Production

**Indicators:**
- ✓ Executing > 1000 workflows/day
- ✓ Need sandboxed code execution for all workflows
- ✓ Workers hitting CPU limits
- ✓ Need auto-scaling
- ✓ Enterprise requirements

**Migration:**
1. Convert workers from `workers:` to `services:` in spec
2. Add `internal_ports: [5679]` to worker services (no http_port for internal-only)
3. Enable runners on workers (N8N_RUNNERS_ENABLED: true, N8N_RUNNERS_MODE: external)
4. Add runner pools for both main and worker services
5. Configure main runners: N8N_RUNNERS_TASK_BROKER_URI: http://n8n-main:5679
6. Configure worker runners: N8N_RUNNERS_TASK_BROKER_URI: http://n8n-worker:5679
7. Optionally switch to dedicated CPU for autoscaling
8. Deploy updated spec
9. Test: all executions (manual and queued) use sandboxed runners

## Scaling Individual Components

### Scaling Workers

**Horizontal Scaling:**
```bash
# Edit your spec, change instance_count
workers:
  - name: n8n-worker
    instance_count: 5  # Was 1, now 5

# Update deployment
doctl apps update YOUR_APP_ID --spec <updated-spec>
```

**When to add workers:**
- Redis queue depth consistently > 50
- Worker CPU > 80%
- Workflows queuing for execution
- Response time degrading

**Rule of thumb:**
- 1 worker = ~10 concurrent workflows
- 5 workers = ~50 concurrent workflows
- 10 workers = ~100 concurrent workflows

### Scaling Runners

**Each worker service has its own runner pool:**
- Start with 1-2 runners per worker service
- Main service runners: Handle manual/webhook executions
- Worker service runners: Handle queued workflow executions
- Scale runners proportionally with workers

**When to add runners:**
- High code execution concurrency
- Workers processing many Code node workflows
- Performance degradation in code execution
- Multiple concurrent workflow executions

**Horizontal Scaling:**
```bash
workers:
  - name: n8n-runner-main
    instance_count: 2  # For main service

  - name: n8n-runner-worker
    instance_count: 2  # For worker service (scale with workers)
```

**Note:** In production mode, each worker service acts as a task runner broker with its own dedicated runner pool for sandboxed code execution.

### Vertical Scaling

**Upgrade instance sizes when:**
- Memory consistently > 80%
- CPU consistently > 70%
- Database connections exhausted
- Slow query performance

**Example upgrade:**
```yaml
# From basic to more memory
instance_size_slug: apps-s-1vcpu-2gb  # Was apps-s-1vcpu-1gb

# To dedicated CPU (for autoscaling)
instance_size_slug: apps-d-1vcpu-1gb
```

### Database Scaling

**PostgreSQL:**
```bash
# Via control panel or doctl
doctl databases resize YOUR_DB_ID --size db-s-2vcpu-4gb
```

**Redis:**
- Start with smallest (db-s-1vcpu-1gb)
- Monitor memory usage
- Upgrade if memory > 80%

## Auto-scaling Configuration

**Requirements:**
- Dedicated CPU instances (`apps-d-*` tier)
- +$22/month per instance vs basic tier

**Configuration:**
```yaml
workers:
  - name: n8n-worker
    instance_size_slug: apps-d-1vcpu-1gb
    autoscaling:
      min_instance_count: 2
      max_instance_count: 10
      metrics:
        cpu:
          percent: 80  # Scale up at 80% CPU
```

**How it works:**
- Monitors CPU usage
- Scales up when > 80% for sustained period
- Scales down when < 80%
- Respects min/max limits

## Cost Optimization

### Development/Staging

**Recommendations:**
- Use Simple mode
- Use Dev databases ($7 vs $15)
- Scale down when not in use
- Use basic tier instances

**Cost:** ~$19/month

### Production

**Recommendations:**
- Use Queue Mode or Production tier
- Use managed databases (HA, backups)
- Enable autoscaling for cost efficiency
- Monitor and right-size instances

**Cost:** $54-$200+/month depending on scale

### Cost Comparison by Tier

| Tier | Base | +1 Worker | +1 Runner | Total Range |
|------|------|-----------|-----------|-------------|
| Simple | $27 | N/A | N/A | $27 |
| Queue | $54 | +$12 | N/A | $54-$174 |
| Runners | $39 | N/A | +$12 | $39-$63 |
| Production | $66 | +$12 | +$12 | $66-$300+ |

## Monitoring and Metrics

### Key Metrics to Watch

**Application:**
- Workflow execution rate
- Workflow duration
- Error rate
- Queue depth (Redis)

**Infrastructure:**
- CPU usage per component
- Memory usage per component
- Database connections
- Redis memory usage

### Setting Up Monitoring

**Enable metrics:**
```yaml
envs:
  - key: N8N_METRICS
    value: "true"
```

**Access metrics:**
```
https://your-app.ondigitalocean.app:9090/metrics
```

**App Platform Insights:**
- CPU, Memory, Request metrics
- Set up alerts in control panel
- Monitor in real-time

## Migration Checklist

### Pre-Migration

- [ ] Backup current deployment
- [ ] Export all workflows
- [ ] Document current configuration
- [ ] Test new spec locally if possible
- [ ] Plan maintenance window

### During Migration

- [ ] Provision new resources (Redis, etc.)
- [ ] Update app spec
- [ ] Deploy with `doctl apps update`
- [ ] Monitor deployment progress
- [ ] Verify components are healthy

### Post-Migration

- [ ] Test workflow execution
- [ ] Verify queue processing
- [ ] Check Code nodes (if using runners)
- [ ] Monitor metrics for 24-48 hours
- [ ] Adjust scaling if needed
- [ ] Update documentation

## Getting Help

- **Slow executions?** See [PRODUCTION.md](PRODUCTION.md)
- **Queue mode issues?** See [docs/QUEUE-MODE.md](docs/QUEUE-MODE.md)
- **Cost concerns?** Review instance sizes
- **Scale questions?** Open an issue

## Next Steps

1. **Assess current usage** using n8n's execution history
2. **Choose your tier** based on decision matrix
3. **Deploy or upgrade** using appropriate spec
4. **Monitor and optimize** based on actual usage
5. **Scale gradually** - start small, grow as needed

---

**Remember:** It's easier to scale up than down. Start with Simple mode and upgrade when you need it!
