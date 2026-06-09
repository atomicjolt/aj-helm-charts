# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Chart Overview

This is the `catalyst-chart` Helm chart for deploying an Elixir (Phoenix/OTP) application called Catalyst. It manages three distinct deployment types (app, worker, lead-worker) plus a migrations Job, optional Redis subchart, and optional DNS sidecar.

## Common Commands

Build chart dependencies (required before packaging or linting):
```sh
helm dependencies build .
```

Lint the chart:
```sh
helm lint .
```

Render templates locally without installing:
```sh
helm template my-release . -f my-values.yaml
```

Package the chart:
```sh
cd ..  # go up to charts/ directory
helm package catalyst
```

Validate a values file renders correctly:
```sh
helm template my-release . -f my-values.yaml --debug
```

## Architecture: Deployment Types

All three deployments run the same container image (`.Values.image`) but are differentiated by their ConfigMap-injected `specific.toml`, which controls which Catalyst subsystems activate.

| Deployment | Config | Scheduler | Regular jobs | Slow jobs | Phoenix endpoints |
|---|---|---|---|---|---|
| `catalyst-app` | `catalyst-app-config` | off | — | — | on |
| `catalyst-worker` | `catalyst-worker-config` | on | on | off | — |
| `catalyst-lead-worker` | `catalyst-lead-worker-config` | on | off | on | — |

The lead-worker is always `replicas: 1` (single instance for cluster coordination and slow job processing). Workers use `strategy: Recreate`.

## Config and Secrets

Each pod mounts a projected volume at `/etc/catalyst` that combines:
1. **Secret** `catalyst-config` — app secrets (database URL, API keys, etc.). Can be populated via `externalSecrets.config`.
2. **ConfigMap** `catalyst-{app|worker|lead-worker}-config` — role-specific TOML that activates/deactivates Catalyst subsystems.

The app also uses `REPLACE_OS_VARS=true` and `ERLANG_NAME` set from the pod IP (for Erlang clustering).

## Migrations

Migrations run as a Helm hook Job (`post-install,pre-upgrade`) that executes `Catalyst.ReleaseTasks.migrate(nil)`. Every deployment also has a `check-migrations` init container that blocks pod startup until migrations are current.

## DNS Sidecar (optional)

When `dnsSidecar.enabled: true`, the worker deployment gets an Unbound DNS caching sidecar (native Kubernetes sidecar: init container with `restartPolicy: Always`, requires Kubernetes 1.29+). It overrides the pod's DNS to `127.0.0.1` and forwards:
- `cluster.local.` → cluster DNS (CoreDNS)
- Zones in `clusterForwardZones` (default: `amazonaws.com.`) → CoreDNS
- All other traffic → `forwarders` (default: AWS Route53 at `169.254.169.253`)

The DNS sidecar config is rendered via `tpl` so it supports Helm template expressions. Metrics are exposed via the `cyb3rjak3/unbound-exporter` container when `dnsSidecar.metrics.enabled: true`.

## Redis (optional subchart)

When `redis.enabled: true`, the chart pulls in the `groundhog2k/redis` chart (v2.1.2). Key notes:
- Redis password is set via a Secret containing a `redis.conf` block with `requirepass`.
- The service name is `<release-name>-redis` (not `-redis-master` as in the old Bitnami chart — see `UPGRADING_REDIS.md`).
- External secret support available via `externalSecrets.redis`.

## Scheduled Scaling

When `scheduledScaling.enabled: true`, the chart creates CronJobs that use `kubectl patch` to adjust `catalyst-worker` replica counts on a schedule. This creates its own ServiceAccount, Role, and RoleBinding (`worker-scaler`).

## Key Values

Required values that have no defaults and must be provided:
- `image` — container image for all Catalyst deployments
- `hostname` — used in the ALB Ingress rule and TLS host
- `appReplicaCount`, `workerReplicaCount` — replica counts
- `appResources`, `workerResources`, `leadWorkerResources` — resource requests/limits
- `appIngressClassName` — ingress class (e.g., `alb`)

The `catalyst-config` Secret must exist before install unless `externalSecrets.config.enabled: true` (which creates it via the External Secrets Operator as a pre-install hook).
