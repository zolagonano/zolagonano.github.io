---
title: Bypassing Internet Censorship Using SSH
layout: post
author: Zola Gonano
series: Censorship
---

As censorship systems like GFW(Great Firewall of China) and Roskomnadzor(Russia's Federal Service for Supervision of Communications, Information Technology) have evolved, people have fought back and developed numerous tools, methods, and protocols to bypass these firewalls and connect to the open and free internet that most people know.

Among these protocols, there is an underrated protocol named SSH, which happens to be my favorite. SSH has been developed as a secure remote shell protocol, offering features that lie somewhere between Telnet and FTP. It enables communication with your server, execution of commands, file transfer, acting as a DNS server, and port forwarding.

SSH is both a critical and uncommon protocol when it comes to bypassing censorship. The cost of blocking this protocol is significant due to its extensive usage for managing Linux and other Unix-like servers, as well as its being used in SFTP, a secure version of FTP that relies on SSH for encryption. Also, it is not worth the time and resources to block SSH since fewer people use it to bypass censorship compared to protocols like VLESS, VMESS, and others that operate over TLS or QUIC.

The feature we will be using to bypass censorship is Dynamic Port Forwarding, also known as SSH tunneling. It enables you to establish an encrypted connection to a remote server running an SSH server, which can be used as a proxy for your network traffic. By routing your network requests through the established SSH connection, it provides a secure and encrypted tunnel for your communication.

# Requirements

To use SSH dynamic port forwarding, you need a server running an SSH server and an SSH client that supports dynamic port forwarding. On Linux and most Unix-like systems, you can find the OpenSSH package, which acts as both the server and client and supports dynamic port forwarding. In many cases, OpenSSH is already installed by default on desktop machines and is always present on servers. To check if SSH is installed on your machine, run the following command:

```bash
ssh -V
```

If the command returns the OpenSSH version, it means you already have it on your machine. If it is not installed, search for instructions on installing OpenSSH specific to your distribution. On Windows, it may be more complicated, and you might need to install additional software like RespiteVPN or NetMod. I don't know much about Windows.

For the server, you can buy a cheap Linux VPS from websites that accept cryptocurrencies like mvps.net if you prefer to make an anonymous purchase. Alternatively, you can buy it from cloud hosting providers such as DigitalOcean or Hetzner.

# Setting it Up

Once you have your server, it's time to proceed with the setup. To do this, you need to connect to your server using SSH with the username and password provided by the provider or reseller from whom you purchased the VPS.

```bash
ssh username@ip -p port
```

The port is optional. Most servers run SSH on port 22, which is the default port for SSH. If you don't specify a port, it will automatically attempt to connect to port 22.

## Creating Users

Once we have successfully logged in to our server with our privileged user (root), we create a new user to disable root access and enhance security against brute force attacks. This user will have sudo privileges for server management. Additionally, we create a separate user with restricted access specifically for dynamic port forwarding. This allows us to grant access to friends and family without compromising the security of our server.

To create the privileged user, run the following command:

```bash
useradd username -m -g sudo
```

Set a password for the privileged user:

```bash
passwd username
```

Next, create the non-privileged user for dynamic port forwarding:

```bash
useradd tunnel_user -s /bin/rbash
```

Set a password for the non-privileged user:

```bash
passwd tunnel_user
```

For the non-privileged user, I have used the Restricted Bash (rbash) as the default shell. However, you can also utilize the [BlackHole shell]() that I [developed in my previous post](/blog/posts/creating-a-blackhole-shell-with-16lines-of-rust-code).

## Setting up SSHD

Next, we need to modify the `/etc/ssh/sshd_config` file to allow specific ports and disable root access to our server. You can use any command-line-based text editor of your choice, such as vim, vi, neo-vim, emacs, or nano. Use the editor that you are most comfortable with.

To open additional ports other than port 22, which is commonly targeted by censorship probes, add the following lines at the end of the file:

```
PermitRootLogin no
Port 8443
Port 2023
```

In this example, I have disabled root login and opened two additional ports (8443 and 2023). You can add more ports as needed.

After editing the file, you may need to allow the newly added ports through the firewall. Use the following commands to allow the ports:

```bash
ufw allow 8443/tcp
ufw allow 2023/tcp
```

## Blocking unauthorized access

For additional security, we can set up [fail2ban](https://www.fail2ban.org/wiki/index.php/Main_Page) to block unauthorized attempts to our server. The installation process depends on the operating system and distribution of your VPS. If you are using Ubuntu or an Ubuntu-based distribution, you can install the fail2ban package by running the following commands:

```bash
sudo apt update
sudo apt install fail2ban
```

Once installed, fail2ban will set up the necessary background services automatically.

Next, you'll need to create the configuration file at `/etc/fail2ban/jail.local`. To do this, simply copy the default configuration file:

```bash
cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

The default settings are generally reasonable, but if you need to make any changes, you can edit the `/etc/fail2ban/jail.local` file.

After creating the configuration file, restart the fail2ban service and check if it is running properly:

```bash
sudo systemctl restart fail2ban.service
sudo systemctl status fail2ban.service
```

The output should resemble the following:

```yaml
● fail2ban.service - Fail2Ban Service
     Loaded: loaded (/lib/systemd/system/fail2ban.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-06-20 12:39:33 CEST; 3 weeks 5 days ago
       Docs: man:fail2ban(1)
   Main PID: 4218 (f2b/server)
      Tasks: 5 (limit: 1074)
     Memory: 20.8M
     CGroup: /system.slice/fail2ban.service
             └─4218 /usr/bin/python3 /usr/bin/fail2ban-server -xf start
```

## Speeding it Up

To improve the speed of our SSH tunnel, we can enable [TCP BBR (Bottleneck Bandwidth and Round-trip time)](https://cloud.google.com/blog/products/networking/tcp-bbr-congestion-control-comes-to-gcp-your-internet-just-got-faster), which is a congestion control algorithm developed by Google to enhance network performance and bandwidth utilization.

To enable TCP BBR, you can execute the following commands:

```bash
sudo sh -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
sudo sh -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sudo sysctl -p
```

Enabling TCP BBR can significantly enhance the performance of the TCP connection.

![GIF from cloud.google.com](/assets/pics/tcp_bbr.gif)

# Connecting to it

After completing the setup process, we can proceed to connect to our server and bypass censorship. Connecting is quite simple, requiring only one command:

```bash
ssh non_privileged_username@ip -p 2023 -qD 2080
```

This command will create a SOCKS5 proxy on `localhost` or `127.0.0.1` with port `2080`, which we can use to establish a connection. In a future post, I will explain how to set up an OpenVPN over SSH.

## Enable compression

If you have a slower network connection, you can enable SSH compression to reduce bandwidth usage and potentially speed up your connection. To enable SSH compression, simply add the `-C` switch when connecting:

```bash
ssh non_privileged_username@ip -p 2023 -qCD 2080
```

By enabling SSH compression, the data transmitted over the SSH connection will be compressed, resulting in reduced bandwidth usage.

## Automatically reconnect

If you have a connection that frequently disconnects, you can use the [autossh](https://github.com/Autossh/autossh) tool to automatically reconnect when a disconnection occurs. To install `autossh`, you can search for the installation guide for your specific distribution.

Once `autossh` is installed, you can use it as follows:

```bash
autossh -M 0 non_privileged_username@ip -p 2023 -qCD 2080
```

The `-M 0` option disables the monitoring of the connection, allowing `autossh` to continuously attempt to reconnect in case of disconnection. This helps to maintain a persistent SSH tunnel for your bypassing needs.

---

As always, the entire blog is available on my [GitHub](https://github.com/zolagonano/zolagonano.github.io), and I would greatly appreciate it if you could point out any inaccuracies, grammatical errors, technical mistakes, or simply give it a star. Your feedback and contributions are valuable to me.
