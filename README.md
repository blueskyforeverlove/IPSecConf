site1:
->ifconfig 10.1.1.1/16 up
->ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:80:9e:77 brd ff:ff:ff:ff:ff:ff
    inet 192.168.69.245/20 brd 192.168.79.255 scope global dynamic noprefixroute ens33
       valid_lft 38374sec preferred_lft 38374sec
    inet6 fe80::20c:29ff:fe80:9e77/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
4: ens38: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:80:9e:8b brd ff:ff:ff:ff:ff:ff
    inet 10.1.1.1/16 brd 10.1.255.255 scope global noprefixroute ens38
       valid_lft forever preferred_lft forever
    inet6 fe80::9aff:7738:f41a:bc6c/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever

->ip route add 10.1.0.0/16 via 192.168.69.245

->ip route
default via 192.168.64.254 dev ens33 proto dhcp metric 102 
10.1.0.0/16 via 192.168.69.245 dev ens33 
192.168.64.0/20 dev ens33 proto kernel scope link src 192.168.69.245 metric 102


->cp /etc/strongswan/ipsec.conf /etc/strongswan/ipsec.conf.orig
->vi /etc/strongswan/ipsec.conf
# ipsec.conf - strongSwan IPsec configuration file

# basic configuration
config setup
        charondebug="all"
        uniqueids=yes

# Add connections here.
conn gateway1-to-gateway2
        type=tunnel
        keyexchange=ikev2
        authby=secret
        auto=start
        left=192.168.69.245
        leftsubnet=10.1.1.1/16
        right=192.168.70.62
        rightsubnet=10.2.1.1/16
        ike=aes256-sha1-modp1024!
        esp=aes256-sha1!
        aggressive=no
        keyingtries=%forever
        ikelifetime=28800s
        lifetime=3600s
        dpddelay=30s
        dpdtimeout=120s
        dpdaction=restart
site2:
->ifconfig 10.2.1.1/16 up
->ip add
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:67:51:1a brd ff:ff:ff:ff:ff:ff
    inet 192.168.70.62/20 brd 192.168.79.255 scope global dynamic noprefixroute ens33
       valid_lft 38105sec preferred_lft 38105sec
    inet6 fe80::20c:29ff:fe67:511a/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
4: ens38: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:67:51:2e brd ff:ff:ff:ff:ff:ff
    inet 10.2.1.1/16 brd 10.2.255.255 scope global noprefixroute ens38
       valid_lft forever preferred_lft forever
    inet6 fe80::b9ec:e8e5:68d4:dfbf/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
       
->ip route add 10.2.0.0/16 via 192.168.70.62
->ip route
default via 192.168.64.254 dev ens33 proto dhcp metric 101 
10.2.0.0/16 via 192.168.70.62 dev ens33 
192.168.64.0/20 dev ens33 proto kernel scope link src 192.168.70.62 metric 101

->vi /etc/sysctl.conf
net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 1
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
->sysctl -p

->vi /etc/strongswan/ipsec.secrets
# ipsec.secrets - strongSwan IPsec secrets file
192.168.69.245 192.168.70.62 : PSK "PT0+IC8gPD09Cgo9PT4gLyA8PT0K"

->cp /etc/strongswan/ipsec.conf /etc/strongswan/ipsec.conf.orig
->vi /etc/strongswan/ipsec.conf
# ipsec.conf - strongSwan IPsec configuration file

# basic configuration
config setup
        charondebug="all"
        uniqueids=yes

# Add connections here.
conn gateway2-to-gateway1
        type=tunnel
        keyexchange=ikev2
        authby=secret
        auto=start
        left=192.168.70.62
        leftsubnet=10.2.1.1/16
        right=192.168.69.245
        rightsubnet=10.1.1.1/16
        ike=aes256-sha1-modp1024!
        esp=aes256-sha1!
        aggressive=no
        keyingtries=%forever
        ikelifetime=28800s
        lifetime=3600s
        dpddelay=30s
        dpdtimeout=120s
        dpdaction=restart
        
->systemctl enable strongswan
->systemctl start strongswan
->systemctl status strongswan
->strongswan start

site1:ping 10.2.1.1
site2:ping 10.1.1.1
