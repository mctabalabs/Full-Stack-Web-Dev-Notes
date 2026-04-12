# Week 26, Day 2: Docker Publishing and Deploys

By the end of today, every push to main builds Docker images and pushes them to a container registry. A small deploy script pulls the latest images on your server and restarts compose. You have a real deploy pipeline.

**Prior concepts:** CI (Week 26 Day 1), Docker (Week 25).

**Estimated time:** 3 hours

---

## Where Do Images Live

Docker images need a home. Options:

- **Docker Hub** -- the default. Free for public images, paid for private.
- **GitHub Container Registry (ghcr.io)** -- free, integrated with GitHub, good for projects hosted there.
- **AWS ECR / GCP Artifact Registry** -- cloud-provider specific, paid.

We use ghcr.io for the Marathon. Free, zero setup, same auth as GitHub.

Your images become URLs like:

```
ghcr.io/yourusername/mctaba-api:latest
ghcr.io/yourusername/mctaba-api:v1.2.3
ghcr.io/yourusername/mctaba-api:sha-a1b2c3d
```

---

## The Publish Workflow

Add `.github/workflows/publish.yml`:

```yaml
name: Publish

on:
  push:
    branches: [main]
    tags: ["v*"]

jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    strategy:
      matrix:
        service:
          - { name: api, context: ./api }
          - { name: notifications, context: ./notification-service }
          - { name: shop, context: ./shop }

    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ghcr.io/${{ github.repository_owner }}/mctaba-${{ matrix.service.name }}
          tags: |
            type=ref,event=branch
            type=sha,prefix=sha-
            type=semver,pattern={{version}}
            type=raw,value=latest,enable={{is_default_branch}}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: ${{ matrix.service.context }}
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

Walk through:

- **Trigger**: pushes to main and tags starting with `v`.
- **Matrix**: three services build in parallel.
- **Login**: use the built-in `GITHUB_TOKEN` -- no secret to manage.
- **Metadata action**: generates tags based on branch, sha, semver, and "latest".
- **Build and push**: the official action, with GitHub Actions cache enabled.

After a push, you get three images on ghcr.io, each with multiple tags.

---

## Versioning Deploys

Tags give you versioned releases:

```bash
git tag v1.0.0
git push --tags
```

The workflow picks up the tag and publishes `mctaba-api:v1.0.0`, `mctaba-api:1.0`, etc. Now your production server can pin to a specific version:

```yaml
# docker-compose.prod.yml
services:
  api:
    image: ghcr.io/you/mctaba-api:v1.0.0
```

Upgrading: change the tag, redeploy. Rollback: change the tag back. This is the whole point of versioned images.

---

## The Deploy Workflow

Add `.github/workflows/deploy.yml`:

```yaml
name: Deploy

on:
  workflow_run:
    workflows: ["Publish"]
    types: [completed]
    branches: [main]
  workflow_dispatch:  # manual trigger button

jobs:
  deploy:
    if: ${{ github.event.workflow_run.conclusion == 'success' || github.event_name == 'workflow_dispatch' }}
    runs-on: ubuntu-latest

    steps:
      - name: Deploy to server
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.DEPLOY_HOST }}
          username: ${{ secrets.DEPLOY_USER }}
          key: ${{ secrets.DEPLOY_SSH_KEY }}
          script: |
            cd /opt/mctaba
            docker compose pull
            docker compose up -d --remove-orphans
            docker image prune -f
```

The workflow SSHes into your production server, pulls the latest images, and restarts compose. The `appleboy/ssh-action` is a popular wrapper around SSH.

Secrets to set:

- `DEPLOY_HOST` -- your server's IP or hostname.
- `DEPLOY_USER` -- the SSH user (usually something like `deploy` with limited privileges, not `root`).
- `DEPLOY_SSH_KEY` -- a private SSH key that can log into the server.

---

## Creating a Deploy Key

On your server:

```bash
sudo useradd -m -s /bin/bash deploy
sudo usermod -aG docker deploy
sudo -u deploy ssh-keygen -t ed25519 -f /home/deploy/.ssh/id_deploy -N ""
cat /home/deploy/.ssh/id_deploy.pub >> /home/deploy/.ssh/authorized_keys
cat /home/deploy/.ssh/id_deploy  # copy this, paste as DEPLOY_SSH_KEY
```

Now `deploy@yourserver` can SSH in using the private key, and has permissions to run Docker. Limit what this user can do -- they should only run compose commands, nothing else. Consider `visudo` restrictions if you want to harden further.

---

## Docker Compose File On The Server

On the server, at `/opt/mctaba/docker-compose.yml`:

```yaml
version: "3.8"

services:
  api:
    image: ghcr.io/you/mctaba-api:latest
    restart: unless-stopped
    # ... rest of your production config

  notifications:
    image: ghcr.io/you/mctaba-notifications:latest
    restart: unless-stopped
    # ...

  shop:
    image: ghcr.io/you/mctaba-shop:latest
    restart: unless-stopped
    # ...
```

Note there is no `build:` -- the server never builds, only pulls. Builds happen in CI.

---

## Pulling Private Images

ghcr.io images are private by default. Your server must log in once to pull them:

```bash
echo "$GHCR_TOKEN" | docker login ghcr.io -u yourusername --password-stdin
```

Create a Personal Access Token with `read:packages` scope on GitHub, use it as the password. This is the only step that needs a human the first time.

After login, `docker compose pull` just works. The credentials live in `~/.docker/config.json` on the server.

---

## Rolling Back

Something broke after a deploy. Rollback:

1. Note the previous working tag (sha or version).
2. SSH to the server.
3. Edit `docker-compose.yml` to use the old tag.
4. `docker compose up -d`.

Or do it via a GitHub Actions workflow_dispatch that accepts a tag as input. For a small team, the manual process is simpler.

---

## Deploy Preview On PRs (Optional)

For each PR, deploy to a temporary environment. This is expensive and usually not worth it for small teams. Vercel and Netlify do this automatically for Next.js apps, but your multi-service setup would need custom infrastructure. Skip for the Marathon.

---

## Checkpoint

1. Pushing to main triggers the publish workflow.
2. Three images appear on ghcr.io with correct tags.
3. The deploy workflow SSHes into the server and runs `docker compose pull`.
4. The server's containers update to the new images.
5. Tagging `v1.0.0` produces versioned images.
6. A manual `workflow_dispatch` can redeploy without pushing new code.

Commit:

```bash
git add .
git commit -m "ci: publish docker images to ghcr and deploy on push"
git push
```

---

## What You Learned

- ghcr.io is free and integrated with GitHub.
- Image tags come from branches, commits, and semver.
- Deploy workflows SSH to the server and pull images.
- Versioned images enable clean rollback.
- The deploy user on the server should have minimum permissions.

Tomorrow is the recap. The rest of the week builds Projects 5 and 6 on this deploy pipeline.
