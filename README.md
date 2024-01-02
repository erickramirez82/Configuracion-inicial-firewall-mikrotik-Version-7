# Configuracion-inicial-firewall-mikrotik-Version-7
Reglas de firewall básicas protección INPUT en mikrotik configuración inicial

```
/ip settings set tcp-syncookies=yes
/ip firewall filter
add action=accept chain=input comment="IN - P - Conexiones establecidas y relacionadas" connection-state=established,related
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
