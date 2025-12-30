---
title: "NetBox Installation"
date: 2025-12-14
draft: false
---

## Pre-requisite {#pre-requisite}

-   [NetBox: The Network Source of Truth]({{< relref "2025-11-30-174834-netbox.md" >}})


## Setting Up NetBox (Ubuntu 24.04/22.04) {#setting-up-netbox--ubuntu-24-dot-04-22-dot-04}

This guide walks you through installing NetBox directly on an Ubuntu LXC or VM ("bare metal" style).


### System Requirements {#system-requirements}

-   **RAM**: 4GB recommended (2GB minimum for very small labs).
-   **Disk**: 20GB recommended (10GB minimum).
-   **CPU**: 2 vCPUs recommended.


### 1. System Preparation {#1-dot-system-preparation}

Update the system and install the required system packages, including PostgreSQL, Redis, and Python build tools.

```bash
apt update && apt upgrade -y
apt install -y postgresql postgresql-common postgresql-client redis-server \
git python3-pip python3-venv python3-dev build-essential libxml2-dev \
libxslt1-dev libffi-dev libpq-dev libssl-dev zlib1g-dev
```


### 2. Database Setup (PostgreSQL) {#2-dot-database-setup--postgresql}

1.  **Start PostgreSQL**:
    ```bash
    systemctl enable --now postgresql
    ```

2.  **Create Database and User**:
    Replace **strong_password** with a real password.
    ```bash
    sudo -u postgres psql
    ```
    Inside the SQL shell:
    ```sql
    CREATE DATABASE netbox WITH TEMPLATE template0 ENCODING 'UTF8';
    CREATE USER netbox WITH PASSWORD 'strong_password';
    ALTER DATABASE netbox OWNER TO netbox;
    \c netbox
    ALTER SCHEMA public OWNER TO netbox;
    GRANT ALL PRIVILEGES ON SCHEMA public TO netbox;
    \q
    ```


### 3. Install NetBox {#3-dot-install-netbox}

1.  **Create the NetBox system user**:
    ```bash
    useradd -r -d /opt/netbox -s /usr/sbin/nologin netbox
    ```

2.  **Clone the NetBox repository**:
    ```bash
    git clone -b main https://github.com/netbox-community/netbox.git /opt/netbox/
    chown -R netbox:netbox /opt/netbox/
    ```

3.  **Configure NetBox**:
    ```bash
    cd /opt/netbox/netbox/netbox/
    cp configuration_example.py configuration.py
    ```
    Generate a secret key:
    ```console
    root@netbox:/opt/netbox/netbox/netbox# python3 -c 'import secrets; print(secrets.token_hex(50))'
    866887ae9b6f118ce7e5a7b7f7a4b2fa7a1979dab810e8b092dd5dba4a3bc1f6b0e1028ac45850c1b55a617c3ff41c932c56
    ```
    Edit \`configuration.py\` (located in \`/opt/netbox/netbox/netbox/\`):

    -   Set \`ALLOWED_HOSTS\` to \`['**']\` for initial testing. **\*(For production, replace \`\*\` with your NetBox server's IP addresses or domain names)****:
        ```python
        ALLOWED_HOSTS = ['*']
        ```
    -   Set \`DATABASE\` user to \`netbox\` and password to \`strong_password\`.
    -   Set \`SECRET_KEY\` to the string generated above.

4.  **Install Dependencies &amp; Run Migrations**:
    ```bash
    /opt/netbox/upgrade.sh
    ```
    This script creates the Python environment, installs requirements, and builds the DB schema.

5.  **Create a Superuser**:
    ```bash
    source /opt/netbox/venv/bin/activate
    cd /opt/netbox/netbox
    python3 manage.py createsuperuser
    ```


### 4. Configure Gunicorn &amp; Systemd {#4-dot-configure-gunicorn-and-systemd}

1.  **Copy the systemd service and config files**:
    ```bash
    cp /opt/netbox/contrib/*.service /etc/systemd/system/
    cp /opt/netbox/contrib/gunicorn.py /opt/netbox/gunicorn.py
    systemctl daemon-reload
    ```

2.  **Start NetBox Services**:
    ```bash
    systemctl enable --now netbox netbox-rq
    ```


### 5. Web Server (Nginx) {#5-dot-web-server--nginx}

To serve NetBox on port 80/443.

1.  **Install Nginx**:
    ```bash
    apt install -y nginx
    ```

2.  **Configure Nginx**:
    Copy the default config provided by NetBox:
    ```bash
    cp /opt/netbox/contrib/nginx.conf /etc/nginx/sites-available/netbox
    rm /etc/nginx/sites-enabled/default
    ln -s /etc/nginx/sites-available/netbox /etc/nginx/sites-enabled/netbox
    ```

3.  **Edit the Nginx Config**:
    Edit **/etc/nginx/sites-available/netbox**. Change **server_name** to your IP or Domain.

4.  **Generate Self-Signed SSL Certificate**:
    The default config enables HTTPS, so you need a certificate.
    ```bash
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/netbox.key \
    -out /etc/ssl/certs/netbox.crt \
    -subj "/C=US/ST=State/L=City/O=NetBox/CN=netbox.local"
    ```

5.  **Restart Nginx**:
    ```bash
    systemctl restart nginx
    ```


### Accessing NetBox {#accessing-netbox}

-   Open \`<http://<your-server-ip>/>\`
-   Log in with the superuser you created.


## Reference List {#reference-list}

1.  <https://netbox.readthedocs.io/en/stable/installation/3-netbox/>
2.  <https://www.apalrd.net/posts/2025/iaac_netbox/>
3.  <https://www.youtube.com/watch?v=p3J3f2QWFGE>
4.  <https://www.howtoforge.com/how-to-install-netbox-irm-on-ubuntu-24-04-server/#google_vignette>
5.  <https://community-scripts.github.io/ProxmoxVE/scripts?id=netbox&category=Network+%26+Firewall>
