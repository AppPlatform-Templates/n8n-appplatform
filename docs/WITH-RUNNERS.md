# With Runners Deployment

Single instance deployment with external task runners for secure JavaScript/Python code execution.

## Architecture

```
User → n8n Instance (UI/API/Execution) ⟷ Task Runners (Sandboxed Code)
          ↓
     PostgreSQL
```

## Components

1. **Main Service**: UI, API, webhooks, workflow execution
2. **Task Runners**: Execute JavaScript/Python Code nodes in sandboxed containers
3. **PostgreSQL**: Data storage

## Cost

- Base: $39/month (Main + 1 Runner + PostgreSQL)
- Scale: +$12/month per additional runner

## Best For

- < 500 workflows/day
- Heavy use of Code nodes (JavaScript/Python)
- Security-critical code execution
- Multi-tenant environments
- Untrusted user scripts

## Deployment

```bash
doctl apps create --spec .do/examples/with-runners.yaml
```

## Configuration

**Key Environment Variables:**
```
N8N_RUNNERS_ENABLED=true
N8N_RUNNERS_MODE=external
N8N_RUNNERS_BROKER_LISTEN_ADDRESS=0.0.0.0
N8N_RUNNERS_AUTH_TOKEN=<secret>
```

**Internal Ports:**
```yaml
services:
  - name: n8n
    internal_ports:
      - 5679  # Task Runner broker port
```

## How Task Runners Work

1. Workflow encounters Code node
2. n8n sends task to broker (port 5679)
3. Runner picks up task
4. Code executes in sandboxed container
5. Result returns to workflow

## Benefits

- ✅ **Security**: Code executes in isolated containers
- ✅ **Stability**: Code crashes don't affect main instance
- ✅ **Resource isolation**: Heavy code execution doesn't impact UI
- ✅ **Multi-tenant safe**: Each execution is isolated

## Limitations

- No queue mode (single instance)
- Limited workflow concurrency
- Not suitable for > 500 workflows/day
- All executions run on main instance

## Scaling Runners

Edit spec and update runner instance count:
```yaml
workers:
  - name: n8n-runner
    instance_count: 3  # Scale based on Code node usage
```

Then:
```bash
doctl apps update YOUR_APP_ID --spec <updated-spec>
```

## Monitoring

- Watch runner CPU/memory usage
- Monitor task broker queue depth
- Check for task timeouts (60s default)

## Troubleshooting

**Task request timed out:**
- Verify `internal_ports: [5679]` is configured
- Check runner logs: `doctl apps logs YOUR_APP_ID n8n-runner`
- Ensure `N8N_RUNNERS_AUTH_TOKEN` matches between main and runners
- Verify runners can connect to `http://n8n:5679`

**Runners not connecting:**
- Check `N8N_RUNNERS_TASK_BROKER_URI=http://n8n:5679` on runners
- Verify broker is ready in main logs: "n8n Task Broker ready"
- Ensure both containers are in same app (internal networking)

**Code execution fails but manual test works:**
- Check runner logs for errors
- Verify runner has required Python packages
- Check memory limits on runner instances

## When to Use Queue Mode Instead

Upgrade to queue-mode if you need:
- > 500 workflows/day
- Horizontal scaling of workflow execution
- High availability
- Separate webhook/UI from execution layer

See [QUEUE-MODE.md](QUEUE-MODE.md) for queue mode deployment.

## When to Use Production Mode

Upgrade to production if you need:
- Queue mode + runners (1000+ workflows/day)
- Both scalability AND code sandboxing
- Enterprise scale

**Note:** Production mode on App Platform has runners only on main service. Workers execute code in-process due to platform limitations. See [PRODUCTION-SETUP.md](PRODUCTION-SETUP.md).
