Disclaimer: This is a little tweak for my taste of using WG easy with AdguardHome. Forked from WeeJeWel/wg-easy and fnazz/docker-adguard-unbound-wireguard

Please refer to those origin for details when you want to cook for yourself. 

# WireGuard Easy

[![Build & Publish Docker Image to Docker Hub](https://github.com/WeeJeWel/wg-easy/actions/workflows/deploy.yml/badge.svg?branch=production)](https://github.com/WeeJeWel/wg-easy/actions/workflows/deploy.yml)
[![Lint](https://github.com/WeeJeWel/wg-easy/actions/workflows/lint.yml/badge.svg?branch=master)](https://github.com/WeeJeWel/wg-easy/actions/workflows/lint.yml)
[![Docker](https://img.shields.io/docker/v/weejewel/wg-easy/latest)](https://hub.docker.com/r/weejewel/wg-easy)
[![Docker](https://img.shields.io/docker/pulls/weejewel/wg-easy.svg)](https://hub.docker.com/r/weejewel/wg-easy)
[![Sponsor](https://img.shields.io/github/sponsors/weejewel)](https://github.com/sponsors/WeeJeWel)

You have found the easiest way to install & manage WireGuard on any Linux host!

<p align="center">
  <img src="./assets/screenshot.png" width="802" />
</p>

## Features

* All-in-one: WireGuard + Web UI.
* Easy installation, simple to use.
* List, create, edit, delete, enable & disable clients.
* Show a client's QR code.
* Download a client's configuration file.
* Statistics for which clients are connected.
* Tx/Rx charts for each connected client.
* Gravatar support.

## Requirements

* A host with a kernel that supports WireGuard (all modern kernels).
* A host with Docker installed.

## Installation

### 1. Install Docker
<pre>
$ sudo apt update && sudo apt upgrade -y
$ sudo apt install docker.io
</pre>

### 2. Install Docker Compose
<pre>
$ sudo apt install python3-pip
$ sudo pip install docker-compose
</pre>

Check installation success
<pre>
$ docker-compose --version
</pre>

### 3. Allow non-root to run command without "sudo"
<pre>
$ sudo groupadd docker
$ sudo usermod -aG docker $USER
</pre>

Reboot to apply

### 4. Run WireGuard Easy + Adguard Home:

To automatically install & run wg-easy, simply run:
<pre>
$ mkdir ~/.wg-easy
$ cd ~/.wg-easy
$ wget https://github.com/timmyvo/wg-easy-adguardhome/raw/master/docker-compose.yml
$ nano docker-compose.yml # To edit some input attribute, modify to suit your needs.
</pre>

> 💡 Replace `YOUR_SERVER_IP` with your WAN IP, or a Dynamic DNS hostname.
> 
> 💡 Replace `YOUR_ADMIN_PASSWORD` with a password to log in on the Web UI.

Ctrl + X then Y to save your "docker-compose.yml" file.
Then enter command: 

<pre>
$ docker-compose up --detach
</pre>

The Web UI will now be available on `http://<YOUR_SERVER_IP>:51821`.

> 💡 Your configuration files will be saved in `~/.wg-easy`

### 5. Add rules into firewall in order to allow internet connection:

Open iptables with this command:
<pre>
sudo nano /etc/iptables/rules.v4
</pre>
and add these below lines after row "-A INPUT -p udp -m udp --sport 123 -j ACCEPT":

<pre>
-A INPUT -p tcp --dport 53 -j ACCEPT
-A INPUT -p udp --dport 53 -j ACCEPT
-A INPUT -p udp --dport 51820 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -p udp -m state --state NEW -m udp --dport 51820 -j ACCEPT
</pre>

Ctrl + X then Y to save your "rules.v4" file then run:

<pre>
$ sudo iptables-restore < /etc/iptables/rules.v4
</pre>

to apply

You now can create user with web UI and then connect your PC to config Adguard Home.

### 5. Access Adguard Home:

Do connect you PC with the wireguard and while connected to WireGuard, navigate to http://10.2.0.100:3000 first to setup AdGuard Home before DNS query and adblocking to work.

## Options

These options can be configured by setting environment variables using `-e KEY="VALUE"` in the `docker run` command.

| Env | Default | Example | Description |
| - | - | - | - |
| `PASSWORD` | - | `foobar123` | When set, requires a password when logging in to the Web UI. |
| `WG_HOST` | - | `vpn.myserver.com` | The public hostname of your VPN server. |
| `WG_PORT` | `51820` | `12345` | The public UDP port of your VPN server. WireGuard will always listen on `51820` inside the Docker container. |
| `WG_MTU` | `null` | `1420` | The MTU the clients will use. Server uses default WG MTU. |
| `WG_PERSISTENT_KEEPALIVE` | `0` | `25` | Value in seconds to keep the "connection" open. |
| `WG_DEFAULT_ADDRESS` | `10.8.0.x` | `10.6.0.x` | Clients IP address range. |
| `WG_DEFAULT_DNS` | `1.1.1.1` | `8.8.8.8, 8.8.4.4` | DNS server clients will use. |
| `WG_ALLOWED_IPS` | `0.0.0.0/0, ::/0` | `192.168.15.0/24, 10.0.1.0/24` | Allowed IPs clients will use. |
| `WG_POST_UP` | `...` | `iptables ...` | See [config.js](https://github.com/WeeJeWel/wg-easy/blob/master/src/config.js#L19) for the default value. |
| `WG_POST_DOWN` | `...` | `iptables ...` | See [config.js](https://github.com/WeeJeWel/wg-easy/blob/master/src/config.js#L26) for the default value. |

> If you change `WG_PORT`, make sure to also change the exposed port.

# Updating

To update to the latest version, simply run:

```bash
docker stop wg-easy
docker rm wg-easy
docker pull weejewel/wg-easy
```

And then run the `docker run -d \ ...` command above again.
