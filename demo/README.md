# Capsule Demo Environment

A Helm chart that provisions a ready-to-use multi-tenant demo cluster using [Capsule](https://capsule.clastix.io).

## What it installs

**Tenants** — five tenants across two energy categories, all defined in `values.yaml`:

| Tenant | Category |
|--------|----------|
| solar  | renewable |
| green  | renewable |
| wind   | renewable |
| gas    | non-renewable |
| coal   | non-renewable |

Each tenant gets:
- A `TenantOwner` (both a `User` and a `ServiceAccount`) with `admin` + `capsule-namespace-deleter` cluster roles
- Namespace quota, resource quotas
- A `LimitRange` distributed via `GlobalTenantResource` (default 200m CPU / 256Mi memory per container)
- A default `NetworkPolicy` distributed via `GlobalTenantResource` (Only traffic inside the tenant is allowed + egress traffic to dns)
- An `ImagePullSecret` replicated into every namespace
- Serviceaccount `default` that uses the imagePullSecret by default

## Prerequisites

- A running Kubernetes cluster with [Capsule installed](https://capsule.clastix.io/docs/tenants/quickstart/)
- `helm` ≥ 3

## Install

```bash
helm install capsule-demo . -n capsule-system
```

## Try it out

Each tenant has a dedicated owner user. Impersonate one to act as a Tenant Owner:

```bash
# Create a namespace as the solar tenant owner
kubectl create ns solar-testns --as solar-owner --as-group projectcapsule.dev

# List namespaces visible to the solar owner
kubectl get ns --as solar-owner --as-group projectcapsule.dev
```

Owners can manage namespaces within their own tenant but cannot see or touch resources belonging to other tenants.

## Configuration

| Value | Description | Default |
|-------|-------------|---------|
| `tenants` | List of tenants to create, each with a `name` and `energy` label | see `values.yaml` |
| `networkpolicies.dns.podSelector.matchLabels` | Label selector for the CoreDNS pods used in the egress DNS rule | `k8s-app: kube-dns` |

## Uninstall

```bash
helm uninstall capsule-demo -n capsule-system
```
