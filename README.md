# firewall-configs

These are the firewalls configs I have for my VPS

ENJOYY!!!!!!!

<details>
<summary><h1>IPv4 Rules</h1></summary>

```
### DDOS Protection Rules

##################
Consider backing up and flushing iptables before doing this
Actually read the rules before you apply them and change ports/limits to your liking
####################

# Flush tables (if you want)
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT
sudo iptables -t nat -F
sudo iptables -t mangle -F
sudo iptables -F
sudo iptables -X

# /etc/sysct.conf configs
net.ipv4.conf.default.rp_filter=1
net.ipv4.conf.all.rp_filter=1
net.ipv4.tcp_syncookies=1
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv6.conf.all.accept_source_route = 0
net.ipv4.conf.all.log_martians = 1

# allow traffic from localhost
sudo iptables -A INPUT -i lo -j ACCEPT

# drop all null packets (NULL attack)
sudo iptables -A INPUT -p tcp --tcp-flags ALL NONE -j DROP

# drop all fragment packets (Fragmentation Flood)
sudo iptables -A INPUT -f -j DROP

# drop malformed xmas packets
sudo iptables -A INPUT -p tcp --tcp-flags ALL ALL -j DROP

# drop new packets that don't use the syn flag
sudo iptables -A INPUT -p tcp ! --syn -m state --state NEW -j DROP

# limit syn packets per IP (connection attacks)
sudo iptables -A INPUT -p tcp --syn -m multiport --dports 2200,443,22 -m limit --limit 10/second --limit-burst 20 -j ACCEPT

# limit udp packets per IP (udp flood)
sudo iptables -A INPUT -p udp -m multiport --dports 2200,443,22 -m limit --limit 15/second --limit-burst 50 -j ACCEPT

# limit icmp echo-request per IP (ping flood)
sudo iptables -I INPUT 7 -p icmp --icmp-type 8 -m limit --limit 1/second --limit-burst 3 -j ACCEPT

# allow traffic on these ports (my public services)
sudo iptables -A INPUT -p tcp -m multiport --dports 2200,443,22 -j ACCEPT

# allow related and established connections
sudo iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT

# drop packets by default
sudo iptables -P INPUT DROP


###################
# DOCKER IPTABLES #
###################

# Only accept connections from host machine (for caddy reverse proxy setup)
sudo iptables -I DOCKER-USER ! -s <HOST-PRIVATE-IP> -j DROP

# allow related and established connections
sudo iptables -A DOCKER-USER -m state --state RELATED,ESTABLISHED -j ACCEPT

# Make sure iptables is enabled in /etc/docker/daemon.json
{"iptables": true}

# Credits
# I took a lot of these rules from these articles and tweaked some of them
https://www.webhostingtalk.com/showthread.php?t=1366995
https://www.cyberciti.biz/tips/linux-iptables-10-how-to-block-common-attack.html
https://gist.github.com/mattia-beta/bd5b1c68e3d51db933181d8a3dc0ba64
```

</details>

<details>
<summary><h1>IPv6 Rules</h1></summary>

```
### DDOS Protection Rules

###
Consider backing up and flushing ip6tables before doing this
Actually read the rules before you apply them and change ports/limits to your liking
###

# Flush tables (if you want)
sudo ip6tables -P INPUT ACCEPT
sudo ip6tables -P FORWARD ACCEPT
sudo ip6tables -P OUTPUT ACCEPT
sudo ip6tables -t nat -F
sudo ip6tables -t mangle -F
sudo ip6tables -F
sudo ip6tables -X

# /etc/sysct.conf configs
net.ipv4.conf.default.rp_filter=1
net.ipv4.conf.all.rp_filter=1
net.ipv4.tcp_syncookies=1
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv6.conf.all.accept_source_route = 0
net.ipv4.conf.all.log_martians = 1

# allow traffic from localhost
sudo ip6tables -A INPUT -i lo -j ACCEPT

# drop all null packets (NULL attack)
sudo ip6tables -A INPUT -p tcp --tcp-flags ALL NONE -j DROP

# drop all fragment packets (Fragmentation Flood)
sudo ip6tables -A INPUT -m frag -j DROP

# drop malformed xmas packets
sudo ip6tables -A INPUT -p tcp --tcp-flags ALL ALL -j DROP

# drop new packets that don't use the syn flag
sudo ip6tables -A INPUT -p tcp ! --syn -m state --state NEW -j DROP

# limit syn packets per IP (connection attacks)
sudo ip6tables -A INPUT -p tcp --syn -m multiport --dports 2200,443,22 -m limit --limit 10/second --limit-burst 20 -j ACCEPT

# limit udp packets per IP (udp flood)
sudo ip6tables -A INPUT -p udp -m multiport --dports 2200,443,22 -m limit --limit 15/second --limit-burst 50 -j ACCEPT

# limit icmp echo-request per IP (ping flood)
sudo ip6tables -I INPUT 7 -p icmpv6 --icmpv6-type 8 -m limit --limit 1/second --limit-burst 3 -j ACCEPT

# allow traffic on these ports (my public services)
sudo ip6tables -A INPUT -p tcp -m multiport --dports 2200,443,22 -j ACCEPT

# allow related and established connections
sudo ip6tables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT

# drop packets by default
sudo ip6tables -P INPUT DROP

# Credits
# I took a lot of these rules from these articles and tweaked some of them
https://www.webhostingtalk.com/showthread.php?t=1366995
https://www.cyberciti.biz/tips/linux-ip6tables-10-how-to-block-common-attack.html
https://gist.github.com/mattia-beta/bd5b1c68e3d51db933181d8a3dc0ba64
```

</details>
