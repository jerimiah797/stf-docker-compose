# SSL/TLS Certificates

Place your SSL certificates in this directory.

## Required Files

- `server.crt` - SSL certificate (or fullchain if using Let's Encrypt)
- `server.key` - Private key

## Generating Self-Signed Certificates (Testing Only)

For testing environments, you can generate a self-signed certificate:

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout server.key -out server.crt \
  -subj "/CN=stf.yourdomain.com"
```

**Warning**: Self-signed certificates will show security warnings in browsers and require `NODE_TLS_REJECT_UNAUTHORIZED=0` in the provider containers (already configured).

## Using Let's Encrypt (Production)

For production deployments, use Let's Encrypt for free, trusted certificates:

```bash
# Install certbot
sudo apt-get install certbot

# Get certificates (requires port 80 to be available temporarily)
sudo certbot certonly --standalone -d stf.yourdomain.com

# Copy certificates to this directory
sudo cp /etc/letsencrypt/live/stf.yourdomain.com/fullchain.pem server.crt
sudo cp /etc/letsencrypt/live/stf.yourdomain.com/privkey.pem server.key

# Set appropriate permissions
sudo chmod 644 server.crt
sudo chmod 600 server.key
```

## Using Commercial Certificates

If you have commercial certificates from a CA:

1. Place the certificate (or certificate bundle) as `server.crt`
2. Place the private key as `server.key`
3. Ensure the certificate includes any necessary intermediate certificates

## Security Notes

- Never commit actual certificates to version control
- Keep private keys secure (600 permissions)
- Certificates are mounted read-only into the nginx container
- Set up automatic renewal for Let's Encrypt certificates (certbot renew)
