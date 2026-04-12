# Week 30, Day 2: Documentation and Runbook

By the end of today, anyone who finds your repo can install, run, deploy, and operate the Capstone without having to ask you questions.

**Estimated time:** 3 hours.

---

## The Four Documents

1. **README.md** -- the front door. What the project is, who it is for, quickstart.
2. **ARCHITECTURE.md** -- how the system is built.
3. **DEPLOY.md** -- how to deploy to production.
4. **RUNBOOK.md** -- how to operate it.

Plus a handful of inline docs (contracts, API spec) already in place.

---

## README.md

The front door. A stranger should understand the project in 60 seconds and be running it in 15 minutes.

Structure:

```markdown
# SME OS -- African SME Platform

[One-paragraph pitch. What it does, who for, what is special.]

## What it does

- Multi-tenant dashboard for small businesses in Kenya.
- USSD, WhatsApp, and M-Pesa integrated out of the box.
- Each tenant gets: a USSD code, a WhatsApp bot, a wallet.

## Screenshot gallery

[5-10 screenshots with captions]

## Quick start

```bash
git clone https://github.com/you/sme-os.git
cd sme-os
cp .env.example .env
docker-compose up -d
# Visit http://localhost:3000
```

## How it works

[One paragraph architecture summary with a link to ARCHITECTURE.md]

## Tech stack

- Next.js 14 (App Router)
- Express + BullMQ
- PostgreSQL with row-level security
- Redis for queues and sessions
- Docker + docker-compose
- CI/CD via GitHub Actions

## Status

Current version: 0.1.0 (Capstone release)

## License

MIT
```

Keep it short. Under 200 lines total. Everyone scrolls.

---

## ARCHITECTURE.md

Already exists from Week 27. Update it now with what actually got built.

Sections:
- System diagram (ASCII art or a linked image).
- Services list (api, notification-service, shop).
- Data model highlights (tenants, RLS).
- Request flow example (USSD purchase end to end).
- Key design decisions (multi-tenancy pattern, payment strategy, notification queueing).
- Known limitations.

Three to five pages. Future maintainers read this first.

---

## DEPLOY.md

Step-by-step deploy instructions. Assume a fresh Ubuntu 22.04 VPS.

```markdown
# Deploying SME OS

## Prerequisites

- Ubuntu 22.04 VPS (minimum 2 GB RAM)
- Docker and docker-compose installed
- Domain name with DNS configured
- GitHub account for image pulls
- API credentials for M-Pesa, WhatsApp, Africa's Talking

## Step 1: Server setup

```bash
ssh root@yourserver
apt update && apt install -y docker.io docker-compose git
usermod -aG docker $USER
```

## Step 2: Clone the repo

```bash
mkdir -p /opt/smeos
cd /opt/smeos
git clone https://github.com/you/sme-os.git .
```

## Step 3: Environment

```bash
cp .env.example .env
nano .env
# Fill in every variable
```

Required variables:
- `DB_PASSWORD`
- `JWT_SECRET` (generate with: `openssl rand -hex 32`)
- `MPESA_CONSUMER_KEY`, `MPESA_CONSUMER_SECRET`, `MPESA_SHORTCODE`, `MPESA_PASSKEY`
- `META_ACCESS_TOKEN`, `META_VERIFY_TOKEN`, `META_APP_SECRET`
- `AT_USERNAME`, `AT_API_KEY`
- `PUBLIC_URL` (your https URL)
- `CRM_SERVER_URL` (internal, probably `http://api:5000`)

## Step 4: Start

```bash
docker-compose -f docker-compose.yml up -d
```

## Step 5: TLS

```bash
docker-compose run --rm certbot certonly \
  --webroot -w /var/www/certbot \
  --email you@example.com --agree-tos --no-eff-email \
  -d yourdomain.co.ke -d www.yourdomain.co.ke
docker-compose exec nginx nginx -s reload
```

## Step 6: Verify

- [ ] `https://yourdomain.co.ke` loads the landing page.
- [ ] `/api/health` returns 200.
- [ ] A test signup creates a tenant.
- [ ] M-Pesa callback URL is reachable.
- [ ] WhatsApp webhook is verified in Meta's dashboard.
- [ ] Africa's Talking USSD callback URL is updated.

## Rollback

```bash
docker-compose -f docker-compose.yml down
git checkout <previous-tag>
docker-compose -f docker-compose.yml up -d
```
```

Twenty-ish commands. Someone who has never seen your project should be able to deploy in an hour.

---

## RUNBOOK.md

The runbook is for when things go wrong at 11pm. Each section is a scenario + the exact commands to fix it.

```markdown
# Runbook

## 1. Site is down

### Symptom
`curl https://yourdomain.co.ke` returns 5xx or times out.

### Diagnose
```bash
ssh root@yourserver
docker-compose ps  # are containers running?
docker-compose logs --tail=100 nginx
docker-compose logs --tail=100 api
```

### Fix
Most common: a container crashed. Restart:
```bash
docker-compose restart api
```

If Postgres is down, check disk space (`df -h`) before restarting.

## 2. Payment callbacks missing

### Symptom
Customers pay but orders stay in `pending`.

### Diagnose
```bash
docker-compose logs api | grep -i callback
docker-compose logs api | grep -i "stk"
```

Check the M-Pesa portal for receipt numbers and compare.

### Fix
1. Run reconciliation manually:
```bash
docker-compose exec api node scripts/reconcile.js $(date -d yesterday +%Y-%m-%d)
```
2. Resolve mismatches from the admin reconciliation page.

## 3. Queue is backed up

### Symptom
Bull Board shows hundreds of waiting jobs.

### Diagnose
```bash
docker-compose logs notification-service --tail=100
docker-compose exec redis redis-cli --no-auth-warning llen bull:whatsapp:wait
```

### Fix
1. Is a channel API rate-limiting you? Check Meta dashboard.
2. Is the worker stuck? Restart it: `docker-compose restart notification-service`.
3. If persistent, scale up: add another worker process.

## 4. Tenant isolation bug

### Symptom
A tenant reports seeing another tenant's data.

### Diagnose
```bash
docker-compose exec postgres psql -U crm_user -d crm
\d tenants
SELECT * FROM pg_policies;  # are RLS policies still in place?
```

### Fix
If policies are missing, this is a severe bug. Halt deploys. Re-apply from `sql/schema.sql`. Audit logs for any queries that might have leaked.

## 5. Out of disk space

### Symptom
Deploys fail, Postgres refuses writes.

### Diagnose
```bash
df -h
docker system df
```

### Fix
```bash
docker image prune -a -f   # remove unused images
docker volume ls           # check for orphans
journalctl --vacuum-size=100M   # shrink systemd journals
```

## Contacts

- Platform owner: you@example.com
- On-call: +254 xxx
```

Five scenarios. Pick the ones most likely to happen. Add more as you see them.

---

## Internal API Docs

Point to `docs/API.md` from Week 27 Day 4. Update it if the spec drifted from the implementation.

Generate a Postman collection if you want, or use the manual doc. Either is fine.

---

## Checkpoint

1. README is a complete front door.
2. DEPLOY.md walks through every step of a fresh deploy.
3. RUNBOOK.md covers the top 5 incident scenarios.
4. Someone not on the team could read the docs and deploy the project.

Commit:

```bash
git add .
git commit -m "docs: readme deploy and runbook for capstone"
```

---

## What You Learned

- Documentation is a demo artifact.
- Runbooks are written before incidents, not during.
- Deploy docs should assume zero institutional knowledge.

Tomorrow: the final deploy.
