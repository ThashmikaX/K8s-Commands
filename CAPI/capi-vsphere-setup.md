# CAPI vSphere Cluster Setup Guide

## Prerequisites
- `kubectl`, `helm`, `clusterctl` installed
- Access to a vSphere environment
- Docker installed (for Kind)

---

## Step 1 — Initialize Kind Cluster

```bash
kind create cluster
```

---

## Step 2 — Install cert-manager

```bash
helm repo add jetstack https://charts.jetstack.io --force-update

helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set crds.enabled=true
```

---

## Step 3 — Create Namespace for CAPI Operator

```bash
kubectl create namespace capi-operator-system
```

---

## Step 4 — Create vSphere Credentials Secret

Create a file `vsphere-credentials.yaml` using the template below, replacing the placeholder values with your Base64-encoded credentials:

```bash
echo -n 'your-username' | base64
echo -n 'your-password' | base64
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: vsphere-credentials
  namespace: capi-operator-system
type: Opaque
data:
  VSPHERE_USERNAME: <BASE64-ENCODED-USERNAME>
  VSPHERE_PASSWORD: <BASE64-ENCODED-PASSWORD>
```

Apply the secret:

```bash
kubectl apply -f vsphere-credentials.yaml
```

---

## Step 5 — Install CAPI Operator

```bash
helm repo add capi-operator https://kubernetes-sigs.github.io/cluster-api-operator

helm install capi-operator capi-operator/cluster-api-operator \
  --create-namespace \
  -n capi-operator-system \
  --set infrastructure.vsphere.enabled=true \
  --set configSecret.name=<NAME-OF-SECRET> \
  --set configSecret.namespace=<NAMESPACE-OF-SECRET> \
  --wait \
  --timeout 90s
```

> Replace `<NAME-OF-SECRET>` with `vsphere-credentials` and `<NAMESPACE-OF-SECRET>` with `capi-operator-system` (or whichever namespace was used in Step 4).

---

## Step 6 — Set Environment Variables

