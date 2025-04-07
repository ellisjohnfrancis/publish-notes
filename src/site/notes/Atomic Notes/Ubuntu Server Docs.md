---
{"dg-publish":true,"permalink":"/atomic-notes/ubuntu-server-docs/"}
---

Tags: [[Tags/education\|education]] [[Tags/code\|code]] [[Tags/ubuntu\|ubuntu]] [[Tags/server\|server]]

LiveSync Pass phease: `sharedvault`
# WebDav

Address: http://141.148.50.228/svn

Creds (user & pass): `admin`
# Rustdesk

Relay Server IP: `141.148.50.228`
Relay Server Key: `HpG0s3n+4QngvhMwAtqvo9OelomR856tzJx33aBsDTY=`

Jacinta Laptop ID: `361 995 017`
Apollo ID: `434 858 829`
# Remote VPS

IP: `141.148.50.228`

Dashboard: https://www.oracle.com/cloud/sign-in.html
# SSH

Setting up SSH involves installing the OpenSSH package, configuring the SSH server (if needed), and generating SSH keys for secure authentication. Here's a step-by-step guide depending on what you need:

---
#### **1. Installing SSH**

##### **On Linux/macOS**

Most distributions come with OpenSSH installed by default. To install or verify:

```sh
### Debian/Ubuntu
sudo apt update && sudo apt install openssh-server -y

### Fedora
sudo dnf install openssh-server -y

### Arch Linux
sudo pacman -S openssh
```

To start and enable the SSH service:

```sh
sudo systemctl enable --now ssh  # (or sshd on some distros)
```

##### **On Windows**

1. Open **Settings** → **Apps** → **Optional Features**.
2. Search for "OpenSSH Server" and install it.
3. Start the SSH service with:
    
    ```powershell
    Start-Service sshd
    ```
    

---

#### **2. Generating SSH Keys**

On the client machine (where you will connect from):

```sh
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

- Press **Enter** to save in `~/.ssh/id_rsa`.
- Set a passphrase for extra security (optional).

---

#### **3. Copying SSH Key to Remote Server**

Run:

```sh
ssh-copy-id user@remote-server
```

or manually:

```sh
scp ~/.ssh/id_rsa.pub user@remote-server:~/.ssh/authorized_keys
```

Ensure correct permissions:

```sh
chmod 600 ~/.ssh/authorized_keys
```

---

## **4. Connecting to Remote Server**

Now you can connect using:

```sh
ssh user@remote-server
```

---

## **5. Securing SSH (Optional but Recommended)**

Edit SSH config:

```sh
sudo nano /etc/ssh/sshd_config
```

Set:

```
PermitRootLogin no
PasswordAuthentication no
AllowUsers your_user
```

Restart SSH:

```sh
sudo systemctl restart ssh
```

---

Let me know if you need help with a specific use case! 🚀

# Wake PC on Lan:

```bash
sudo -S <<< "AdminHermesFlowshift9!" etherwake -i eno1 70:85:C2:5A:FD:DC
```

---
# Mounting LVM Drives
###### 1. Scan for LVM Location:

```bash
sudo pvscan
```

###### 2. Scan for Volume Groups:

```bash
sudo vgscan
```

###### 3. List the Logical Volumes:

```bash
sudo lvscan
```
###### 4. Server Mount Point:

```
/mnt/server-storage
```

###### 5. Mount the Logical Volume at desired point:

```bash
sudo mount /dev/vg01/server-storage /mnt/server-storage
```
###### 6. Check if successful by listing mounted drives

```
lsblk
```

###### 7. Add drive mount point to f-stab

1. Find UUID of desired drive

```bash
sudo tune2fs -l /dev/vg01/server-storage | grep UUID
```

3. Open f-stab file

```bash
sudo nano /etc/fstab
```

4. Paste into f-stab

```bash
UUID=DESIRED-DRIVE-UUID /mnt/server-storage ext4 defaults,nofail 0 2
```

5. Apply Changes Without Rebooting

```bash
sudo mount -a
```

6. Verify the Mount

```bash
df -h | grep server-storage
```

##### 8. Mount server on Laptop

```bash
sudo mount -t cifs //hermes/server /mnt/server -o credentials=/etc/smbcredentials,iocharset=utf8,rw,uid=1000,gid=1000
```

---

# Adding Additional Storage to LVM

To add a new drive to an existing LVM setup, follow these steps:

##### 1. Identify the New Drive

First, find the name of the new disk using:

```bash
lsblk
```

Let's assume the new drive is `/dev/sdb`.

##### **2. Create a New Physical Volume (PV)**

Initialize the new drive as an LVM physical volume:

```bash
pvcreate /dev/sdb
```

Verify:

```bash
pvdisplay
```

##### **3. Extend the Volume Group (VG)**

Find your existing volume group name:

```bash
vgdisplay
```

Then, extend it with the new physical volume:

```bash
vgextend vg01 /dev/sdb
```

Verify:

```bash
vgdisplay
```

##### **4. Extend the Logical Volume (LV)**

Find your logical volume name:

```bash
lvdisplay
```

Extend the LV (adjust size as needed):

```bash
lvextend -l +100%FREE /dev/vg01/server-storage
```

or specify a size, e.g., +50G

##### **5. Resize the Filesystem**

```bash
resize2fs /dev/vg01/server-storage
```

##### **6. Verify Changes**

Check new space:

```bash
df -h
```

---

# WireGuard VPN

### Resources:

https://www.procustodibus.com/blog/2020/12/wireguard-site-to-site-config/

https://www.procustodibus.com/blog/2020/10/wireguard-topologies/#site-to-site

https://www.procustodibus.com/blog/2020/11/wireguard-point-to-point-config/

https://www.procustodibus.com/blog/2020/11/wireguard-point-to-site-config/

https://www.procustodibus.com/blog/2021/04/wireguard-point-to-site-routing/


change order of dns resoltion in wg config, include both lan and vpn dns
### SOP:

###### 1. Generate a New Set Of Keys on Ubuntu Server:

```bash
wg genkey | tee /etc/wireguard/clients/jacinta_iphone_PRIVATE.key | wg pubkey >        /etc/wireguard/clients/jacinta _iphone_PUBLIC.key
```

###### 2. Create a New Client Configuration File and Set IP:

```bash
[Interface]
PrivateKey = NEW_CLIENT_NAME_PRIVATE.key
Address = NEW_CLIENT_IP/24
DNS = 10.8.0.1

