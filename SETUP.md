# Setup Guide for Separate Manifest Repository

## Step 1: Create the Manifest Repository

Run these commands to initialize the `k3-manifests` repository:

```bash
cd /path/to/k3-manifests
echo "# k3-manifests" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:argahv/k3-manifests.git
git push -u origin main
```

## Step 2: Copy Manifest Files

Copy the manifest files from `testproj/manifests/K8s-Manifest/testproj/` to your `k3-manifests` repository:

```bash
# From the testproj repository
cp -r manifests/K8s-Manifest/testproj /path/to/k3-manifests/K8s-Manifest/
cd /path/to/k3-manifests
git add K8s-Manifest/
git commit -m "Add testproj manifests"
git push origin main
```

## Step 3: Set Up ArgoCD Application

Apply the ArgoCD Application manifest to your ArgoCD instance:

```bash
kubectl apply -f manifests/argocd-application.yaml
```

Or manually create the application in ArgoCD UI with these settings:
- **Name**: testproj
- **Project**: default
- **Repository URL**: git@github.com:argahv/k3-manifests.git
- **Path**: ./K8s-Manifest/testproj
- **Target Revision**: HEAD
- **Namespace**: testproj
- **Auto-sync**: Enabled
- **Prune**: Enabled
- **Self-heal**: Enabled

## Step 4: Configure GitHub Secrets

Ensure your GitHub repository has these secrets configured:
- `DOCKER_USERNAME`: Docker Hub username
- `DOCKER_PASSWORD`: Docker Hub password/token
- `GITHUB_TOKEN`: GitHub token (default, automatically provided)

The workflow will use `GITHUB_TOKEN` to push to the manifest repository.

## Step 5: Test the Workflow

1. Make a change to your code
2. Push to `main` branch
3. Watch GitHub Actions workflow:
   - Builds Docker image with tag: `main-{short-sha}-{run-id}`
   - Updates manifest repository
   - Pushes changes
4. ArgoCD should detect the change and sync automatically
5. Argo Rollout will perform gradual canary deployment

## Troubleshooting

### ArgoCD Not Syncing

1. Check ArgoCD can access the repository:
   ```bash
   argocd repo get git@github.com:argahv/k3-manifests.git
   ```

2. Manually refresh ArgoCD:
   ```bash
   argocd app get testproj
   argocd app sync testproj
   ```

### Workflow Fails to Push

1. Verify `GITHUB_TOKEN` has write access to `k3-manifests` repository
2. Check repository permissions in GitHub settings
3. Ensure the manifest repository exists and is accessible

### Rollout Not Progressing

1. Check Rollout status:
   ```bash
   kubectl get rollout testproj -n testproj
   kubectl describe rollout testproj -n testproj
   ```

2. Check pod status:
   ```bash
   kubectl get pods -n testproj -l app=testproj
   ```

3. Check services:
   ```bash
   kubectl get svc -n testproj
   ```

