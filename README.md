# Infrastructure as Code (IaC)

Declarative, repeatable infrastructure powered by **Terraform**, **Ansible** and **Kubernetes**.

> *All infrastructure is self‑documented, version‑controlled and quick to redeploy.*

---

## Repo layout

| Folder | Purpose |
|--------|---------|
| `terraform/` | Terraform modules, state files and bootstrap scripts |
| `ansible/`   | Ansible playbooks, roles, inventory and variables |
| `k8s/`       | Kubernetes manifests (YAML), Helm charts, kustomize overlays |

---

## Quick start

1. **Install prerequisites**

    ```bash
    # Linux / macOS
    brew install terraform ansible kubectl helm
    ```

2. **Clone the repo**

    ```bash
    git clone https://github.com/Kory-Albert/infra.git
    cd infra
    ```

3. **Provision infrastructure**

    ```bash
    cd terraform
    terraform init
    terraform apply   # (use -auto-approve for non‑interactive runs)
    ```

4. **Configure the cluster**

    ```bash
    cd ../ansible
    ansible-playbook site.yml -i inventory.ini
    ```

5. **Deploy workloads**

    ```bash
    cd ../k8s
    kubectl apply -f .
    # or helm/kustomize as preferred
    ```

---

## Kubernetes Environment Details

### Cluster Overview

This environment features a production-grade Kubernetes cluster built with:
- **3 Talos Linux control plane nodes** (immutable, API-driven)
- **5 Talos Linux worker nodes**, including:
  - 4 general-purpose workers
  - 1 GPU-accelerated worker
- **Talos Linux** provides secure, minimal nodes managed via API and `talosctl`
- All infrastructure is provisioned and managed through Terraform and Ansible

### Key Components

- **Container Runtime**: containerd (Talos default)
- **CNI**: Calico for networking and network policies
- **CSI**: Longhorn for distributed, resilient block storage
- **Gateway**: Envoy Gateway for modern, Kubernetes-native ingress
- **Database**: CloudNative-PG for PostgreSQL clusters with streaming replication
- **GPU Support**: NVIDIA GPU Operator and device plugin for AI workloads
- **Certificate Management**: cert-manager with Let's Encrypt integration
- **Monitoring**: Prometheus/Grafana/loki stack (deployed via Helm)

### Cluster Access

To interact with the Talos-managed cluster:

1. **Access Talos Nodes**:
   ```bash
   # Install talosctl if needed
   brew install talosctl
   
   # Configure talosctl for your cluster
   talosctl config endpoint add <control-plane-ip>
   talosctl config node add <control-plane-ip>
   ```

2. **Get kubeconfig**:
   ```bash
   talosctl kubeconfig .
   export KUBECONFIG=$(pwd)/kubeconfig
   ```

3. **Verify access**:
   ```bash
   kubectl get nodes
   kubectl get pods -A
   talosctl dashboard  # For node health
   ```

### Workload Deployment Structure

Applications follow a cloud-native structure in the `k8s/` directory:

```
k8s/
├── <application-name>/
│   ├── deployment.yaml           # Deployment or StatefulSet
│   ├── service.yaml              # ClusterIP service
│   ├── gateway.yaml              # Envoy Gateway HTTPRoute/TCPRoute
│   ├── pvc.yaml                  # PersistentVolumeClaim (Longhorn)
│   ├── secret.yaml               # Sensitive data (SOPS/SealedSecrets)
│   ├── configmap.yaml            # Non-sensitive configuration
│   ├── networkpolicy.yaml        # Calico network policies
│   └── hpa.yaml                  # HorizontalPodAutoscaler (optional)
```

**AI Workload Examples**:
- **Vision Models**: TensorRT/PyTorch services with GPU requests
- **LLM Automation**: vLLM/TGI deployments with tensor parallelism
- **Image Generation**: Stable Diffusion workers with GPU affinity
- **Batch Jobs**: Argo Workflows for preprocessing pipelines

### Environment Add-ons

- **Longhorn CSI**: Provides resilient block storage with snapshots/backups
  - StorageClass: `longhorn` (default) and `longhorn-replicated`
  - Supports volume expansion and backup schedules

- **Envoy Gateway**: Modern ingress replacing traditional Ingress
  - Supports Gateway API (HTTPRoute, TCPRoute, TLSRoute)
  - Integrated with cert-manager for automatic TLS
  - Rate limiting, request/response transformation

- **CloudNative-PG**: PostgreSQL operator for HA clusters
  - Streaming replication with automatic failover
  - Backup/restore via volume snapshots or object storage
  - Connection pooling and monitoring

- **NVIDIA GPU Support**:
  - GPU Operator installed via Helm
  - Node Feature Discovery labels GPU nodes
  - Workloads request GPUs via `resources.limits.nvidia.com/gpu`

### Extending the Environment

To add new applications (especially AI workloads):

1. **Create application directory**:
   ```bash
   mkdir -p k8s/<your-app>
   ```

2. **Copy base manifests** from a similar application:
   ```bash
   cp -r k8s/bookstack/* k8s/<your-app>/  # Example using existing template
   ```

3. **Customize for your workload**:
   - **AI/ML workloads**: Add GPU requests:
     ```yaml
     resources:
       limits:
         nvidia.com/gpu: 1  # or fractional for MIG
       requests:
         nvidia.com/gpu: 1
     ```
   - **Storage**: Choose Longhorn StorageClass:
     ```yaml
     storageClassName: longhorn
     ```
   - **Networking**: Use Envoy Gateway API:
     ```yaml
     apiVersion: gateway.networking.k8s.io/v1
     kind: HTTPRoute
     metadata:
       name: <your-app>-http
     spec:
       parentRefs:
       - name: envoy
         namespace: gateway-system
       hostnames:
       - <your-app>.yourdomain.com
       rules:
       - matches:
         - path:
             type: PathPrefix
             value: /
         forwardTo:
         - serviceName: <your-app>
           port: 80
     ```
   - **Secrets**: Use SOPS or SealedSecrets for GitOps-friendly secret management

4. **Test and Deploy**:
   ```bash
   # Dry run first
   kubectl apply -f k8s/<your-app>/ --dry-run=client
   
   # Apply when ready
   kubectl apply -f k8s/<your-app>/
   ```

### Important Notes

- **Talos Specifics**: Nodes are immutable - modify via `talosctl` or machine configs
- **GPU Worker**: Ensure workloads use `nodeSelector` or `affinity` for GPU node:
  ```yaml
  nodeSelector:
    node.kubernetes.io/instance-type: gpu-worker
  # or
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: feature.node.kubernetes.io/custom-gpu
            operator: In
            values:
            - "true"
  ```
- **Storage**: Longhorn volumes are replicated - ensure sufficient disk space on workers
- **Networking**: Calico policies default to deny - explicitly allow required traffic
- **GPU Sharing**: For fractional GPUs, configure MIG or time-slicing via GPU Operator
- **Monitoring**: Use `kubectl top nodes` and `talosctl dashboard` for node metrics
- **Troubleshooting**: 
  - Talos: `talosctl -n <node> logs kubelet`
  - Kubernetes: `kubectl describe pod <pod> -n <namespace>`
  - Longhorn: `longhornctl` or via UI at `http://<longhorn-ip>:8000`

---

## Contributing

Pull requests are welcome.
- Follow the existing style and folder conventions.
- Run the CI tests (`terraform fmt`, `ansible-lint`, etc.) before pushing.

---

## License

MIT © 2026 Kory Albert
Feel free to fork and adapt for your own environment.