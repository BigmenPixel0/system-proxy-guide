# How to route all traffic through a proxy

**Attention! Everything has been tested only on the ArchLinux distribution! It is possible that the commands will be different.**

First, we need to supply some packages:
```
sudo pacman -S nftables iptables-nft firewalld
```
and redsocks from the AUR:
```
git clone https://aur.archlinux.org/redsocks.git
cd redsocks/
makepkg -sric
```

It is assumed that you already have a local socks5 / http proxy configured. Most likely, this will not work with TOR, because we need to add the IP address of the server to which you want to conduct traffic to the exception, it is impossible to add all the input nodes.

Next, we need to change the redsocks config and bring it to this form:
```
base {
 log_debug = on;
 log_info = on;
 log = "stderr";
 daemon = off;
 redirector = iptables;
}
redsocks {
    // Local IP listen to
    local_ip = 127.0.0.1;
    // Port to listen to
    local_port = 1081;
    // Remote proxy address
    ip = 127.0.0.1;
    port = YOUR_SOCKS5_LOCAL_PORT;
    // Proxy type
    type = socks5;
}
```
You need to change "YOUR_SOCKS5_LOCAL_PORT" to your value.


Save the config and turn on firewalld and redsocks services:
```
sudo systemctl enable --now firewalld.service redsocks.service
```

Now we need to configure firewalld to route traffic on port 1081:
```
sudo firewall-cmd --direct --add-chain ipv4 nat REDSOCKS

sudo firewall-cmd --direct --add-rule ipv4 nat REDSOCKS 0 -d 0.0.0.0/8 -j RETURN
sudo firewall-cmd --direct --add-rule ipv4 nat REDSOCKS 0 -d 10.0.0.0/8 -j RETURN
sudo firewall-cmd --direct --add-rule ipv4 nat REDSOCKS 0 -d 127.0.0.0/8 -j RETURN
sudo firewall-cmd --direct --add-rule ipv4 nat REDSOCKS 0 -d 169.254.0.0/16 -j RETURN
sudo firewall-cmd --direct --add-rule ipv4 nat REDSOCKS 0 -d 172.16.0.0/12 -j RETURN
sudo firewall-cmd --direct --add-rule ipv4 nat REDSOCKS 0 -d 192.168.0.0/16 -j RETURN
sudo firewall-cmd --direct --add-rule ipv4 nat REDSOCKS 0 -d 224.0.0.0/4 -j RETURN
sudo firewall-cmd --direct --add-rule ipv4 nat REDSOCKS 0 -d 240.0.0.0/4 -j RETURN

sudo firewall-cmd --direct --add-rule ipv4 nat REDSOCKS 0 -d 123.123.123.123/32 -j RETURN

sudo firewall-cmd --direct --add-rule ipv4 nat REDSOCKS 0 -p tcp -j REDIRECT --to-ports 1081

sudo firewall-cmd --direct --add-rule ipv4 nat OUTPUT 0 -p tcp -j REDSOCKS
```
Instead of 123.123.123.123, write the IP address of the **proxy server** without changing the port.
Now try curl ifconfig.me. If that worked, run:
```
sudo firewall-cmd --runtime-to-permanent
```
in order to save all changes.
**Enjoy!**
