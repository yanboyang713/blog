---
title: "prevent laptop lid suspending"
draft: false
---

To prevent Proxmox from suspending when the laptop lid is closed, you can modify the system's power management settings. Here are the steps to achieve this:

1.  ****Access Proxmox via SSH or Console:**** Log in to your Proxmox server using SSH or the console.

2.  ****Edit Logind Configuration:****
    -   Open the \`logind.conf\` file for editing. You can use a text editor like \`nano\` or \`vi\`. The command might look like this:
        \`\`\`bash
        nano /etc/systemd/logind.conf
        \`\`\`
    -   In this file, find the line that starts with \`#HandleLidSwitch=suspend\`. If it's commented out (as indicated by the \`#\` at the beginning), you need to uncomment it (remove the \`#\`) and change its value.
    -   Change it to:
        \`\`\`bash
        HandleLidSwitch=ignore
        \`\`\`
    -   This tells the system to ignore the lid switch, thereby preventing it from suspending when the lid is closed.

3.  ****Restart the Systemd Logind Service:****
    -   After saving the changes, restart the systemd logind service to apply the new settings. This can be done with the following command:
        \`\`\`bash
        systemctl restart systemd-logind
        \`\`\`

4.  ****Verify the Changes:****
    -   To ensure the changes have taken effect, close the laptop lid and check if Proxmox continues to run without going into suspend mode.

5.  ****Check for Other Power Management Settings:****
    -   Sometimes, other power management tools or settings might override this configuration. Ensure no other power management settings are conflicting with your new configuration.

6.  ****Regularly Update Your System:****
    -   Keep your Proxmox system updated, as updates might overwrite your configuration. After updates, it's a good idea to check if the \`logind.conf\` file still has your desired settings.

7.  ****Documentation and Help:****
    -   For more detailed information or troubleshooting, refer to the Proxmox documentation or community forums.

Remember that changing system settings can have unintended consequences, so it's always a good idea to have backups and understand the changes you're making.


## Run Script {#run-script}

```bash
wget https://raw.githubusercontent.com/yanboyang713/proxmox-helper/main/disable-lid-suspend.sh
chmod +x disable-lid-suspend.sh
./disable-lid-suspend.sh
```
