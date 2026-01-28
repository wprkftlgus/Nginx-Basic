# Nginx Basic Setup (Ubuntu / EC2)

Basic Nginx configuration for an EC2 server with HTTPS and reverse proxy.

```
# Connect to EC2
ssh -i .pem ubuntu@[Your EC2 Public IPv4 Ip] #Enter EC2 server

# Download Nginx
sudo apt update
sudo apt install nginx -y

# Create API config file
sudo nano /etc/nginx/sites-available/api

# Example configuration
server {
    listen 80; 
    server_name sihyeonzon.online www.sihyeonzon.online api.sihyeonzon.online;
    return 301 https://$host$request_uri;
}  # When users enter http, Forcing user to enter https for security reason. 

server {
    listen 443 ssl;
    server_name api.sihyeonzon.online;

    ssl_certificate /etc/letsencrypt/live/api.sihyeonzon.online/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.sihyeonzon.online/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
} # api server listen at 443, ssl is mendatory,  http://127.0.0.1:3000 is node.js IP,  proxy_set_header X-Real-IP $remote_addr : get the user IP, proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for : Make the user IP list 

# Enable the site (symbolic link)
sudo ln -s /etc/nginx/sites-available/api \
           /etc/nginx/sites-enabled/

# Check the status
ls -l /etc/nginx/sites-enabled/

# See the nginx working well + apply the setting to Nginx
sudo nginx -t
sudo systemctl reload nginx
```
