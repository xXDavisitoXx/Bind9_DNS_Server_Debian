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

Test service



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

## Create DNS zones in named.conf.local: 
```bash
nano /etc/bind/conf/named.conf.local
```

```bash
//
// Do any local configuration here
//

zone "lan" {
        type master;
        file "/etc/bind/zones/db.homelab.guide.org";
};

zone "1.168.192.in-addr.arpa" {
        type master;
        file "/etc/bind/zones/db.192.168.1.in-addr.arpa";
};
```

Create file zones:

```bash
nano /etc/bind/zones/db.homelab.guide.org
```

```bash
$TTL 86400
@       IN      SOA     ns.homelab.guide.org. admin.homelab.guide.org. (
                        2026060301 ; serial
                        3600       ; refresh
                        1800       ; retry
                        604800     ; expire
                        86400 )    ; minimum TTL

        IN      NS      ns.homelab.guide.org.

; --- Infraestructura Core (Reservados) ---
router  IN      A       192.168.1.1
ns      IN      A       192.168.1.10
nas     IN      A       192.168.1.20
proxmox IN      A       192.168.1.30
zabbix  IN      A       192.168.1.40
grafana IN      A       192.168.1.41

; --- IoT & Dispositivos Varios (1-100) ---
; Bloque 1: Networking y Seguridad
ap-salita    IN      A       192.168.1.2
ap-jardin    IN      A       192.168.1.3
switch-poe   IN      A       192.168.1.4
cctv-nvr     IN      A       192.168.1.5
cam-entrada  IN      A       192.168.1.6
cam-patio    IN      A       192.168.1.7
alarma-gen   IN      A       192.168.1.8
cerradura-int IN     A       192.168.1.9

; Bloque 2: Domótica e Iluminación (11-19)
luz-sala      IN     A       192.168.1.11
luz-cocina    IN     A       192.168.1.12
luz-dorm      IN     A       192.168.1.13
termostato    IN     A       192.168.1.14
sensor-humo   IN     A       192.168.1.15
sensor-fuga   IN     A       192.168.1.16
persiana-sala IN     A       192.168.1.17
purificador   IN     A       192.168.1.18
humificador   IN     A       192.168.1.19

; Bloque 3: Multimedia y Otros (21-29)
apple-tv      IN     A       192.168.1.21
shield-tv     IN     A       192.168.1.22
ampli-audio   IN     A       192.168.1.23
chromecast    IN     A       192.168.1.24
impresora-3d  IN     A       192.168.1.25
corte-laser   IN     A       192.168.1.26
tablet-muro   IN     A       192.168.1.27
echo-dot      IN     A       192.168.1.28
google-home   IN     A       192.168.1.29

; Bloque 4: Servidores de apoyo y VM (31-39)
pihole-sec    IN     A       192.168.1.31
wireguard     IN     A       192.168.1.32
mqtt-broker   IN     A       192.168.1.33
db-influx     IN     A       192.168.1.34
web-proxy     IN     A       192.168.1.35
vaultwarden   IN     A       192.168.1.36
home-assistant IN    A       192.168.1.37
nodered       IN     A       192.168.1.38
teslamate     IN     A       192.168.1.39

; Bloque 5: Miscelánea y dispositivos restantes (42-100)
ups-pwr       IN     A       192.168.1.42
estacion-met  IN     A       192.168.1.43
frigo-smart   IN     A       192.168.1.44
lavadora      IN     A       192.168.1.45
secadora      IN     A       192.168.1.46
aspirador     IN     A       192.168.1.47
proyector     IN     A       192.168.1.48
consola-game  IN     A       192.168.1.49
pc-escritorio IN     A       192.168.1.50
laptop-work   IN     A       192.168.1.51
macbook-pro   IN     A       192.168.1.52
monitor-smart IN     A       192.168.1.53
bici-estatica IN     A       192.168.1.54
balanza-iot   IN     A       192.168.1.55
gateway-zigbee IN    A       192.168.1.56
gateway-ble   IN     A       192.168.1.57
switch-tasm   IN     A       192.168.1.58
esphome-wall  IN     A       192.168.1.59
sensor-co2    IN     A       192.168.1.60
control-ac    IN     A       192.168.1.61
radio-wifi    IN     A       192.168.1.62
tira-led-tv   IN     A       192.168.1.63
ventilador    IN     A       192.168.1.64
regleta-smart IN     A       192.168.1.65
timbre-video  IN     A       192.168.1.66
sensor-puerta IN     A       192.168.1.67
sensor-mov    IN     A       192.168.1.68
interruptor-ext IN   A       192.168.1.69
riego-jardin  IN     A       192.168.1.70
sensor-suelo  IN     A       192.168.1.71
cam-garaje    IN     A       192.168.1.72
server-backup IN     A       192.168.1.73
nube-priv     IN     A       192.168.1.74
vlan-iot      IN     A       192.168.1.75
vpn-cliente   IN     A       192.168.1.76
db-mariadb    IN     A       192.168.1.77
auth-server   IN     A       192.168.1.78
log-server    IN     A       192.168.1.79
monitor-temp  IN     A       192.168.1.80
pc-media      IN     A       192.168.1.81
switch-l2     IN     A       192.168.1.82
ap-garaje     IN     A       192.168.1.83
drone-base    IN     A       192.168.1.84
impresora-laser IN   A       192.168.1.85
scanner-doc   IN     A       192.168.1.86
tablet-cocina IN     A       192.168.1.87
echo-show     IN     A       192.168.1.88
fire-stick    IN     A       192.168.1.89
roku-tv       IN     A       192.168.1.90
server-test   IN     A       192.168.1.91
lab-docker    IN     A       192.168.1.92
k8s-node1     IN     A       192.168.1.93
k8s-node2     IN     A       192.168.1.94
k8s-master    IN     A       192.168.1.95
dev-sandbox   IN     A       192.168.1.96
nas-backup    IN     A       192.168.1.97
sensor-luz-ext IN    A       192.168.1.98
switch-core   IN     A       192.168.1.99
router-isp    IN     A       192.168.1.100
```

