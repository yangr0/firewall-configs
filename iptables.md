# DDOS Protection Rules

> [!CAUTION]
> Consider backing up and flushing iptables before doing this. Actually read the rules before you apply them and change ports/limits to your liking

#### Flush tables (if you want)
```bash
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT
sudo iptables -t nat -F
sudo iptables -t mangle -F
sudo iptables -F
sudo iptables -X
```

#### /etc/sysctl.conf configs
```bash
net.ipv4.conf.default.rp_filter=1
net.ipv4.conf.all.rp_filter=1
net.ipv4.tcp_syncookies=1
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv6.conf.all.accept_source_route = 0
net.ipv4.conf.all.log_martians = 1
```

#### allow traffic from localhost
```bash
sudo iptables -A INPUT -i lo -j ACCEPT
```

#### port scan trap (use a common port that you DO NOT use)
```bash
sudo iptables -A INPUT -p tcp -m tcp --dport 21 -m state --state NEW -m recent --name banip --rsource --set -j LOG
sudo iptables -A INPUT -m recent --name banip --rsource --update --reap --seconds 300 -j DROP
```

#### drop all null packets (NULL attack)
```bash
sudo iptables -A INPUT -p tcp --tcp-flags ALL NONE -j DROP
```

#### drop all fragment packets (Fragmentation Flood)
```bash
sudo iptables -A INPUT -f -j DROP
```

#### drop malformed xmas packets
```bash
sudo iptables -A INPUT -p tcp --tcp-flags ALL ALL -j DROP
```

#### drop new packets that don't use the syn flag
```bash
sudo iptables -A INPUT -p tcp ! --syn -m state --state NEW -j DROP
```

#### limit syn packets per IP (connection attacks)
```bash
sudo iptables -A INPUT -p tcp --syn -m multiport --dports 80,2200,443,22,21,8388,2005 -m limit --limit 8/second --limit-burst 5 -j ACCEPT
```

#### limit udp packets per IP (udp flood)
```bash
sudo iptables -A INPUT -p udp -m multiport --dports 51820,443,8388 -m limit --limit 50/second --limit-burst 10 -j ACCEPT
```

#### limit icmp echo-request per IP (ping flood)
```bash
sudo iptables -A INPUT -p icmp --icmp-type 8 -m limit --limit 1/second --limit-burst 3 -j ACCEPT
```

#### allow related and established connections
```bash
sudo iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
```

#### drop packets by default
```bash
sudo iptables -P INPUT DROP
```


# DOCKER IPTABLES

#### Only accept connections from host machine (for caddy reverse proxy setup)
```bash
sudo iptables -I DOCKER-USER -j DROP
sudo iptables -I DOCKER-USER -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -I DOCKER-USER -s 172.23.0.1,172.23.0.3 -j ACCEPT
```

#### Make sure iptables is enabled in /etc/docker/daemon.json
```json
{
    ...
    "iptables": true
}
```

#### SAVE IPTABLES #

```bash
sudo netfilter-persistent save
```

# Credits

I took a lot of these rules from these articles and tweaked some of them

<https://www.webhostingtalk.com/showthread.php?t=1366995>

<https://www.cyberciti.biz/tips/linux-iptables-10-how-to-block-common-attack.html>

<https://gist.github.com/mattia-beta/bd5b1c68e3d51db933181d8a3dc0ba64>
