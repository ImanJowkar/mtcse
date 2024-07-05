# MTCSE (MiKroTik Certified Security Engineer)

## Steps to secure Mikrotik:

* `always update your router-os to the latest stable release.`


* `If the bandwidth test server is not being used, it should be disabled.`
```
tool/bandwidth-server/set enabled=no
```

* `disable mac ping `
```
/tool mac-server ping
set enabled=no
```

* `if you're not using mac-telnet then disable it.`


* `Always generate a new user with administrative privileges and remove the default admin user.`


* `if you're not using RoMon then disable it. by default this service is disable.`


* `If your MikroTik serves as a DNS server within your network, block incoming DNS queries from the WAN interface.`
![img](img/1.PNG)

```
/ip firewall filter
add action=drop chain=input dst-port=53 in-interface=WAN-Interface protocol=udp

```

* `disable unused interfaces`

* `always use a syslog server and send logs to a syslog server`


[step by step to install rsyslog on rocky linux](https://linuxhint.com/setup-syslog-rocky-linux-9/)

```
/system logging action
add name=SyslogServerRockyLinux remote=192.168.56.102 src-address=192.168.56.105 target=remote

/system logging
set 0 action=SyslogServerRockyLinux
set 1 action=SyslogServerRockyLinux
set 2 action=SyslogServerRockyLinux

```

## packet-flow
It is good to understand the packet flow in MikroTik devices.
![img](img/packetflow-digram.PNG)
------------- ---------- 
![img](img/Bridging.jpg)
-------------- ----------
![img](img/MPLS.jpg)
-------------- ----------
![img](img/Routing.jpg)


-------------- -----------
![img](img/chains.png)

It is advantageous to discard packets in the raw table as it consumes fewer resources on the device.


`MikroTik suggests adding the following policy in the INPUT chain  to your router for enhanced security.`
```

/ip firewall filter
add action=accept chain=input connection-state=established,related
add action=accept chain=input src-address-list=allowed_to_router
add action=accept chain=input protocol=icmp
add action=drop chain=input

/ip firewall address-list
add address=10.10.10.1-10.10.10.30 list=allowed_to_router

```

## Invalid WAN IP addresses
`The following address list is invalid in the WAN network.`

```
/ip firewall address-list
add address=0.0.0.0/8 comment=RFC6890 list=not_in_internet
add address=172.16.0.0/12 comment=RFC6890 list=not_in_internet
add address=192.168.0.0/16 comment=RFC6890 list=not_in_internet
add address=10.0.0.0/8 comment=RFC6890 list=not_in_internet
add address=169.254.0.0/16 comment=RFC6890 list=not_in_internet
add address=127.0.0.0/8 comment=RFC6890 list=not_in_internet
add address=224.0.0.0/4 comment=Multicast list=not_in_internet
add address=198.18.0.0/15 comment=RFC6890 list=not_in_internet
add address=192.0.0.0/24 comment=RFC6890 list=not_in_internet
add address=192.0.2.0/24 comment=RFC6890 list=not_in_internet
add address=198.51.100.0/24 comment=RFC6890 list=not_in_internet
add address=203.0.113.0/24 comment=RFC6890 list=not_in_internet
add address=100.64.0.0/10 comment=RFC6890 list=not_in_internet
add address=240.0.0.0/4 comment=RFC6890 list=not_in_internet
add address=192.88.99.0/24 comment="6to4 relay Anycast [RFC 3068]" list=not_in_internet



```

`now add below firewall rules for your clients.`
```
/ip firewall filter
add action=fasttrack-connection chain=forward comment=FastTrack connection-state=established,related
add action=accept chain=forward connection-state=established,related
add action=drop chain=forward connection-state=invalid log=yes log-prefix=invalid
add action=drop chain=forward dst-address-list=not_in_internet in-interface=bridge1 log=yes log-prefix=!public_from_LAN out-interface=!bridge1
add action=drop chain=forward connection-nat-state=!dstnat connection-state=new in-interface=ether1 log=yes log-prefix=!NAT
add action=drop chain=forward in-interface=ether1 log=yes log-prefix=!public src-address-list=not_in_internet
add action=drop chain=forward in-interface=bridge1 log=yes log-prefix=LAN_!LAN src-address=!192.168.88.0/24


```


## port-scan
`The following rules are employed to identify the source IP address attempting to port-scan our router and block it`
```
/ip firewall mangle
add action=add-src-to-address-list address-list=port-scan address-list-timeout=\
    1d chain=prerouting protocol=tcp psd=21,3s,3,1
/ip firewall raw
add action=drop chain=prerouting src-address-list=port-scan

# or we can use action=tarpit
```
its good to drop packets in `raw table` like above.



# detect vpn login failure more than 3 times
`Identify the source IP that is making repeated attempts with incorrect usernames and passwords through VPNS.`

```
# find vpns login failure.

/ip firewall mangle
add action=add-dst-to-address-list address-list=vpn-authentication-faild3 \
    address-list-timeout=1h1m chain=output comment=\
    vpn-authentication-failure-finder3 content="authentication failed" \
    dst-address-list=vpn-authentication-faild2
add action=add-dst-to-address-list address-list=vpn-authentication-faild2 \
    address-list-timeout=1m chain=output comment=\
    vpn-authentication-failure-finder2 content="authentication failed" \
    dst-address-list=vpn-authentication-faild1
add action=add-dst-to-address-list address-list=vpn-authentication-faild1 \
    address-list-timeout=1m chain=output comment=\
    vpn-authentication-failure-finder1 content="authentication failed"

/ip firewall filter
add action=drop chain=input src-address-list=vpn-authentication-faild3


```



## DDOS attack finder and block

![img](img/ddos.png)

```

/ip firewall filter
add action=jump chain=input connection-state=new jump-target=detect-ddos
add action=return chain=detect-ddos dst-limit=32,32,src-and-dst-addresses/10s
add action=add-dst-to-address-list address-list=ddos-target address-list-timeout=10m chain=detect-ddos
add action=add-src-to-address-list address-list=ddos-attackers address-list-timeout=10m chain=detect-ddos

/ip firewall raw
add action=drop chain=prerouting dst-address-list=ddos-target src-address-list=ddos-attackers






```



## IP range in each country
You can visit this resource to get the full range of IP addresses used by each country.
[get ip address](https://mikrotikconfig.com/firewall/)


```
import IP-Firewall-Address-List.rsc
# Now you can restrict access to a specific range of source addresses.


```



## detect and drop TCP SYN attack

```
# sloution 1
/ip firewall filter
add action=add-src-to-address-list address-list="SYN Attacker" address-list-timeout=none-dynamic chain=input connection-limit=20,32 protocol=tcp tcp-flags=syn
add action=tarpit chain=input protocol=tcp src-address-list="SYN Attacker“

------------------------------

# sloution2: is the same as ddos attack solution

/ip firewall filter
add action=jump chain=input connection-state=new jump-target=detect-ddos
add action=return chain=detect-ddos dst-limit=32,32,src-and-dst-addresses/10s protocol=tcp tcp-flags=syn,ack
add action=add-dst-to-address-list address-list=ddos-target address-list-timeout=10m chain=detect-ddos
add action=add-src-to-address-list address-list=ddos-attackers address-list-timeout=10m chain=detect-ddos

/ip firewall raw
add action=drop chain=prerouting dst-address-list=ddos-target src-address-list=ddos-attackers



```



## Strong Crypto in ssh
If you have opened the SSH port on your MikroTik, make sure to enable strong cryptography.

![img](img/ssh.png)

```
ip/ssh/set strong-crypto=yes 
system/reboot

```


## DHCP 

If you run a DHCP server on MikroTik RouterOS, always enable DHCP alerts.
```


/ip dhcp-server alert
add disabled=no interface=WAN-Interface valid-server=13:32:27:5:82:AC

```


## port knocking

```









```


## detect Brute force attack

```

/ip firewall filter
add action=drop chain=input log=yes src-address-list="Black List (SSH)"
add action=jump chain=input dst-port=22 jump-target="SSH-Chain " protocol=tcp
add action=add-src-to-address-list address-list="Black List (SSH)" address-list-timeout=4w2d chain="SSH-Chain " connection-state=new src-address-list=G3
add action=add-src-to-address-list address-list=G3 address-list-timeout=1m chain="SSH-Chain " connection-state=new src-address-list=G2
add action=add-src-to-address-list address-list=G2 address-list-timeout=1m chain="SSH-Chain " connection-state=new src-address-list=G1
add action=add-src-to-address-list address-list=G1 address-list-timeout=1m chain="SSH-Chain " connection-state=new
add action=return chain="SSH-Chain "



```


## detect ping flood

```

/ip firewall filter
add action=accept chain=input limit=2,5:packet protocol=icmp
add action=drop chain=input protocol=icmp

-------------------

/ip firewall filter
add action=drop chain=input dst-address-type=broadcast protocol=icmp

```


## lets-encrypt certifacte

```



```


## self-sgin certificate

```

# 1) enable ntp client on your router
set enabled=yes
/system ntp client servers
add address=ntp.server.com


# 2) set correct time-zone
/system clock
set time-zone-name=<saflkjsdf>



# 3) generate CA





```