Create inverse zone
```bash
nano /etc/bind/zones/db.192.168.1.in-addr.arpa
```

```bash
nano /etc/bind/zones/db.192.168.1.in-addr.arpa
```

```bash
$TTL 86400
@       IN      SOA     ns.homelab.guide.org. admin.homelab.guide.org. (
                        2026060301 ; serial
                        3600       ; refresh
                        1800       ; retry
                        604800     ; expire
                        86400 )    ; minimum TTL

        IN      NS      ns.homelab.guide.org.

; --- PTR Records (Ordenados por el último octeto de la IP) ---
1       IN      PTR     router.homelab.guide.org.
2       IN      PTR     ap-salita.homelab.guide.org.
3       IN      PTR     ap-jardin.homelab.guide.org.
4       IN      PTR     switch-poe.homelab.guide.org.
5       IN      PTR     cctv-nvr.homelab.guide.org.
6       IN      PTR     cam-entrada.homelab.guide.org.
7       IN      PTR     cam-patio.homelab.guide.org.
8       IN      PTR     alarma-gen.homelab.guide.org.
9       IN      PTR     cerradura-int.homelab.guide.org.
10      IN      PTR     ns.homelab.guide.org.
11      IN      PTR     luz-sala.homelab.guide.org.
12      IN      PTR     luz-cocina.homelab.guide.org.
13      IN      PTR     luz-dorm.homelab.guide.org.
14      IN      PTR     termostato.homelab.guide.org.
15      IN      PTR     sensor-humo.homelab.guide.org.
16      IN      PTR     sensor-fuga.homelab.guide.org.
17      IN      PTR     persiana-sala.homelab.guide.org.
18      IN      PTR     purificador.homelab.guide.org.
19      IN      PTR     humificador.homelab.guide.org.
20      IN      PTR     nas.homelab.guide.org.
21      IN      PTR     apple-tv.homelab.guide.org.
22      IN      PTR     shield-tv.homelab.guide.org.
23      IN      PTR     ampli-audio.homelab.guide.org.
24      IN      PTR     chromecast.homelab.guide.org.
25      IN      PTR     impresora-3d.homelab.guide.org.
26      IN      PTR     corte-laser.homelab.guide.org.
27      IN      PTR     tablet-muro.homelab.guide.org.
28      IN      PTR     echo-dot.homelab.guide.org.
29      IN      PTR     google-home.homelab.guide.org.
30      IN      PTR     proxmox.homelab.guide.org.
31      IN      PTR     pihole-sec.homelab.guide.org.
32      IN      PTR     wireguard.homelab.guide.org.
33      IN      PTR     mqtt-broker.homelab.guide.org.
34      IN      PTR     db-influx.homelab.guide.org.
35      IN      PTR     web-proxy.homelab.guide.org.
36      IN      PTR     vaultwarden.homelab.guide.org.
37      IN      PTR     home-assistant.homelab.guide.org.
38      IN      PTR     nodered.homelab.guide.org.
39      IN      PTR     teslamate.homelab.guide.org.
40      IN      PTR     zabbix.homelab.guide.org.
41      IN      PTR     grafana.homelab.guide.org.
42      IN      PTR     ups-pwr.homelab.guide.org.
43      IN      PTR     estacion-met.homelab.guide.org.
44      IN      PTR     frigo-smart.homelab.guide.org.
45      IN      PTR     lavadora.homelab.guide.org.
46      IN      PTR     secadora.homelab.guide.org.
47      IN      PTR     aspirador.homelab.guide.org.
48      IN      PTR     proyector.homelab.guide.org.
49      IN      PTR     consola-game.homelab.guide.org.
50      IN      PTR     pc-escritorio.homelab.guide.org.
51      IN      PTR     laptop-work.homelab.guide.org.
52      IN      PTR     macbook-pro.homelab.guide.org.
53      IN      PTR     monitor-smart.homelab.guide.org.
54      IN      PTR     bici-estatica.homelab.guide.org.
55      IN      PTR     balanza-iot.homelab.guide.org.
56      IN      PTR     gateway-zigbee.homelab.guide.org.
57      IN      PTR     gateway-ble.homelab.guide.org.
58      IN      PTR     switch-tasm.homelab.guide.org.
59      IN      PTR     esphome-wall.homelab.guide.org.
60      IN      PTR     sensor-co2.homelab.guide.org.
61      IN      PTR     control-ac.homelab.guide.org.
62      IN      PTR     radio-wifi.homelab.guide.org.
63      IN      PTR     tira-led-tv.homelab.guide.org.
64      IN      PTR     ventilador.homelab.guide.org.
65      IN      PTR     regleta-smart.homelab.guide.org.
66      IN      PTR     timbre-video.homelab.guide.org.
67      IN      PTR     sensor-puerta.homelab.guide.org.
68      IN      PTR     sensor-mov.homelab.guide.org.
69      IN      PTR     interruptor-ext.homelab.guide.org.
70      IN      PTR     riego-jardin.homelab.guide.org.
71      IN      PTR     sensor-suelo.homelab.guide.org.
72      IN      PTR     cam-garaje.homelab.guide.org.
73      IN      PTR     server-backup.homelab.guide.org.
74      IN      PTR     nube-priv.homelab.guide.org.
75      IN      PTR     vlan-iot.homelab.guide.org.
76      IN      PTR     vpn-cliente.homelab.guide.org.
77      IN      PTR     db-mariadb.homelab.guide.org.
78      IN      PTR     auth-server.homelab.guide.org.
79      IN      PTR     log-server.homelab.guide.org.
80      IN      PTR     monitor-temp.homelab.guide.org.
81      IN      PTR     pc-media.homelab.guide.org.
82      IN      PTR     switch-l2.homelab.guide.org.
83      IN      PTR     ap-garaje.homelab.guide.org.
84      IN      PTR     drone-base.homelab.guide.org.
85      IN      PTR     impresora-laser.homelab.guide.org.
86      IN      PTR     scanner-doc.homelab.guide.org.
87      IN      PTR     tablet-cocina.homelab.guide.org.
88      IN      PTR     echo-show.homelab.guide.org.
89      IN      PTR     fire-stick.homelab.guide.org.
90      IN      PTR     roku-tv.homelab.guide.org.
91      IN      PTR     server-test.homelab.guide.org.
92      IN      PTR     lab-docker.homelab.guide.org.
93      IN      PTR     k8s-node1.homelab.guide.org.
94      IN      PTR     k8s-node2.homelab.guide.org.
95      IN      PTR     k8s-master.homelab.guide.org.
96      IN      PTR     dev-sandbox.homelab.guide.org.
97      IN      PTR     nas-backup.homelab.guide.org.
98      IN      PTR     sensor-luz-ext.homelab.guide.org.
99      IN      PTR     switch-core.homelab.guide.org.
100     IN      PTR     router-isp.homelab.guide.org.
```

