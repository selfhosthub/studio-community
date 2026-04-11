# Super Admin Guide

Deploy and run Studio in production with security, backups, monitoring, and upgrades.

Choose **Docker Compose** for simplest setup, **Kubernetes** for scaling. Add **RunPod Serverless** workers if you need GPUs for AI workloads.

---

## Common tasks

- Get Studio running fast → [Quick start](#quick-start)
- Pick a deployment style → [Deployment choices](#deployment-choices)
- Generate required secrets → [Generate secrets](#generate-secrets)
- Put Studio behind TLS → [TLS and reverse proxy](#tls-and-reverse-proxy)
- Set CORS correctly → [CORS](#cors)
- Back up and restore → [Backups](#backups)
- Monitor health → [Health checks](#health-checks)
- Scale workers → [Scaling workers](#scaling-workers)
- Expose Studio publicly → [Cloudflare Tunnel](#cloudflare-tunnel-setup)
- Protect API from unauthorized access → [Cloudflare Zero Trust](#cloudflare-zero-trust-service-tokens)
- Add GPU workers → [RunPod Serverless](#runpod-serverless)
- Set up custom domains → [Custom domains](#custom-domains)
- Configure OAuth (Google/GitHub) → [OAuth integrations](#oauth-integrations)
- Troubleshoot common outages → [Troubleshooting](#troubleshooting)

---

## Quick start

The fastest way to get Studio running is with Docker.

### Prerequisites

| Software | Purpose | Verify |
|----------|---------|--------|
| **Docker** | Container runtime | `docker --version` |

System requirements: 8GB RAM minimum (16GB recommended), 10GB free disk.

> **Docker Compose** is only needed if you run services separately (e.g., external Postgres, separate workers). The quick start uses a single container.

### Steps

```bash
docker run -d --name studio selfhosthub/studio:latest
```

What happens:
1. Pulls the Studio image from DockerHub
2. Starts PostgreSQL, API, and worker containers
3. Creates database tables
4. Creates super admin account

**Default login:** `admin@localhost` / `admin` (you'll be forced to change on first login)

First time takes 2-5 minutes (downloading the image). After that, under a minute.

> **Building from source?** If you're a developer contributing to Studio, clone the repo and use `make prod` instead. See the [Developer Guide](../../docs/development/developers-guide.md).

### Verify it works

1. Open http://localhost:8000/health — should show `{"status": "healthy"}`
2. Open http://localhost:8000/docs — interactive API docs
3. Log in with the default credentials
4. Create a workflow, add a step, run it

---

## Deployment choices

Not sure which setup is right for you? Start with two questions:

**1. Do you need GPU for AI workloads?** (ComfyUI, TTS, image generation)

**2. Do you have your own GPU hardware?** (NVIDIA GPU desktop, Apple Silicon Mac, etc.)

| I want to... | Use this | Networking |
|---|---|---|
| **Try it on my machine** | Docker Compose (local) | localhost |
| **Run it for my team** | Docker Compose on a VPS | Cloudflare Tunnel |
| **Run AI workflows on my own GPU machine** | Docker Compose (local, with GPU workers) | Cloudflare Tunnel if public |
| **Run AI workflows, no GPU hardware** | Docker Compose + RunPod Serverless | Cloudflare Tunnel + RunPod Serverless |
| **API at home, GPU workers in the cloud** | Docker Compose + RunPod Serverless | Cloudflare Tunnel |
| **Scale for production** | Kubernetes | Ingress + cert-manager |
| **Production + GPU** | Kubernetes + RunPod Serverless | Ingress + RunPod Serverless |

> **New to self-hosting?** Join the [SelfHostHub Skool community](https://www.skool.com/selfhosthub) — we'll help you pick the right setup and get running.

### Try it on my machine

1. [ ] Install [Docker](#prerequisites)
2. [ ] Run — [Quick start](#quick-start)
3. [ ] Verify — [health check + login](#verify-it-works)

### Run it for my team (VPS)

1. [ ] Get a VPS (any provider — Hetzner, DigitalOcean, Linode, etc.)
2. [ ] SSH in, install [Docker + Docker Compose](#prerequisites)
3. [ ] Run — [Quick start](#quick-start)
4. [ ] [Generate real secrets](#generate-secrets) (don't use defaults)
5. [ ] Set up [Cloudflare Tunnel](#cloudflare-tunnel-setup) to expose publicly
6. [ ] Set [CORS](#cors) to your domain

### Own GPU machine (NVIDIA on Linux or Windows)

1. [ ] Install NVIDIA drivers for your GPU
2. [ ] Install [CUDA Toolkit](https://developer.nvidia.com/cuda-downloads)
3. [ ] Install [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) (lets Docker access the GPU)
4. [ ] Install [Docker](#prerequisites)
5. [ ] Verify GPU access: `docker run --rm --gpus all nvidia/cuda:12.0-base nvidia-smi`
6. [ ] Run — [Quick start](#quick-start)
7. [ ] Start GPU workers — [GPU workers](#gpu-workers)
8. [ ] Optional: [Cloudflare Tunnel](#cloudflare-tunnel-setup) if you want public access

### Own GPU machine (Apple Silicon Mac)

1. [ ] Install [Docker Desktop for Mac](https://www.docker.com/products/docker-desktop/)
2. [ ] Run — [Quick start](#quick-start)
3. [ ] GPU workers use Metal/MPS acceleration automatically
4. [ ] Optional: [Cloudflare Tunnel](#cloudflare-tunnel-setup) if you want public access

### API at home + GPU in the cloud (RunPod Serverless)

1. [ ] Set up Studio on your server — [Docker deployment](#docker-deployment)
2. [ ] Set up [Cloudflare Tunnel](#cloudflare-tunnel-setup) on your server
3. [ ] Set up [Cloudflare Zero Trust service tokens](#cloudflare-zero-trust-service-tokens) to protect internal API endpoints
4. [ ] Create [RunPod Serverless endpoints](#runpod-serverless) for your GPU worker types
5. [ ] Configure RunPod endpoint IDs in Studio settings
6. [ ] [Generate real secrets](#generate-secrets) (don't use defaults)

### Kubernetes

1. [ ] Set up a Kubernetes cluster (GKE, EKS, self-managed, etc.)
2. [ ] Deploy Studio — [Kubernetes deployment](#kubernetes-deployment) (detailed guide coming soon)
3. [ ] Configure [TLS via ingress](#tls-and-reverse-proxy)
4. [ ] Optional: add [RunPod Serverless](#runpod-serverless) for GPU workloads

---

## Production checklist

If you do only one thing from this guide, do this:

1. Generate secrets (JWT, worker shared secret, DB password)
2. Put Studio behind TLS
3. Change default admin password (if seeded)
4. Configure backups (database + workspace files)
5. Add monitoring/alerts for health endpoints

Everything else is optimization.

---

## Generate secrets

Generate these with a real CSPRNG:

```bash
openssl rand -hex 32
```

Required secrets (typical):

* `SHS_JWT_SECRET_KEY` (signs auth tokens)
* `SHS_WORKER_SHARED_SECRET` (workers authenticate to API)
* `POSTGRES_PASSWORD` (if you manage Postgres yourself)

Store them in your secret manager:

* Kubernetes Secret
* Docker secrets / env management
* RunPod Secrets (for serverless workers)
* Vault / SSM Parameter Store / etc.

---

## Docker deployment

Use Docker when you want the simplest stable deployment.

### Minimal steps

1. Clone repo / obtain deployment bundle
2. Create a production env file
3. Set required variables:

   * `SHS_DATABASE_URL`
   * `SHS_JWT_SECRET_KEY`
   * `SHS_WORKER_SHARED_SECRET`
4. Start the stack (`make prod` if provided, or `docker compose up -d`)

### What to verify

* Web UI loads
* You can log in
* Creating a workflow works
* Running a workflow creates an instance and completes
* Worker health shows active

---

## GPU workers

If your workflows use AI services like ComfyUI, TTS, or image generation, those workers need GPUs.

Two approaches:

1. **Local GPU** — run GPU workers on the same machine as Studio (or on your network). Workers poll the API for jobs and write output files to the local `/workspace` directory.
2. **RunPod Serverless** — Studio's orchestrator sends GPU jobs to RunPod's serverless platform. RunPod spins up GPU containers on demand, executes the work, and the handler pushes results back to Studio over HTTPS. No shared storage, no persistent pods.

### Local GPU workers

For machines with NVIDIA GPUs co-located with your Studio server:

```bash
docker run -d \
  --name worker-comfyui \
  --gpus all \
  -e SHS_API_BASE_URL=http://localhost:8000 \
  -e SHS_WORKER_SHARED_SECRET=<same-as-studio> \
  -v ~/workspace:/workspace \
  -v ~/models:/workspace/models \
  selfhosthub/studio-worker-comfyui:latest
```

Local workers share the `/workspace` filesystem with the API. They register, poll for jobs, execute, write files locally, and post results. This is the simplest setup if you have GPU hardware.

### RunPod Serverless (cloud GPUs)

See [RunPod Serverless](#runpod-serverless) for full setup.

---

## Networking

### How Studio networking works

Studio uses a simple architecture for remote GPU workers:

- **All traffic is outbound HTTP/HTTPS.** Workers call the API — the API never calls workers.
- **No shared storage needed for remote workers.** Workers push generated files to the API via an upload endpoint, then post the job result.
- **No mesh networking needed.** No Tailscale, no WireGuard, no VPN. Workers only need to reach the API's public URL.

This means the only networking component you need is a way to expose your Studio API to the internet. For most self-hosters, that's Cloudflare Tunnel.

### Recommended combinations

| Setup | Public access | GPU workers |
|-------|-------------|-------------|
| **Homelab, single server** | Cloudflare Tunnel | Local GPU workers (same machine) |
| **Homelab + cloud GPUs** | Cloudflare Tunnel | RunPod Serverless |
| **VPS, no GPU** | Cloudflare Tunnel | RunPod Serverless |
| **Kubernetes** | Ingress controller | RunPod Serverless (or GPU nodes) |

---

## Cloudflare Tunnel setup

Expose Studio to the internet without opening ports on your server. Cloudflare Tunnel creates an outbound connection from your server to Cloudflare's edge, so your origin IP stays hidden and you get DDoS protection, access policies, and TLS for free.

Use this when:
- You're running Studio on a homelab or VPS and want public access
- You want access controls (require login, restrict by email domain) before traffic even reaches Studio
- You need custom domains with automatic TLS

### Setup steps

1. Create a Cloudflare account and add your domain
2. Go to [Cloudflare Zero Trust](https://dash.cloudflare.com/) → Networks → Tunnels
3. Create a tunnel, name it (e.g., `studio`)
4. Copy the tunnel token
5. Run the connector:

```bash
docker run -d \
  --name cloudflared \
  --network host \
  cloudflare/cloudflared:latest \
  tunnel --no-autoupdate run --token <your-tunnel-token>
```

6. In the tunnel config, add a public hostname:
   - Subdomain: `studio` (or whatever you want)
   - Domain: `yourdomain.com`
   - Service: `http://localhost:8000`

   Cloudflare automatically creates a **CNAME record** (`studio.yourdomain.com → <tunnel-id>.cfargotunnel.com`) in your DNS. If you don't see it, add it manually in **DNS → Records**.

7. **(If using OAuth providers)** Some providers (e.g., TikTok) require **domain verification** via a **TXT record**. Add the TXT record the provider gives you to your domain's DNS in Cloudflare:
   - Go to **DNS → Records → Add Record**
   - Type: `TXT`, Name: `yourdomain.com` (or subdomain), Content: the verification string from the provider

8. Verify:

```bash
curl https://studio.yourdomain.com/health
```

9. Set environment variables to match the tunnel domain:

```bash
# Required — tells the API its public URL (used for OAuth callbacks, worker URLs)
SHS_API_BASE_URL=https://studio.yourdomain.com

# Required — tells the API where the frontend lives (used for post-OAuth redirects)
SHS_FRONTEND_URL=https://studio.yourdomain.com

# Restrict CORS to your tunnel domain
SHS_CORS_ORIGINS=https://studio.yourdomain.com
```

> **OAuth providers (TikTok, Google, etc.) require `SHS_API_BASE_URL` to match exactly what you registered as the callback URL in the provider's developer portal.** If `SHS_API_BASE_URL` is not set, it defaults to `http://localhost:8000` and OAuth callbacks will fail with redirect URI mismatch errors.

### Local config file

The steps above use the Docker/token approach. If you run `cloudflared` directly on your machine (common for homelabs), use a `config.yml` instead:

```yaml
tunnel: <tunnel-id>
credentials-file: /Users/<you>/.cloudflared/<tunnel-id>.json

ingress:
  - hostname: studio.yourdomain.com
    path: /api/*
    service: http://localhost:8000
  - hostname: studio.yourdomain.com
    path: /health
    service: http://localhost:8000
  - hostname: studio.yourdomain.com
    path: /ws/*
    service: http://localhost:8000
  - hostname: studio.yourdomain.com
    service: http://localhost:3000
  - hostname: other-app.yourdomain.com
    service: http://localhost:5679
  - service: http_status:404
```

A single tunnel can serve **multiple hostnames**, including hostnames from different domains — as long as each domain is a zone in your Cloudflare account.

Run the tunnel:

```bash
cloudflared tunnel run <tunnel-name>
```

> **Multi-domain tunnels:** If your tunnel was created under zone A (`domainA.com`) and you want to route a hostname from zone B (`domainB.com`), `cloudflared tunnel route dns` will append the hostname to zone A's domain — creating `studio.domainB.com.domainA.com`. Instead, add the CNAME record **manually** in the Cloudflare dashboard for zone B: Name = your subdomain, Target = `<tunnel-id>.cfargotunnel.com`, Proxied.

### Split UI/API routing

If you run the UI separately (dev mode on port 3000 or a separate container), you need path-based routing so `/api/*`, `/health`, and `/ws/*` go to the API port and everything else goes to the UI port. The config example above shows this pattern.

The UI also needs to know where to find the API. Set `NEXT_PUBLIC_API_URL` in the UI environment to the public tunnel URL:

```bash
# In the UI container / .env
NEXT_PUBLIC_API_URL=https://studio.yourdomain.com
```

Do **not** set this to `http://localhost:8000` — the browser makes requests from the user's machine, not the server, so it needs the public URL.

---

## Cloudflare Zero Trust service tokens

Protect your internal API endpoints (`/internal/*`, `/workers/*`) so only authorized workers can reach them. Without this, anyone who knows your API URL can send requests to internal endpoints — even if they fail authentication, your server still processes them.

With service tokens, Cloudflare blocks unauthorized requests at the edge. They never reach your server.

### When you need this

- You have remote workers (RunPod Serverless, remote machines) connecting through your Cloudflare Tunnel
- You want to prevent DOS attacks on internal API endpoints
- You want defense-in-depth on top of `SHS_WORKER_SHARED_SECRET` JWT auth

You don't need this if everything runs on localhost (no Cloudflare Tunnel).

### Step 1: Create a service token

1. Go to [Cloudflare Zero Trust](https://dash.cloudflare.com/) → Access controls → Service credentials → Service Tokens
2. Click **Create Service Token**
3. Name it `studio-workers`
4. Choose a duration (1 year is the default)
5. Click **Generate token**
6. **Save both values immediately** — Cloudflare only shows the secret once:
   - `CF-Access-Client-Id` (e.g., `abc123.access`)
   - `CF-Access-Client-Secret` (long hex string)

### Step 2: Create an Access Application for internal paths

1. Go to Access → Applications → **Add an Application** → Self-hosted
2. Application domain: `studio.yourdomain.com`
3. Add path rules to protect:
   - `studio.yourdomain.com/internal/*`
   - `studio.yourdomain.com/workers/*`
4. Under **Policies**, add a policy:
   - **Name:** `Worker Service Auth`
   - **Action:** `Service Auth` (not "Allow" — "Allow" redirects to identity provider login)
   - **Include rule:** Service Token → select `studio-workers`
5. Save

### Step 3: Keep UI paths accessible

If you access the Studio web UI through the same domain, either:

- **Option A:** Create a separate Access Application for the root path (`studio.yourdomain.com`) with your normal human authentication (email OTP, Google login, etc.)
- **Option B:** Only protect `/internal/*` and `/workers/*` paths — leave everything else unprotected at the Cloudflare level (Studio's own JWT auth handles the rest)

### Step 4: Add headers to your workers

For any worker connecting through Cloudflare, add two environment variables:

```bash
CF_ACCESS_CLIENT_ID=abc123.access
CF_ACCESS_CLIENT_SECRET=<your-secret>
```

For RunPod Serverless, add these as **RunPod Secrets** on each endpoint.

For Docker workers:

```bash
docker run -d \
  --name worker-comfyui \
  --gpus all \
  -e SHS_API_BASE_URL=https://studio.yourdomain.com \
  -e SHS_WORKER_SHARED_SECRET=<same-as-studio> \
  -e CF_ACCESS_CLIENT_ID=abc123.access \
  -e CF_ACCESS_CLIENT_SECRET=<your-secret> \
  selfhosthub/studio-worker-comfyui:latest
```

Workers co-located with the API (connecting to `localhost`) don't need these headers — they don't go through Cloudflare.

### What this gives you

| Without service tokens | With service tokens |
|---|---|
| Attacker hits `/internal/jobs/claim` → API processes and rejects (wasted CPU) | Attacker hits `/internal/jobs/claim` → Cloudflare returns 403 (API never sees it) |
| All protection is at the application layer | Two layers: Cloudflare edge + application JWT |
| Workers only need `SHS_WORKER_SHARED_SECRET` | Workers need `SHS_WORKER_SHARED_SECRET` + CF service token headers |

---

## RunPod Serverless

RunPod Serverless provides on-demand GPU compute for AI workloads. Instead of managing persistent GPU pods, Studio sends jobs to RunPod's serverless platform. RunPod spins up GPU containers, executes the work, and the handler pushes results back.

### Why serverless instead of pods

| Persistent pods | Serverless |
|---|---|
| You manage pod lifecycle (launch, monitor, terminate) | RunPod manages everything |
| Pay while idle | Pay only for compute time |
| Fixed GPU type per pod | Choose GPU tier per endpoint |
| Single pod = sequential jobs | Auto-scales for parallel jobs |
| Needs network storage for shared files | No shared storage for output — workers push files via HTTP |

### How it works

```
1. User runs workflow with GPU step
2. Studio orchestrator checks endpoint health across zones
3. Orchestrator picks the healthiest endpoint and dispatches the job
4. RunPod spins up GPU container (models loaded from Network Volume)
5. Handler executes (generates images, renders video, etc.)
6. Handler pushes output files to Studio API over HTTPS
7. Handler posts job result to Studio API
8. Container shuts down
```

For iteration jobs (e.g., generate 15 images), Studio dispatches 15 jobs to RunPod. RunPod processes them in parallel across multiple GPU containers. You get 15x parallelism instead of sequential processing.

### Setup steps

#### 1. Create a RunPod account

Sign up at [runpod.io](https://www.runpod.io/). Add a payment method.

#### 2. Create network volumes for your models

AI models (checkpoints, LoRAs) are too large to bake into Docker images and too slow to download on every cold start. RunPod Network Volumes provide persistent storage that mounts instantly when a worker starts.

**Important:** Each network volume pins its endpoint to one datacenter/zone. If that zone runs out of GPUs, jobs queue indefinitely. To avoid this, create identical volumes in 2-3 zones so Studio can route to whichever zone has available GPUs.

For each zone you want to support:

1. Go to RunPod → Storage → **Create Network Volume**
2. Choose a zone (e.g., US-TX-3, EU-RO-1, US-CA-1)
3. Set size large enough for your models (e.g., 100GB)
4. Create the volume
5. Launch a temporary pod in that zone, attach the volume, download your models to `/runpod-volume/`, then terminate the pod

Approximate cost: $0.07/GB/month for the first 1TB. 100GB across 3 zones = ~$21/month.

#### 3. Create serverless endpoints

Create one endpoint per worker type per zone:

| Worker type | What it does | Suggested GPU |
|---|---|---|
| **ComfyUI** | Image generation (Flux, Stable Diffusion) | RTX 4090 / RTX 5090 |
| **Video** | Video rendering, FFmpeg processing | RTX 4090 / A100 |
| **Audio** | TTS, voice cloning (Chatterbox) | RTX 4090 |

For each endpoint:
1. Go to RunPod → Serverless → **New Endpoint**
2. Select your Docker image (e.g., `selfhosthub/studio-serverless-comfyui:latest`)
3. Choose GPU type
4. Attach the network volume for that zone
5. Set max workers (controls parallelism — e.g., 15 for 15 concurrent image generations)
6. Set idle timeout (how long to keep a warm container — 5-30 seconds is typical)

For multi-zone availability, repeat for each zone (e.g., comfyui-us-tx, comfyui-eu-ro, comfyui-us-ca).

#### 4. Add secrets to each endpoint

In RunPod → Serverless → your endpoint → **Secrets**, add:

| Secret | Value |
|---|---|
| `SHS_API_BASE_URL` | `https://studio.yourdomain.com` |
| `SHS_WORKER_SHARED_SECRET` | Same secret as your Studio API |
| `CF_ACCESS_CLIENT_ID` | From [Cloudflare Zero Trust setup](#step-1-create-a-service-token) |
| `CF_ACCESS_CLIENT_SECRET` | From [Cloudflare Zero Trust setup](#step-1-create-a-service-token) |

These are injected as environment variables into every serverless container.

#### 5. Connect Studio to RunPod

1. In Studio, go to Settings → Infrastructure → **RunPod Serverless**
2. Enter your **RunPod API key** (from RunPod → Settings → API Keys)
3. Click **Save** — Studio automatically discovers all endpoints on your account
4. For each discovered endpoint, select its **worker type** (ComfyUI, Video, Audio)
5. Drag to set **priority order** — Studio tries the highest-priority endpoint first, falls back to the next if GPUs are unavailable
6. Click **Save Configuration**

That's it. Studio handles routing, health checking, and fallback across zones automatically.

#### 6. Test

1. Create a workflow with a ComfyUI image generation step
2. Run it
3. The orchestrator should dispatch to RunPod instead of waiting for a local GPU worker
4. Check the instance view — the step should complete and show output files

### Multi-zone routing

Studio checks each endpoint's health before dispatching a job. If a zone has no idle workers and a growing queue, Studio skips it and tries the next zone in priority order. If a job gets dispatched but sits in queue too long (no GPU picks it up), Studio cancels it and retries on another zone.

This means GPU availability in any single zone doesn't block your workflows — as long as at least one zone has capacity, your jobs run.

### Multiple GPU tiers

You can create multiple endpoints for the same worker type with different GPUs:

- **Basic tier:** RTX 5090 endpoint (fast, cost-effective for drafts)
- **Premium tier:** H100 endpoint (maximum quality for final renders)

Map both in Studio and set priority. Users can select the GPU tier when running a workflow, or you can set defaults per workflow.

### Monitoring

Studio's RunPod Serverless page shows live health for each endpoint:
- Workers: idle / running
- Queue depth: jobs waiting / in progress / completed / failed
- Status indicator: green (healthy), yellow (busy), red (no workers)

For deeper metrics (cost, execution time breakdown, cold start frequency), use the RunPod dashboard directly.

---

## TLS and reverse proxy

In production, terminate TLS at:

* Kubernetes ingress, or
* your load balancer, or
* a reverse proxy (nginx/caddy/traefik)

Make sure:

* HTTP → HTTPS redirect
* WebSocket support (Studio uses real-time events)

If live updates don't work, it's usually proxy WebSocket settings.

Example Kubernetes ingress TLS (cert-manager):

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - studio.yourdomain.com
    secretName: studio-tls
```

---

## CORS

Set CORS to your real domains in production.

```bash
SHS_CORS_ORIGINS=https://studio.yourdomain.com,https://admin.yourdomain.com
```

---

## Custom domains

If you support branded landing pages per org, you need a routing layer that forwards requests by Host header.

A common approach is Cloudflare Tunnel.

### Cloudflare Tunnel quick setup

1. Create a tunnel in Cloudflare Zero Trust
2. Get the tunnel token
3. Run the connector (`cloudflared`) with that token
4. Create DNS records (CNAME) pointing your domain to the tunnel

Studio then reads the `Host` header to determine which org branding to load.

---

## OAuth integrations

Providers that use OAuth (Google, TikTok, GitHub, etc.) require you to register your app with the provider and set environment variables so Studio can handle the OAuth flow.

### How OAuth works with Cloudflare Tunnel

```
User clicks "Authorize" in Studio UI
  → Studio API builds OAuth URL with callback = {SHS_API_BASE_URL}/api/v1/oauth/{provider}/callback
  → User is redirected to provider (Google, TikTok, etc.)
  → User authorizes
  → Provider redirects to {SHS_API_BASE_URL}/api/v1/oauth/{provider}/callback
  → Studio exchanges code for tokens
  → Studio redirects user to {SHS_FRONTEND_URL}/providers?oauth_success=true
```

Both `SHS_API_BASE_URL` and `SHS_FRONTEND_URL` must match your tunnel domain. If you're running locally with a Cloudflare Tunnel at `https://app.example.com`:

```bash
SHS_API_BASE_URL=https://app.example.com
SHS_FRONTEND_URL=https://app.example.com
```

The callback URL you register with each provider must match `SHS_API_BASE_URL` exactly — no trailing slashes, no `http` vs `https` mismatch.

### Google (Drive, Sheets, Gmail, Calendar)

1. Create a [Google Cloud](https://console.cloud.google.com/) project
2. Enable the APIs you need (Drive, Sheets, Gmail, Calendar)
3. Create OAuth 2.0 credentials (Web application type)
4. Add authorized redirect URI: `{SHS_API_BASE_URL}/api/v1/oauth/google/callback`

```bash
GOOGLE_CLIENT_ID=...
GOOGLE_CLIENT_SECRET=...
```

### TikTok

1. Create a [TikTok Developer](https://developers.tiktok.com/) app
2. Set redirect URI: `{SHS_API_BASE_URL}/api/v1/oauth/tiktok/callback` (no trailing slash)
3. Add required scopes: `user.info.basic`, `video.publish`, `video.upload`, etc.
4. Submit for app review (requires Terms of Service and Privacy Policy URLs)

```bash
TIKTOK_CLIENT_ID=...
TIKTOK_CLIENT_SECRET=...
```

> TikTok requires your app to be reviewed before it can access production APIs. During development, you can test with sandbox users added in the TikTok developer portal.

### GitHub

```bash
GITHUB_CLIENT_ID=...
GITHUB_CLIENT_SECRET=...
```

### OAuth security notes

* PKCE (Proof Key for Code Exchange) is always-on for all OAuth providers
* State/CSRF protection is built-in
* Tokens are encrypted at rest and scoped to organization
* Rotate client secrets immediately if exposed

---

## Health checks

Useful endpoints (typical):

* `/health` (basic)
* `/api/v1/infrastructure/health` (detailed)
* `/api/v1/infrastructure/health/workers`
* `/api/v1/infrastructure/health/full`

Alert on:

* API not healthy
* Worker heartbeats missing

---

## Monitoring and logs

### Logs

Docker:

```bash
docker logs <api-container-name> --tail 200
```

Kubernetes:

```bash
kubectl logs -n studio deployment/studio-api --tail=200
```

Set `SHS_DEBUG=false` in production to reduce noise.

### Worker logging configuration

Workers support structured logging with environment variables:

| Variable | Values | Default | Notes |
|----------|--------|---------|-------|
| `SHS_LOG_LEVEL` | DEBUG, INFO, WARNING, ERROR | INFO | Controls verbosity |
| `SHS_LOG_FORMAT` | pretty, json | pretty | Use `json` for ELK/Prometheus |
| `LOG_PREFIX` | any string | auto-detect | Override container name in logs |
| `LOG_COLORS` | true, false | true | Disable for non-TTY output |

**Pretty format** (development):
```
shs-worker-comfyui (172.17.0.2) | 15:33:19.086 | INFO | Job completed
```

**JSON format** (production/ELK):
```json
{"timestamp":"2024-12-30T15:30:45.123Z","level":"INFO","service":"shs-worker-comfyui","host":"172.17.0.2","message":"Job completed","job_id":"abc123"}
```

For production with log aggregation (ELK, Grafana Loki, etc.):
```bash
SHS_LOG_FORMAT=json SHS_LOG_LEVEL=INFO
```

### What to monitor

* request latency
* error rate
* queue depth / job backlog
* worker heartbeats (local workers)
* RunPod Serverless queue depth and execution time
* Postgres connections
* storage usage (workspace)

### Audit logging

Studio has two separate audit systems serving different purposes:

| System | Destination | Purpose | Events |
|--------|-------------|---------|--------|
| **Security Audit** | stdout/stderr | Real-time monitoring, SIEM | Login, logout, access denied, token refresh |
| **Business Audit** | PostgreSQL | Compliance, forensics | Credential CRUD, user changes, settings, templates |

**Security Audit (stdout)**

Logs appear in your container/pod logs:

```
SECURITY_AUDIT: LOGIN_SUCCESS | user=admin | org=acme-corp | ip=192.168.65.1 | [SUCCESS]
SECURITY_AUDIT: ACCESS_DENIED_ROLE | user=viewer | org=acme-corp | ip=192.168.65.1 | [FAILURE]
```

For production (JSON format for SIEM ingestion):
```bash
SHS_LOG_FORMAT=json
```

Output:
```json
{"log_type":"security_audit","timestamp":"2026-01-30T16:23:54Z","who":{"user_id":"...","username":"admin"},"what":{"event":"LOGIN_SUCCESS"},"result":{"status":"success"}}
```

Security audit answers: **"Who accessed the system?"**

**Business Audit (database)**

Stored in the `audit_events` table, accessible via:
- Admin UI at `/audit`
- API at `GET /api/v1/audit`

Business audit answers: **"What did they change?"** with full before/after values.

**Why two systems?**

- Security events need real-time visibility (watch login failures now)
- Business changes need queryable history (who changed this credential last month?)
- Database audit stays in your controlled retention policy
- Stdout audit integrates with your existing log aggregation (ELK, Loki, CloudWatch)

---

## Backups

Studio has three categories of data to back up:

1. **Database** — PostgreSQL (users, orgs, workflows, instances, credentials)
2. **User files** — workspace org directories (uploads, generated files, instance outputs)
3. **Marketplace files** — providers, blueprints, ComfyUI workflows, schemas (repo-level)

### Makefile commands

All backups go to `backups/<timestamp>/` by default. Override with `BACKUP_DIR=<path>`.

```bash
# Database
make backup-db                          # → backups/<timestamp>/database.sql
make restore-db                         # restore from most recent backup
make restore-db FROM=backups/20260223_210016/database.sql  # restore specific file

# User files (org uploads, instance outputs)
make list-orgs                          # show org UUIDs and sizes
make backup-files                       # archive all orgs
make backup-files ORG=<uuid>            # archive one org
make restore-files                      # restore from most recent backup
make restore-files FROM=backups/<ts>/orgs-all.tar.gz

# Marketplace (providers, catalog, studio-cat content)
make backup-marketplace                 # archive providers/ + studio-cat/
make restore-marketplace                # restore from most recent backup
make restore-marketplace FROM=backups/<ts>/marketplace.tar.gz
```

Restore commands auto-select the most recent backup when `FROM=` is not specified.

After restoring the database, restart the API: `docker compose restart api`

### User files — manual alternative

Org files live at `<WORKSPACE>/orgs/<org-uuid>/`. On the local filesystem you can just `cp` or `rsync` the directory directly. The Makefile targets are convenience wrappers around `tar`.

### Production environments

In production, `WORKSPACE` is typically an absolute path like `/workspace` (not `~/workspace-dev`). Pass it explicitly:

```bash
make backup-files WORKSPACE=/workspace
```

For Kubernetes or cloud storage (S3, NFS, block volumes), use your platform's snapshot/backup tools for workspace files — the Makefile targets are designed for Docker Compose deployments.

### Backup schedule

A backup you've never restored is not a backup.

| Install size | Database | User files | Restore test |
|---|---|---|---|
| Development | Before destructive commands (`make -f dev/Makefile.dev clean`, `make -f dev/Makefile.dev seed-menu`) | As needed | — |
| Small team | Daily | Weekly | Quarterly |
| Production | Daily (automated) | Daily (automated) | Monthly |

---

## Scaling workers

### Local workers

Local workers scale horizontally.

Docker Compose:

```bash
docker compose up -d --scale worker=3
```

Kubernetes:

```bash
kubectl scale -n studio deployment/studio-worker --replicas=5
```

If instances are stuck pending, scale workers.

### RunPod Serverless workers

Scaling is automatic. Set the **max workers** on your RunPod Serverless endpoint to control maximum parallelism. RunPod handles spin-up and spin-down.

For burst workloads (e.g., 15 image iterations), set max workers >= the iteration count to get full parallelism.

For GPU availability, add endpoints in multiple zones. Studio automatically routes to whichever zone has healthy workers. More zones = more availability, at the cost of duplicating network volumes (~$7/month per 100GB per zone).

---

## Kubernetes deployment

Kubernetes is the path for production scaling with separate API, workers, web, and database.

**Note:** Detailed Kubernetes deployment guide coming soon.

Key concepts:

* Ingress controller for routing
* TLS termination (cert-manager or load balancer)
* Managed or in-cluster Postgres
* Persistent volumes for database and workspace files
* Secrets for sensitive configuration

---

## Upgrades

Treat upgrades like a small deployment:

1. Back up database + workspace
2. Deploy new version
3. Run a small smoke test:

   * login
   * run a simple workflow
   * verify workers process jobs
4. Roll back if needed

---

## Troubleshooting

### API won't start

Most common causes:

* missing required env vars
* invalid `SHS_DATABASE_URL`
* secrets are empty or incorrectly quoted

Start by checking logs.

### Database connection failed

* confirm Postgres is reachable from the API container/pod
* verify credentials in `SHS_DATABASE_URL`
* check DNS/service name (Kubernetes)

### Instances stuck on pending

* workers not running (local) or RunPod endpoint not configured
* worker can't authenticate (wrong `SHS_WORKER_SHARED_SECRET`)
* RunPod Serverless: check endpoint status in RunPod dashboard
* Check orchestrator logs for dispatch errors

### WebSockets don't update live UI

* reverse proxy not configured for WebSockets
* TLS termination or headers dropping upgrade requests

### RunPod Serverless jobs failing

* Check RunPod dashboard for container logs
* Verify secrets are set correctly on the endpoint (API_BASE_URL, WORKER_SHARED_SECRET, CF headers)
* Verify your Cloudflare Tunnel is running and the API is reachable at the configured URL
* Check Cloudflare Access logs — if you see 403s, the service token may be misconfigured
* Test connectivity: from any machine, `curl -H "CF-Access-Client-Id: <id>" -H "CF-Access-Client-Secret: <secret>" https://studio.yourdomain.com/health`

### RunPod jobs stuck or slow to start

* Check Studio's RunPod Serverless page — it shows per-endpoint health (workers idle/running, queue depth)
* If all endpoints show zero workers and growing queues, GPUs are scarce — Studio will try each zone in priority order and cancel jobs that sit too long
* Check that your network volume is in the same zone as the endpoint — mismatched zones mean the endpoint can't mount the volume
* Increase max workers on your RunPod endpoint to allow more parallelism
* Consider adding endpoints in additional zones for better GPU availability

### Cloudflare Tunnel issues

* **Browser redirects to wrong domain:** If you previously had Cloudflare redirect rules (Page Rules, Redirect Rules) on the domain, browsers cache 301 permanent redirects. Clear browser cache or test in incognito. Also check **Rules → Redirect Rules** and **Rules → Page Rules** in the Cloudflare dashboard for the domain.
* **UI shows "site down" or maintenance page through tunnel:** The UI can't reach the API. Check that path-based routing is configured in your tunnel config and `NEXT_PUBLIC_API_URL` points to the tunnel URL (e.g., `https://studio.yourdomain.com`), not `http://localhost:8000`.
* **`cloudflared tunnel route dns` creates record in wrong zone:** If your tunnel was created under zone A and you're routing a hostname from zone B, `route dns` appends to zone A's domain. Add the CNAME manually in zone B's DNS instead: Name = subdomain, Target = `<tunnel-id>.cfargotunnel.com`, Proxied.
* **Tunnel hostname not resolving or returning wrong IP:** Cloudflare may add a default A record for your domain (e.g., pointing to a parking page or placeholder IP). This overrides the CNAME record for the tunnel. Go to **DNS → Records** for the domain and delete any A or AAAA records that conflict with your tunnel's CNAME record.

### Cloudflare service token issues

* **403 Forbidden from Cloudflare:** Service token headers missing or wrong. Double-check `CF_ACCESS_CLIENT_ID` and `CF_ACCESS_CLIENT_SECRET` values.
* **Redirected to login page:** Access policy action is set to "Allow" instead of "Service Auth". Change the policy action.
* **Local workers broken after adding Access policies:** Make sure your Access Application only covers `/internal/*` and `/workers/*` paths — not the entire domain.

---

## Environment variables reference

### Required

| Variable               | What it does                 |
| ---------------------- | ---------------------------- |
| `SHS_DATABASE_URL`         | PostgreSQL connection URL    |
| `SHS_JWT_SECRET_KEY`       | Signs auth tokens            |
| `SHS_WORKER_SHARED_SECRET` | Auth between API and workers |

Generate secrets with:

```bash
openssl rand -hex 32
```

### Common optional variables

| Variable                  | Typical default | Notes                                           |
| ------------------------- | --------------: | ----------------------------------------------- |
| `SHS_ENV`                 |   `development` | Use `production` in prod                        |
| `SHS_DEBUG`               |          `true` | Set `false` in prod                             |
| `PORT`                    |          `8000` | API port                                        |
| `HOST`                    |       `0.0.0.0` | Bind address                                    |
| `SHS_WORKSPACE_ROOT`      |      `/workspace` | Where organization files live                   |
| `SHS_CREATE_TABLES`       |          varies | Disable in production if migrations are managed |
| `SHS_API_BASE_URL`            | `http://localhost:8000` | Public URL of the API (OAuth callbacks, worker URLs) |
| `SHS_FRONTEND_URL`            | `http://localhost:3000` | Public URL of the frontend (post-OAuth redirects)    |
| `SHS_CORS_ORIGINS`        |             `*` | Restrict in production                          |
| `CLOUDFLARE_TUNNEL_TOKEN` |               - | For custom domains                              |
| `SHS_MAX_UPLOAD_SIZE_MB`  |           `100` | Max file size for worker uploads                |

### RunPod Serverless variables (API-side)

| Variable                   | Notes                                       |
| -------------------------- | ------------------------------------------- |
| `RUNPOD_API_KEY`           | RunPod API key (or set via UI)              |
| `STUDIO_API_URL`           | Public URL for workers to call back to      |

Note: Individual endpoint IDs are not configured via env vars. Studio auto-discovers endpoints from RunPod's API when you enter your API key. Endpoint-to-worker-type mappings are managed in the Studio UI and stored in the database.

### Worker variables (set on worker containers or RunPod Secrets)

| Variable                   | Notes                                       |
| -------------------------- | ------------------------------------------- |
| `SHS_API_BASE_URL`             | Studio API URL the worker calls             |
| `SHS_WORKER_SHARED_SECRET`     | Must match API's secret                     |
| `CF_ACCESS_CLIENT_ID`      | Cloudflare service token (remote workers)   |
| `CF_ACCESS_CLIENT_SECRET`  | Cloudflare service token (remote workers)   |
| `SHS_LOG_LEVEL`                | DEBUG, INFO, WARNING, ERROR                 |
| `SHS_LOG_FORMAT`               | `pretty` or `json`                          |
| `SHS_LOG_PREFIX`           | Override container name in log prefix       |
| `SHS_LOG_COLORS`           | `true` / `false`                            |

---

## Compliance and data handling

At minimum, be able to answer these:

* Where is user data stored?
* How are secrets encrypted?
* How do we delete user data?
* How do we export data for an org?

Typical storage:

* DB (PostgreSQL): orgs, users, workflows, instances, secrets (encrypted)
* Workspace/storage: files and artifacts

### Instance deletion

Admins and users can permanently delete terminal instances (completed, failed, cancelled) from the instances list page. Deletion removes:

* All output files and thumbnails from disk
* All database records: `job_output_resources`, `queued_jobs`, `job_executions`, `instance_steps`, `instances`
* The instance directory (`workspace/orgs/{org_id}/instances/{instance_id}/`)

Admins can delete any instance in their org. Users can only delete instances they created. This is a hard delete — no soft-delete or archive step.

If you need GDPR-style behavior, document:

* deactivation/offboarding
* data export
* retention policy
* deletion policy (instance deletion covers per-run cleanup; org-level data removal is manual via DB)

---

## Next steps

* **[Admins Guide](admin.md)**: org setup, users, credentials, billing, limits
* **[Users Guide](user.md)**: build workflows, run instances, debug failures