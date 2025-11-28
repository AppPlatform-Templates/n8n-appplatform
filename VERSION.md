# n8n Version Information

## Current Version

This repository is based on **n8n v1.122.2** (latest stable release as of deployment).

## Version Source

The version was determined by checking the GitHub releases API:
```bash
gh repo view n8n-io/n8n --json latestRelease --jq '.latestRelease.tagName'
```

## Update Policy

### How to Check for Updates

1. Visit the [n8n releases page](https://github.com/n8n-io/n8n/releases)
2. Or use the GitHub CLI:
   ```bash
   gh repo view n8n-io/n8n --json latestRelease
   ```

### How to Update

To update this deployment to a newer version of n8n:

1. **Check the latest release** on GitHub
2. **Review the changelog** for breaking changes
3. **Update the Dockerfile** if needed (base image, node version, etc.)
4. **Test locally** using:
   ```bash
   docker build -t n8n-test -f docker/images/n8n/Dockerfile .
   docker run -p 5678:5678 n8n-test
   ```
5. **Update this VERSION.md** file with the new version number
6. **Redeploy** to App Platform

### Breaking Changes to Watch For

When updating n8n, pay attention to:
- Database migration requirements
- Environment variable changes
- Node.js version requirements
- Dependency updates

## Deployment Information

- **Deployed on**: DigitalOcean App Platform
- **Region**: Amsterdam (ams3)
- **Database**: PostgreSQL 17
- **Instance Size**: apps-s-1vcpu-1gb (1 vCPU, 1 GB RAM)

## Upstream Repository

- **Source**: https://github.com/n8n-io/n8n
- **Documentation**: https://docs.n8n.io
- **License**: Sustainable Use License (Fair-code)
