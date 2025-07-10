# Odoo Installation Guide

---

## Linux (Ubuntu 22.04)

---

### 1. Connect to the Server

Using username and password:

```bash
ssh username@IP_Address -p Port_Number
```

Using a private key:

```bash
ssh -i /PATH_TO_PRIVATE_KEY username@IP_Address -p Port_Number
```

---

### 2. Update Server

```bash
sudo apt-get update
sudo apt-get upgrade -y
```

---

### 3. Secure Server from SSH Attacks (Optional)

```bash
sudo apt-get install -y openssh-server fail2ban
```

---

### 4. Install Dependencies and Packages

Install pip:

```bash
sudo apt-get install -y python3-pip
```

Development tools and libraries:

```bash
sudo apt-get install -y python3-dev libxml2-dev libxslt1-dev zlib1g-dev libsasl2-dev libldap2-dev build-essential libssl-dev libffi-dev libmysqlclient-dev libjpeg-dev libpq-dev libjpeg8-dev liblcms2-dev libblas-dev libatlas-base-dev
```

Install npm:

```bash
sudo apt-get install -y nodejs npm
```

Install git:

```bash
sudo apt-get install -y git
```

Install less and node-less:

```bash
sudo npm install -g less less-plugin-clean-css
sudo apt-get install -y node-less
```

---

### 5. Initialize Database (PostgreSQL)

Install PostgreSQL:

```bash
sudo apt-get install -y postgresql
```

Configure PostgreSQL:

```bash
sudo su - postgres
createuser --createdb --username postgres --no-createrole --no-superuser --pwprompt odoo
psql
```

you now enter SQL interface

```sql
ALTER USER odoo WITH SUPERUSER;
\q
```

\q : quit from sql interface

exit user

```bash
exit
```

> **Note:** The username and password set here are needed for Odoo configuration.

---

### 6. Create Odoo System User

```bash
sudo adduser --system --home=/opt/odoo --group odoo
```

Switch to Odoo user environment:

```bash
sudo su - odoo -s /bin/bash
```

---

### 7. Clone Odoo Repository or Use Enterprise Package

Clone Odoo from GitHub:

```bash
git clone https://www.github.com/odoo/odoo --depth 1 --branch 18.0 --single-branch .
```

