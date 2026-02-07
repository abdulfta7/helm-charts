# Helm Charts CI Pipeline

This project contains a Helm chart for a DevOps application with an automated CI/CD pipeline that verifies, lints, scans, and packages the chart.

## 📁 Project Structure

```
helm-charts/
├── devops-app/                 # Main Helm chart
│   ├── Chart.yaml             # Chart metadata
│   ├── values.yaml            # Default configuration values
│   └── templates/             # Kubernetes resource templates
│       ├── backend-*          # Backend services (deployment, config, secrets, etc.)
│       ├── db-*              # Database services (PostgreSQL, Redis)
│       ├── frontend-*        # Frontend services
│       └── other-*           # Additional resources (ingress, namespaces, policies, etc.)
├── .github/workflows/
│   └── helm-ci.yml           # GitHub Actions CI pipeline
└── README.md                  # This file
```

## 🚀 What This Project Does

The Helm chart defines a complete DevOps application with:
- **Backend services** - Application deployment, configuration, and secrets
- **Databases** - PostgreSQL and Redis stateful sets
- **Frontend services** - Web application deployment
- **Networking** - Ingress rules, network policies, and services
- **Security** - TLS secrets and image pull secrets
- **Resource management** - Horizontal Pod Autoscaling and resource quotas

## ⚙️ CI/CD Pipeline (GitHub Actions)

The workflow automatically runs on every push to `main` branch with 2 sequential jobs:

### Job 1: **Verify and Lint** ✅
- Verifies the Helm chart structure
- Checks that required files exist:
  - `Chart.yaml`
  - `values.yaml`
  - `templates/` directory with YAML files
- Validates YAML syntax
- Runs Helm lint to check for best practices

**Status**: Must pass before proceeding to Job 2

### Job 2: **Scan and Package** 📦
(Only runs if Job 1 passes)
- Renders Helm templates with values
- Validates rendered manifests with `kubeval` against Kubernetes v1.26
- Packages the chart as a `.tgz` file

## 🔍 What is Kubeval?

**Kubeval** is a command-line tool that validates Kubernetes YAML manifests against the official Kubernetes schema. It ensures your Kubernetes resources are correctly structured and follow Kubernetes API standards.

### How Kubeval Works in This Project

1. **Renders Templates**: First, Helm converts your template files (with variables like `{{ .Values.name }}`) into actual Kubernetes YAML
2. **Validates Against Schema**: Kubeval checks each rendered YAML file against Kubernetes v1.26 schema
3. **Reports Issues**: If any manifest doesn't match the schema, it reports errors

### What Kubeval Checks

✅ Resource kind and API version are valid  
✅ Required fields are present  
✅ Field types are correct (string, integer, boolean, etc.)  
✅ Field values follow constraints (e.g., port numbers must be 1-65535)  
✅ Resource structure matches Kubernetes standards  

### Example

**Invalid manifest** (Kubeval will catch this):
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: my-container
      image: nginx
      ports:
        - containerPort: 99999  # ❌ Invalid! Port must be 1-65535
```

**Valid manifest** (Kubeval will pass):
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: my-container
      image: nginx
      ports:
        - containerPort: 8080  # ✅ Valid port number
```

### Kubeval in Our Pipeline

In the workflow, kubeval:
- Validates against **Kubernetes v1.26** schema
- Uses `--ignore-missing-schemas` flag to skip resources without schemas
- Validates all rendered template files in the `templates/` directory
- Ensures your Helm chart can be safely deployed to Kubernetes



## 📝 How to Use

### 1. **View the Workflow**
Check `.github/workflows/helm-ci.yml` to see the CI pipeline configuration.

### 2. **Make Changes**
Edit files in the `devops-app/` directory:
- Modify `Chart.yaml` for chart metadata
- Update `values.yaml` for configuration changes
- Edit templates for Kubernetes resource definitions

### 3. **Commit and Push**
Push changes to the `main` branch to trigger the CI pipeline automatically.

```bash
git add .
git commit -m "Update Helm chart"
git push origin main
```

### 4. **Check Pipeline Results**
Go to GitHub Actions tab to see:
- ✅ Job logs for verification and linting
- 📦 Packaged Helm chart artifact

## 🔍 Key Files

| File | Purpose |
|------|---------|
| `devops-app/Chart.yaml` | Helm chart metadata (name, version, description) |
| `devops-app/values.yaml` | Default configuration values for all components |
| `devops-app/templates/` | Kubernetes resource templates |
| `.github/workflows/helm-ci.yml` | Automated CI/CD pipeline configuration |

## 📋 Requirements

- Kubernetes v1.26+
- Helm v3.12.0+
- Docker images for frontend, backend, and databases
- Valid TLS certificates (for ingress)
- Image pull credentials (if using private registries)

## ✨ Features

✅ Automated validation on every commit  
✅ Helm linting for best practices  
✅ Kubernetes manifest validation  
✅ Automatic chart packaging  
✅ Modular template structure  
✅ Security policies and network isolation  
✅ Resource quotas and autoscaling  

## 🛠️ Troubleshooting

If the CI pipeline fails:

1. **Verify Job**: Check that all required files exist and YAML is valid
2. **Lint Job**: Run locally: `helm lint devops-app`
3. **Scan Job**: Run locally: `helm template devops-app devops-app`
4. **Check logs**: View GitHub Actions workflow logs for detailed error messages