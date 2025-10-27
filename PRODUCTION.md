# Production Deployment Guide

This guide covers best practices for deploying n8n to production on DigitalOcean App Platform.

## 🔒 Security Hardening

### 1. Database Security

#### Enable Trusted Sources

After initial deployment and testing, restrict database access:

```bash
# Using doctl CLI
doctl databases firewalls replace YOUR_DATABASE_ID \
  --rule app:YOUR_APP_ID

# Or via Control Panel:
# 1. Go to Databases → Your database
# 2. Settings → Trusted Sources
# 3. Remove "All IPv4" (0.0.0.0/0)
# 4. Add your app from dropdown
# 5. Save changes
```

#### Use VPC for Additional Isolation

For maximum security:

1. Create a VPC in your region
2. Add database to VPC
3. Add app to same VPC
4. All traffic stays on private network

### 2. Encryption Key Management

**Critical Security Practice**:

```bash
# Generate a strong encryption key
openssl rand -hex 32

# Store securely - NEVER:
# - Commit to git
# - Share publicly
# - Reuse across environments
# - Use weak/predictable keys
```

**Key Rotation** (if needed):
1. Create backup of workflows
2. Generate new key
3. Update environment variable
4. Re-encrypt credentials manually
5. Test all workflows

### 3. User Authentication

#### Option 1: Built-in User Management (Recommended)

Enable in n8n settings:
- First user = Owner (full permissions)
- Invite additional users via email
- Assign roles: Owner, Admin, Member
- Enable 2FA when available

#### Option 2: Basic Authentication

Set environment variables:
```bash
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=<strong-password>
```

⚠️ **Note**: Built-in user management is more secure and feature-rich.

### 4. HTTPS & SSL

✅ **Automatic with App Platform**:
- Free SSL/TLS certificates
- Auto-renewal
- HTTPS enforcement
- HTTP → HTTPS redirect

No configuration needed!

### 5. Webhook Security

**Best Practices**:
1. Use authentication in webhook workflows
2. Validate incoming data
3. Use webhook authentication headers:
   ```javascript
   // In webhook node settings
   if ($request.headers['x-api-key'] !== 'your-secret-key') {
     throw new Error('Unauthorized');
   }
   ```
