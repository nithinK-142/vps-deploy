# How to Deploy and Host a MERN App on a Linux VPS

This guide will walk you through the steps to deploy and host a MERN (MongoDB, Express.js, React.js, Node.js) application on a Linux VPS. By the end of this guide, you will have a fully functional MERN app accessible via your domain.

## Prerequisites

Before you begin, ensure you have the following:

1. **A Linux VPS**: You should have root or sudo access to the server.
2. **A Domain Name**: Point your domain to the VPS's IP address.
3. **Node.js and npm**: Installed on your local machine and the VPS.
4. **MongoDB**: Installed on the VPS or using a cloud-based MongoDB service like MongoDB Atlas.
5. **Git**: Installed on the VPS for cloning your repository.
6. **PM2**: A process manager for Node.js applications (optional but recommended).
7. **Nginx**: A web server to act as a reverse proxy (optional but recommended).

---

## Step 1: Prepare Your MERN App for Deployment

1. **Build the React App**:
   Navigate to the `client` directory of your MERN app and run:
   ```bash
   npm run build
   ```
   This will create a `build` folder containing the production-ready static files.

2. **Set Environment Variables**:
   Ensure your `.env` files (for both the server and client) are configured with the correct production values (e.g., database connection strings, API keys, etc.).

3. **Push Code to a Git Repository**:
   Commit your code to a Git repository (e.g., GitHub, GitLab) if you haven't already. This will make it easier to clone the code onto your VPS.

---

## Step 2: Set Up Your Linux VPS

1. **Connect to Your VPS**:
   Use SSH to connect to your VPS:
   ```bash
   ssh root@your_vps_ip
   ```

2. **Update the System**:
   Update the package list and upgrade installed packages:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

3. **Install Node.js and npm**:
   Install Node.js and npm using the following commands:
   ```bash
   curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
   sudo apt-get install -y nodejs
   ```

4. **Install MongoDB** (if not using a cloud service):
   ```bash
   sudo apt install mongodb
   sudo systemctl enable mongodb
   sudo systemctl start mongodb
   ```

5. **Install Git**:
   ```bash
   sudo apt install git
   ```

6. **Install PM2** (optional but recommended):
   ```bash
   sudo npm install -g pm2
   ```

7. **Install Nginx** (optional but recommended):
   ```bash
   sudo apt install nginx
   ```

---

## Step 3: Deploy Your MERN App

1. **Clone Your Repository**:
   Clone your MERN app repository to the VPS:
   ```bash
   git clone https://github.com/your-username/your-repo.git
   cd your-repo
   ```

2. **Install Dependencies**:
   Install dependencies for both the server and client:
   ```bash
   npm install
   cd client
   npm install
   cd ..
   ```

3. **Build the React App**:
   Build the React app for production:
   ```bash
   cd client
   npm run build
   cd ..
   ```

4. **Start the Node.js Server**:
   Use PM2 to start your Node.js server:
   ```bash
   pm2 start server.js --name "mern-app"
   ```

5. **Set Up Nginx as a Reverse Proxy**:
   - Edit the Nginx configuration file:
     ```bash
     sudo nano /etc/nginx/sites-available/default
     ```
   - Add the following configuration:
     ```nginx
     server {
         listen 80;
         server_name yourdomain.com;

         location / {
             proxy_pass http://localhost:5000; # Replace 5000 with your Node.js server port
             proxy_http_version 1.1;
             proxy_set_header Upgrade $http_upgrade;
             proxy_set_header Connection 'upgrade';
             proxy_set_header Host $host;
             proxy_cache_bypass $http_upgrade;
         }

         location /static {
             alias /path/to/your/client/build/static;
         }
     }
     ```
   - Test the Nginx configuration:
     ```bash
     sudo nginx -t
     ```
   - Restart Nginx:
     ```bash
     sudo systemctl restart nginx
     ```

---

## Step 4: Secure Your App with SSL

1. **Install Certbot**:
   ```bash
   sudo apt install certbot python3-certbot-nginx
   ```

2. **Obtain an SSL Certificate**:
   ```bash
   sudo certbot --nginx -d yourdomain.com
   ```

3. **Automate Certificate Renewal**:
   Certbot will automatically set up a cron job to renew the certificate.

---

## Step 5: Access Your MERN App

Once everything is set up, you should be able to access your MERN app at `https://yourdomain.com`.

---

## Troubleshooting

- **Check PM2 Logs**:
  ```bash
  pm2 logs
  ```
- **Check Nginx Logs**:
  ```bash
  sudo tail -f /var/log/nginx/error.log
  ```
- **Firewall Configuration**:
  Ensure ports 80 (HTTP) and 443 (HTTPS) are open:
  ```bash
  sudo ufw allow 80
  sudo ufw allow 443
  ```

---
