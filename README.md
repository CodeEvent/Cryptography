# Cryptography — COMP09106

**Module:** COMP09106 Cryptography  
**Degree:** BEng (Hons) Cyber Security, University of the West of Scotland  
**Instructor:** Dr. Althaff Mohideen  
**Grade:** First-class band  

A 9-week practical cryptography module covering symmetric and asymmetric encryption, Public Key Infrastructure, certificate lifecycle management, hashing, digital signatures, and IKEv2 VPN deployment. All labs completed on Linux (Kali) using OpenSSL, GPG, and strongSwan.

---

## Labs

| Week | Topic | Key Tools |
|------|-------|-----------|
| 1 | Getting started — CrypTool 2 and classical cipher introduction | CrypTool 2, Cryptophane |
| 2 | Classical cipher cryptanalysis — Caesar and monoalphabetic substitution | CrypTool 2 |
| 3 | Symmetric encryption — AES-256-CBC, DES, Base64 encoding | OpenSSL |
| 4 | Self-signed certificates — RSA key generation, CSR, X.509 signing | OpenSSL |
| 5 | RSA asymmetric encryption — key pair generation, encrypt/decrypt | OpenSSL |
| 7 | Public Key Infrastructure — root CA, X.509 certificate lifecycle, HTTPS deployment | OpenSSL |
| 8 | Hashing and digital signatures — SHA-256, SHA-3, GPG, OpenSSL dgst, password cracking | OpenSSL, GPG, John the Ripper |
| 9 | IKEv2 IPsec VPN — 4096-bit PKI, strongSwan server configuration end to end | strongSwan, OpenSSL PKI |

---

## Skills Demonstrated

**PKI and Certificate Lifecycle**
- Built a complete root Certificate Authority with OpenSSL (custom `openssl.cnf`, `index.txt`, `serial` management)
- Issued and signed X.509 certificates from a client CSR; deployed the signed certificate on a live HTTPS server
- Diagnosed and explained certificate trust chain failure in Firefox; imported CA certificate into browser trust store
- Generated 4096-bit PKI root CA and server certificate using `strongswan-pki` for IKEv2 VPN

**Asymmetric Cryptography**
- Generated 2048-bit RSA private keys with and without AES-256 passphrase protection
- Extracted public keys; encrypted and decrypted messages using RSA public/private key pairs
- Created Certificate Signing Requests (CSR) and signed them with both a CA and a self-signing key

**Symmetric Encryption**
- Encrypted and decrypted files using AES-256-CBC and DES via OpenSSL
- Applied Base64 encoding for safe transport of ciphertext
- Compared cipher mode security: CBC versus ECB

**Hashing**
- Produced and compared SHA-256 and SHA-3-256/SHA-3-512 digests; observed avalanche effect
- Identified hashing algorithms from hash values using `hash-identifier`
- Cracked MD5 password hashes using John the Ripper with a targeted wordlist (CUPP)

**Digital Signatures**
- Created and verified detached digital signatures using GPG (`--detach-sig`, `--verify`)
- Created and verified detached signatures using OpenSSL `dgst` with RSA private/public key pair

**IKEv2 VPN**
- Configured kernel packet forwarding and deployed strongSwan on Ubuntu 20.04
- Generated 4096-bit root CA and signed VPN server certificate
- Configured `/etc/ipsec.conf` with IKEv2 tunnel, EAP-MSCHAPv2 authentication, cipher suites (ChaCha20-Poly1305, AES-256-GCM), and client IP pool

---

## Tools and Commands Reference

### Symmetric Encryption (Week 3)

```bash
# AES-256-CBC encryption
openssl enc -aes-256-cbc -e -in letter_to_granny.txt -out letter_to_granny.enc -base64

# AES-256-CBC decryption
openssl enc -aes-256-cbc -d -in letter_to_granny.enc -out decrypted.txt -base64

# DES encryption
openssl enc -des -e -in letter_to_granny.txt -out letter_des.enc -base64
```

### Self-Signed Certificates (Week 4)

```bash
# Generate 2048-bit RSA private key (no passphrase)
openssl genrsa -out private.pem 2048

# Generate with AES-256 passphrase protection
openssl genrsa -aes256 -out private.pem 2048

# Generate a CSR
openssl req -new -key private.pem -out signreq.csr

# Self-sign the CSR (valid 365 days)
openssl x509 -req -days 365 -in signreq.csr -signkey private.pem -out certificate.pem

# Direct self-signed certificate (skips intermediate CSR)
openssl req -x509 -newkey rsa:2048 -keyout private.pem -out certificate.pem -days 365

# Verify certificate contents
openssl x509 -text -noout -in certificate.pem
```

### RSA Asymmetric Encryption (Week 5)