Export all required CAPI/vSphere environment variables. Refer to the [clusterctl vSphere docs](https://github.com/kubernetes-sigs/cluster-api-provider-vsphere) for the full list. Key variables include:

```bash
export VSPHERE_USERNAME="username"
export VSPHERE_PASSWORD="password"
export VSPHERE_SERVER="url"
export VSPHERE_DATACENTER="datacenter"
export VSPHERE_DATASTORE="Datastore"
export VSPHERE_NETWORK="Network"
export VSPHERE_RESOURCE_POOL="*/Resources"
export VSPHERE_FOLDER="folder-name"
export VSPHERE_TEMPLATE="photon-5-kube-v1.33.9-SL"
export CONTROL_PLANE_ENDPOINT_IP="192.168.1.100"
export VSPHERE_TLS_THUMBPRINT="thumbprint"
export EXP_CLUSTER_RESOURCE_SET="true"
export VSPHERE_SSH_AUTHORIZED_KEY="$(cat ~/.ssh/id_rsa.pub)"
export VSPHERE_STORAGE_POLICY=""
export CPI_IMAGE_K8S_VERSION="v1.33.0"
export KUBERNETES_VERSION="1.33.9" 
# ... add all other required variables
```

---

## Step 7 — Create a Namespace for the New Cluster

Using a dedicated namespace per cluster is best practice:

```bash
kubectl create namespace <namespace>
```

---

## Step 8 — Generate Cluster YAML Configuration

```bash
clusterctl generate cluster --infrastructure vsphere:v1.15.2 <cluster-name> \
  -n <namespace> \
  --worker-machine-count 1 \
  > cluster.yaml
```

Review the generated `cluster.yaml` before applying.

---

## Step 9 — Apply Cluster Configuration

```bash
kubectl apply -f cluster.yaml
```

---

## Step 10 — Wait for Cluster Provisioning

Monitor the cluster creation progress:

```bash
kubectl get cluster -n <namespace> --watch
kubectl get machines -n <namespace>
```

The cluster is ready when the control plane shows `Provisioned` status.

---

## Step 11 — Retrieve the Cluster kubeconfig

```bash
kubectl get secret <cluster-name>-kubeconfig \
  -n <namespace> \
  -o jsonpath='{.data.value}' | base64 -d > kubeconfig
```

Verify connectivity to the new cluster:

```bash
kubectl --kubeconfig=kubeconfig get nodes
```

---

## Step 12 — Install Cilium CNI on the New Cluster

```bash
helm repo add cilium https://helm.cilium.io/

helm install cilium cilium/cilium \
  --kubeconfig=kubeconfig \
  --namespace kube-system \
  --set envoy.enabled=false
```

Wait for Cilium pods to be ready:

```bash
kubectl --kubeconfig=kubeconfig get pods -n kube-system --watch
```

---

## Step 13 — Scale Worker Machine Count

To adjust the number of worker nodes, edit the `MachineDeployment` resource:

```bash
kubectl edit machinedeployment <cluster-name> -n <namespace>
```

Update the `spec.replicas` field to the desired count, then save and exit. Alternatively, use `kubectl scale`:

```bash
kubectl scale machinedeployment <cluster-name> --replicas=<count> -n <namespace>
```

---

---

## Pivot — Move Management to Workload Cluster

Once the workload cluster is up (Steps 1–13), pivot the CAPI management plane into it so the Kind bootstrap cluster is no longer needed.

### Step 14 — Bootstrap the Workload Cluster as a Management Cluster

Run Steps 2–5 again, this time **targeting the workload cluster** using its kubeconfig:

```bash
# Point kubectl at the workload cluster for all commands in this section
export KUBECONFIG=./kubeconfig

# Install cert-manager
helm repo add jetstack https://charts.jetstack.io --force-update
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set crds.enabled=true

# Create namespace
kubectl create namespace capi-operator-system

# Apply vSphere credentials secret
kubectl apply -f vsphere-credentials.yaml

# Install CAPI operator
helm install capi-operator capi-operator/cluster-api-operator \
  --create-namespace \
  -n capi-operator-system \
  --set infrastructure.vsphere.enabled=true \
  --set configSecret.name=vsphere-credentials \
  --set configSecret.namespace=capi-operator-system \
  --wait \
  --timeout 90s
```

### Step 15 — Switch Context Back to the Management (Kind) Cluster

```bash
kubectl config use-context kind-kind
```

Verify you are on the correct cluster before moving:

```bash
kubectl config current-context
kubectl get clusters -A
```

### Step 16 — Move CAPI Objects to the Workload Cluster

```bash
clusterctl move -n <namespace> --to-kubeconfig ./kubeconfig
```

This transfers all CAPI custom resources (Cluster, Machines, etc.) from the Kind management cluster to the workload cluster. After this completes, the workload cluster manages itself.

### Step 17 — Verify the Move

Switch to the workload cluster and confirm all objects are present:

```bash
kubectl --kubeconfig=./kubeconfig get clusters -A
kubectl --kubeconfig=./kubeconfig get machines -A
```

The Kind bootstrap cluster can now be safely deleted:

```bash
kind delete cluster
```

---

## Summary

| Step | Action | Command/File |
|------|--------|-------------|
| 1 | Create Kind cluster | `kind create cluster` |
| 2 | Install cert-manager | `helm install cert-manager` |
| 3 | Create namespace | `kubectl create namespace capi-operator-system` |
| 4 | Create vSphere secret | `kubectl apply -f vsphere-credentials.yaml` |
| 5 | Install CAPI operator | `helm install capi-operator` |
| 6 | Set env variables | `export VSPHERE_*=...` |
| 7 | Create cluster namespace | `kubectl create namespace <namespace>` |
| 8 | Generate cluster YAML | `clusterctl generate cluster ...` |
| 9 | Apply cluster config | `kubectl apply -f cluster.yaml` |
| 10 | Wait for provisioning | `kubectl get cluster --watch` |
| 11 | Get kubeconfig | `kubectl get secret ... \| base64 -d` |
| 12 | Install Cilium CNI | `helm install cilium` |
| 13 | Scale worker nodes | `kubectl scale machinedeployment` |
| 14 | Bootstrap workload cluster (Steps 2–5) | `export KUBECONFIG=./kubeconfig` |
| 15 | Switch context to Kind | `kubectl config use-context kind-kind` |
| 16 | Move CAPI objects | `clusterctl move --to-kubeconfig ./kubeconfig` |
| 17 | Verify & delete Kind | `kind delete cluster` |