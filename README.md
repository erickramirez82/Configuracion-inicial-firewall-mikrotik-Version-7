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

- Reglas de firewall básicas protección INPUT en mikrotik configuración inicial

```
/ip settings set tcp-syncookies=yes
/ip firewall filter
/ip firewall filter
add chain=input action=accept protocol=icmp comment="Permitir ICMP"
add chain=input action=accept connection-state=established,related comment="Permitir conexiones establecidas y relacionadas"
add chain=input action=drop in-interface=WAN comment="Bloquear todo otro tráfico de entrada"
add chain=forward action=accept connection-state=established,related comment="Permitir conexiones establecidas y relacionadas"
add chain=forward action=drop connection-state=invalid comment="Descartar conexiones inválidas"




add action=drop chain=input comment="IN - R - Conexiones invalidas" connection-state=invalid
add action=accept chain=input comment="IN - P - Acceso a src-address-list \"Soporte\"" src-address-list=Soporte
add action=drop chain=input comment="IN - R - Detecta y rechaza escaneadores de puertos" protocol=tcp psd=10,3s,3,1 in-interface=WAN1
add action=drop chain=input comment="IN - R - Rechaza escaneadores de puertos" src-address-list=EscaneadoresPuertos in-interface=WAN1
add action=add-src-to-address-list address-list=EscaneadoresPuertos address-list-timeout=none-static chain=input comment="IN - R - Detecta y lista escaneadores de puertos" protocol=tcp psd=10,3s,3,1 in-interface=WAN1
add action=tarpit chain=input comment="IN - R - Suprime ataques DoS" connection-limit=3,32 protocol=tcp src-address-list=AtaqueDoS in-interface=WAN1
add action=add-src-to-address-list address-list=AtaqueDoS address-list-timeout=2w1d chain=input comment="IN - R - Detecta ataques DoS" connection-limit=10,32 protocol=tcp in-interface=WAN1
add action=accept chain=input comment="IN - P - TCP Trafico DNS" dst-port=53 protocol=tcp src-address-list=Redes_Lan
add action=accept chain=input comment="IN - P - UDP Trafico DNS" dst-port=53 protocol=udp src-address-list=Redes_Lan
add action=drop chain=input comment="IN - R - Todo lo demas"
```
