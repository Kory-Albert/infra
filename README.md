
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

## Contributing

Pull requests are welcome.  
- Follow the existing style and folder conventions.  
- Run the CI tests (`terraform fmt`, `ansible-lint`, etc.) before pushing.  

---

## License

MIT © 2026 Kory Albert
Feel free to fork and adapt for your own environment.