4. Rate limit webhooks (use n8n's built-in settings)

## 📊 Monitoring & Observability

### 1. Enable Application Metrics

Set environment variables:
```bash
N8N_METRICS=true
N8N_METRICS_PORT=9090
```

Access Prometheus metrics:
```
https://your-app.ondigitalocean.app:9090/metrics
```

### 2. Configure Logging

**Production Settings**:
```bash
N8N_LOG_LEVEL=info
N8N_LOG_OUTPUT=console
N8N_LOG_FILE_LOCATION=/home/node/.n8n/logs/
```

**Log Levels**:
- `error` - Errors only
- `warn` - Warnings + errors
- `info` - Standard (recommended for production)
- `verbose` - Detailed
- `debug` - Very detailed (development only)

### 3. Execution Data

Configure what to save:
```bash
# Save all executions
EXECUTIONS_DATA_SAVE_ON_ERROR=all
EXECUTIONS_DATA_SAVE_ON_SUCCESS=all
EXECUTIONS_DATA_SAVE_MANUAL_EXECUTIONS=true

# Or save errors only (saves space)
EXECUTIONS_DATA_SAVE_ON_SUCCESS=none
```

### 4. DigitalOcean Monitoring

Enable in App Platform:
1. Go to your app → Insights
2. View CPU, Memory, Request metrics
3. Set up alerts:
   - CPU > 80%
   - Memory > 90%
   - 5xx errors > threshold
   - Pod crashes

## 💾 Backup Strategy

### 1. Database Backups

**Automatic (Managed Database)**:
- Daily automated backups
- 7-day retention (standard)
- Point-in-time recovery
- One-click restore

**Manual Backup**:
```bash
# Using pg_dump
pg_dump "$DATABASE_URL" > n8n_backup_$(date +%Y%m%d).sql

# Compress
gzip n8n_backup_*.sql

# Upload to Spaces
s3cmd put n8n_backup_*.sql.gz s3://your-bucket/backups/
```

### 2. Workflow Backup

**Method 1: Export via UI**
1. Go to Workflows
2. Select workflows
3. Click "Export"
4. Save JSON file

**Method 2: API Export** (automated)
```bash
# Get all workflows
curl https://your-app.ondigitalocean.app/api/v1/workflows \
  -H "X-N8N-API-KEY: your-api-key" \
  > workflows_backup.json
```

### 3. Credentials Backup

⚠️ **Important**: Credentials are encrypted with `N8N_ENCRYPTION_KEY`

**Backup Process**:
1. Note your `N8N_ENCRYPTION_KEY` (store securely)
2. Export database
3. Store backup encrypted
4. Test restore procedure

**Recovery**:
- Same encryption key required
- Import database backup
- Credentials decrypt automatically

## 🚀 Performance Optimization

### 1. Instance Sizing

**Current**: apps-s-1vcpu-1gb ($12/mo)

**Upgrade When**:
- CPU consistently > 70%
- Memory consistently > 80%
- Workflows timing out
- Slow UI performance

**Recommended Sizes**:
- **Light use** (< 100 workflows/day): apps-s-1vcpu-1gb
- **Medium use** (100-1000/day): apps-s-1vcpu-2gb
- **Heavy use** (1000+/day): apps-s-2vcpu-4gb
- **Production** (high availability): apps-d-* (dedicated CPU)

### 2. Database Sizing

**Current**: db-s-1vcpu-1gb ($15/mo)

**Upgrade When**:
- Database CPU > 80%
- Slow query performance
- Connection pool exhausted
- Storage > 80% used

**Monitoring**:
```bash
# Check database size
SELECT pg_database_size('defaultdb');

# Check table sizes
SELECT schemaname, tablename, pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename))
FROM pg_tables
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

### 3. Workflow Optimization

**Best Practices**:
1. **Batch processing**: Process items in groups
2. **Use filters**: Filter early, process less
3. **Caching**: Cache API responses when possible
4. **Pagination**: Handle large datasets in chunks
5. **Async operations**: Use webhooks for long-running tasks

**Example - Batch Processing**:
```javascript
// Instead of 1000 separate API calls
// Group into batches of 50
const batchSize = 50;
const items = $input.all();
const batches = [];

for (let i = 0; i < items.length; i += batchSize) {
  batches.push(items.slice(i, i + batchSize));
}

return batches.map(batch => ({ json: { items: batch } }));
```

### 4. Database Optimization

**Connection Pooling**:
```bash
# Environment variables
DB_POSTGRESDB_POOL_SIZE=20
DB_POSTGRESDB_IDLE_TIMEOUT_MILLIS=10000
```

**Query Optimization**:
- Add indexes for frequently queried fields
- Use EXPLAIN to analyze slow queries
- Archive old execution data

**Cleanup Old Executions**:
```sql
-- Delete executions older than 30 days
DELETE FROM execution_entity
WHERE "startedAt" < NOW() - INTERVAL '30 days';
```

## 🔄 Scaling Strategies

### Vertical Scaling (Single Instance)

**When**: Up to ~1000 executions/day

**How**:
1. Upgrade app instance size
2. Upgrade database size
3. Optimize workflows
4. Add caching

**Cost**: $50-100/month

### Horizontal Scaling (Multiple Instances)

**When**: > 1000 executions/day, need HA

**Requirements**:
- Queue mode enabled
- Redis for coordination
- Shared storage (Spaces)
- Load balancer

**Configuration**:
```bash
# Main instance (API + UI)
N8N_MODE=main
EXECUTIONS_MODE=queue
QUEUE_BULL_REDIS_HOST=your-redis.db.ondigitalocean.com
QUEUE_BULL_REDIS_PASSWORD=your-redis-password

# Worker instances (execution only)
N8N_MODE=worker
EXECUTIONS_MODE=queue
QUEUE_BULL_REDIS_HOST=your-redis.db.ondigitalocean.com
```

**Cost**: $100-200+/month

### Read Replicas (Database)

**When**: High read load

**How**:
1. Add read replica to database
2. Configure connection URL
3. Route read queries to replica

**Cost**: + $15/month per replica

## 🔍 Health Checks

### Application Health

**Endpoint**: `/healthz`

**Expected Response**:
```json
{
  "status": "ok"
}
```

**App Platform Configuration**:
```yaml
health_check:
  http_path: /healthz
  initial_delay_seconds: 60
  period_seconds: 10
  timeout_seconds: 3
```

### Database Health

**Check Connection**:
```bash
# Via psql
psql "$DATABASE_URL" -c "SELECT 1"

# Via n8n workflow
# Postgres node: SELECT 1
```

### Manual Testing

**Checklist**:
- [ ] UI loads correctly
- [ ] Can create workflow
- [ ] Can execute workflow
- [ ] Database queries work
- [ ] Webhooks receive data
- [ ] Credentials authenticate
- [ ] Email notifications sent

## 📈 Capacity Planning

### Expected Usage

| Workflows/Day | Instance | Database | Est. Cost/Month |
|---------------|----------|----------|-----------------|
| < 100 | apps-s-1vcpu-1gb | db-s-1vcpu-1gb | $27 |
| 100-500 | apps-s-1vcpu-2gb | db-s-1vcpu-2gb | $46 |
| 500-2000 | apps-s-2vcpu-4gb | db-s-2vcpu-4gb | $110 |
| 2000+ | Multiple workers | db-s-4vcpu-8gb | $200+ |

### Growth Planning

**Monitoring Triggers**:
1. **CPU > 70%** for 1 hour → Plan upgrade
2. **Memory > 80%** → Upgrade soon
3. **DB connections > 80%** → Increase pool size
4. **Execution queue > 100** → Add workers

## 🆘 Incident Response

### Common Issues

#### 1. Application Crash

**Symptoms**:
- 502/503 errors
- "Service Unavailable"
- Pod restart loops

**Diagnosis**:
```bash
# Check logs
doctl apps logs YOUR_APP_ID --type run

# Check status
doctl apps get YOUR_APP_ID
```

**Solutions**:
- Check environment variables
- Verify database connection
- Review recent changes
- Check resource limits

#### 2. High Memory Usage

**Symptoms**:
- Slow performance
- Crashes under load

**Solutions**:
- Upgrade instance size
- Optimize workflows
- Clear old executions
- Review code nodes for memory leaks

#### 3. Database Connection Pool Exhausted

**Symptoms**:
- "Too many connections" errors
- Slow workflow execution

**Solutions**:
- Increase pool size
- Optimize long-running queries
- Add connection timeout
- Upgrade database

#### 4. Webhook Failures

**Symptoms**:
- 404 on webhook URLs
- Workflows not triggering

**Solutions**:
- Verify `WEBHOOK_URL` setting
- Check workflow is active
- Test with curl
- Review firewall rules

## 🔐 Compliance & Data Privacy

### Data Residency

**Control Where Data Lives**:
- Choose region (ams3 for GDPR compliance)
- Database in same region
- Spaces bucket in same region

### GDPR Compliance

**Requirements**:
- User consent for data processing
- Right to access data
- Right to deletion
- Data encryption (✅ automatic)
- Audit logging (enable via n8n settings)

### SOC 2 / ISO 27001

**DigitalOcean Certifications**:
- SOC 2 Type II compliant
- ISO 27001 certified
- GDPR compliant

**Your Responsibilities**:
- Secure credentials
- Regular security updates
- Access control
- Backup and recovery procedures

## 📝 Maintenance Checklist

### Daily
- [ ] Check application status
- [ ] Review error logs
- [ ] Monitor execution queue

### Weekly
- [ ] Review resource usage
- [ ] Check database size
- [ ] Test critical workflows
- [ ] Review security logs

### Monthly
- [ ] Update environment variables if needed
- [ ] Review and cleanup old executions
- [ ] Test backup restore procedure
- [ ] Security audit
- [ ] Performance review

### Quarterly
- [ ] Review capacity planning
- [ ] Update n8n version
- [ ] Security assessment
- [ ] Disaster recovery drill
- [ ] Cost optimization review

## 🆕 Updating n8n

See [VERSION.md](VERSION.md) for detailed update instructions.

**Quick Steps**:
1. Review changelog for breaking changes
2. Test in development environment
3. Backup database and workflows
4. Update deployment configuration
5. Monitor for issues
6. Roll back if needed

## 📚 Additional Resources

- [n8n Production Best Practices](https://docs.n8n.io/hosting/best-practices/)
- [DigitalOcean App Platform Limits](https://docs.digitalocean.com/products/app-platform/details/limits/)
- [Database Scaling Guide](https://docs.digitalocean.com/products/databases/postgresql/how-to/scale/)
- [App Platform Monitoring](https://docs.digitalocean.com/products/app-platform/how-to/manage-deployments/)

---

**Questions?** Open an issue on [GitHub](https://github.com/AppPlatform-Templates/n8n-appplatform/issues)

## Deployment Tiers

n8n on App Platform supports multiple deployment tiers. Choose based on your needs:

### Simple Mode ($27/month)
- Single instance
- Best for: < 100 workflows/day
- See [docs/SIMPLE-MODE.md](docs/SIMPLE-MODE.md)

### Queue Mode ($54/month base)
- Main + Workers + Redis
- Best for: 100-1000 workflows/day  
- Horizontal scaling
- See [docs/QUEUE-MODE.md](docs/QUEUE-MODE.md)

### Production Mode ($66/month base)
- Queue + Workers + Runners
- Best for: 1000+ workflows/day
- Full scalability
- See [docs/PRODUCTION-SETUP.md](docs/PRODUCTION-SETUP.md)

**Migration guide:** See [SCALING.md](SCALING.md)

## Queue Mode Architecture

### How Queue Mode Works

```
User Request
     ↓
Main Service (UI/API)
     ↓
Redis Queue
     ↓
Worker Pool (pulls jobs)
     ↓
Executes Workflow
     ↓
Stores Results (PostgreSQL)
```

### Benefits
- **Horizontal scalability**: Add workers as needed
- **Reliability**: Jobs persist in Redis if worker fails
- **Load distribution**: Automatic via Redis
- **Better UX**: API returns immediately

### Configuration

**Main Service:**
- Runs UI/API/webhooks
- Adds jobs to Redis queue
- Does NOT execute workflows

**Workers:**
- Pull jobs from Redis
- Execute workflows
- Report results
- Scale horizontally

**Redis:**
- Job queue coordinator
- Maintains execution state
- Handles worker coordination

### Scaling Workers

**Rule of Thumb:**
- 1 worker = ~10 concurrent workflows
- Scale based on queue depth
- Monitor CPU/memory per worker

**How to Scale:**
```bash
# Edit your app spec
workers:
  - name: n8n-worker
    instance_count: 5  # Increase as needed

# Update deployment
doctl apps update YOUR_APP_ID --spec <spec-file>
```

## Task Runners

### Runner Architecture

```
Worker hits Code node
        ↓
Sends to Main's Runner Broker (port 5679)
        ↓
Runner Pool picks up task
        ↓
Executes JavaScript/Python in sandbox
        ↓
Returns result to Worker
        ↓
Worker continues workflow
```

### When to Use Runners

✅ **Use runners when:**
- Workflows use Code nodes extensively
- Need secure code execution
- Running untrusted/user-provided code
- Security/compliance requirements

❌ **Skip runners if:**
- No Code nodes in workflows
- Only using built-in nodes
- Cost-sensitive (saves $12/month per runner)

### Scaling Runners

**Matching Workers:**
- Code-light: 1 runner per 2 workers
- Code-moderate: 1 runner per worker  
- Code-heavy: 2 runners per worker

**Monitor:**
- Runner CPU usage
- Code execution times
- Queue wait times

## Redis Performance

### Monitoring Redis

**Key Metrics:**
- Memory usage
- Connection count
- Command latency
- Evicted keys

**App Platform Redis:**
- Managed Redis service
- Automatic backups
- High availability option

### Sizing Redis

**Start Small:**
- db-s-1vcpu-1gb ($15/month)
- Good for < 1000 jobs/day

**Scale Up When:**
- Memory > 80%
- High eviction rate
- Connection limit reached

**Larger Sizes:**
- db-s-1vcpu-2gb ($30/month)
- db-s-2vcpu-4gb ($60/month)

## Database in Queue Mode

### Considerations

**Increased Load:**
- Queue mode = more concurrent executions
- More database connections
- Higher write throughput

**Connection Pooling:**
```yaml
envs:
  - key: DB_POSTGRESDB_POOL_SIZE
    value: "20"  # Per worker
```

**Scaling Database:**
- Monitor connection usage
- Watch for slow queries
- Consider read replicas for reporting

### Database HA

For production:
```yaml
databases:
  - name: n8n-db
    engine: PG
    version: "17"
    production: true  # Enables HA
```

## Complete Production Checklist

### Architecture
- [ ] Deployed with production.yaml spec
- [ ] Main service healthy
- [ ] 2+ workers deployed
- [ ] 2+ runners deployed (if needed)
- [ ] Redis operational
- [ ] PostgreSQL with HA enabled

### Configuration
- [ ] Strong encryption key
- [ ] Secure runner token
- [ ] Database trusted sources enabled
- [ ] Environment variables documented
- [ ] Backup procedures tested

### Monitoring
- [ ] App Platform Insights configured
- [ ] CPU/Memory alerts set
- [ ] Queue depth monitoring
- [ ] Error rate tracking
- [ ] Database performance monitored

### Security
- [ ] Encryption keys rotated
- [ ] User authentication enabled
- [ ] Database firewall configured
- [ ] Runner authentication secured
- [ ] Regular security audits scheduled

### Operations
- [ ] Scaling procedures documented
- [ ] Incident response plan
- [ ] Backup/restore tested
- [ ] Monitoring dashboards created
- [ ] Team trained on operations

## Migration to Queue Mode

See [SCALING.md](SCALING.md) for step-by-step migration guide from Simple to Queue mode.

## Cost Optimization for Production

### Right-Sizing
- Start with basic instances
- Monitor actual usage
- Scale based on metrics
- Use auto-scaling (dedicated CPU)

### Worker Optimization
- Tune concurrency limits
- Optimize workflow logic
- Cache where possible
- Monitor execution times

### Database Optimization
- Regular vacuum/analyze
- Index optimization
- Archive old executions
- Connection pool tuning

See [SCALING.md](SCALING.md) for complete cost optimization strategies.
