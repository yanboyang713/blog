---
title: "enable copy-paste functionality in the Proxmox noVNC console"
date: 2025-05-04
draft: false
---

To enable copy-paste functionality in the [Proxmox VE]({{< relref "20230228043925-proxmox_ve.md" >}}) noVNC console, you can use a [Tampermonkey]({{< relref "2025-05-04-043920-tampermonkey.md" >}}) user script developed by amunchet.


## Step 1: Install Tampermonkey Extension {#step-1-install-tampermonkey-extension}

-   **For Chrome or Edge**: Visit the [Chrome Web Store](https://chrome.google.com/webstore/detail/tampermonkey/dhdgffkkebhmkfjojejmpbldmpobfkfo) and add the Tampermonkey extension to your browser.
-   **For Firefox**: Go to the [Firefox Add-ons page](https://addons.mozilla.org/firefox/addon/tampermonkey/) and install the Tampermonkey extension.


## Step 2: Install the noVNC Paste Script {#step-2-install-the-novnc-paste-script}

1.  Open the raw version of the script: [noVNCCopyPasteProxmox.user.js](https://gist.github.com/amunchet/4cfaf0274f3d238946f9f8f94fa9ee02/raw/0b84970f89e1f282f09b86d46227eda71178c040/noVNCCopyPasteProxmox.user.js).
2.  Tampermonkey should prompt you to install the script. Click **Install**.


## Step 3: Use the Script in Proxmox noVNC Console {#step-3-use-the-script-in-proxmox-novnc-console}

1.  Copy the desired text to your clipboard (e.g., using **Ctrl+C**).
2.  Navigate to your Proxmox noVNC console.
3.  Right-click inside the console window. The script will simulate typing the clipboard contents into the console.


## Reference List {#reference-list}

1.  <https://gist.github.com/amunchet/4cfaf0274f3d238946f9f8f94fa9ee02>
