# Helm Commands Cheat Sheet (Most Used)

Helm is the package manager for Kubernetes. This cheat sheet lists the most commonly used commands for day-to-day work.

---

## 🔎 Helm Basics

### Check Helm version
```bash
helm version
```

### Get help
```bash
helm help
helm <command> --help
```

---

## 📦 Repositories

### Add a repo
```bash
helm repo add <repo-name> <repo-url>
```

### List repos
```bash
helm repo list
```

### Update repo index
```bash
helm repo update
```

### Search charts in repos
```bash
helm search repo <keyword>
```

---

## 🧾 Search & Inspect Charts

### Show chart details
```bash
helm show chart <chart>
```

### Show default values
```bash
helm show values <chart>
```

### Show full manifest templates (rendered locally)
```bash
helm template <release-name> <chart>
```

---

## ⬇️ Pull / Download Charts

### Pull a chart from repo
```bash
helm pull <chart-name> --version <version>
```

### Pull and untar
```bash
helm pull <chart-name> --version <version> --untar
```

### Pull from OCI registry (example)
```bash
helm pull oci://dipscloudacr.azurecr.io/helm/appointmentsms-webclient --version 1.7.26
```

---

## 🚀 Install / Upgrade / Rollback

### Install a release
```bash
helm install <release-name> <chart>
```

### Install with custom values file
```bash
helm install <release-name> <chart> -f values.yaml
```

### Install with namespace
```bash
helm install <release-name> <chart> -n <namespace> --create-namespace
```

### Upgrade a release
```bash
helm upgrade <release-name> <chart>
```

### Upgrade with values
```bash
helm upgrade <release-name> <chart> -f values.yaml
```

### Upgrade + install if not exists
```bash
helm upgrade --install <release-name> <chart>
```

### Rollback to previous revision
```bash
helm rollback <release-name> <revision>
```

---

## 📋 List Releases

### List all releases in current namespace
```bash
helm list
```

### List releases in a namespace
```bash
helm list -n <namespace>
```

### List all releases across namespaces
```bash
helm list -A
```

---

## 🔍 Get Release Info

### Show status
```bash
helm status <release-name>
```

### Show history
```bash
helm history <release-name>
```

### Get values (user-supplied)
```bash
helm get values <release-name>
```

### Get all values (including defaults)
```bash
helm get values <release-name> --all
```

### Get manifest (rendered YAML)
```bash
helm get manifest <release-name>
```

### Get notes
```bash
helm get notes <release-name>
```

---

## 🧪 Dry Run & Debugging

### Dry-run install
```bash
helm install <release-name> <chart> --dry-run
```

### Dry-run upgrade
```bash
helm upgrade <release-name> <chart> --dry-run
```

### Debug output
```bash
helm install <release-name> <chart> --debug --dry-run
```

### Lint a chart
```bash
helm lint <chart-directory>
```

---

## 🧹 Uninstall / Cleanup

### Uninstall a release
```bash
helm uninstall <release-name>
```

### Uninstall from a namespace
```bash
helm uninstall <release-name> -n <namespace>
```

---

## 🛠 Chart Development

### Create a new chart skeleton
```bash
helm create <chart-name>
```

### Package a chart
```bash
helm package <chart-directory>
```

### Validate chart dependencies
```bash
helm dependency list <chart-directory>
```

### Build dependencies (download subcharts)
```bash
helm dependency build <chart-directory>
```

### Update dependencies
```bash
helm dependency update <chart-directory>
```

---

## 🧩 Values & Overrides

### Set values inline
```bash
helm install <release-name> <chart> --set key=value
```

### Set nested values
```bash
helm install <release-name> <chart> --set image.tag=1.0.0
```

### Multiple values files
```bash
helm upgrade --install <release-name> <chart> -f values.yaml -f values-prod.yaml
```

---

## 🔐 OCI Registry Login (Azure ACR Example)

### Login to OCI registry
```bash
helm registry login <registry-url>
```

Example:
```bash
helm registry login dipscloudacr.azurecr.io
```

---

## 🧠 Useful Extras

### Render templates with specific namespace
```bash
helm template <release-name> <chart> -n <namespace>
```

### Render with values file
```bash
helm template <release-name> <chart> -f values.yaml
```

### Show release in JSON format
```bash
helm list -A -o json
```

---

✅ Tip: Most day-to-day usage revolves around:
- `helm pull`
- `helm install`
- `helm upgrade --install`
- `helm list`
- `helm status`
- `helm rollback`
- `helm uninstall`
- `helm template`
