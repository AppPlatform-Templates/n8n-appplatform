# Deploy n8n to DigitalOcean App Platform

This guide explains how to deploy n8n to DigitalOcean App Platform using the Deploy-to-DO button.

## 🚀 One-Click Deployment

[![Deploy to DO](https://www.deploytodo.com/do-btn-blue.svg)](https://cloud.digitalocean.com/apps/new?repo=https://github.com/AppPlatform-Templates/n8n-appplatform/tree/main)

## Prerequisites

Before deploying, you'll need:

1. **DigitalOcean Account**: [Sign up here](https://cloud.digitalocean.com/registrations/new)
2. **GitHub Account**: For repository access
3. **PostgreSQL Database** (included in deployment)
4. **Credit Card** on file (for billing)

## Estimated Monthly Cost

| Component | Size | Cost/Month |
|-----------|------|------------|
| n8n Service | apps-s-1vcpu-1gb | $12.00 |
| PostgreSQL Database | db-s-1vcpu-1gb | $15.00 |
| **Total** | | **$27.00** |

> Prices are estimates. Actual costs may vary based on usage and region.

## Deployment Steps

### 1. Click the Deploy Button

Click the blue "Deploy to DO" button above. This will:
- Fork/reference the repository
- Pre-fill the App Platform configuration
- Set up the build and deployment pipeline

### 2. Authorize GitHub

When prompted:
- Click "Authorize DigitalOcean"
- Grant access to the repository
- You may need to install the DigitalOcean GitHub App

### 3. Configure Your App

**App Name**: Choose a unique name (e.g., `my-n8n-automation`)

**Region**: Select closest to your users:
- Amsterdam (ams3) - Europe
- New York (nyc3) - US East
- San Francisco (sfo3) - US West
- Singapore (sgp1) - Asia
- London (lon1) - UK
- Frankfurt (fra1) - Germany

### 4. Configure Environment Variables

**CRITICAL**: You MUST change these values:

#### Required Changes

**N8N_ENCRYPTION_KEY** (⚠️ MUST CHANGE):
```bash
# Generate a secure key:
openssl rand -hex 32
```
Copy the output and paste it as the `N8N_ENCRYPTION_KEY` value.

#### Database Configuration

The Deploy-to-DO button will automatically create a PostgreSQL database and set these variables:
- ✅ `DB_TYPE` = `postgresdb`
- ✅ `DB_POSTGRESDB_HOST` = Auto-configured
- ✅ `DB_POSTGRESDB_PORT` = Auto-configured
- ✅ `DB_POSTGRESDB_DATABASE` = Auto-configured
- ✅ `DB_POSTGRESDB_USER` = Auto-configured
- ✅ `DB_POSTGRESDB_PASSWORD` = Auto-configured

#### Application URLs

The Deploy-to-DO process will automatically set:
- ✅ `N8N_HOST` = Your app's URL
- ✅ `WEBHOOK_URL` = Webhook base URL

### 5. Optional: Configure File Storage

If you need persistent file storage (for uploads, binary data):

1. **Create a Spaces Bucket**:
   - Go to Spaces in DigitalOcean control panel
   - Create bucket (e.g., `n8n-storage-ams3`)
   - Note the region

2. **Get Spaces Credentials**:
   - Go to API → Spaces Keys
   - Generate new key
   - Note Access Key and Secret

3. **Add Environment Variables**:
   ```
   N8N_DEFAULT_BINARY_DATA_MODE=filesystem
   AWS_ACCESS_KEY_ID=<your-spaces-key>
   AWS_SECRET_ACCESS_KEY=<your-spaces-secret>
   AWS_S3_BUCKET=n8n-storage-ams3
   AWS_S3_ENDPOINT=https://ams3.digitaloceanspaces.com
   ```

### 6. Deploy!

Click **"Deploy"** button and wait:
- ⏱️ Build time: 5-15 minutes
- ⏱️ Database creation: 10-15 minutes
- ⏱️ Total: ~20-30 minutes

You can monitor progress in the "Activity" tab.

## Post-Deployment

### 1. Access Your n8n Instance

Once deployed, visit:
```
https://your-app-name.ondigitalocean.app
```

### 2. Create Owner Account

On first visit:
1. Enter your email
2. Choose a strong password
3. Set up your profile
4. Click "Get Started"

### 3. Verify Database Connection

Create a test workflow:
1. Add "Postgres" node
2. Select "Execute Query"
3. Run: `SELECT version();`
4. Execute to verify connection

### 4. Enable Database Firewall (Recommended)

For production, enable trusted sources:

1. Go to your database in control panel
2. Click "Settings"
3. Under "Trusted Sources":
   - Remove "All IPv4" (0.0.0.0/0)
   - Add your app's name from dropdown
4. Save changes

This restricts database access to only your app.

## Updating Your Deployment

### Via Control Panel

1. Go to Apps → Your n8n app
2. Click on "Settings"
3. Update environment variables
4. Click "Save" (triggers redeployment)

### Via Git Push

If you forked the repository:
1. Make changes locally
2. Commit and push to GitHub
3. App Platform auto-deploys

### Updating n8n Version

See [VERSION.md](VERSION.md) for version update instructions.

## Troubleshooting

### Build Fails

**Symptom**: Build fails with error message

**Solutions**:
- Check build logs in "Runtime Logs"
- Verify Dockerfile path is correct
- Ensure Node.js version is supported
- Check for dependency issues

### Can't Access After Deployment

**Symptom**: URL shows 404 or connection refused

**Solutions**:
- Wait for deployment to complete (check status)
- Verify service is running in "Components" tab
- Check health check configuration
- Review application logs

### Database Connection Failed

**Symptom**: n8n shows database error on startup

**Solutions**:
- Verify database is online (check Databases tab)
- Ensure environment variables are correct
- Check database firewall (temporarily allow all for testing)
- Verify region matching (database and app in same region works best)

### Webhooks Not Working

**Symptom**: Webhooks return 404 or don't trigger

**Solutions**:
- Verify `WEBHOOK_URL` environment variable
- Ensure webhook workflow is active
- Check webhook URL format: `https://your-app.ondigitalocean.app/webhook/your-path`
- Test with curl

### Application Crashed

**Symptom**: Service shows "crashed" status

**Solutions**:
- Check runtime logs for error messages
- Verify `N8N_ENCRYPTION_KEY` is set
- Ensure database connection is working
- Check resource limits (may need larger instance)

## Scaling Your Deployment

### Vertical Scaling (More Resources)

If experiencing performance issues:

1. Go to your app → n8n service
2. Click "Edit Plan"
3. Select larger instance:
   - `apps-s-1vcpu-2gb` ($18/mo) - More memory
   - `apps-s-2vcpu-4gb` ($50/mo) - More CPU & memory

### Database Scaling

1. Go to Databases → Your database
2. Click "Resize"
3. Select larger configuration
4. Confirm resize

### Horizontal Scaling

⚠️ **Important**: n8n requires additional configuration for multiple instances:
- Use queue mode
- Configure Redis for coordination
- See [n8n scaling docs](https://docs.n8n.io/hosting/scaling/)

## Cost Optimization

### Development/Testing

For non-production use:
- Use **Dev Database** ($7/mo instead of $15/mo)
  - Note: PostgreSQL only, single database
- Use smallest instance size
- Disable when not in use

### Production

For production workloads:
- Use managed database with HA
- Enable autoscaling (dedicated CPU instances only)
- Set up monitoring and alerts
- Regular backups (included with managed DB)

## Security Best Practices

### 1. Encryption Key
- ✅ Use strong, random encryption key
- ✅ Never reuse between environments
- ✅ Store securely (DO encrypts environment variables)

### 2. Database
- ✅ Enable trusted sources after testing
- ✅ Use strong password (auto-generated)
- ✅ Enable VPC for additional isolation
- ✅ Regular backups (automatic with managed DB)

### 3. Application
- ✅ Use HTTPS only (automatic with App Platform)
- ✅ Enable user authentication
- ✅ Review workflow permissions
- ✅ Regular security updates

### 4. Monitoring
- ✅ Enable app metrics
- ✅ Set up alerts for crashes
- ✅ Monitor execution logs
- ✅ Track database performance

## Additional Resources

- **n8n Documentation**: [docs.n8n.io](https://docs.n8n.io)
- **App Platform Docs**: [docs.digitalocean.com/products/app-platform](https://docs.digitalocean.com/products/app-platform/)
- **Getting Started Guide**: [HOW_TO_USE_N8N.md](HOW_TO_USE_N8N.md)
- **Environment Variables**: [ENV_TEMPLATE.md](ENV_TEMPLATE.md)
- **Version Information**: [VERSION.md](VERSION.md)

## Support

- **n8n Issues**: [github.com/n8n-io/n8n/issues](https://github.com/n8n-io/n8n/issues)
- **n8n Community**: [community.n8n.io](https://community.n8n.io)
- **DigitalOcean Support**: Via control panel
- **This Template**: [github.com/AppPlatform-Templates/n8n-appplatform](https://github.com/AppPlatform-Templates/n8n-appplatform)

---

**Ready to automate?** Click the Deploy button above and start building workflows! 🚀