Alternatively, download enterprise package from [odoo-community.ir](https://odoo-community.ir/odoo-enterprise):

```bash
sudo wget https://dl.odoo-community.ir/odoo-source/odoo18/deb_18e.deb
```

Exit Odoo user:

```bash
exit
```

---

### 8. Install Python Requirements

Install Python packages required by Odoo:

```bash
sudo pip install -r /opt/odoo/requirements.txt
```

> **Note:** If errors occur, install packages individually via `apt-get`.

---

### 9. Install wkhtmltopdf

Required for PDF exports:

```bash
sudo wget http://security.ubuntu.com/ubuntu/pool/universe/w/wkhtmltopdf/wkhtmltopdf_0.12.6-2build2_amd64.deb
sudo apt-get install ./wkhtmltopdf_0.12.6-2build2_amd64.deb
sudo apt install -f
```

---

### 9.1. RTL (Right-to-Left) Interface Support (Optional)

For RTL languages (Persian, Arabic, Hebrew):

```bash
sudo npm install -g rtlcss
```

---

### 10. Configure Odoo (`odoo.conf`)

Copy and edit configuration:

```bash
sudo cp /opt/odoo/debian/odoo.conf /etc/odoo.conf
sudo nano /etc/odoo.conf
```

Example configuration:

```ini
[options]
admin_passwd = admin
db_user = odoo
addons_path = /opt/odoo/addons
logfile = /var/log/odoo/odoo.log
```

Set permissions:

```bash
sudo chown odoo: /etc/odoo.conf
sudo chmod 640 /etc/odoo.conf
```

Create log directory:

```bash
sudo mkdir /var/log/odoo
sudo chown odoo:root /var/log/odoo
```

---

### 11. Configure Odoo as a System Service (`odoo.service`)

Create and edit service file:

```bash
sudo nano /etc/systemd/system/odoo.service
```

Content:

```ini
[Unit]
Description=Odoo
Documentation=http://www.odoo.com

[Service]
Type=simple
User=odoo
ExecStart=/opt/odoo/odoo-bin -c /etc/odoo.conf

[Install]
WantedBy=default.target
```

Set permissions:

```bash
sudo chmod 755 /etc/systemd/system/odoo.service
sudo chown root: /etc/systemd/system/odoo.service
```

---

### 12. Start and Manage Odoo Service

Start Odoo:

```bash
sudo systemctl start odoo.service
```

Check status:

```bash
sudo systemctl status odoo.service
```

Enable Odoo service on reboot:

```bash
sudo systemctl enable odoo.service
```

Restart Odoo after configuration changes:

```bash
sudo systemctl restart odoo.service
```

Access Odoo via browser:

```
http://<your_domain_or_IP_address>:8069
```

---

### 13. Monitor Odoo Logs

View logs in real-time:

```bash
sudo tail -f /var/log/odoo/odoo.log
```

---

## Best Practices for Production Deployment (Odoo 18)

### 1. Use a Reverse Proxy (Nginx)

```bash
sudo apt install nginx
```

Configure Nginx to proxy requests securely to Odoo.

### 2. Enable SSL with Let's Encrypt

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d your_domain
```

### 3. Optimize PostgreSQL Server

Edit PostgreSQL configuration:

```bash
sudo nano /etc/postgresql/{version}/main/postgresql.conf
```

Adjust parameters like `shared_buffers`, `work_mem`, and others based on server resources.

### 4. Configure Proper Logging

Install `logrotate` for managing log files:

```bash
sudo apt install logrotate
```

Configure log rotation to manage disk space efficiently.

### 5. Enable Workers for Performance

Add to `odoo.conf`:

```
workers = (CPU cores × 2) + 1
```

Example for 4 CPU cores:

```
workers = 9
```

### 6. Use Redis for Caching

Install Redis:

```bash
sudo apt install redis-server
```

Configure Odoo with Redis support in `odoo.conf`:

```ini
[options]
longpolling_port = 8072
proxy_mode = True
dbfilter = ^%h$
```

### 7. Automate Database Backups

Create a backup script (`backup_odoo.sh`):

```bash
#!/bin/bash
backup_dir="/backup/db"
date=$(date +%F)
filename="odoo_backup_$date.sql"
pg_dump odoo_db > "$backup_dir/$filename"
```

Schedule this script using cron jobs.

---

Following these guidelines ensures a secure, stable, and high-performing Odoo deployment suitable for production environments.


✅ Step-by-Step: Use the 2.18TB Datastore Inside Your Ubuntu VM
🧱 1. Add a New Virtual Disk to the VM
Power off your Ubuntu VM (optional, but safer).

In the ESXi Web UI:

Go to Virtual Machines → select your VM → Edit.

Click Add hard disk → New hard disk.

Set Size = e.g. 2000 GB (or whatever part of the 2.18TB you want).

Disk Provisioning: Thick Lazy Zeroed or Thin (your choice).

Storage: Select the 2.18TB datastore.

Click Save.

✅ This creates a second virtual disk (.vmdk) on your large HDD datastore and attaches it to your Ubuntu VM.

🐧 2. Boot into Ubuntu and Format the Disk
SSH into the VM or open console.

Run:

bash
Copy
Edit
lsblk
You'll see something like:

cpp
Copy
Edit
sda      25G
└─sda1   ...
sdb     2T     ← your new disk!
Create a partition:

bash
Copy
Edit
sudo parted /dev/sdb
(parted) mklabel gpt
(parted) mkpart primary ext4 0% 100%
(parted) quit
Format it:

bash
Copy
Edit
sudo mkfs.ext4 /dev/sdb1
📁 3. Mount the Disk for Odoo
Create a mount point:

bash
Copy
Edit
sudo mkdir /mnt/odoo_data
Mount the disk:

bash
Copy
Edit
sudo mount /dev/sdb1 /mnt/odoo_data
To make it mount automatically on boot:

bash
Copy
Edit
sudo blkid
# Copy the UUID of /dev/sdb1
sudo nano /etc/fstab
# Add this line at the end:
UUID=your-uuid-here /mnt/odoo_data ext4 defaults 0 2
🗃️ 4. Store Odoo DB Here (Optional)
If you're using PostgreSQL locally:

Edit the PostgreSQL config or initialize the PostgreSQL data directory on /mnt/odoo_data.

Example:

bash
Copy
Edit
sudo systemctl stop postgresql
sudo mv /var/lib/postgresql/14/main /mnt/odoo_data/main
sudo chown -R postgres:postgres /mnt/odoo_data/main
sudo nano /etc/postgresql/14/main/postgresql.conf
# Change:
data_directory = '/mnt/odoo_data/main'
sudo systemctl start postgresql
