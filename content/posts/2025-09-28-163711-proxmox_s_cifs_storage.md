---
title: "Proxmox’s CIFS storage"
date: 2025-09-28
draft: false
---

[Proxmox VE]({{< relref "20230228043925-proxmox_ve.md" >}})


## pvesm scan {#pvesm-scan}

Running pvesm scan is very useful before adding CIFS or NFS storage in Proxmox because it automatically detects what shares or exports are actually available on the remote server. This way you avoid guessing and misconfiguring --share or --subdir

```console
root@server1:~# pvesm scan cifs samba.cs.virginia.edu --username YOUR_UVA_USERNAME --password YOUR_PASSWORD
b       server
bigtemp pigtemp
```


## add cifs {#add-cifs}

```bash
pvesm add cifs uva_proj \
  --server samba.cs.virginia.edu \
  --share p \
  --subdir /nmg5g/pve \
  --username YOUR_UVA_USERNAME \
  --password YOUR_PASSWORD \
  --domain CSDOM \
  --smbversion 3 \
  --content images,iso,rootdir,backup,vztmpl,snippets
```

Explanation of flags

| Option                             | Description                                                                                          |
|------------------------------------|------------------------------------------------------------------------------------------------------|
| \`cifs\`                           | Storage type (CIFS/SMB share).                                                                       |
| \`uva_proj\`                       | The ****storage ID**** in Proxmox — name it whatever you want.                                       |
| \`--server samba.cs.virginia.edu\` | UVA’s SMB server hostname.                                                                           |
| \`--share b\`                      | CIFS share to use (get available ones with pvesm scan cifs &lt;address&gt; or the web UI). Required. |
| \`--subdir nmg5g\`                 | Points directly to \`/p/nmg5g\` instead of listing all projects.                                     |
| \`--username\`                     | Your UVA CS computing username.                                                                      |
| \`--password\`                     | Your UVA password (or app password, if required).                                                    |
| \`--smbversion 3\`                 | Forces SMBv3 for better performance and security.                                                    |
| \`--content iso,vztmpl,backup\`    | Allow storage of ****ISO files****, ****container templates****, and ****backups**** only.           |


## Reference List {#reference-list}

1.  <https://pve.proxmox.com/wiki/Storage%3A_CIFS?utm_source=chatgpt.com>
2.  <https://www.cs.virginia.edu/computing/doku.php?id=project_directories>
