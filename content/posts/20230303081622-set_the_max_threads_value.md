---
title: "Set the max threads value"
draft: false
---

There are two ways to set the maximum number of threads:


## Editing the /etc/security/limits.conf file {#editing-the-etc-security-limits-dot-conf-file}

You can also set the maximum number of threads by editing the /etc/security/limits.conf file. This file contains a list of resource limits for different users and groups. To set the maximum number of threads for a specific user, add the following line to the file:

```file
<username> hard nproc <max_threads_value>
```

Replace &lt;username&gt; with the desired user, and &lt;max_threads_value&gt; with the desired maximum number of threads.

or,

```file
/* hard nproc <max_threads_value>
```

After making changes to the /etc/security/limits.conf file, you may need to log out and log back in for the changes to take effect.


## Using the **ulimit** command {#using-the-ulimit-command}

The ulimit command is used to set various resource limits for a user or a process. To set the maximum number of threads for a process, you can use the following command:

```bash
ulimit -u <max_threads_value>
```

Replace &lt;max_threads_value&gt; with the desired maximum number of threads.

Note that the maximum number of threads allowed can vary based on the system configuration, and the value specified may not always be achievable.


### Get ulimit setting value {#get-ulimit-setting-value}

You can use the ulimit command with the -a option to display the current resource limits for the shell and its child processes. To display the current maximum number of threads limit, you can use the following command:

```bash
ulimit -a | grep "max user processes"
```

This will display the current maximum number of threads limit in the output.

Alternatively, you can use the ulimit command with the -u option to display the current maximum number of threads limit directly. For example:

```bash
ulimit -u
```

This will display the current maximum number of threads limit for the current shell session. If you want to check the limit for a specific user, you can switch to that user and run the command.


## Reference List {#reference-list}

1.  <https://stackoverflow.com/questions/344203/maximum-number-of-threads-per-process-in-linux>
