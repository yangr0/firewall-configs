# DDOS Protection Rules

> [!CAUTION]
> Consider backing up and flushing ip6tables before doing this. Actually read the rules before you apply them and change ports/limits to your liking

#### Flush tables (if you want)
```bash
sudo ip6tables -P INPUT ACCEPT
sudo ip6tables -P FORWARD ACCEPT
sudo ip6tables -P OUTPUT ACCEPT
sudo ip6tables -t nat -F
sudo ip6tables -t mangle -F
sudo ip6tables -F
sudo ip6tables -X
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
sudo ip6tables -A INPUT -i lo -j ACCEPT
```

#### port scan trap (use a common port that you DO NOT use)
```bash
sudo ip6tables -A INPUT -p tcp -m tcp --dport 21 -m state --state NEW -m recent --name banip --rsource --set -j LOG
sudo ip6tables -A INPUT -m recent --name banip --rsource --update --seconds 300 --reap -j DROP
```

#### drop all null packets (NULL attack)
```bash
sudo ip6tables -A INPUT -p tcp --tcp-flags ALL NONE -j DROP
```

#### drop all fragment packets (Fragmentation Flood)
```bash
sudo ip6tables -A INPUT -m frag -j DROP
```

#### drop malformed xmas packets
```bash
sudo ip6tables -A INPUT -p tcp --tcp-flags ALL ALL -j DROP
```

#### drop new packets that don't use the syn flag
```bash
sudo ip6tables -A INPUT -p tcp ! --syn -m state --state NEW -j DROP
```

#### limit syn packets per IP (connection attacks)
```bash
sudo ip6tables -A INPUT -p tcp --syn -m multiport --dports 2200,443,22,21,8388,2005 -m limit --limit 8/second --limit-burst 5 -j ACCEPT
```

#### limit udp packets per IP (udp flood)
```bash
sudo ip6tables -A INPUT -p udp -m multiport --dports 51820,443,8388 -m limit --limit 50/second --limit-burst 10 -j ACCEPT
```

#### limit icmp echo-request per IP (ping flood)
```bash
sudo ip6tables -I INPUT 7 -p icmpv6 --icmpv6-type 8 -m limit --limit 1/second --limit-burst 3 -j ACCEPT
```

#### allow related and established connections
```bash
sudo ip6tables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
```

#### drop packets by default
```bash
sudo ip6tables -P INPUT DROP
```

# SAVE IPTABLES

```bash
sudo netfilter-persistent save
```

# Credits

I took a lot of these rules from these articles and tweaked some of them

<https://www.webhostingtalk.com/showthread.php?t=1366995>

<https://www.cyberciti.biz/tips/linux-ip6tables-10-how-to-block-common-attack.html>

<https://gist.github.com/mattia-beta/bd5b1c68e3d51db933181d8a3dc0ba64>