[Peer]
PublicKey = YXngS9Gl9f2icJViewJHRJtVvygqPlbv17lL6DUY9AY=
AllowedIPs = 10.8.0.0/24
Endpoint = flowshift.duckdns.org:51820
PersistentKeepalive = 25
```

###### 3. Add the client to the VPN server:

```bash
sudo nano /etc/wireguard/wg0.conf
```

```bash
[Peer]
PublicKey = feACopIbHMqP5a33/CQSSlGbiZx7vtfLIdrbRuxm1zc=
AllowedIPs = 10.8.0.10/32
```

###### 4. Restart WireGuard on server:

```bash
sudo systemctl restart wg-quick@wg0
```

###### 5. Use the config file to set up connection on client

###### 6. Check VPN connection on server

```
sudo wg show
```

#### Reference Config Templates:

##### Server:

```
[Interface]
Address = 10.8.0.1/24
Address = fd04:f8b1:c511::/64
DNS = 10.8.0.1
PostUp = ufw route allow in on wg0 out on eno1
PostUp = iptables -t nat -I POSTROUTING -o eno1 -j MASQUERADE
PostUp = ip6tables -t nat -I POSTROUTING -o eno1 -j MASQUERADE
PreDown = ufw route delete allow in on wg0 out on eno1
PreDown = iptables -t nat -D POSTROUTING -o eno1 -j MASQUERADE
PreDown = ip6tables -t nat -D POSTROUTING -o eno1 -j MASQUERADE
ListenPort = 51820
PrivateKey = QMsuue9dw7+nsTmzq4u5fsZaaoSfSNHJrm74fyz1SFE=

[Peer]
PublicKey = txt0mvDj8YFp6d/0fuUR3EY7FoqnZKqsGxB+JPL0H04=
AllowedIPs = 10.8.0.2/32, fd04:f8b1:c511::2/128
Endpoint = 174.209.96.87:7986

[Peer]
PublicKey = 1gUElpGnKMf6Z2v2baxa9khciBg2P45n8z/NIme3tTc=
AllowedIPs = 10.8.0.3/32
Endpoint = 174.209.96.87:7979

[Peer]
PublicKey = R7Mv5wQaghdsyxw0Lu6fuL0rwL6vAjn/D6dYUel8zz8=
AllowedIPs = 10.8.0.4/32
```


##### Client:

```
[Interface]
PrivateKey = mB4Q6yZ2KVLwBinrqG9Au7bJ4byPewx32EBdge5/fWA=
Address = 10.8.0.4/24
DNS = 10.8.0.1

