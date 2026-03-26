# n8n-personal

Self-hosted n8n instance running on Fly.io. Used for personal automation workflows: email processing, webhook integrations, API orchestration, and more.

## Stack

- **n8n** — workflow automation platform
- **Fly.io** — hosting with persistent volume for workflow/credential storage
- **GitHub Actions** — CI/CD on push to main

## One-Time Setup

### 1. Install Fly CLI and authenticate

```bash
brew install flyctl
fly auth login
```

### 2. Create the Fly app

```bash
fly apps create personal-n8n
```

### 3. Create persistent volume

```bash
fly volumes create n8n_data --size 1 --region sjc -a personal-n8n
```

### 4. Set secrets

```bash
fly secrets set \
  N8N_BASIC_AUTH_USER=admin \
  N8N_BASIC_AUTH_PASSWORD=your_secure_password \
  N8N_ENCRYPTION_KEY=$(openssl rand -hex 32) \
  -a personal-n8n
```

### 5. Add GitHub secret

Generate a Fly.io deploy token:

```bash
fly tokens create deploy -x 999999h -a personal-n8n
```

Add it to your GitHub repo:
- Settings → Secrets and variables → Actions → New repository secret
- Name: `FLY_API_TOKEN`
- Value: paste the token

### 6. Deploy

```bash
fly deploy -a n8n-{unique-ref}
```

Or push to main — GitHub Actions will handle it automatically.

## Accessing n8n

Once deployed, visit:

```
https://n8n-{unique-ref}.fly.dev
```

Log in with the `N8N_BASIC_AUTH_USER` and `N8N_BASIC_AUTH_PASSWORD` you set above.

## Custom Domain (Optional)

To use a custom subdomain like `n8n.yourdomain.com`:

```bash
fly certs add n8n.yourdomain.com -a n8n-{unique-ref}
```

Then add a DNS A record pointing to your Fly.io IPv4:

```bash
fly ips list -a personal-n8n
```

Update `N8N_HOST` and `WEBHOOK_URL` in `fly.toml` to match your custom domain, then redeploy.

## Adding New Workflows

All workflows are configured directly in the n8n UI and stored in the persistent volume. No code changes needed for new workflows.

For workflows that call external services, add credentials via n8n's built-in credential manager — they are encrypted using your `N8N_ENCRYPTION_KEY`.

## Workflows

Workflows are managed directly in the n8n UI.
See the `workflows/` directory for exported workflow JSON files.

## Useful Commands

```bash
# View logs
fly logs -a personal-n8n

# SSH into the instance
fly ssh console -a personal-n8n

# Check app status
fly status -a personal-n8n

# List volumes
fly volumes list -a personal-n8n
```
