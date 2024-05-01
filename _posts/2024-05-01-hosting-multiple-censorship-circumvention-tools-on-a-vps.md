---
title: Hosting Multiple Censorship Circumvention Tools on a VPS
layout: post
series: Censorship
---

This is a guide for those who live under heavy internet censorship and restrictions and want to host their own censorship circumvention tools and services to bypass the firewalls and access the free internet. To understand how these censorship systems and firewalls work, you can check out my previous post by [clicking here.](./how-governments-detect-and-block-your-internet-traffic)

To host your own VPN and Proxy services, you will need a VPS with  unrestricted internet access (and in some extreme cases a VPS inside  your country to communicate with that VPS with unrestricted access,  which I'll explain why and how later in this post).

## Using Sing-box to create our proxy servers

Sing-box is a fast universal proxy platform with straightforward configuration. It can be used to run many services like Trojan, VMess, VLess, Shadowsocks, Hysteria, TUIC, and other censorship circumvention tools.

For this post, I'll be creating a Shadowsocks server and a Trojan server using Sing-box. However, you can check [Sing-box's configuration guide](https://sing-box.sagernet.org/configuration/) and choose the protocol that works best for you.

For the Trojan server, we will need a domain name with a certificate. If the domain name is not available, we can generate a self-signed certificate instead.

To create the self-signed certificates, run these `openssl` commands:

```bash
openssl genrsa -out privkey.pem 2048
openssl req -new -key privkey.pem -out csr.pem
openssl x509 -req -days 365 -in csr.pem -signkey privkey.pem -out fullchain.pem
```

This should generate two files, `privkey.pem` and `fullchain.pem`, which we will use for TLS encryption in Sing-box.

Now, let's create the Sing-box config file:

```json
{
  "inbounds": [
    {
      "type": "trojan",
      "listen": "127.0.0.1",
      "listen_port": 4431,
      "users": [
        {
          "name": "example",
          "password": "password"
        }
      ],
      "tls": {
        "enabled": true,
        "server_name": "example.org",
        "key_path": "privkey.pem",
        "certificate_path": "fullchain.pem"
      }
    },
    {
      "type": "shadowsocks",
      "listen": "127.0.0.1",
      "listen_port": 4432,
      "network": "tcp",
      "method": "2022-blake3-aes-128-gcm",
      "password": "<server_password>",
      "users": [
        {
          "name": "username",
          "password": "<user_password>"
        }
      ]
    }
  ]
}
```

Now, our config file is ready. It will run a Shadowsocks server for us on `127.0.0.1:4432` and a Trojan server on `127.0.0.1:4431`, which we will redirect traffic to using HAProxy.

To run our Sing-box server, we need to execute the Sing-box command:

```bash
sing-box run -c config.json
```

Optionally, you can create a systemd service for it as well to make it automatically run on boot:

To set up the systemd service, create and open a file at `/etc/systemd/system/sing-box.service` and add the following configuration (change the paths according to where your config file and Sing-box executable are located):

```ini
[Unit]
Description=Sing-box Service
After=network.target

[Service]
Type=simple
ExecStart=/path/to/sing-box run -c /path/to/config.json
WorkingDirectory=/path/to/sing-box
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

Then, reload the systemd services and enable and start the Sing-box service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable sing-box.service --now
```

## Setting-Up HAProxy to host multiple services on one Port

Most of the time, you can only use common ports like 443 (which is for TLS and it's not suspicious to have encrypted communications over this port). So, we might need to run all of our services on this port. HAProxy can help us achieve that.

[HAProxy](https://www.haproxy.org/) is a high-performance TCP/HTTP load balancer. It can spread requests across multiple servers and separate requests based on things like SNI and headers.

Here is the config file for HAProxy, which takes requests from port 443 and reroutes them based on the SNI of the request to our Trojan or Shadowsocks server:

```
frontend https
    bind <server_ip>:443
    mode tcp
    tcp-request inspect-delay 5s
    tcp-request content accept if { req_ssl_hello_type 1 }

    use_backend trojan_server if { req_ssl_sni -i example.org }
    default_backend shadowsocks_server

backend trojan_server
    mode tcp
    server trojan_server 127.0.0.1:4431

backend shadowsocks_server
    mode tcp
    server shadowsocks_server 127.0.0.1:4432

```

You need to add this to the end of your HAProxy config file located at `/etc/haproxy/haproxy.cfg`.

Then, you can start and enable HAProxy:

```bash
systemctl enable --now haproxy
```

## Now your proxies should be ready to use

After setting up Sing-box and HAProxy, you can now access your proxies on the same port and hopefully bypass censorship. Additionally, you can add more services like obfs4 and OpenVPN (to be used with Stunnel or obfs4 if it's blocked) and other stuff on the same port and the same VPS using HAProxy.

---

My blog and all of its content are available under the `CC by SA 4.0` License on my [GitHub](https://github.com/zolagonano/zolagonano.github.io). If you notice any problems or have any improvements for the blog or its content, you're always welcome to open a pull request.
