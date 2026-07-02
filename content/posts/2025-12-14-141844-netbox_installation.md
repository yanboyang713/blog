---
title: "NetBox Installation"
date: 2025-12-14
draft: false
---

## Pre-requisite {#pre-requisite}

-   [NetBox: The Network Source of Truth]({{< relref "2025-11-30-174834-netbox.md" >}})


## Using [Proxmox VE (PVE)]({{< relref "20230228043925-proxmox_ve.md" >}}) Helper-Script {#using-proxmox-ve--pve----20230228043925-proxmox-ve-dot-md--helper-script}


### Installation {#installation}

```bash
var_net="192.168.1.26/24" \
var_gateway="192.168.1.4" \
var_ns="-nameserver=192.168.1.23" \
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/netbox.sh)"
```


### Access {#access}

<https://192.168.1.26>
**NOTE**: Apache2 handling **:80** and **:443**


### Location of config file {#location-of-config-file}

/opt/netbox/netbox/netbox/configuration.py


### Show login and database credentials {#show-login-and-database-credentials}

```bash
cat netbox.creds
```


### [NetBox]({{< relref "2025-11-30-174834-netbox.md" >}}) Re-issue a self-signed cert that works for IP and/or hostname {#netbox--2025-11-30-174834-netbox-dot-md--re-issue-a-self-signed-cert-that-works-for-ip-and-or-hostname}


#### Original CA {#original-ca}

-   Cert: /etc/ssl/certs/netbox.crt
-   Key: /etc/ssl/private/netbox.key (don’t copy this off the NetBox LXC)


#### Confirm whether it’s self-signed {#confirm-whether-it-s-self-signed}

```console
root@netbox:~# openssl x509 -in /etc/ssl/certs/netbox.crt -noout -subject -issuer
subject=C=US, O=NetBox, OU=Certificate, CN=localhost
issuer=C=US, O=NetBox, OU=Certificate, CN=localhost
```

If subject == issuer, treat /etc/ssl/certs/netbox.crt as the CA.
As you can found CN=localhost


#### Back up Original CA {#back-up-original-ca}

```bash
cp -a /etc/ssl/certs/netbox.crt /etc/ssl/certs/netbox.crt.bak.$(date +%F)
cp -a /etc/ssl/private/netbox.key /etc/ssl/private/netbox.key.bak.$(date +%F)
```


#### Generate New CA {#generate-new-ca}

```bash
openssl req -x509 -nodes -days 825 -newkey rsa:2048 \
      -keyout /etc/ssl/private/netbox.key \
      -out /etc/ssl/certs/netbox.crt \
      -subj "/C=US/O=NetBox/OU=Certificate/CN=netbox.testbed.com" \
      -addext "subjectAltName=DNS:netbox.testbed.com,IP:192.168.1.26"
```

**NOTE**:

-   CN=netbox.testbed.com is there because the certificate needs an identity. Historically TLS used the Common Name, but modern clients
-   validate the SAN (Subject Alternative Name). In our command, the SAN is the part that really matters:
    -   subjectAltName=DNS:netbox.example.com,IP:192.168.1.26
-   Pick the CN to match the primary name you plan to use (usually the hostname), but as long as SAN is correct, CN is mostly cosmetic.


#### Set Apache vhost as ServerName netbox.testbed.com {#set-apache-vhost-as-servername-netbox-dot-testbed-dot-com}

<!--list-separator-->

-  Edit the vhost (set the name in both \*:80 and \*:443 blocks)

    vi /etc/apache2/sites-available/netbox.conf

    ```file
    Make sure it includes:
      ServerName netbox.example.com
    ```

<!--list-separator-->

-  Reload Apache (after checking config)

    ```bash
    apachectl configtest
    systemctl reload apache2
    ```

<!--list-separator-->

-  Verify which vhost is active

    ```bash
    apachectl -S
    ```

    **Important**: for clients to reach <https://netbox.example.com>, that name must resolve to 192.168.1.26 (via PowerDNS or a client /etc/hosts entry).

    <https://community-scripts.github.io/ProxmoxVE/scripts?id=netbox>


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
