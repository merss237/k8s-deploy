# Reusable GitHub Workflow: Kubernetes App Deployment with ArgoCD

> ⚠️ **Beta Notice**: This workflow is currently in beta. While the structure is stable, it has not yet been fully tested in production environments. Use with caution and feel free to contribute improvements via pull request.

This reusable GitHub Actions workflow deploys a Kubernetes application by rendering manifests using `kustomize` and syncing them to an ArgoCD-managed cluster.

---

## 🔧 Inputs

| Name                | Required | Type   | Description                                                                 |
|---------------------|----------|--------|-----------------------------------------------------------------------------|
| `environment`       | ✅        | string | Environment name (e.g. `dev`, `prod`)                                       |
| `application`       | ✅        | string | Application name                                                            |
| `namespace`         | ✅        | string | Kubernetes namespace to deploy into                                         |
| `repo`              | ✅        | string | GitHub repo where source manifests live (e.g. `my-org/my-app`)             |
| `repo_path`         | ✅        | string | Subdirectory in the repo containing manifests (e.g. `manifests/`)          |
| `repo_commit_id`    | ❌        | string | Commit ID to checkout (takes precedence over branch if both set)           |
| `branch_name`       | ❌        | string | Branch to checkout (default: `main`)                                       |
| `overlay_dir`       | ✅        | string | Path to the kustomize overlay directory within the repo                    |
| `image_tag`         | ❌        | string | Optional image tag to set on one or more container images                  |
| `image_base_name`   | ❌        | string | Single image base name to patch (e.g. `my-app`)                            |
| `image_base_names`  | ❌        | array  | Multiple image base names to patch                                         |
| `kustomize_version` | ❌        | string | Kustomize version (default: `5.0.1`)                                       |
| `skip_status_check` | ❌        | string | If `true`, skips waiting for ArgoCD sync to complete. Only confirms sync was accepted. |
| `runner`            | ❌        | string | Runner label to execute the workflow on (e.g. `ubuntu-latest`, `self-hosted`) |


---

## 🔐 Secrets

These secrets must be defined in the calling repository:

| Name                   | Description |
|------------------------|-------------|
| `ARGO_CD_ADMIN_USER`   | ArgoCD admin username |
| `ARGO_CD_ADMIN_PASSWORD` | ArgoCD admin password |

---

## 🔧 Actions Used

The `gitopsmanager` reusable workflow uses the following GitHub Actions:

