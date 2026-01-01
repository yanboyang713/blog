---
title: "Proxmox VE Get “Universal Trust” TLS with Let’s Encrypt (ACME)"
draft: false
---

## Goal {#goal}

If you access [Proxmox VE]({{< relref "20230228043925-proxmox_ve.md" >}}) at \`<https://172.27.135.44:8006/>\`, your browser warns **“Not secure”** because Proxmox ships with a self-signed (cluster) CA by default. That traffic is still encrypted, but it is not **publicly trusted**.

Requirements:

-   **Universal trust**: no installing a custom CA into every client OS/browser.
-   **No /etc/hosts hacks**.
-   **Proxmox management stays private (VPN-only / internal-only)**.
-   DNS provider: [cloudflare]({{< relref "2023-11-20-155714-cloudflare.md" >}}) (domain: **yanboyang.com**).
-   Proxmox VE: **9.1.4**, **cluster**.
-   Current access IP (internal): **172.27.135.44**, port **8006**.

This post describes the simplest model: **DNS-01 via [cloudflare]({{< relref "2023-11-20-155714-cloudflare.md" >}})**, direct access to \`<https://pve.yanboyang.com:8006/>\`, with management still private.


## Why DNS-01 is the right fit for “private Proxmox, publicly trusted certs” {#why-dns-01-is-the-right-fit-for-private-proxmox-publicly-trusted-certs}

Let’s Encrypt supports multiple ACME challenge types:

-   **HTTP-01** requires the CA to reach your server on **port 80**, and the hostname must resolve to a reachable public IP.
-   **DNS-01** proves you control DNS by creating a TXT record at **\_acme-challenge.&lt;your-domain&gt;**. It works even if the service is private / not publicly reachable.

Proxmox VE supports both challenge types and explicitly documents DNS-01 as the option when external validation access “is not possible or desired,” requiring TXT provisioning “via an API.”


## Why you need the Cloudflare API (even if you already added an A record) {#why-you-need-the-cloudflare-api--even-if-you-already-added-an-a-record}

Adding **pve.yanboyang.com -&gt; 172.27.135.44** as “DNS only” is **NOT** enough for Let’s Encrypt issuance/renewal.
With DNS-01, every issuance/renewal requires creating (and later removing) a temporary TXT record under:

-   **\_acme-challenge.pve.yanboyang.com**

Let’s Encrypt strongly recommends DNS-01 **only when your DNS provider has an API**, because renewals must be automated to be practical.

So the Cloudflare API token is what allows Proxmox (via its ACME integration) to programmatically add/remove the TXT record during issuance and renewal.


## Important privacy note (read before proceeding) {#important-privacy-note--read-before-proceeding}

1.  **Your hostname will be public.** Let’s Encrypt certificates are publicly observable via Certificate Transparency systems.
2.  **Your management UI stays private.** Clients will resolve **pve.yanboyang.com** to **172.27.135.44** (a private address). Only users on your internal network/VPN can connect.
3.  Publishing a private RFC1918 address in public DNS is unusual, but technically valid. The main “leak” is that it advertises your internal addressing choice—not actual reachability.

If that trade-off is acceptable, proceed.


## Step-by-step: Cloudflare + Proxmox DNS-01 (Model 1) {#step-by-step-cloudflare-plus-proxmox-dns-01--model-1}


### Step 0 — Pick your final URL and commit to using it {#step-0-pick-your-final-url-and-commit-to-using-it}

Once you install a publicly trusted cert for \`pve.yanboyang.com\`, you should always use:

-   <https://pve.yanboyang.com:8006/>

If you keep using the IP URL, the browser will continue to warn (hostname mismatch).


### Step 1 — Create the Cloudflare DNS record (DNS-only) {#step-1-create-the-cloudflare-dns-record--dns-only}

In Cloudflare DNS for \`yanboyang.com\`:

-   **Type**: A
-   **Name**: pve
-   **IPv4**: 172.27.135.44
-   **Proxy status**: **DNS only** (gray cloud)

Notes:

-   “Proxied” does **not** work for this use case anyway (Cloudflare won’t proxy arbitrary port 8006 like this).
-   DNS propagation speed matters later; low TTL can help, but Cloudflare is usually fast.


### Step 2 — Create a least-privilege Cloudflare API token {#step-2-create-a-least-privilege-cloudflare-api-token}

You will need:

-   **API Token**
-   **Account ID** and/or **Zone ID** (On section **Overview**)

{{< figure src="https://res.cloudinary.com/dkvj6mo4c/image/upload/v1767230022/API-Tokens-Cloudflare-12-31-2025_08_12_PM_xc4rx7.png" >}}

**Recommended token scope (typical DNS-01 needs):**

-   Permissions:
    -   Zone → DNS → Edit
    -   Zone → Zone → Read (commonly required by many DNS automation clients)
-   Zone Resources:
    -   Restrict to **yanboyang.com** zone only

Keep the token secret safe.


### Step 3 — Add the ACME DNS plugin in Proxmox (cluster-wide) {#step-3-add-the-acme-dns-plugin-in-proxmox--cluster-wide}

Proxmox stores ACME plugin configuration in **/etc/pve/priv/acme/plugins.cfg**, and the plugin is available to all nodes in a cluster.
It also reuses **acme.sh** DNS plugins for provider-specific APIs.

1.  **Datacenter → ACME**
2.  Under **Challenge Plugins**, click **Add**
3.  Set:
    -   **Challenge Type**: **DNS**
    -   **DNS API**: **Cloudflare**
    -   **Plugin ID**: e.g. **cf-pve-yanboyang**
    -   You don't need Cloudflare global api key
    -   Account ID (require)
    -   Email (require)
    -   Token (require)
    -   Zone ID (require)


### Step 4 — Register an ACME account (use staging first) {#step-4-register-an-acme-account--use-staging-first}

Proxmox supports Let’s Encrypt production and staging endpoints, and recommends staging first due to rate limits.

GUI:

1.  Datacenter → ACME → Accounts → Add
2.  Use Let’s Encrypt V2 Staging initially
3.  Provide email, accept ToS

CLI (documented example):

```bash
pvenode acme account register default you@example.com
```


### Step 5 — Configure the domain on the node (node-specific) {#step-5-configure-the-domain-on-the-node--node-specific}

In a cluster:

-   The plugin is shared cluster-wide.
-   The domain binding is node-specific.

GUI:

1.  Select a node (e.g., the one you access at 172.27.135.44), my one server1
2.  Node → Certificates
3.  Add an ACME domain:
4.  Challenge Type: DNS
5.  Plugin: cf-pve-yanboyang
6.  Domain: pve.yanboyang.com
7.  Account: your staging account (for testing)

CLI pattern (from Proxmox docs):

```bash
pvenode config set -acmedomain0 pve.yanboyang.com,plugin=cf-pve-yanboyang
```


### Step 6 — Order the certificate and install it {#step-6-order-the-certificate-and-install-it}

GUI:

-   In Node → Certificates, click Order Certificate (ACME)

CLI:

```bash
pvenode acme cert order
```

On success, Proxmox restarts pveproxy (you’ll see this in the task output).
Proxmox uses:

-   /etc/pve/local/pveproxy-ssl.pem and its key for the web UI TLS cert, and notes /etc/pve/local is node-specific.


### Step 7 — Switch from staging to production {#step-7-switch-from-staging-to-production}

Once the staging flow works end-to-end:

1.  Create a production ACME account (Let’s Encrypt Production).
2.  Select it for the node domain entry.
3.  Order the certificate again.

This avoids rate-limit surprises during debugging.


### Renewal behavior {#renewal-behavior}

Proxmox states: “Renewal will happen automatically.”
Operationally, you should still verify this once:

```bash
# Show configured domains and plugin binding
pvenode config get | grep -i acme

# (Optional) Force a renewal test (typically unnecessary in normal ops)
pvenode acme cert renew --force
```


## Reference List {#reference-list}

1.  <https://www.youtube.com/watch?v=2_PhwHOxytM&t=667s>
