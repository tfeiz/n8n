# n8n Installation on Ubuntu with Docker & Subdomain


## Step 1: Create n8n Docker Compose File

```bash
mkdir ~/n8n && cd ~/n8n
nano docker-compose.yml
```

Paste this configuration:

```yaml
version: '3'

services:
  n8n:
    image: n8nio/n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      - N8N_BASE_URL=https://n8n.yourdomain.com
      - N8N_PROTOCOL=https
      - NODE_ENV=production
      - N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
      - N8N_RUNNERS_ENABLED=true
      - GENERIC_TIMEZONE=Asia/Tehran
    volumes:
      - ./.n8n:/home/node/.n8n
    networks:
      - n8n_network

networks:
  n8n_network:
    driver: bridge
```

## Step 2: Configure Nginx Reverse Proxy

Install Nginx if not already installed:

```bash
sudo apt install -y nginx
```

Create Nginx configuration:

```bash
sudo nano /etc/nginx/sites-available/n8n.yourdomain.com
```

Add this configuration (replace with your domain):

```nginx
server {
    server_name n8n.yourdomain.com;

    location / {
        proxy_pass http://localhost:5678;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    listen 80;
}
```

Enable the site:

```bash
sudo ln -s /etc/nginx/sites-available/n8n.yourdomain.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

## Step 3: Set Up SSL with Let's Encrypt

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d n8n.yourdomain.com
```

Follow the prompts to configure SSL.

## Step 4: Start n8n

```bash
docker-compose up -d
```

## Step 5: Verify Installation

Access your n8n instance at:
```
https://n8n.yourdomain.com
```


This README provides a complete guide with proper formatting for GitHub. Adjust domain names and timezone as needed for your setup.
