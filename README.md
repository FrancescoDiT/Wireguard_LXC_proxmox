# Creating a container LXC with a WireguardVPN server
## **REQUIREMENTS**
* **OS**: a Debian template (you can find it by clicking the button "Templates" in *Datacenter* **>** *your_node_name* **>** *local(your_node_name)* **>** *CT Templates*)
  
(I'm using *debian-12-standard_12.7-1_amd64.tar.zst*).

* **Container Hardware**
  * **CPU**: 1 core
  * **Memory**: 128MB
  * **Bootdisk**: 2GB

### 1. **Setup your container configuration file**

In your pve shell, run :

```bash
nano /etc/pve/lxc/<your_container_id>.conf
```

Then add :

```ini
 lxc.cgroup2.devices.allow: c 10:200 rwm
 lxc.mount.entry: /dev/net dev/net none bind,create=dir
```

Save and exit and run :
```bash
chown 100000:100000 /dev/net/tun
```

Now start your container :

```bash
pct start 123
```

### 2. **Download Wireguard and run it**

In your container shell run :

```bash
apt update && apt upgrade -y
```

then install wireguard :
```bash
wget https://git.io/wireguard -O wireguard-install.sh && bash wireguard-install.sh
```

* Go through the guided configuration setup;

* In the end you will have your qr code for mobile devices (don't use it if you want to add a DDNS);

* now you need to open the UDP port 51820 on your router.

### 3. **Add a DDNS (optional for thoose who don't have a static IP)**

Follow my guide to setup a [DDNS LXC container](https://github.com/FrancescoDiT/DDNS_LXC_proxmox "DDNS LXC Container setup")

Open your peer configuration file and change the **Endpoint** value:

```bash
nano your_peer_name.conf
```

```ini
[Interface]
Address = auto_assigned_ip/24 #don't change this
DNS = dns_choose_during_setup #don't change this
PrivateKey = private_key #don't change this

[Peer]
PublicKey = public_key #don't change this
PresharedKey = preshared_key #don't change this
AllowedIPs = ips_you_want_to_allow #change only if you know what you do 
Endpoint = subdomain.domain:51820 #---change this to your DDNS subdomain---
PersistentKeepalive = persistent_keep_alive #don't change this
```

regenerate the QR code for your client

```bash
sudo apt install qrencode -y
```

```bash
qrencode -t ansiutf8 < your_client.conf
```

scan it with your device to add the tunnel

### 4. **Create a new peer**
