---
title: "finger"
tags: ["definition", "linux"]
draft: false
---

## Definition {#definition}

<https://www.howtogeek.com/440391/how-to-use-the-finger-command-on-linux/>
<https://www.tecmint.com/find-user-account-info-and-login-details-in-linux/>


## Examples {#examples}

```bash

awk -F: '{ print $1,$6}' /etc/passwd

awk -F: '{ if ($6 == "/home/"*) print $6;}' /etc/passwd

awk '{ if($3 == "B6") print $0;}' geeksforgeeks.txt

string='My long string'
if [[ $string == *"My long"* ]]; then
  echo "It's there!"
fi

awk '{match($0,/\[([^]]+)\]/, a); print $1,$3,a[1]}' file



awk -F: '{ if (match($6,"^/home")) print $6;}' /etc/passwd

```
