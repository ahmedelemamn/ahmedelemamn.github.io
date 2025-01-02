---
layout: post
title: "Initializing a HashiCorp Vault Instance with Let's Encrypt Certificates"
date: 2024-01-02 19:25:00 +0000
categories: [DevOps, Hashicorp]
tags: [DevOps, Hashicorp, vault]
---

# Initializing a HashiCorp Vault Instance with Let's Encrypt Certificates

In this tutorial, I'll guide you through the process of setting up a HashiCorp Vault instance and securing it with a wildcard SSL certificate from Let's Encrypt. We'll be using DNS validation for the certificate and configuring Vault to use the SSL certificate for HTTPS communication.

## Prerequisites

1. **Ubuntu 20.04 or later** (or any Linux-based system)
2. A domain name (In this example, we'll use **Cloudflare** for DNS validation)
3. **Cloudflare API token** for DNS validation or equivilant

### Steps

### 1. Install HashiCorp Vault

To install HashiCorp Vault on Ubuntu, follow these commands:

```bash
# Update and install Vault
sudo apt update
sudo apt install vault

# Install the Vault repository (if the previous command failed)
sudo apt update
sudo apt install -y wget curl gnupg software-properties-common
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor > /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com ubuntu main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
```

Once the installation is complete, start and enable Vault to run on boot:

```bash
# Start Vault
sudo systemctl start vault

# Enable Vault to start on boot
sudo systemctl enable vault
```

### 2. Install Certbot and Obtain a Let's Encrypt Certificate

Since we are using DNS validation for Let's Encrypt, you'll need to install **Certbot** and configure it to use Cloudflare as the DNS provider.

1. **Install Certbot and the Cloudflare DNS plugin:**

   ```bash
   sudo apt install certbot python3-certbot-dns-cloudflare
   ```

2. **Configure DNS API for Cloudflare:**

   - Log into your Cloudflare account and create an API token with DNS edit permissions.
   - Save the API token securely (you’ll need it for Certbot).

3. **Save your Cloudflare API Token to a file:**

   ```bash
   # Create a file to save your token
   sudo vim /etc/letsencrypt/cloudflare-api-token.conf
   # Add your token to the file and save it

   # Make the file read-only to protect the token
   sudo chmod 600 /etc/letsencrypt/cloudflare-api-token.conf
   ```

4. **Obtain the SSL Certificate:**

   Use Certbot to obtain a wildcard SSL certificate for your domain:

   ```bash
   sudo certbot certonly --dns-cloudflare --dns-cloudflare-credentials /etc/letsencrypt/cloudflare-api-token.conf -d "*.example.com" -d "example.com" --agree-tos --non-interactive --preferred-challenges dns
   ```

   The `certbot` tool will automatically create a DNS record to prove domain ownership and retrieve the wildcard certificate. The process typically takes around 60 seconds to complete.

### 3. Set Up the Vault Configuration

Now, we'll configure Vault to use the obtained Let's Encrypt certificate. First, copy the certificate files into the Vault directory:

```bash
# Copy the certificate and key into the Vault configuration directory
sudo cp /etc/letsencrypt/live/example.com/fullchain.pem /opt/vault/tls/tls.crt
sudo cp /etc/letsencrypt/live/example.com/privkey.pem /opt/vault/tls/tls.key
```

Next, open the Vault configuration file (`/etc/vault.d/vault.hcl`) and specify the TLS settings:

```bash
sudo nano /etc/vault.d/vault.hcl
```

In the configuration file, set the listener section for HTTPS:

```hcl
# Vault configuration

ui = true

storage "raft" {
  path = "/opt/vault/data"
  node_id = "homelab_vault"
}

# HTTPS listener
listener "tcp" {
  address       = "vault.example.com:8200"
  tls_cert_file = "/opt/vault/tls/tls.crt"
  tls_key_file  = "/opt/vault/tls/tls.key"
}

api_addr = "https://vault.example.com:8200"
cluster_addr = "https://vault.example.com:8201"
cluster_id = "homelab_vault_cluster"
```

#### Notes:
- Replace `vault.example.com` with your own domain name.
- Ensure that the domain resolves correctly by adding the necessary DNS records and confirming that your Vault VM can resolve the domain. You can verify it using `dig` or `ping`.

```bash
# Verify DNS resolution
dig vault.example.com

# Verify the network connection
ping vault.example.com
```

- The Vault address (`api_addr`) needs to be defined as a variable in your shell. You can either add `export VAULT_ADDR="https://vault.example.com:8200"` to your `.bashrc` or `.zshrc` config files, or set it in the current session.

### 4. Restart Vault

After updating the configuration file, restart the Vault service to apply the changes:

```bash
sudo systemctl restart vault
```

### 5. Verify Vault Status

To check if Vault is running correctly with the SSL certificate, run:

```bash
vault status
```

If everything is set up correctly, you should see Vault’s status without any TLS errors.

### 6. Automate Certificate Renewal

Let's Encrypt certificates are valid for 90 days, so it's important to set up automatic renewal. Certbot automatically installs a cron job or systemd timer to handle renewals. To verify, run the following command:

```bash
sudo systemctl list-timers | grep certbot
```

You should see a timer for the `certbot` service. You can also test the renewal process with:

```bash
sudo certbot renew --dry-run
```

This command will simulate a renewal to ensure everything is working as expected.

### Conclusion

By following these steps, you have successfully initialized a HashiCorp Vault instance secured with a wildcard SSL certificate from Let's Encrypt using DNS validation. This setup ensures that your Vault instance is secure, using trusted certificates for encrypted communication.

Remember, automatic certificate renewal is crucial to avoid certificate expiration. Regularly verify that the renewal process is functioning correctly to ensure uninterrupted service.
