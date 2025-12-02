# k3-manifests

Kubernetes manifests repository for ArgoCD deployments.

## Structure

```
K8s-Manifest/
└── testproj/
    ├── Argo-Rollout/
    │   └── testproj.yaml      # Argo Rollout resource
    ├── svc.yaml                # Preview and Active services
    └── ingress.yaml            # Ingress configuration
```

## Image Tag Format

Images are tagged with the format: `main-{short-sha}-{run-id}`

Example: `argahv/testproj:main-75d20dc-0123`

- `main`: Environment/branch name
- `75d20dc`: Short commit SHA (7 characters)
- `0123`: Run ID (first 4 characters)

## ArgoCD Application

The ArgoCD Application manifest (`argocd-application.yaml`) should be applied to your ArgoCD instance to watch this repository.

## Deployment Flow

1. Code is pushed to `testproj` repository
2. GitHub Actions builds Docker image with tag `main-{short-sha}-{run-id}`
3. Workflow updates this repository's Rollout manifest
4. ArgoCD detects the change and syncs automatically
5. Argo Rollout performs gradual canary deployment (20% → 50% → 100%)

# k3-manifests
