# Bind9_DNS_Server_Debian

## Install software
Refresh APT:
```bash
apt update
```

Install Bind9: 
```bash
apt install bind9
```

Check the service started and enable:

```bash
systemctl status bind9
```

```bash
● named.service - BIND Domain Name Server
     Loaded: loaded (/usr/lib/systemd/system/named.service; enabled; preset: enabled)
     Active: active (running) since Wed 2026-06-03 10:53:15 CEST; 7min ago
 Invocation: 72a9302fa91f4b9ca16b3518e1650d84
       Docs: man:named(8)
   Main PID: 3927 (named)
     Status: "running"
      Tasks: 8 (limit: 4593)
     Memory: 27.9M (peak: 29.9M)
        CPU: 86ms
     CGroup: /system.slice/named.service
             └─3927 /usr/sbin/named -f -u bind
```

Check version:
```bash
named -v
```

```bash
BIND 9.20.23-1~deb13u1-Debian (Stable Release) <id:>
```

## Backup template files: 

```bash
cp /etc/bind/named.conf.options /etc/bind/named.conf.options.bak
cp /etc/bind/named.conf.local /etc/bind/named.conf.local.bak
```
