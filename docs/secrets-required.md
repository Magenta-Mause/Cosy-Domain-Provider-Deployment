# Required Kubernetes Secrets

Apply these manually to each namespace before ArgoCD can deploy successfully.
Never commit secret values to this repository.

## namespace: cosy-staging / cosy-prod

### `backend-secrets`

```bash
kubectl create secret generic backend-secrets \
  --namespace=cosy-staging \
  --from-literal=AWS_ACCESS_KEY_ID=... \
  --from-literal=AWS_SECRET_ACCESS_KEY=... \
  --from-literal=AWS_HOSTED_ZONE_ID=... \
  --from-literal=AWS_DOMAIN=... \
  --from-literal=MAIL_API_URL=... \
  --from-literal=MAIL_API_KEY=... \
  --from-literal=STRIPE_API_KEY=... \
  --from-literal=STRIPE_WEBHOOK_SECRET=... \
  --from-literal=STRIPE_PRODUCT_ID=... \
  --from-literal=STRIPE_PRICE_ID=... \
  --from-literal=OAUTH_GOOGLE_CLIENT_ID=... \
  --from-literal=OAUTH_GOOGLE_CLIENT_SECRET=... \
  --from-literal=OAUTH_GOOGLE_CALLBACK_URI=https://staging.cosy-hosting.net/api/v1/auth/oauth/google/callback \
  --from-literal=OAUTH_GITHUB_CLIENT_ID=... \
  --from-literal=OAUTH_GITHUB_CLIENT_SECRET=... \
  --from-literal=OAUTH_GITHUB_CALLBACK_URI=https://staging.cosy-hosting.net/api/v1/auth/oauth/github/callback \
  --from-literal=OAUTH_DISCORD_CLIENT_ID=... \
  --from-literal=OAUTH_DISCORD_CLIENT_SECRET=... \
  --from-literal=OAUTH_DISCORD_CALLBACK_URI=https://staging.cosy-hosting.net/api/v1/auth/oauth/discord/callback \
  --from-literal=COSY_DOMAIN_PROVIDER_JWT_SECRET_KEY=... \
  --from-literal=ADMIN_SECRET_KEY=... \
  --from-literal=TURNSTILE_SECRET_KEY=... \
  --from-literal=STAGING_AUTH_USERNAME=... \
  --from-literal=STAGING_AUTH_PASSWORD=... \
  --from-literal=FRONTEND_URL=https://staging.cosy-hosting.net \
  --from-literal=SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/cosy \
  --from-literal=SPRING_DATASOURCE_USERNAME=cosy \
  --from-literal=SPRING_DATASOURCE_PASSWORD=...
```

For prod: set `FRONTEND_URL=https://cosy-hosting.net`.

Auth cookies are `Secure` by default. `AUTH_COOKIE_SECURE=false` (or the legacy
`OAUTH_SECURE_COOKIE=false`) can override this, but should never be set in staging or prod —
both are served over HTTPS.

### `postgres-secrets`

```bash
kubectl create secret generic postgres-secrets \
  --namespace=cosy-staging \
  --from-literal=POSTGRES_PASSWORD=...
```

The `SPRING_DATASOURCE_PASSWORD` in `backend-secrets` must match this value.

### `ghcr-pull-secret` (only needed if the ghcr.io packages are private)

```bash
kubectl create secret docker-registry ghcr-pull-secret \
  --namespace=cosy-staging \
  --docker-server=ghcr.io \
  --docker-username=<github-username> \
  --docker-password=<github-pat-with-read:packages>
```

## namespace: monitoring

### `pushgateway-basic-auth` (für den Systemtest-Pushgateway)

Schützt den `pushgateway.jannekeipert.de`-Ingress per Basic-Auth, damit nur der
Systemtest-CronJob Metriken pushen kann. Erwartet ein htpasswd-File unter dem
Key `auth`:

```bash
# htpasswd erzeugen (User + Passwort frei wählbar — müssen mit
# PUSHGATEWAY_USERNAME/PASSWORD im cosy-systemtest-secrets übereinstimmen):
htpasswd -nbB cosy-systemtest '<starkes-passwort>' > auth

kubectl create secret generic pushgateway-basic-auth \
  --namespace=monitoring \
  --from-file=auth
rm auth
```

Das `pushgateway-tls`-Secret legt cert-manager (`letsencrypt-prod`) automatisch an.

## GitHub Actions secrets (set in each app repo)

| Secret | Description |
|--------|-------------|
| `DEPLOY_REPO_TOKEN` | GitHub PAT with `contents: write` on `cosy-domain-provider-deployment` |
| `VITE_TURNSTILE_SITE_KEY_PROD` | Cloudflare Turnstile site key for production (frontend repo only) |
