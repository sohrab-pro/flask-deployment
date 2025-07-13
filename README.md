# Flask App Deployment Guide - Ubuntu

Deploy Flask app on Ubuntu

**App Location**: `/home/ubuntu/my_project`

## Prerequisites
- Instance running Ubuntu
- Domain access to configure DNS (yourdomain.com)
- Flask app with `requirements.txt` and `app.py`

## Step 1: Prepare Your Ubuntu Instance

Update system and install required packages:

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install python3 python3-pip python3-venv nginx -y
```

## Step 2: Set Up Virtual Environment and Install Dependencies

Navigate to Flask app directory and create virtual environment:

```bash
cd /home/ubuntu/my_project
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
pip install gunicorn
```

## Step 3: Test Your Flask App

Verify Flask app runs correctly:

```bash
python app.py
```

Test locally, then stop the development server (Ctrl+C).

## Step 4: Create a Gunicorn Service File

Create systemd service file:

```bash
sudo nano /etc/systemd/system/my-project.service
```

Add this content:

```ini
[Unit]
Description=Gunicorn instance to serve my-project
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/my_project
Environment="PATH=/home/ubuntu/my_project/venv/bin"
ExecStart=/home/ubuntu/my_project/venv/bin/gunicorn --workers 3 --bind unix:my-project.sock -m 007 app:app
ExecReload=/bin/kill -s HUP $MAINPID
Restart=always

[Install]
WantedBy=multi-user.target
```

## Step 5: Start and Enable the Service

```bash
sudo systemctl start my-project
sudo systemctl enable my-project
sudo systemctl status my-project
```

## Step 6: Configure Nginx

Create Nginx configuration:

```bash
sudo nano /etc/nginx/sites-available/yourdomain.com
```

Add this configuration:

```nginx
server {
    listen 80;
    server_name yourdomain.com;

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/ubuntu/my_project/my-project.sock;
    }
}
```

## Step 7: Enable the Nginx Site

```bash
sudo ln -s /etc/nginx/sites-available/yourdomain.com /etc/nginx/sites-enabled
sudo nginx -t
sudo systemctl restart nginx
```

## Step 8: Configure Domain DNS

In your domain registrar/DNS provider:

1. Create an A record for the domain:
   - **Name/Host**: `@` (or leave blank, depending on provider)
   - **Type**: `A`
   - **Value/Points to**: Your Ubuntu instance's public IP address
   - **TTL**: 300 (or default)
   - Note: The @ symbol represents the root domain (e.g., yourdomain.com). Some DNS providers may use a blank field instead. Refer to their specific instructions.


## Step 9: Test Your Deployment

Wait for DNS propagation (5-30 minutes), then test:

```bash
curl -I http://yourdomain.com
```

## Step 10: Set Up SSL with Let's Encrypt (Recommended)

Install Certbot and get SSL certificate:

```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d yourdomain.com
```

Set up automatic renewal:

```bash
sudo certbot renew --dry-run
```

## Step 11: Final Verification

Your site should now be accessible at:
- `http://yourdomain.com` (redirects to HTTPS if SSL configured)
- `https://yourdomain.com` (if SSL configured)

## Troubleshooting Commands

If you encounter issues:

```bash
# Check service status
sudo systemctl status my-project

# Check Nginx status
sudo systemctl status nginx

# View service logs
sudo journalctl -u my-project

# Check Nginx error logs
sudo tail -f /var/log/nginx/error.log

# Grant execute permissions, allowing Nginx to traverse the directory path
sudo chmod o+x /home/ubuntu /home/ubuntu/your-project

# Test Nginx configuration
sudo nginx -t
```

## Important Notes

1. ⚠️ Make sure your Flask app's `app.py` has the correct application object (usually `app = Flask(__name__)`)
2. ⚠️ Remove or comment out `app.run()` calls in production
3. 💡 Ensure your instance has adequate resources for expected traffic
4. 📊 Consider setting up monitoring and automated backups
5. 🔒 SSL/HTTPS is highly recommended for production deployments

## Final Result

Your Flask app should now be live at: **https://yourdomain.com**