| Action                                                                 | Version | Description                                                                 |
|------------------------------------------------------------------------|---------|-----------------------------------------------------------------------------|
| ![Checkout](https://img.shields.io/badge/actions--checkout-v4-blue?logo=github) [actions/checkout](https://github.com/actions/checkout) | `v4` | Checks out the repository so workflow jobs can access its contents. |
| ![Envsubst](https://img.shields.io/badge/envsubst-v1-blue?logo=gnu) [nowactions/envsubst](https://github.com/nowactions/envsubst) | `v1` | Substitutes environment variables in files using the `envsubst` CLI. |
| ![Kustomize](https://img.shields.io/badge/setup--kustomize-v2-blue?logo=kubernetes) [imranismail/setup-kustomize](https://github.com/imranismail/setup-kustomize) | `v2` | Sets up Kustomize CLI for building Kubernetes manifests. |
| ![Artifact](https://img.shields.io/badge/upload--artifact-v4-blue?logo=github) [actions/upload-artifact](https://github.com/actions/upload-artifact) | `v4` | Uploads generated workflow artifacts (e.g., rendered YAML files). |
| ![YQ](https://img.shields.io/badge/yq-v4-blue?logo=yaml) [mikefarah/yq](https://github.com/mikefarah/yq) | `v4` | Installs the `yq` YAML processor for working with manifests. |

---

## 🧱 Assumptions

1. Your **GitHub runners** (e.g., via GitHub ARC) have:
   - Read access to all source repos
   - Write access to the `continuous-deployment` repo
   - Authentication is handled via a GitHub App in the runner pod

2. Your `kustomization.yaml` must include image replacement like:
```yaml
images:
- name: my-app
  newName: ghcr.io/my-org/my-app
  newTag: "999"
```

3. The `continuous-deployment` repo contains a directory called `config/` with a file named `env-map.yaml`:
```yaml
config/env-map.yaml:
  dev:
    cluster: weu-eks03
    dns_zone: prod.aws.demo.internal
```
This file defines environment-specific values used in the workflow:
- `cluster`: used to construct the Argo CD ingress hostname
- `dns_zone`: used as the DNS suffix to fully resolve the Argo CD hostname

The workflow uses this data to template Ingress resources and build the ArgoCD endpoint URL:
```text
https://<cluster>-argocd-argocd-web-ui.<dns_zone>
```
So in the example above, the Argo CD instance will be located at:
```
https://weu-eks03-argocd-argocd-web-ui.prod.aws.demo.internal
```

> 💡 **Assumption**: Your Argo CD installation uses this naming convention for its ingress hostname. If your setup differs, you should update the URL construction logic in the workflow accordingly.

---

## 📁 CD Repo Structure

Manifests will be rendered and saved to:
```text
continuous-deployment/<cluster>/<namespace>/<application>/
```
This structure allows ArgoCD to track each environment and app cleanly.

The `overlay_dir` within this path will be used for `kustomize build`.

---

```markdown
## 🧩 Manifest Templating

Before running `kustomize build`, the workflow uses `envsubst` to template all manifest files in your specified `repo_path`. This is especially useful for dynamically generating Ingress or other environment-aware resources with values such as:
- `APPLICATION`
- `CLUSTER_NAME`
- `NAMESPACE`
- `DNS_ZONE`

For example, in an Ingress manifest, you could use:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ${APPLICATION}
  namespace: ${NAMESPACE}
spec:
  rules:
```
These values are injected by the workflow using the [`nowactions/envsubst`](https://github.com/nowactions/envsubst) action, based on your workflow inputs and values resolved from `env-map.yaml`.

> This approach allows you to reuse the same Kubernetes manifests across multiple environments, dynamically substituting names, hostnames, and namespaces to reflect the target cluster configuration.

---

## 📦 Artifacts

If the build fails (or for auditing), you will get:
- `built-kustomize-manifest`: final YAML from `kustomize build`
- `templated-source-manifests`: templated input files after `envsubst`

These artifacts are downloadable in the Actions UI.

---

## 🚀 ArgoCD Sync Behavior

- The workflow constructs the ArgoCD web UI domain as:
  ```
  https://<cluster>-argocd-argocd-web-ui.<dns_zone>
  ```

- It then calls the REST API to trigger a sync of the app:
  ```
  POST /api/v1/applications/<application>-<cluster>-<namespace>/sync
  ```

- It polls `.status.sync.status` for up to 2 minutes until the app is marked `Synced`

---

## 🧪 Example Call

```yaml
jobs:
  deploy:
    uses: gitopsmanager/k8s-deploy/.github/workflows/deploy.yaml@main
    with:
      environment: dev
      application: my-app
      namespace: my-namespace
      repo: my-org/my-app
      repo_path: manifests
      overlay_dir: overlays/dev
    secrets:
      ARGO_CD_ADMIN_USER: ${{ secrets.ARGO_CD_ADMIN_USER }}
      ARGO_CD_ADMIN_PASSWORD: ${{ secrets.ARGO_CD_ADMIN_PASSWORD }}

---

## 🏷 Versioning

We recommend pinning to a version tag once stable. Example:
```yaml
uses: gitopsmanager/k8s-deploy/.github/workflows/deploy.yaml@v1
```
Tags will be published using semantic versioning (`v1`, `v1.0.0`, etc.).

---

## 📝 License

© 2025 Affinity7 Consulting Ltd. This project is licensed under the [MIT License](LICENSE). You are free to use, modify, and distribute it with attribution.