[Peer]
PublicKey = YXngS9Gl9f2icJViewJHRJtVvygqPlbv17lL6DUY9AY=
AllowedIPs = 10.8.0.0/24
Endpoint = flowshift.duckdns.org:51820
PersistentKeepalive = 25
```



#### WireGuard Control:

```bash
wgc start # Connect to wireguard
wgc stop # Disconnect from wireguard
wgc restart # Restart wiregiard
```

###### Script Location:

```bash
cd /usr/local/bin
```

###### Script Contents:

```bash
#!/bin/bash

CONFIG="wg0"  # Change this if your config name is different

if [ "$1" == "start" ]; then
    sudo wg-quick up $CONFIG
    echo "WireGuard started ✅"
elif [ "$1" == "stop" ]; then
    sudo wg-quick down $CONFIG
    echo "WireGuard stopped ⛔"
elif [ "$1" == "restart" ]; then
    sudo wg-quick down $CONFIG
    sudo wg-quick up $CONFIG
    echo "WireGuard restarted 🔄"
else
    echo "Usage: ./wg.sh {start|stop|restart}"
fi
```

###### Make Executable:

```bash
chmod +x wgc

```

---
# Frigate NVR:

Amcrest camera info:

Model Number:  IP2M-841B
IP: `192.168.1.182`
User: `admin`
Pass: `weloverafi1!`

NVR Users:

admin
jacinta (argosisadog)

---
# Git Docs:

To initialize a Git repository in a local folder and pull the `main` branch from your SSH remote repository, follow these steps:

#### 1. **Navigate to Your Project Folder**

Open a terminal and go to the folder where you want to initialize the repository:

```bash
cd /path/to/your/local/folder
```

#### 2. **Initialize Git in the Folder**

Run the following command to initialize Git in the directory:

```bash
git init
```

#### 3. **Add the SSH Remote**

Set the remote repository using SSH:

```bash
git remote add origin git@hermes:my-projects/strapi-next-js-winnoc-mills.git
```

#### 4. **Verify the Remote URL** _(Optional)_

Confirm that the remote was added correctly:

```bash
git remote -v
```

#### 5. **Pull the `main` Branch**

Fetch and pull the `main` branch from the remote repository:

```bash
git pull origin main
```

> **Note:** If you get a warning about pulling into an empty repository, you can set the upstream branch with:

```bash
git pull --set-upstream origin main
```

#### 6. **Check SSH Authentication if Needed**

If you encounter authentication issues, ensure your SSH key is set up correctly:

- Check if your SSH key is added:
    
    ```bash
    ssh-add -l
    ```
    
- If no keys are listed, add your key:
    
    ```bash
    ssh-add ~/.ssh/id_rsa
    ```
    
- Test your connection:
    
    ```bash
    ssh -T git@hermes
    ```
    


#### **7.  Reset Local Repository

##### Force Pull Using `reset --hard`**

```bash
git fetch origin 
git reset --hard origin/main
```

#####  Using Rebase

```bash
git pull --rebase origin main
```

---
# Adding Swap File to Server:

```bash
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
sudo bash -c "echo '/swapfile none swap sw 0 0' >> /etc/fstab"
```

```bash
sudo dd if=/dev/zero of=/swapfile bs=1G count=4 status=progress
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile 
```
```bash
export NODE_OPTIONS="--max-old-space-size=4096"
npm run build
```

If it still fails, try increasing the limit further:

```bash
export NODE_OPTIONS="--max-old-space-size=8192"
npm run build
```

---

# Running A Next.js Project With pm2:

Yes, you **can** run a Next.js **dev server** with `pm2`, but it’s **not recommended** for production because the `dev` mode is meant for development only (it’s slower and uses more resources). However, if you want to use `pm2` for convenience during development, you can do this:

---

#### **Running Next.js in Dev Mode with PM2**

1. **Install pm2 (if not installed yet)**
    
    ```bash
    npm install -g pm2
    ```
    
2. **Start the Next.js dev server**
    
    ```bash
    pm2 start npm --name "nextjs-dev" -- run dev
    ```
    
    - `npm` runs the `dev` script from `package.json`.
    - `--name "nextjs-dev"` gives it a recognizable name.
3. **Check status**
    
    ```bash
    pm2 list
    ```
    
4. **View logs (optional)**
    
    ```bash
    pm2 logs nextjs-dev
    ```
    
5. **Stop the server**
    
    ```bash
    pm2 stop nextjs-dev
    ```
    
6. **Delete the process**
    
    ```bash
    pm2 delete nextjs-dev
    ```
    

---

##### **Should You Use PM2 for Development?**

- ✅ **Useful if you want to keep your dev server running even after disconnecting SSH.**
- ❌ **Not recommended for production** → Instead, use:
    
    ```bash
    pm2 start npm --name "nextjs-prod" -- run start
    ```
    
    (This runs `next build` first, then serves the optimized app.)



#### **To see all running PM2 processes, use:**

```bash
pm2 list
```

This will display all running applications with their **status, memory usage, and CPU usage**.

##### **Other Useful PM2 Commands:**

- **Detailed process info:**
    
    ```bash
    pm2 status
    ```
    
- **View logs for all processes:**
    
    ```bash
    pm2 logs
    ```
    
- **View logs for a specific process:**
    
    ```bash
    pm2 logs <process_name_or_id>
    ```
    
- **Check specific process details:**
    
    ```bash
    pm2 show <process_name_or_id>
    ```


---

# Installing and Using Caddy


#### Install

```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https 
```

```bash
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
```

```bash
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list 
```

```bash
sudo apt update 
```

```bash
sudo apt install caddy
```


#### Setup

###### 1. Edit caddyfile

```bash
sudo nano /etc/caddy/Caddyfile
```

###### 2. Add config for reverse proxy

```bash
winnocmills.ca {
    reverse_proxy localhost:3000
}

