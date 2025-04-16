# Deploying FastAPI on IONOS VPS with Plesk

## Introduction
This guide explains how to set up your FastAPI-based web application on an IONOS VPS using Plesk.

## Step 1: Accessing Plesk SSH Terminal
Log in to your Plesk admin dashboard at:
```
SERVER_IP:8443
```
Navigate to **"Tools & Settings" > "SSH Terminal"**. Use the following shortcuts:
- **Copy:** `Ctrl + Insert`
- **Paste:** `Shift + Insert`

## Step 2: Update the System
Run the following command to update packages:
```bash
sudo apt update && sudo apt upgrade
```

## Step 3: Install MongoDB
Run the following commands to install MongoDB:
```bash
sudo apt install -y gnupg curl
curl -fsSL https://www.mongodb.org/static/pgp/server-8.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg --dearmor
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
sudo apt update
sudo apt install -y mongodb-org
sudo systemctl start mongod
sudo systemctl enable mongod
sudo systemctl status mongod
```

### Step 3.1: Set Up MongoDB Authentication
```bash
mongosh
```
Then run:
```javascript
> use admin
> db.createUser({
  user: "PUT USERNAME HERE",
  pwd: "PUT PASSWORD HERE",
  roles: [ { role: "userAdminAnyDatabase", db: "admin" } , "readWriteAnyDatabase" ]
})
> exit
```
Edit the MongoDB configuration file:
```bash
sudo nano /etc/mongod.conf
```
Add:
```yaml
security:
  authorization: "enabled"
```
MongoDB connection string:
```
mongodb://PUT USERNAME HERE:PUT PASSWORD HERE@localhost:27017
```

## Step 4: Set Up FastAPI
Create a project directory, for example:
```bash
cd /var/www/html
mkdir webapp && cd webapp
```
Create a virtual environment:
```bash
sudo apt install -y python3-venv
python3 -m venv venv
source venv/bin/activate
```
Install dependencies:
```bash
pip3 install fastapi uvicorn motor pymongo
```
Upload your FastAPI web app files to `/var/www/html/webapp`.

## Step 5: Create a Systemd Service for FastAPI
Create a service file:
```bash
sudo nano /etc/systemd/system/fastapi.service
```
Paste the following(ensure proper path first for your web app):
```ini
[Unit]
Description=FastAPI App
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/var/www/html/webapp
ExecStart=/var/www/html/webapp/venv/bin/uvicorn app:app --host 0.0.0.0 --port 8000 --workers 4
Restart=always

[Install]
WantedBy=multi-user.target
```
Save and exit, then enable and start the service:
```bash
sudo systemctl daemon-reload
sudo systemctl start fastapi
sudo systemctl enable fastapi
sudo systemctl status fastapi
```
To restart the service after changes:
```bash
sudo systemctl restart fastapi.service
```

## Step 6: Configure Apache2 as a Reverse Proxy
Edit the Apache2 configuration file:
```bash
sudo nano /etc/apache2/sites-available/000-default.conf
```
Modify it with:
```apache
<VirtualHost 127.0.0.1:7080>
        ServerName SERVER_IP
        ProxyRequests Off
        ProxyPreserveHost On
        ProxyPass / http://0.0.0.0:8000/
        ProxyPassReverse / http://0.0.0.0:8000/

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html
</VirtualHost>
```
Restart Apache2 for changes to take effect:
```bash
sudo systemctl restart apache2
```

## Step 7: Access Your FastAPI Web App
Navigate to your server's homepage, and your FastAPI app should be running successfully.

Enjoy! 🚀
