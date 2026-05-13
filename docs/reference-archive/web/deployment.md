# Deployment

Clawboard Dashboard is a static site (Vite build output in `dist/`). It can be deployed anywhere that serves static files.

## Docker (recommended)

### Build and run

```bash
docker build \
  --build-arg VITE_API_URL=https://api.example.com/api/v1 \
  -t clawboard-web:latest .

docker run -p 80:80 clawboard-web:latest
```

### Pre-built images (GHCR)

```bash
docker pull ghcr.io/moltar-forge/clawboard-web:latest
```

Available tags: `latest`, `main`, `sha-<short>`, `v1.2.3`.

## Full stack via Docker Compose

The easiest way to run everything together:

```bash
cd clawboard-api
make up
# Dashboard available at http://localhost:5173
```

## Static hosting (S3, Netlify, Vercel, etc.)

```bash
npm run build
# Upload dist/ to your static host
```

For S3 + CloudFront, see the template workflow in `.github/workflows/build-deploy.yml`.

## nginx configuration

The included `nginx.conf` handles:

- SPA routing (all paths fall back to `index.html`)
- Long-lived cache headers for hashed assets
- Security headers (`X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy`)

## Production checklist

- [ ] `VITE_API_URL` points to the production API URL
- [ ] API is running with `CORS_ORIGIN` set to the dashboard's exact origin
- [ ] Served over HTTPS
- [ ] No secrets in `VITE_*` variables
