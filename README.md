# Argo Workflows Dev

## Environment

- Control-Plane
  - OS: Ubuntu Server 20.04.2 LTS
  - K8s: v1.20.9
  - CRI: CRI-O v1.20.3
- Worker (NVIDIA Jetson Nano)
  - OS: Ubuntu 18.04.5 LTS
  - Jetpack: 4.5.1
  - K8s: v1.20.9
  - CRI: containerd v1.5.2
- Kubernetes Resources
  - Helm: v3.6.3
  - argo-helm: 0.2.12
  - argo-workflows: v3.0.7

## Installation

### Installer

```bash
wget "https://github.com/argoproj/argo-helm/archive/refs/tags/argo-workflows-0.2.12.tar.gz"

tar -zxvf ./argo-workflows-0.2.12.tar.gz

ln -s argo-helm-argo-workflows-0.2.12 argo-helm-argo-workflows
```

### CLI

```bash
# Download the binary
curl -sLO https://github.com/argoproj/argo-workflows/releases/download/v3.1.2/argo-linux-amd64.gz

# Unzip
gunzip argo-linux-amd64.gz

# Make binary executable
chmod +x argo-linux-amd64

# Move binary to path
mv ./argo-linux-amd64 /usr/local/bin/argo

# Test installation
argo version
```

### Override Chart Values

```bash
cat > argo-workflow-override.yaml << EOF
---
workflow:
  serviceAccount:
    create: true
    name: argo-workflow
controller:
  containerRuntimeExecutor: k8sapi  # use emissary ?
  workflowNamespaces:
    - argo
server:
  serviceType: NodePort
EOF
```

### Install Argo-workflows Using Helm

```bash
kubectl create ns argo

helm install -f argo-workflow-override.yaml argo-workflow argo-helm-argo-workflows/charts/argo-workflows
```

### Grant Admin Permission to the argo-workflow

```bash
kubectl create rolebinding default-admin --clusterrole=admin --serviceaccount=argo:argo-workflow -n argo
```

### Verification

```bash
argo submit -n argo --serviceaccount argo-workflow --watch https://raw.githubusercontent.com/argoproj/argo-workflows/master/examples/hello-world.yaml
```

