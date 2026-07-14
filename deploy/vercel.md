# Frontend deploy handoff (Vercel)

You deploy this part in the Vercel dashboard or CLI. API stays on your VPS — see [`deploy/vps/README.md`](./vps/README.md).

## Prerequisites

- Frontend production build already verified locally (`cd frontend && npm run build`).
- Public API base known, e.g. `https://api.yourdomain.com` (must expose `GET /api/health`).

## Option A — Dashboard

1. Import https://github.com/CemAyyildiz/Quittance  
2. **Root Directory:** `frontend`  
3. Framework: Next.js (`frontend/vercel.json`)  
4. Production env vars:

| Variable | Value |
|----------|--------|
| `NEXT_PUBLIC_API_URL` | `https://api.YOURDOMAIN.com/api` |
| `NEXT_PUBLIC_STELLAR_NETWORK` | `TESTNET` |
| `NEXT_PUBLIC_HORIZON_URL` | `https://horizon-testnet.stellar.org` |
| `NEXT_PUBLIC_APP_URL` | `https://YOUR-PROJECT.vercel.app` (update after first deploy if needed) |
| `NEXT_PUBLIC_APP_NAME` | `Quittance` |
| `NEXT_PUBLIC_USE_MOCK` | `false` |

5. Deploy → copy the production URL.

## Option B — CLI

```bash
cd frontend
vercel login
vercel link   # root = this frontend folder / monorepo rootDir frontend
vercel env add NEXT_PUBLIC_API_URL production
vercel env add NEXT_PUBLIC_STELLAR_NETWORK production   # TESTNET
vercel env add NEXT_PUBLIC_HORIZON_URL production
vercel env add NEXT_PUBLIC_APP_URL production
vercel env add NEXT_PUBLIC_USE_MOCK production          # false
vercel --prod
```

## After deploy — CORS wire-up

1. On VPS `.env`, set `FRONTEND_URL` to the **exact** Vercel origin (no trailing slash).  
2. `sudo systemctl restart quittance-api`  
3. From a browser on the Vercel site, confirm API calls succeed (Network tab).  
4. Smoke: `curl -sS https://api.YOURDOMAIN.com/api/health`  
5. Paste both URLs into [`EVIDENCE.md`](../EVIDENCE.md) and GrantFox official links.

Checklist (copy/paste): [`wire-cors.md`](./wire-cors.md).
