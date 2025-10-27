# Simple Mode Deployment

Simple mode is the default n8n deployment - a single instance handling all operations.

## What You Get

- All-in-one n8n instance (UI + API + execution)
- PostgreSQL database
- SSL/TLS encryption
- **Cost:** $27/month

## Architecture

Single instance executes workflows directly:
```
User → n8n Instance → PostgreSQL
```

## Best For

- Personal use, testing
- < 100 workflows/day
- Small teams (1-5 users)
- Simple workflows

## Deployment

Use the Deploy-to-DO button in the README.

## Limitations

- No horizontal scaling
- All workflows execute in main process
- Limited concurrency
- Code nodes less secure (no sandboxing)

## When to Upgrade

See [SCALING.md](../SCALING.md) if you need:
- > 100 workflows/day
- Multiple concurrent executions
- Code execution sandboxing
- High availability

## Troubleshooting

See [DEPLOY_TO_DO.md](../DEPLOY_TO_DO.md) for common issues.
