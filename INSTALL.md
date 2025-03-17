# Excalidraw Installation Guide for Ubuntu Server with Nginx

This guide will walk you through the process of installing and deploying Excalidraw on an Ubuntu server with Nginx, using your domain `draw.marketcalls.in` with Cloudflare.

## Prerequisites

- Ubuntu server (Ubuntu 20.04 LTS or newer recommended)
- Root or sudo access
- Domain with A record pointing to your server IP (draw.marketcalls.in)
- Basic knowledge of terminal/command line operations

## Step 1: Set Up the Server Environment

Update your system and install required dependencies:

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y git curl wget nginx certbot python3-certbot-nginx
```

## Step 2: Install Node.js 22.x

Since fnm installation had issues, let's install Node.js 22.x directly using NodeSource:

```bash
# Install Node.js 22.x directly
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs

# Verify the Node.js version
node -v  # Should print "v22.x.x"

# Verify npm version
npm -v  # Should print "10.x.x"

# Install Yarn through npm
sudo npm install -g yarn
yarn --version  # Verify yarn installation
```

## Step 3: Clone the Excalidraw Repository

```bash
# Create directory for the application
mkdir -p /var/www/excalidraw
cd /var/www/excalidraw
git clone https://github.com/excalidraw/excalidraw.git .
```

## Step 4: Install Dependencies and Build the Application

```bash
# Navigate to the application directory
cd /var/www/excalidraw

# Install dependencies
yarn install

# Build the application for production
yarn build:app
```

This will create a production build in the `excalidraw-app/build` directory.

## Step 5: Configure Nginx

Create an Nginx server block for your domain:

```bash
sudo nano /etc/nginx/sites-available/draw.marketcalls.in
```

Add the following configuration:

```nginx
server {
    listen 80;
    server_name draw.marketcalls.in;
    root /var/www/excalidraw/excalidraw-app/build;
    index index.html;

    # Excalidraw specific headers
    location / {
        try_files $uri $uri/ /index.html;
        add_header X-Content-Type-Options "nosniff";
        add_header Referrer-Policy "origin";
    }

    # Cache static assets
    location ~* \.(css|js|jpg|jpeg|png|gif|ico|svg|woff|woff2)$ {
        expires 7d;
        add_header Cache-Control "public, max-age=604800";
        access_log off;
    }
}
```

Enable the site and test Nginx configuration:

```bash
sudo ln -s /etc/nginx/sites-available/draw.marketcalls.in /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

## Step 6: Set Up SSL with Let's Encrypt

Since you're using Cloudflare, you have two options:
1. Use Cloudflare's SSL (recommended if you're using Cloudflare's proxy)
2. Install a Let's Encrypt certificate

For Let's Encrypt:

```bash
sudo certbot --nginx -d draw.marketcalls.in
```

Follow the prompts to complete the SSL certificate installation.

## Step 7: Set Proper Permissions

Ensure proper permissions for your web files:

```bash
sudo chown -R www-data:www-data /var/www/excalidraw
sudo chmod -R 755 /var/www/excalidraw
```

## Step 8: Configure Firewall (Optional but Recommended)

```bash
sudo ufw allow 'Nginx Full'
sudo ufw enable
```

## Updating Excalidraw

To update Excalidraw to the latest version:

```bash
cd /var/www/excalidraw
git pull
yarn install
yarn build:app
sudo systemctl restart nginx
```

## Alternative: Docker Installation (Optional)

If you'd like to try Docker installation in the future, here are the steps:

```bash
# Install Docker
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install -y docker-ce docker-compose

# Clone and build with Docker
cd /var/www/excalidraw
docker build -t excalidraw:latest .
docker run -d --name excalidraw -p 8080:80 --restart unless-stopped excalidraw:latest

# Configure Nginx as reverse proxy
# Update your Nginx config to proxy_pass to http://localhost:8080
```

## Troubleshooting

### Site Not Loading

Check Nginx error logs:
```bash
sudo tail -f /var/nginx/error.log
```

### SSL Issues

If you're using Cloudflare, ensure the SSL/TLS encryption mode is set correctly in your Cloudflare dashboard (Full or Full (strict) recommended).

### Application Errors

Check for any build errors:
```bash
cd /var/www/excalidraw
yarn build:app
```

## Additional Configuration

### Environment Variables

For production deployments, you might want to configure environment variables based on your needs. Create a `.env.production` file with the necessary configurations before building.

### Persistent Storage (Optional)

Excalidraw is primarily a client-side application, but if you want to enable collaborative features, you'll need to set up the backend services as well, which requires additional configuration.
