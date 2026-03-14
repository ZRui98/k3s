# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a GitOps repository for a self-hosted K3s cluster, managed by FluxCD. Changes pushed to the `master` branch are automatically reconciled by Flux (poll interval: 10 minutes). The cluster is at `zrui.dev`.

## Architecture

**GitOps flow:** FluxCD watches this repo (`flux-system/gotk-sync.yaml`) and applies everything from the root path. All Kubernetes manifests are applied directly — there is no Kustomize layering beyond what Flux itself uses.

**Secret management:** Secrets are encrypted with Bitnami Sealed Secrets. The public key is `pub-sealed-secrets.pem`. Raw secret files (`.secret` extension, `mysecret.*`, `mysealedsecret.*`) must never be committed.

**Ingress & TLS:** Traefik is the ingress controller. TLS certificates are issued by cert-manager using the `letsencrypt-prod` ClusterIssuer (HTTP01 challenge via Traefik). All ingress resources annotate with `cert-manager.io/cluster-issuer: letsencrypt-prod`.

**Helm via Flux:** Helm charts are managed through `HelmRelease` resources referencing either `HelmRepository` (HTTP) or `OCIRepository` (OCI) sources. Drift detection is enabled on all releases.

## Services

| Directory | Service | Domain |
|---|---|---|
| `forgejo/` | Forgejo (Git hosting) | git.zrui.dev |
| `woodpecker/` | Woodpecker CI | ci.zrui.dev |
| `miniflux-configs/` | Miniflux RSS + Postgres | — |
| `victoriametrics/` | VictoriaMetrics monitoring stack | — |
| `vector/` | Vector log agent (DaemonSet) | — |
| `kube-state-metrics/` | kube-state-metrics | — |
| `cert-manager/` | cert-manager + ClusterIssuer | — |
| `sealed-secrets/` | Sealed Secrets controller | — |
| `system-upgrade-controller/` | K3s auto-upgrade controller | — |
| `git-services/` | Small self-hosted git apps | — |
| `flux-system/` | FluxCD bootstrap manifests | — |

## Common Operations

### Sealing a secret

```bash
# Create a secret manifest without applying it
kubectl create secret generic my-secret \
  --from-literal=key=value \
  --dry-run=client -o yaml > mysecret.yaml

# Seal it using the cluster's public key
kubeseal --cert pub-sealed-secrets.pem \
  --format yaml < mysecret.yaml > mysealedsecret.yaml
```

### Applying changes manually (bypassing Flux wait)

```bash
kubectl apply -f <file-or-directory>
```

### Checking Flux reconciliation status

```bash
flux get all
flux logs --follow
```

### Updating the Vector DaemonSet

```bash
kubectl apply -f vector/vector-agent.yaml
kubectl rollout restart ds vector -n vector
kubectl rollout status daemonset vector -n vector
```

### Upgrading K3s

Edit the `version` field in `k3s/plan.yaml` for both `server-plan` and `agent-plan`, then commit and push.

### VictoriaMetrics auth setup

```bash
htpasswd -c auth.txt.secret adminuser
kubectl create secret generic vmui-auth-secret --from-file=auth.txt.secret -n vm
```

## Secrets Convention

- Files ending in `.secret` are raw secret values — never commit them.
- `mysecret.yaml` / `mysecret.json` — unsecured Kubernetes secret manifests, not for committing.
- `mysealedsecret.yaml` — output of `kubeseal`, safe to commit.
- `pub-sealed-secrets.pem` — the cluster's Sealed Secrets public key, used for sealing.

## Node Affinity

Workloads with persistent storage (Forgejo, Miniflux Postgres, VictoriaMetrics) prefer nodes labeled `type=storage`.
