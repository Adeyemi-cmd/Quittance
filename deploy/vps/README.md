# Quittance MVP API — VPS runbook

Self-host `server-mvp.ts` (in-memory). Do **not** use `backend/vercel.json` / Postgres `server.ts` for the public demo.

## Requirements

- Ubuntu/Debian (or similar) VPS
- Node.js **18+**
- nginx + certbot (recommended for HTTPS)
- DNS: `api.yourdomain.com` → VPS public IP

## 1. Install Node (example)

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs
node -v   # >= 18
```

## 2. App user and clone

```bash
sudo useradd -r -m -d /opt/quittance -s /bin/bash quittance || true
sudo mkdir -p /opt/quittance
sudo chown quittance:quittance /opt/quittance

sudo -u quittance -H bash -lc '
  cd /opt/quittance
  git clone https://github.com/CemAyyildiz/Quittance.git .
  cd backend
  npm install
  cp env.mvp.example .env
'
```

## 3. Configure `.env`

Edit `/opt/quittance/backend/.env`:

| Variable | Value |
|----------|--------|
| `NODE_ENV` | `production` |
| `PORT` | `3001` |
| `STELLAR_NETWORK` | `TESTNET` |
| `STELLAR_HORIZON_URL` | `https://horizon-testnet.stellar.org` |
| `FRONTEND_URL` | exact Vercel origin, no trailing slash (e.g. `https://quittance.vercel.app`) |
| `ALLOW_SIMULATE` | `false` |

`FRONTEND_URL` must match the browser origin of the Next.js app (CORS + payment links).

## 4. Smoke test

```bash
sudo -u quittance -H bash -lc 'cd /opt/quittance/backend && npm run start:mvp'
# elsewhere:
curl -sS http://127.0.0.1:3001/api/health
# Ctrl+C when OK, then enable systemd
```

## 5. systemd

```bash
sudo cp /opt/quittance/deploy/vps/quittance-api.service /etc/systemd/system/
# If the repo landed only under backend, copy from this folder after scp/git pull
sudo systemctl daemon-reload
sudo systemctl enable --now quittance-api
sudo systemctl status quittance-api --no-pager
```

Adjust `WorkingDirectory` / `User` in the unit if your paths differ. Restart after every `.env` change:

```bash
sudo systemctl restart quittance-api
```

**Note:** In-memory storage — process restart clears all invoices. Expected for the demo.

## 6. nginx + TLS

```bash
sudo apt-get install -y nginx certbot python3-certbot-nginx
sudo cp /opt/quittance/deploy/vps/nginx-quittance-api.conf /etc/nginx/sites-available/quittance-api
sudo sed -i 's/api.example.com/api.YOURDOMAIN.com/g' /etc/nginx/sites-available/quittance-api
sudo ln -sf /etc/nginx/sites-available/quittance-api /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
sudo certbot --nginx -d api.YOURDOMAIN.com
```

Public health check: `https://api.YOURDOMAIN.com/api/health`

## 7. Wire frontend (Vercel)

Owner deploys Vercel — see [`../vercel.md`](../vercel.md). Then:

1. Set `NEXT_PUBLIC_API_URL=https://api.YOURDOMAIN.com/api`
2. Set `NEXT_PUBLIC_APP_URL` to the Vercel URL
3. Redeploy frontend
4. Ensure VPS `FRONTEND_URL` equals that Vercel origin (no trailing slash)
5. `sudo systemctl restart quittance-api`
6. Follow [`../wire-cors.md`](../wire-cors.md) until browser API calls succeed

## Files in this folder

- `quittance-api.service` — systemd unit
- `nginx-quittance-api.conf` — reverse proxy template
