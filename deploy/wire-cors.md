# Wire CORS — Vercel ↔ VPS

Do this after both sides are reachable.

## Values

| Side | Variable | Example |
|------|----------|---------|
| Vercel | `NEXT_PUBLIC_API_URL` | `https://api.example.com/api` |
| Vercel | `NEXT_PUBLIC_APP_URL` | `https://quittance-xxx.vercel.app` |
| VPS `.env` | `FRONTEND_URL` | `https://quittance-xxx.vercel.app` |

Rules:

- No trailing slash on `FRONTEND_URL` or `NEXT_PUBLIC_APP_URL`
- `NEXT_PUBLIC_API_URL` **must** end with `/api`
- Origins must match exactly (scheme + host + port)

## Checklist

- [ ] VPS: `curl -sS https://api.YOURDOMAIN.com/api/health` → JSON/OK
- [ ] VPS: `FRONTEND_URL` = live Vercel origin
- [ ] VPS: `sudo systemctl restart quittance-api`
- [ ] Vercel: `NEXT_PUBLIC_API_URL` points at VPS `/api`
- [ ] Vercel: redeploy after any `NEXT_PUBLIC_*` change
- [ ] Browser on Vercel origin: create invoice / dashboard XHR not blocked by CORS
- [ ] Payment URLs in API responses use the Vercel host (`FRONTEND_URL`)

## Quick CORS debug

From any machine:

```bash
curl -si -X OPTIONS 'https://api.YOURDOMAIN.com/api/health' \
  -H 'Origin: https://YOUR-APP.vercel.app' \
  -H 'Access-Control-Request-Method: GET' | head -n 30
```

Expect `Access-Control-Allow-Origin` equal to your Vercel origin.
