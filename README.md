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

Create a structure bind folders:
Stop service
```bash
systemctl stop bind9
```

```bash
cd /etc/bind
mkdir conf zones keys backup
```

Move config files:
mv /etc/bind/named* /etc/bind/conf/

Move key file:
mv /etc/bind/rndc.key /etc/bind/keys/

## Backup template files: 

```bash
cp /etc/bind/config/named.conf.options /etc/bind/backup/named.conf.options.bak
cp /etc/bind/config/named.conf.local /etc/bind/backup/named.conf.local.bak
```

Edit /etc/bind/conf/named.conf file
```bash
nano /etc/bind/conf/named.conf
```
Edit config path and add rndc.key file:
```bash

// This is the primary configuration file for the BIND DNS server named.
//
// Please read /usr/share/doc/bind9/README.Debian for information on the
// structure of BIND configuration files in Debian, *BEFORE* you customize
// this configuration file.
//
// If you are just adding zones, please do that in /etc/bind/named.conf.local

include "/etc/bind/conf/named.conf.options";
include "/etc/bind/conf/named.conf.local";
include "/etc/bind/conf/named.conf.root-hints";
include "/etc/bind/keys/rndc.key";
```

Create simbolic link named.conf
```bash
 ln -s /etc/bind/conf/named.conf /etc/bind/
```

Configure options file:
```bash
nano /etc/bind/conf/named.conf.options
```

```bash
options {
        directory "/var/cache/bind";

        recursion yes;
        listen-on {
                127.0.0.1;
                192.168.1.10; # YOUR IP DNS SERVER
        };

        allow-query {
                localhost;
                192.168.1.0/24;
        };

        allow-recursion {
                localhost;
                192.168.1.0/24;
        };

        forwarders {
                8.8.8.8;
                8.8.4.4;
        };

        dnssec-validation auto;

        auth-nxdomain no;
};
```
Check config: 

```bash
named-checkconf
```
Restart service:

```bash
systemctl restart bind9
```

Create DNS zones: 

