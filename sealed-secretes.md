# Kubernetes Sealed Secrets Commands

This document contains useful commands for working with Kubernetes sealed secrets using kubeseal.

## Getting Public Key

Get the public key from the sealed-secrets controller:

```bash
kubeseal --fetch-cert --controller-name=sealed-secrets --controller-namespace=secrets > /tmp/sealed-secrets-public.pem
```

## Encrypting Text

Encrypt text using the public key:

```bash
echo -n "vm_monitor" | kubeseal --raw --from-file=/dev/stdin --namespace=monitoring --name=vsphere-secret --cert=/tmp/sealed-secrets-public.pem
```

## Managing Secrets

### List Sealed Secrets

List all sealed secrets in the secrets namespace:

```bash
kubectl -n secrets get secret | grep -i sealed
```

### Extract Private Key

Extract private key from secrets:

```bash
kubectl -n secrets get secret <SECRET_NAME> -o jsonpath='{.data.tls\.key}' | base64 --decode > /tmp/sealed-secrets-tls.key
```

### Decode Base64 Password

Decode base64 encoded password:

```bash
echo dm1fbW9uaXRvcg== | base64 --decode
```

### Get Decrypted Secret

Get secret in decrypted form:

```bash
kubectl get secret -n monitoring <SECRET_NAME> -o yaml
```

## Creating and Applying Sealed Secrets

### Generate Secret File

Create a secret file (dry-run):

```bash
kubectl create secret generic database -n default --from-literal=DB_PASSWORD=password123 --dry-run=client -o yaml > secret.yaml
```

### Generate Sealed Secret File

Convert regular secret to sealed secret using controller (when connected to cluster):

```bash
kubeseal --controller-name sealed-secrets --controller-namespace secrets --format yaml < secret.yaml > sealed-secret.yaml
```

### Generate Sealed Secret File Using Public Key (.pem)

Convert regular secret to sealed secret using the public key file (offline mode):

```bash
kubeseal --format yaml --cert /tmp/sealed-secrets-public.pem < secret.yaml > sealed-secret.yaml
```

**Note:** Using the .pem public key allows you to create sealed secrets without being connected to the Kubernetes cluster.

### Apply Sealed Secret

Apply the sealed secret to the cluster:

```bash
kubectl apply -f sealed-secret.yaml
```

## Decrypting Sealed Secrets

### Decrypt Using Private Key

If you have a sealed secret YAML file and the private key, you can decrypt it offline:

```bash
kubeseal --recovery-unseal --recovery-private-key /tmp/sealed-secrets-tls.key < sealed-secret.yaml
```

### Alternative Method with Output to File

Decrypt and save the output to a file:

```bash
kubeseal --recovery-unseal --recovery-private-key /tmp/sealed-secrets-tls.key < sealed-secret.yaml > decrypted-secret.yaml
```

### Decrypt Specific Field

If you only want to decrypt a specific field from the sealed secret:

```bash
kubeseal --recovery-unseal --recovery-private-key /tmp/sealed-secrets-tls.key --recovery-unseal-field <FIELD_NAME> < sealed-secret.yaml
```

**Note:** The private key should be the same one used by the sealed-secrets controller that encrypted the secret.