winnocmills.ca/admin {
    reverse_proxy localhost:1337
}
```

###### 3. Restart

``` bash
sudo systemctl restart caddy
```

###### 6. Check status

```bash
sudo systemctl status caddy
```

---
# NextDNS

Config Access:

```
https://my.nextdns.io/
```

DNS Servers:

```
45.90.28.12
```

```
45.90.30.12
```

---
# Samba

#### **1️. Install Samba**

Run the following command to install Samba:

```bash
sudo apt update
sudo apt install samba -y
```

---
#### **2. Configure Samba**

Edit the **Samba configuration file**:

```bash
sudo nano /etc/samba/smb.conf
```

Add a SMB share:

```ini
[Shared]
   path = /srv/samba/shared
   browseable = yes
   writable = yes
   guest ok = yes
   create mask = 0777
   directory mask = 0777
```

---

#### **3. Restart Samba Service**

Apply changes by restarting Samba:

```bash
sudo systemctl restart smbd
```

Check if Samba is running:

```bash
sudo systemctl status smbd
```

---

#### **4. Allow Samba in Firewall (if enabled)**

```bash
sudo ufw allow samba
```

---

#### **6. Set Up User Authentication**

If you want to require login, create a Samba user:

```bash
sudo smbpasswd -a your_username
```

Then, modify your **`smb.conf`** entry like this:

```ini
[Shared]
   path = /srv/samba/shared
   browseable = yes
   writable = yes
   valid users = your_username
