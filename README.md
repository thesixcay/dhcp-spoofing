# DHCP Spoofing con Scapy

## Descripción

Este laboratorio demuestra cómo realizar un ataque de **DHCP Spoofing** utilizando **Python y Scapy** en un entorno controlado. El atacante responde a las solicitudes DHCP antes que el servidor legítimo, entregando parámetros de red falsificados a la víctima.

También se muestra cómo mitigar este ataque mediante la implementación de **DHCP Snooping** en un switch Cisco.

---

# Objetivo del Laboratorio

- Comprender el funcionamiento del protocolo DHCP.
- Implementar un ataque DHCP Spoofing utilizando Scapy.
- Analizar el impacto del ataque en una estación víctima.
- Implementar medidas de seguridad para prevenir servidores DHCP no autorizados.

---

# Topología
<img width="441" height="434" alt="photo_2026-06-05_07-37-13" src="https://github.com/user-attachments/assets/8ca9301d-6500-4526-967b-6740276f816d" />

# Configuración del Router DHCP Legítimo (R2)

```bash
configure terminal

interface Ethernet0/0
 ip address 192.23.61.1 255.255.255.0
 no shutdown
 exit

ip dhcp excluded-address 192.23.61.1 192.23.61.19
ip dhcp excluded-address 192.23.61.100

ip dhcp pool LEGITIMO_POOL
 network 192.23.61.0 255.255.255.0
 default-router 192.23.61.1
 dns-server 8.8.8.8
 exit

end
write
```

---

# Configuración Inicial del Switch (R1)

```bash
configure terminal

interface range Ethernet0/0 - 2
 no shutdown
 exit

end
write
```

---

# Configuración de Kali Linux

Asignar una IP estática al equipo atacante:

```bash
sudo ip addr add 192.23.61.100/24 dev eth0 2>/dev/null || true
sudo ip link set dev eth0 up
```

Verificar la configuración:

```bash
ip addr show eth0
```
# Ejecución del Ataque

En Kali Linux:
<img width="676" height="240" alt="image" src="https://github.com/user-attachments/assets/5050d243-f8a4-4d12-8565-49770c28db6e" />

La víctima utilizará al atacante como puerta de enlace predeterminada.
<img width="719" height="310" alt="image" src="https://github.com/user-attachments/assets/cbe60336-e9ac-48ab-b347-ece56c34341c" />