```bash
# Generate 2048-bit RSA private key
openssl genpkey -algorithm RSA -out private_key.pem -pkeyopt rsa_keygen_bits:2048

# Extract public key
openssl rsa -pubout -in private_key.pem -out public_key.pem

# Encrypt a file with the public key
openssl rsautl -encrypt -inkey public_key.pem -pubin -in message.txt -out encrypted_message.bin

# Decrypt with the private key
openssl rsautl -decrypt -inkey private_key.pem -in encrypted_message.bin -out decrypted.txt
```

### PKI and Root CA (Week 7)

```bash
# Set up CA directory structure
mkdir -p ~/lab09CA/{certs,crl,newcerts,private}
touch ~/lab09CA/index.txt
echo "0001" > ~/lab09CA/serial

# Generate self-signed root CA certificate
openssl req -new -x509 -keyout private/cakey.pem -out cacert.pem -config /etc/ssl/openssl.cnf

# Verify the CA certificate
openssl x509 -in cacert.pem -text -noout

# Generate server private key (on Server VM)
openssl genrsa -des3 -out server.key 2048

# Generate CSR (on Server VM)
openssl req -new -key server.key -out server.csr -config /etc/ssl/openssl.cnf

# Transfer CSR to CA VM
scp server.csr kali@<CA_VM_IP>:~

# Sign the CSR with the root CA
openssl ca -in server.csr -out newcerts/server.crt \
  -cert cacert.pem -keyfile private/cakey.pem \
  -config /etc/ssl/openssl.cnf

# View the signed certificate
openssl x509 -in newcerts/server.crt -noout -text

# Combine key and certificate for OpenSSL s_server
cp server.key server.pem
cat server.crt >> server.pem

# Launch HTTPS server on port 4433
openssl s_server -cert server.pem -www
```

### Hashing and Digital Signatures (Week 8)

```bash
# SHA-256 hash
openssl sha256 file1.txt

# SHA-3 hashes
openssl sha3-256 file1.txt
openssl sha3-512 file1.txt

# GPG: generate key pair
gpg --full-generate-key

# GPG: create detached signature
gpg --output my_file.sig --detach-sig my_file.txt

# GPG: verify signature
gpg --verify my_file.sig my_file.txt

# OpenSSL: create RSA digital signature (SHA-256)
openssl dgst -sha256 -sign private_key.pem -out my_file.sig my_file.txt

# OpenSSL: extract public key and verify signature
openssl pkey -in private_key.pem -pubout -out public_key.pem
openssl dgst -sha256 -verify public_key.pem -signature my_file.sig my_file.txt
```

### IKEv2 VPN with strongSwan (Lab 9)

```bash
# Install strongSwan and PKI tools
sudo apt install strongswan strongswan-pki libcharon-extra-plugins \
  libcharon-extauth-plugins libstrongswan-extra-plugins libtss2-tcti-tabrmd-dev

# Create PKI directory structure
sudo mkdir -p /root/pki/{cacerts,certs,private}

# Generate 4096-bit root CA private key
sudo pki --gen --type rsa --size 4096 --outform pem > /root/pki/private/ca-key.pem

# Create self-signed root CA certificate (10-year lifetime)
sudo pki --self --ca --lifetime 3650 \
  --in /root/pki/private/ca-key.pem --type rsa \
  --dn "CN=VPN root CA" --outform pem > /root/pki/cacerts/ca-cert.pem

# Generate 4096-bit VPN server private key
sudo pki --gen --type rsa --size 4096 --outform pem > /root/pki/private/server-key.pem

# Create and sign server certificate (5-year lifetime)
sudo pki --pub --in /root/pki/private/server-key.pem --type rsa \
  | pki --issue --lifetime 1825 \
    --cacert /root/pki/cacerts/ca-cert.pem \
    --cakey /root/pki/private/ca-key.pem \
    --dn "CN=<server-ip>" --san <server-ip> \
    --flag serverAuth --flag ikeIntermediate \
    --outform pem > /root/pki/certs/server-cert.pem

# Deploy certificates to ipsec.d
sudo cp -r /root/pki/* /etc/ipsec.d/
```

---

## Related Repos

- [Network-Security](https://github.com/CodeEvent/Network-Security) — second independent OpenVPN PKI built with EasyRSA as part of COMP10014 (FreeRADIUS AAA, Snort IDS, GRE tunnelling)
- [SmartGuard](https://github.com/CodeEvent/SmartGuard) — honours dissertation applying cryptographic security analysis to DeFi smart contracts; accepted at SIGiST 2026
- [Secure-Programming](https://github.com/CodeEvent/Secure-Programming) — SEI CERT C++ remediation including MSC51-CPP (cryptographic randomness seeding)
