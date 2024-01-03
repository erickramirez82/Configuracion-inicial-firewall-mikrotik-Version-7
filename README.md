# Configuracion-inicial-firewall-mikrotik-Version-7

- Nombramos la interfaces 
```
/interface ethernet
  set [ find default-name=ether1 ] name=ether1-wan comment="IPS1"
 ```

- Definimos ip address la ip de la wan
```
/ip address
add address=192.168.1.19/24 comment=defconf interface=ether1-wan network=192.168.1.0
```
 
- Definirmos bridge en caso usarlo

```
/interface bridge
add  comment=defconf name=bridge   
```
- Definimos interfaces o puertos en el bridge
```
/interface bridge port
add bridge=bridge interface=ether2 
add bridge=bridge interface=ether3
```

- Definimos la lista de interfaces
```
/interface list
add comment=defconf name=WAN
add comment=defconf name=LAN
/interface list member
add comment=defconf interface=bridge list=LAN
add comment=defconf interface=ether1-wan list=WAN
```


- Definir en ip address la ip Router

```
/ip address
add address=192.168.88.1/24 comment=defconf interface=bridge network=192.168.88.0
```

- Definimos el pool

```
/ip pool
add name=dhcp ranges=192.168.88.10-192.168.88.50
```

- Crearmos el servidor de DCHP

```
/ip dhcp-server
add address-pool=dhcp interface=bridge lease-time=10m name=defconf
/ip dhcp-server network
add address=192.168.88.0/24 comment=defconf dns-server=192.168.88.1 gateway=192.168.88.1 netmask=24
```
### Seguridad y Firewall

- Desactivamos servicios no vamos a usar y sugerencia cambiar el puerto winbox y www que vienen por default

```
/ip service
set telnet disabled=yes
set ftp disabled=yes
set www port=80
set ssh port=22 disabled=yes
set api disabled=yes
set winbox port=8291
set api-ssl disabled=yes
```

- Reglas de firewall básicas protección INPUT en mikrotik configuración inicial

```
/ip settings set tcp-syncookies=yes
/ip firewall filter
add action=drop chain=input comment="Denegar Conexiones Invalidas" connection-state=invalid
add action=accept chain=input comment="Permitir Conexiones establecidas y relacionadas" connection-state=established,related
add action=accept chain=input protocol=icmp comment="accept ICMP packets"
add chain=input action=drop comment="Denegar todo desde Internet"
```

Para agregar mas reglas de seguridad ir al siguiente repositio https://github.com/erickramirez82/bloqueo-botnets-en-mikrotik

### Activamos navegacion a internet al equipo

- Definimos lo Dns de navegacion

```
/ip dns
set allow-remote-requests=yes servers=1.1.1.1,8.8.8.8
```

- Configuramos en firewall dentro del NAT
- 
```
/ip firewall nat
add chain=srcnat out-interface=ether1-wan action=masquerade
```

- Definimos las rutas Ip->Route

1. Monitor al Dns del  operador claro:190.157.8.33, movistar:200.21.200.10, etb:200.75.51.132) 

- Definir las salidas dos maneras 

Usando la puerta enlace del modem
```
/ip/route/
add distance=1 gateway=192.168.1.1  check-gateway=ping  comment="Default IPS1 out"
```
o con los DNS del operador

```
/ip/route/
add distance=1 gateway=200.21.200.10 target-scope=11 check-gateway=ping  comment="Default DNS IPS1 out"
```

check dns 

```
/ip/route/
add check-gateway=ping dst-address=200.21.200.10 gateway=192.168.1.1  scope=10   comment="Monitor DNS IPS1"
```