### Check service
```bash
sudo named-checkconf
```

# check dns zones:

```bash
named-checkzone homelab.guide.org /etc/bind/zones/db.homelab.guide.org
zone homelab.guide.org/IN: loaded serial 2026060301
OK
```

```bash
named-checkzone 1.168.192.in-addr.arpa /etc/bind/zones/db.192.168.1.in-addr.arpa
zone 1.168.192.in-addr.arpa/IN: loaded serial 2026060301
OK
```

Create RPZ:
```bash
nano /etc/bind/zones/db.rpz.telemetry
```

```bash
$TTL 60
@ IN SOA localhost. root.localhost. (
        2026060301 3600 1800 604800 60 )

    IN NS localhost.

; Telemetría bloqueada mediante política nativa
telemetry.microsoft.com.rpz.local.      CNAME   rpz-nxdomain.
vortex.data.microsoft.com.rpz.local.    CNAME   rpz-nxdomain.
telemetry.google.com.rpz.local.         CNAME   rpz-nxdomain.
stats.g.doubleclick.net.rpz.local.      CNAME   rpz-nxdomain.
analytics.google.com.rpz.local.         CNAME   rpz-nxdomain.
```

Declare Zone in named.conf.local


```bash
 nano /etc/bind/conf/named.conf.local
```

```bash
//
// Do any local configuration here
//

zone "lan" {
        type master;
        file "/etc/bind/zones/db.homelab.guide.org";
};

zone "1.168.192.in-addr.arpa" {
        type master;
        file "/etc/bind/zones/db.192.168.1.in-addr.arpa";
};

zone "rpz.telemetry" {
    type master;
    file "/etc/bind/zones/db.rpz.telemetry";
    allow-query { none; };
};
```

Active response policy in named.conf.options

```bash
nano named.conf.local nano /etc/bind/conf/named.conf.options
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


        response-policy {
        zone "rpz.telemetry";
        };

        dnssec-validation auto;

        auth-nxdomain no;

};
```

Check service
```bash
sudo named-checkconf
```

Restart service
```bash
sudo systemctl restart bind9
```
