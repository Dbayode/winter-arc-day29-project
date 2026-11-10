# Day 29/90 — Kubernetes Application Deployment with Helm and Minikube

## Project Overview

This project demonstrates how to package, deploy, configure, upgrade, and roll back a containerised Nginx application using Helm on a local Kubernetes cluster managed by Minikube.

It forms part of my **90-Day DevOps Winter Arc Challenge**, focused on developing practical skills in Linux, containers, Kubernetes, cloud infrastructure, and deployment automation.

## Project Objectives

- Install and configure Helm on Ubuntu.
- Create a reusable Kubernetes Helm chart.
- Deploy Nginx to Minikube.
- Manage application configurations using `values.yaml`.
- Scale an application through a Helm upgrade.
- Restore a previous deployment using Helm rollback.
- Validate application availability.
- Document the deployment process.

## Tools and Technologies

| Technology | Purpose |
|---|---|
| Ubuntu Linux | Development environment |
| Minikube | Local Kubernetes cluster |
| VirtualBox | Virtualisation driver |
| Kubernetes | Container orchestration |
| Helm | Kubernetes package management |
| Nginx | Web server application |
| kubectl | Kubernetes administration |
| Git and GitHub | Version control and documentation |

## Architecture

The project uses the following architecture:

```text
Ubuntu Host
    |
    v
Minikube (VirtualBox)
    |
    v
Kubernetes Cluster
    |
    v
Namespace: devops-lab
    |
    v
Helm Release: nginx-release
    |
    +---- Nginx Pod 1
    |
    +---- Nginx Pod 2
    |
    v
Kubernetes ClusterIP Service
    |
    v
kubectl port-forward
    |
    v
Browser: http://localhost:8080
```

## 1. Verify the Kubernetes Environment

```bash
minikube status
kubectl get nodes
kubectl config current-context
```

The Kubernetes context should be `minikube`, and the cluster node should report `Ready`.

## 2. Install Helm

Helm was installed using the official installation script.

```bash
curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 -o get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh
```

Verify:

```bash
helm version
```

## 3. Create a Helm Chart

```bash
helm create nginx-chart
```

The generated chart contains:

- `Chart.yaml` — chart metadata.
- `values.yaml` — configurable application values.
- `templates/deployment.yaml` — Kubernetes Deployment template.
- `templates/service.yaml` — Kubernetes Service template.
- `templates/serviceaccount.yaml` — ServiceAccount template.
- `templates/_helpers.tpl` — reusable template definitions.

## 4. Configure the Application

Update the following settings in `nginx-chart/values.yaml`:

```yaml
replicaCount: 2

image:
  repository: nginx
  pullPolicy: IfNotPresent
  tag: "stable"
```

These settings configure two Nginx replicas and the container image.

## 5. Validate the Helm Chart

```bash
cd nginx-chart

helm lint .
helm template nginx-release .
```

**Validation result:**

```text
1 chart(s) linted, 0 chart(s) failed
```

The chart passed validation, and Helm successfully rendered the Kubernetes manifests.

## 6. Deploy Nginx Using Helm

```bash
helm install nginx-release . \
  --namespace devops-lab \
  --create-namespace \
  --wait \
  --timeout 5m
```

Verify the release:

```bash
helm list -n devops-lab
kubectl get pods -n devops-lab
kubectl get deployments -n devops-lab
kubectl get svc -n devops-lab
```

**Verified deployment result:**

- Release: `nginx-release`
- Namespace: `devops-lab`
- Status: `deployed`
- Running pods: `2/2`
- Pod restarts: `0`

## 7. Test Application Accessibility

Use Kubernetes port forwarding:

```bash
kubectl port-forward -n devops-lab \
  svc/nginx-release-nginx-chart 8080:80
```

Open:

http://localhost:8080

**Result:** The Nginx welcome page was successfully displayed in the browser.

This confirmed that the application was accessible through the Kubernetes Service.

## 8. Perform a Helm Upgrade

Modify `nginx-chart/values.yaml`:

```yaml
replicaCount: 3
```

Apply the upgrade:

```bash
helm upgrade nginx-release . \
  --namespace devops-lab \
  --wait \
  --timeout 5m
```

Verify:

```bash
kubectl get pods -n devops-lab
helm history nginx-release -n devops-lab
```

The expected outcome is three running Nginx replicas and a new Helm release revision.

## 9. Perform a Helm Rollback

Restore the initial deployment configuration:

```bash
helm rollback nginx-release 1 \
  --namespace devops-lab \
  --wait \
  --timeout 5m
```

Verify:

```bash
kubectl get deployments -n devops-lab
helm history nginx-release -n devops-lab
```

The expected outcome is a return to two replicas and a new release revision recording the rollback.

## 10. Troubleshooting

### Helm package not found using APT

**Issue:**

```text
E: Unable to locate package helm
```

**Resolution:** Installed Helm using its official installation script.

### Incorrect Helm template syntax

**Issue:**

```text
Error: expected at most two arguments, unexpected arguments: .
```

**Resolution:** Corrected the command to:

```bash
helm template nginx-release .
```

### Accessing a ClusterIP Service

A ClusterIP Service is intended for internal cluster access.

**Solution:** Used `kubectl port-forward` to access Nginx from the Ubuntu browser without exposing the service publicly.

## 11. Evidence and Screenshots

The `screenshots/` directory is intended to contain evidence of:

1. Successful Helm chart validation.
2. Deployed Helm release and running pods.
3. Nginx welcome page in the browser.
4. Helm upgrade and replica scaling.
5. Helm rollback and release history.

Screenshots should be added after verifying each operation.

## 12. Key Lessons Learned

- Helm simplifies Kubernetes application management.
- Helm charts provide reusable deployment templates.
- Configuration values can be maintained separately from templates.
- Helm upgrades allow application configuration changes to be applied consistently.
- Helm rollback provides a straightforward way to restore previous release configurations.
- Minikube offers a practical environment for Kubernetes experimentation without requiring cloud infrastructure.

## Conclusion

This project demonstrates the use of Helm to manage a containerised Nginx application on Kubernetes.

The deployment and browser-access tests were successful. The upgrade and rollback exercises demonstrate Helm's release-management workflow and should be supported with command output and screenshots in the final project submission.

**90-Day DevOps Winter Arc**

Learn → Build → Deploy → Automate → Improve.
