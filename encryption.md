# Encryption Key Verification Commands

## Check if Private Key and Public Key Match

### 1. RSA Key Pair Verification

#### Compare key modulus values
```bash
# Extract modulus from private key
openssl rsa -in private_key.pem -modulus -noout

# Extract modulus from public key
openssl rsa -in public_key.pem -pubin -modulus -noout

# Extract modulus from certificate
openssl x509 -in certificate.crt -modulus -noout
```

#### Compare using MD5 hash
```bash
# Hash private key modulus
openssl rsa -in private_key.pem -modulus -noout | openssl md5

# Hash public key modulus
openssl rsa -in public_key.pem -pubin -modulus -noout | openssl md5

# Hash certificate modulus
openssl x509 -in certificate.crt -modulus -noout | openssl md5
```

#### One-liner comparison
```bash
# Compare private key and certificate
diff <(openssl rsa -in private_key.pem -modulus -noout) <(openssl x509 -in certificate.crt -modulus -noout)

# Compare private key and public key
diff <(openssl rsa -in private_key.pem -modulus -noout) <(openssl rsa -in public_key.pem -pubin -modulus -noout)
```

### 2. ECDSA Key Pair Verification

#### Extract public key from private key and compare
```bash
# Extract public key from ECDSA private key
openssl ec -in private_key.pem -pubout -out extracted_public.pem

# Compare with existing public key
diff extracted_public.pem public_key.pem

# View key details
openssl ec -in private_key.pem -text -noout
openssl ec -in public_key.pem -pubin -text -noout
```

### 3. Test with Encryption/Decryption

#### RSA encryption test
```bash
# Create test message
echo "test message" > test.txt

# Encrypt with public key
openssl rsautl -encrypt -inkey public_key.pem -pubin -in test.txt -out encrypted.bin

# Decrypt with private key
openssl rsautl -decrypt -inkey private_key.pem -in encrypted.bin -out decrypted.txt

# Compare original and decrypted
diff test.txt decrypted.txt
```

#### Sign and verify test
```bash
# Sign with private key
openssl dgst -sha256 -sign private_key.pem -out signature.bin test.txt

# Verify with public key
openssl dgst -sha256 -verify public_key.pem -signature signature.bin test.txt
```

## Kubernetes TLS Secret Verification

### Check TLS secret key pair matching
```bash
# Extract certificate and key from Kubernetes secret
kubectl get secret tls-secret -o jsonpath='{.data.tls\.crt}' | base64 -d > cert.crt
kubectl get secret tls-secret -o jsonpath='{.data.tls\.key}' | base64 -d > key.pem

# Verify they match
openssl x509 -in cert.crt -modulus -noout | openssl md5
openssl rsa -in key.pem -modulus -noout | openssl md5
```

### Verify certificate chain and key
```bash
# Check certificate details
openssl x509 -in cert.crt -text -noout

# Check private key details
openssl rsa -in key.pem -check -noout

# Verify certificate chain
openssl verify -CAfile ca.crt cert.crt
```

## Certificate Authority (CA) Verification

### Check if certificate was signed by CA
```bash
# Verify certificate against CA
openssl verify -CAfile ca.crt server.crt

# Check certificate chain
openssl x509 -in server.crt -text -noout | grep -A 5 "Issuer:"

# Verify certificate signature
openssl x509 -in server.crt -noout -issuer -subject
```

### Extract CA from certificate
```bash
# Get certificate chain
openssl s_client -connect hostname:443 -showcerts

# Extract intermediate certificates
openssl x509 -in cert.crt -text -noout | grep -A 10 "Certificate:"
```

## Common Verification Scripts

### Automated key pair verification script
```bash
#!/bin/bash
verify_keypair() {
    local private_key=$1
    local public_key=$2
    
    private_hash=$(openssl rsa -in "$private_key" -modulus -noout 2>/dev/null | openssl md5)
    public_hash=$(openssl rsa -in "$public_key" -pubin -modulus -noout 2>/dev/null | openssl md5)
    
    if [ "$private_hash" = "$public_hash" ]; then
        echo "✓ Keys match"
        return 0
    else
        echo "✗ Keys do not match"
        return 1
    fi
}

# Usage: verify_keypair private_key.pem public_key.pem
```

### Certificate and key verification script
```bash
#!/bin/bash
verify_cert_key() {
    local cert=$1
    local key=$2
    
    cert_hash=$(openssl x509 -in "$cert" -modulus -noout | openssl md5)
    key_hash=$(openssl rsa -in "$key" -modulus -noout | openssl md5)
    
    if [ "$cert_hash" = "$key_hash" ]; then
        echo "✓ Certificate and private key match"
        return 0
    else
        echo "✗ Certificate and private key do not match"
        return 1
    fi
}

# Usage: verify_cert_key certificate.crt private_key.pem
```

## PowerShell Equivalents (Windows)

### Using PowerShell with OpenSSL
```powershell
# Compare key modulus values
$privateModulus = & openssl rsa -in private_key.pem -modulus -noout
$publicModulus = & openssl rsa -in public_key.pem -pubin -modulus -noout

if ($privateModulus -eq $publicModulus) {
    Write-Host "Keys match" -ForegroundColor Green
} else {
    Write-Host "Keys do not match" -ForegroundColor Red
}
```

### PowerShell function for verification
```powershell
function Test-KeyPair {
    param(
        [string]$PrivateKeyPath,
        [string]$PublicKeyPath
    )
    
    $privateHash = & openssl rsa -in $PrivateKeyPath -modulus -noout | & openssl md5
    $publicHash = & openssl rsa -in $PublicKeyPath -pubin -modulus -noout | & openssl md5
    
    return $privateHash -eq $publicHash
}

# Usage: Test-KeyPair -PrivateKeyPath "private_key.pem" -PublicKeyPath "public_key.pem"
```

## Troubleshooting

### Common issues and solutions
```bash
# If key is encrypted, provide passphrase
openssl rsa -in encrypted_private_key.pem -modulus -noout -passin pass:your_passphrase

# Check key format and type
file private_key.pem
openssl rsa -in private_key.pem -text -noout | head -5

# Convert key formats if needed
openssl rsa -in private_key.pem -outform DER -out private_key.der
openssl rsa -in private_key.der -inform DER -outform PEM -out private_key_converted.pem
```

### Validate key integrity
```bash
# Check private key validity
openssl rsa -in private_key.pem -check -noout

# Check certificate validity
openssl x509 -in certificate.crt -noout -dates

# Check if certificate is expired
openssl x509 -in certificate.crt -checkend 0
```
