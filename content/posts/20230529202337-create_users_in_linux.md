---
title: "Create Users in Linux"
draft: false
---

## useradd Command {#useradd-command}

The general syntax for the useradd command is as follows:

```bash
useradd [OPTIONS] USERNAME
```

Only root or users with sudo privileges can use the useradd command to create new user accounts.

When invoked, useradd creates a new user account according to the options specified on the command line and the default values set in the /etc/default/useradd file.

The variables defined in this file differ from distribution to distribution, which causes the useradd command to produce different results on different systems.

useradd also reads the content of the /etc/login.defs file. This file contains configuration for the shadow password suite such as password expiration policy, ranges of user IDs used when creating system and regular users, and more.


## How to Create a New User in Linux {#how-to-create-a-new-user-in-linux}

To create a new user account, invoke the useradd command followed by the name of the user.

For example to create a new user named username you would run:

```bash
sudo useradd username
```

When executed without any option, useradd creates a new user account using the default settings specified in the /etc/default/useradd file.
The command adds an entry to the /etc/passwd, /etc/shadow, /etc/group and /etc/gshadow files.

To be able to log in as the newly created user, you need to set the user password. To do that run the passwd command followed by the username:

```bash
sudo passwd username
```

You will be prompted to enter and confirm the password. Make sure you use a strong password.

```console

Changing password for user username.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```


## How to Add a New User and Create Home Directory {#how-to-add-a-new-user-and-create-home-directory}

On most Linux distributions, when creating a new user account with useradd, the user’s home directory is not created.

Use the -m (--create-home) option to create the user home directory as /home/username:

```bash
sudo useradd -m username
```

The command above creates the new user’s home directory and copies files from /etc/skel directory to the user’s home directory. If you list the files in the /home/username directory, you will see the initialization files:

```console
ls -la /home/username/
drwxr-xr-x 2 username username 4096 Dec 11 11:23 .
drwxr-xr-x 4 root     root     4096 Dec 11 11:23 ..
-rw-r--r-- 1 username username  220 Apr  4  2018 .bash_logout
-rw-r--r-- 1 username username 3771 Apr  4  2018 .bashrc
-rw-r--r-- 1 username username  807 Apr  4  2018 .profile
```

Within the home directory, the user can write, edit and delete files and directories.


## Adding User to the [sudo]({{< relref "20230616143930-sudo.md" >}}) Group {#adding-user-to-the-sudo--20230616143930-sudo-dot-md--group}

On Ubuntu, the easiest way to grant sudo privileges to a user is by adding the user to the “sudo” group. Members of this group can execute any command as root via sudo and prompted to authenticate themselves with their password when using sudo.

We’re assuming that the user already exists. If you want to create a new user, check this guide.

To add the user to the group run the command below as root or another sudo user. Make sure you change “username” with the name of the user that you want to grant permissions to.

```bash
usermod -aG sudo username
```

Granting sudo access using this method is sufficient for most use cases.

To ensure that the user has sudo privileges, run the whoami command:

```bash
sudo whoami
```

You will be prompted to enter the password. If the user has sudo access, the command will print “root”:

```file
root
```

If you get an error saying “user is not in the sudoers file”, it means that the user doesn’t have sudo privileges.


## Remove a Linux user {#remove-a-linux-user}

Switch to the root user:

```bash
sudo su -
```

Use the userdel command to remove the old user:

```bash
userdel user's username
```

Optional: You can also delete that user's home directory and mail spool by using the -r flag with the command:

```bash
userdel -r user's username
```


## Reference List {#reference-list}

1.  <https://linuxize.com/post/how-to-create-users-in-linux-using-the-useradd-command/>
2.  <https://ostechnix.com/add-delete-and-grant-sudo-privileges-to-users-in-arch-linux/>
