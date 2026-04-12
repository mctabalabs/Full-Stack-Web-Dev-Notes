# Week 30, Day 3: Final Deploy

By the end of today, the SME OS is live on a real domain, serving HTTPS, with monitoring, alerting, and backups. It is production-ready enough to show real users.

**Estimated time:** 4-5 hours.

---

## The Deploy Plan

In order. Do not skip steps.

1. Provision a fresh VPS.
2. Point DNS at it.
3. Install Docker.
4. Pull the latest images.
5. Set environment variables.
6. Start the stack.
7. Get TLS certs.
8. Register webhooks with providers.
9. Run a test purchase.
10. Set up monitoring and alerts.
11. Set up backups.
12. Celebrate for 30 seconds, then test again.

---

## Provision The Server

Any Linux VPS with 2 GB RAM and Docker capability. For the Capstone, Hetzner Cloud ($4/month) or DigitalOcean ($6/month).

```bash
ssh root@<new-server-ip>
apt update
apt upgrade -y
apt install -y docker.io docker-compose fail2ban ufw
ufw allow 22
ufw allow 80
ufw allow 443
ufw enable
useradd -m -s /bin/bash deploy
usermod -aG docker deploy
mkdir -p /home/deploy/.ssh
# copy your public key into authorized_keys
```

---

## DNS

Update your domain's DNS to point at the new server's IP. Use an `A` record for the apex and a `CNAME` for `www`.

Wait for propagation. Check with `dig yourdomain.co.ke`. It should resolve to the server's IP within a few minutes.

---

## Deploy The Stack

```bash
su - deploy
mkdir -p /opt/smeos
cd /opt/smeos
git clone https://github.com/you/sme-os.git .
cp .env.example .env
nano .env
```

Fill in every variable. Double-check `JWT_SECRET` is a strong random string (`openssl rand -hex 32`), not a placeholder.

```bash
echo "$GHCR_TOKEN" | docker login ghcr.io -u yourusername --password-stdin
docker-compose -f docker-compose.yml up -d
```

Wait for all containers to start. `docker-compose ps` should show all healthy.

---

## TLS

```bash
docker-compose run --rm certbot certonly \
  --webroot -w /var/www/certbot \
  --email you@example.com \
  --agree-tos --no-eff-email \
  -d smeos.co.ke -d www.smeos.co.ke

docker-compose exec nginx nginx -s reload
```

Curl your domain. Green padlock. `curl -I https://smeos.co.ke` returns 200.

---

## Register Webhooks

Update every provider's dashboard to point at the new domain:

- **Daraja**: STK Push callback URL = `https://smeos.co.ke/webhook/mpesa/callback`.
- **Meta**: WhatsApp webhook = `https://smeos.co.ke/webhook/whatsapp`, verify token from your `.env`.
- **Africa's Talking**: USSD callback URL = `https://smeos.co.ke/webhook/ussd`.
- **Stripe**: webhook endpoint = `https://smeos.co.ke/api/webhooks/stripe`.

Test each one:

- Daraja: use Safaricom's "test" button.
- Meta: Meta auto-verifies.
- AT: dial the code from their simulator.
- Stripe: use the CLI `stripe listen` or trigger a test event.

---

## End-To-End Test On Production

1. Sign up a real tenant.
2. Create a real product.
3. Dial the USSD code from a real phone.
4. Complete the STK prompt.
5. Receive the WhatsApp confirmation on a real phone.
6. See the order in the dashboard.

If anything fails, do not announce the launch until it is fixed.

---

## Monitoring

Minimal monitoring that catches the obvious issues:

### Uptime

Use https://betteruptime.com or https://uptimerobot.com (both have free tiers). Add:

- A GET check on `/api/health` every minute.
- A GET check on the shop home page every minute.
- A TCP check on port 443 every minute.

Alerts go to email and a phone number.

### Error tracking

Sign up for https://sentry.io (free tier). Install `@sentry/node`:

```bash
npm install @sentry/node
```

Add at the top of each service entry:

```javascript
const Sentry = require("@sentry/node");
Sentry.init({ dsn: process.env.SENTRY_DSN, environment: "production" });
```

Every uncaught error lands in Sentry with a stack trace. This is the difference between "we had an outage" and "we had an outage and I know exactly what line broke".

### Dashboards

Bull Board is already at `/admin/queues` behind admin auth. `/admin/cron` shows cron health. `/admin/reconciliation` shows money reconciliation. That is enough.

---

## Backups

Postgres backups are non-negotiable.

Add a cron job on the host:

```bash
0 3 * * * /opt/smeos/scripts/backup.sh
```

Where `backup.sh` is:

```bash
#!/bin/bash
cd /opt/smeos
docker-compose exec -T postgres pg_dump -U crm_user crm | gzip > /backups/crm-$(date +%Y%m%d).sql.gz
find /backups -name "crm-*.sql.gz" -mtime +7 -delete

# Upload to remote storage (S3, Backblaze, etc)
aws s3 cp /backups/crm-$(date +%Y%m%d).sql.gz s3://yourbucket/backups/
```

Daily backups, keep 7 days locally, ship to remote storage. Test restore monthly.

### Restore drill

```bash
gunzip < /backups/crm-20260501.sql.gz | docker-compose exec -T postgres psql -U crm_user crm_restore
```

Restore into a separate database to verify the backup is valid. Do this at least once.

---

## Launch Checklist

Before telling anyone:

- [ ] Site loads on HTTPS.
- [ ] All webhooks registered.
- [ ] Test purchase succeeds.
- [ ] Uptime monitor is green.
- [ ] Sentry is receiving events.
- [ ] Backups ran overnight.
- [ ] Restore drill worked.
- [ ] RUNBOOK.md is accessible.
- [ ] Contact email for incidents is published.

---

## Checkpoint

1. SME OS is live at `https://smeos.co.ke` (or whatever your domain is).
2. A real test purchase goes through real M-Pesa sandbox.
3. Monitoring is configured.
4. Backups are running.
5. Nothing is in the error inbox.

Commit:

```bash
git add .
git commit -m "chore: final deploy configuration and backup scripts"
git tag v1.0.0
git push --follow-tags
```

---

## What You Learned

- A production deploy is a checklist, not an adventure.
- Monitoring, alerting, backups are as important as the code.
- Tags mark the launch point; rollback is a tag checkout.

Tomorrow: demo day rehearsal.