```

Restart Samba again:

```bash
sudo systemctl restart smbd
```

---

#### **6. Access the SMB share**

- **From Windows:** Open `\\your-ubuntu-ip\Shared` in File Explorer.
- **From another Linux machine:**
    
    ```bash
    smbclient //your-ubuntu-ip/Shared -U your_username
    ```


---
# DNSmasq

#### **Summary of Basic Commands:**

1. **Install dnsmasq**: `sudo apt install dnsmasq`
2. **Configure dnsmasq**:  `sudo nano /etc/dnsmasq.conf`
3. **Restart dnsmasq**: `sudo systemctl restart dnsmasq`
4. **Check status**: `sudo systemctl status dnsmasq`
5. **Test**: `dig` or `nslookup`

#### 1. **Install dnsmasq**

First, you need to install `dnsmasq`:

```bash
sudo apt-get update
sudo apt-get install dnsmasq
```

This installs the DNS and DHCP server `dnsmasq`.

#### 2. **Configure dnsmasq**

By default, `dnsmasq` works out of the box for most basic uses, but you may want to customize its configuration. The configuration file for `dnsmasq` is located at `/etc/dnsmasq.conf`.

##### Edit the configuration:

```bash
sudo nano /etc/dnsmasq.conf
```

Some common configurations you might add:

**DNS Forwarding:** Forward DNS queries to an upstream DNS server. For example, you can configure `dnsmasq` to use Google’s DNS servers (8.8.8.8 and 8.8.4.4):

```ini
server=8.8.8.8
server=8.8.4.4
```

**Local DNS Entries:** You can set static DNS records for local addresses, domains, and subdomains :

```ini
address=/mydomain.local/192.168.1.100
```

```
address=/drive.flowshiftmedia.com/192.168.1.100
```

```
address=/flowshiftmedia.com/192.168.1.100
```


**DHCP Range Configuration:** To configure `dnsmasq` as a DHCP server, you can specify a DHCP range. For example, for IPs in the range `192.168.1.100` to `192.168.1.150`:

```ini
dhcp-range=192.168.1.100,192.168.1.150,12h
```

 **Set DHCP Hostnames:** You can also define a hostname for each DHCP client, either by MAC address or by IP address:
 
```ini
dhcp-host=00:11:22:33:44:55,mydevice,192.168.1.101
```

(This ties the MAC address `00:11:22:33:44:55` to the hostname `mydevice` and assigns it the IP `192.168.1.101`.)

**DNS Caching:** By default, `dnsmasq` caches DNS queries. You can configure the cache size:

```ini
cache-size=1000
```

(This will set the cache size to 1000 entries.)

#### 3. **Restart dnsmasq**

After editing the configuration, you’ll need to restart `dnsmasq` for the changes to take effect:

```bash
sudo systemctl restart dnsmasq
```

You can check the status to ensure it’s running correctly:

```bash
sudo systemctl status dnsmasq
```

#### 4. **Configure Network to Use dnsmasq**

For `dnsmasq` to function as your DNS server, you need to configure your network or device to use it. If you’re running `dnsmasq` on your local machine, configure your system to use it for DNS resolution.

##### On Ubuntu, you can edit the `/etc/resolv.conf` file to use `localhost` (127.0.0.1) as the DNS server:

```bash
sudo nano /etc/resolv.conf
```

Add this line to the file:

```ini
nameserver 127.0.0.1
```

You can also configure your router or other devices on the network to use your `dnsmasq` server by pointing their DNS settings to your server’s IP address.

#### 5. **Testing dnsmasq**

To test that `dnsmasq` is working correctly, you can use the `dig` or `nslookup` command.

```bash
dig mydomain.local
```

This should return the IP address you specified in the `dnsmasq.conf` file.

You can also check that your device is getting an IP from the DHCP server (if you have DHCP set up):

```bash
ifconfig
```

Look for the IP address assigned to your interface. If you're using `dnsmasq` as a DHCP server, the IP address should be within the specified range.

#### 6. **Useful `dnsmasq` Commands**

- **Check the dnsmasq log:**
    
    If you have logging enabled in `/etc/dnsmasq.conf`, you can view the logs for troubleshooting.
    
    ```bash
    tail -f /var/log/syslog
    ```
    
- **Manually restart dnsmasq**:
    
    ```bash
    sudo systemctl restart dnsmasq
    ```
    
- **Stop dnsmasq**:
    
    ```bash
    sudo systemctl stop dnsmasq
    ```
    
- **Disable dnsmasq at startup**:
    
    ```bash
    sudo systemctl disable dnsmasq
    ```
    






#### 7. **Redirect All Requests for a Subdomain to an IP**

You can redirect all subdomains of a particular domain to a specific IP address. This is helpful if you want to direct all subdomains of `example.com` to a local or remote server.

##### Example:

```ini
address=/example.com/192.168.1.100
address=/.example.com/192.168.1.100
```

This reroutes all subdomains under `example.com` to the IP `192.168.1.100`.

#### 8. **Redirect a Specific Domain to a Different DNS Server (for Forwarding)**

If you want to redirect DNS queries for a domain to a different DNS server (for example, for external domains), you can use the `server` directive to forward those queries.

##### Example:

```ini
server=/example.com/8.8.8.8
```

This tells `dnsmasq` to forward queries for `example.com` to Google's DNS server (`8.8.8.8`).


#### 9. **Additional Advanced Options**

##### Wildcard DNS Redirection

If you want to catch all DNS queries for a domain (including subdomains), you can use the wildcard `*`:

```ini
address=/.example.com/192.168.1.100
```

This catches any request for `anysubdomain.example.com` and reroutes it to `192.168.1.100`.

---

