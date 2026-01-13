# Basic kubectl Commands Reference

## Cluster Information

```bash
# Display cluster information
kubectl cluster-info

# Show the current context
kubectl config current-context

# List all contexts
kubectl config get-contexts

# Switch context
kubectl config use-context <context-name>

# View cluster nodes
kubectl get nodes
```

## Working with Pods

```bash
# List all pods in the current namespace
kubectl get pods

# List all pods in all namespaces
kubectl get pods --all-namespaces
kubectl get pods -A

# Get detailed information about a pod
kubectl describe pod <pod-name>

# View pod logs
kubectl logs <pod-name>

# View logs of a specific container in a pod
kubectl logs <pod-name> -c <container-name>

# Stream logs in real-time
kubectl logs -f <pod-name>

# Execute a command in a pod
kubectl exec <pod-name> -- <command>

# Get an interactive shell in a pod
kubectl exec -it <pod-name> -- /bin/bash

# Delete a pod
kubectl delete pod <pod-name>
```

## Working with Deployments

```bash
# List all deployments
kubectl get deployments

# Create a deployment
kubectl create deployment <name> --image=<image-name>

# Get detailed information about a deployment
kubectl describe deployment <deployment-name>

# Scale a deployment
kubectl scale deployment <deployment-name> --replicas=<number>

# Update deployment image
kubectl set image deployment/<deployment-name> <container-name>=<new-image>

# Rollout status
kubectl rollout status deployment/<deployment-name>

# Rollout history
kubectl rollout history deployment/<deployment-name>

# Undo a rollout
kubectl rollout undo deployment/<deployment-name>

# Delete a deployment
kubectl delete deployment <deployment-name>
```

## Working with Services

```bash
# List all services
kubectl get services
kubectl get svc

# Describe a service
kubectl describe service <service-name>

# Expose a deployment as a service
kubectl expose deployment <deployment-name> --port=<port> --type=<type>

# Delete a service
kubectl delete service <service-name>
```

## Working with Namespaces

```bash
# List all namespaces
kubectl get namespaces
kubectl get ns

# Create a namespace
kubectl create namespace <namespace-name>

# Set default namespace for current context
kubectl config set-context --current --namespace=<namespace-name>

# Delete a namespace
kubectl delete namespace <namespace-name>
```

## Working with ConfigMaps and Secrets

```bash
# List ConfigMaps
kubectl get configmaps
kubectl get cm

# Create a ConfigMap from literal values
kubectl create configmap <name> --from-literal=<key>=<value>

# Create a ConfigMap from a file
kubectl create configmap <name> --from-file=<path-to-file>

# List Secrets
kubectl get secrets

# Create a generic secret
kubectl create secret generic <name> --from-literal=<key>=<value>

# Describe a secret
kubectl describe secret <secret-name>
```

## Applying and Managing Resources

```bash
# Apply a configuration from a file
kubectl apply -f <filename.yaml>

# Apply all YAML files in a directory
kubectl apply -f <directory>/

# Delete resources defined in a file
kubectl delete -f <filename.yaml>

# Get resources defined in a file
kubectl get -f <filename.yaml>
```

## Viewing Resources

```bash
# Get resources with wide output (more details)
kubectl get <resource> -o wide

# Get resources in YAML format
kubectl get <resource> <name> -o yaml

# Get resources in JSON format
kubectl get <resource> <name> -o json

# Watch resources for changes
kubectl get <resource> --watch
kubectl get <resource> -w

# List all resource types
kubectl api-resources
```

## Labels and Selectors

```bash
# Show labels
kubectl get pods --show-labels

# Filter by label
kubectl get pods -l <key>=<value>

# Add a label to a resource
kubectl label <resource> <name> <key>=<value>

# Remove a label from a resource
kubectl label <resource> <name> <key>-
```

## Debugging and Troubleshooting

```bash
# Get events in the current namespace
kubectl get events

# Sort events by timestamp
kubectl get events --sort-by=.metadata.creationTimestamp

# Check resource usage
kubectl top nodes
kubectl top pods

# Port forward to a pod
kubectl port-forward <pod-name> <local-port>:<pod-port>

# Copy files to/from a pod
kubectl cp <local-path> <pod-name>:<pod-path>
kubectl cp <pod-name>:<pod-path> <local-path>
```

## Common Resource Shortcuts

| Resource Type | Short Name |
|--------------|------------|
| pods | po |
| services | svc |
| deployments | deploy |
| replicasets | rs |
| namespaces | ns |
| configmaps | cm |
| persistentvolumes | pv |
| persistentvolumeclaims | pvc |
| nodes | no |

## Useful Flags

- `-n <namespace>` - Specify namespace
- `--all-namespaces` or `-A` - All namespaces
- `-o wide` - Additional details
- `-o yaml` - Output in YAML format
- `-o json` - Output in JSON format
- `--watch` or `-w` - Watch for changes
- `--dry-run=client` - Test without executing
- `-f <file>` - Specify a file
- `-l <label>` - Filter by label selector

## Quick Examples

```bash
# Create a nginx deployment with 3 replicas
kubectl create deployment nginx --image=nginx --replicas=3

# Expose the deployment as a NodePort service
kubectl expose deployment nginx --port=80 --type=NodePort

# Scale the deployment
kubectl scale deployment nginx --replicas=5

# Update the image
kubectl set image deployment/nginx nginx=nginx:1.21

# Check rollout status
kubectl rollout status deployment/nginx
```
