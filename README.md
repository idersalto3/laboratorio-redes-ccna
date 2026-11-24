# Laboratorio de redes con Cisco Packet Tracer | Recorrido CCNA

Laboratorio práctico de una red corporativa pequeña, desarrollado en Cisco Packet Tracer. Comenzó con una topología basada en contenidos de ITN y evolucionó hacia una arquitectura con VLAN, enlaces trunk 802.1Q, Router-on-a-Stick y una red de gestión, incorporando contenidos de SRWE.

El objetivo es aplicar fundamentos de switching, routing, direccionamiento y servicios de red, y documentar las pruebas y los problemas encontrados durante la implementación.

## Evolución del laboratorio

| Versión | Enfoque | Características principales |
|---|---|---|
| **v1 · ITN** | Red inicial de oficina | VLSM, IPv4/IPv6, DHCPv4, SLAAC, SSH, Port Security, DNS y HTTP |
| **v2 · SRWE** | Red segmentada | VLAN 10, 20, 30 y 99; trunks 802.1Q; Router-on-a-Stick; gestión separada |

La versión 1 se conserva como punto de partida. La versión 2 es la topología principal para continuar practicando.

## Versión 2 — Red corporativa con VLAN y Router-on-a-Stick

### Topología

![Topología lógica v2: Router-on-a-Stick, cuatro VLAN y trunks 802.1Q](docs/img/05-topologia-v2.png)

Archivo de simulación: [red-corporativa-v2.pkt](packet-tracer/red-corporativa-v2.pkt).

La topología utiliza un router Cisco ISR 4331, dos switches Catalyst 2960-24TT, cuatro PCs y un servidor interno. R1 se conecta a SW-Access-01 mediante un trunk 802.1Q. El enlace entre switches también transporta las VLAN configuradas.

### Segmentación y direccionamiento

| VLAN | Nombre | Subred IPv4 | Máscara | Prefijo IPv6 | Gateway IPv4 |
|---:|---|---|---|---|---|
| 10 | ADMIN | 192.168.1.0/25 | 255.255.255.128 | 2001:db8:acad:10::/64 | 192.168.1.1 |
| 20 | SOPORTE | 192.168.1.128/26 | 255.255.255.192 | 2001:db8:acad:20::/64 | 192.168.1.129 |
| 30 | SERVIDORES | 192.168.1.192/27 | 255.255.255.224 | 2001:db8:acad:30::/64 | 192.168.1.193 |
| 99 | GESTION | 192.168.1.224/28 | 255.255.255.240 | 2001:db8:acad:99::/64 | 192.168.1.225 |

Las subinterfaces de R1 utilizan `fe80::1` como dirección IPv6 link-local. Las direcciones de gestión de los switches son `192.168.1.226` (SW-Access-01) y `192.168.1.227` (SW-Access-02), con gateway `192.168.1.225`. Las SVI de VLAN 1 quedaron apagadas en la v2.

### Mapeo de puertos

| Equipo / puerto | Conexión | Modo / VLAN | Función |
|---|---|---|---|
| R1 G0/0/0 | SW-Access-01 Gi0/2 | Trunk 802.1Q | Transporte de VLAN 10, 20, 30 y 99 |
| SW-Access-01 Gi0/1 | SW-Access-02 Gi0/1 | Trunk 802.1Q | Transporte de VLAN entre switches |
| SW-Access-01 Fa0/1–Fa0/2 | PCs de Administración | Access VLAN 10 | Puestos de Administración |
| SW-Access-01 Fa0/24 | ServerEmpresa Fa0 | Access VLAN 30 | Servicios DNS y HTTP |
| SW-Access-02 Fa0/1–Fa0/2 | PCs de Soporte | Access VLAN 20 | Puestos de Soporte |

Los trunks tienen permitidas las VLAN 10, 20, 30 y 99, con negociación DTP deshabilitada mediante `nonegotiate`. R1 utiliza las subinterfaces `.10`, `.20`, `.30` y `.99` sobre G0/0/0. Las interfaces físicas G0/0/1 y G0/0/2 quedaron administrativamente apagadas.

### Servicios y configuraciones implementadas

- **VLAN y switching:** segmentación de Administración, Soporte, Servidores y Gestión.
- **Routing inter-VLAN:** Router-on-a-Stick mediante subinterfaces 802.1Q en R1.
- **IPv4:** direccionamiento VLSM y DHCPv4 para los puestos de usuario.
- **IPv6:** prefijos /64 y SLAAC configurados para la topología.
- **Gestión:** direcciones de administración en VLAN 99 y acceso SSHv2 a ambos switches.
- **Servicios internos:** ServerEmpresa mantiene DNS y HTTP, con el registro `www.ider.com` apuntando a `192.168.1.194`.

### Pruebas realizadas en la v2

| Comprobación | Resultado documentado |
|---|---|
| Estado de subinterfaces de R1 | G0/0/0.10, .20, .30 y .99 en up/up |
| Interfaces físicas no utilizadas | G0/0/1 y G0/0/2 en administratively down/down |
| Trunks | VLAN 10, 20, 30 y 99 en forwarding |
| DHCPv4 | Asignación recuperada en Administración y Soporte después de corregir VLAN y puertos en SW-Access-02 |
| DNS y conectividad al servidor | Desde PC-Soporte-1, `ping www.ider.com` resolvió 192.168.1.194 y recibió 4/4 respuestas |
| Gestión SSH | Acceso desde PC-Soporte-1 a 192.168.1.226 y 192.168.1.227 |

**Alcance de las pruebas:** el ping por nombre comprueba la resolución DNS y la conectividad IP hasta el servidor; no reemplaza una prueba HTTP desde el navegador. En la v2 todavía falta guardar evidencia de acceso web y una prueba de conectividad IPv6 de extremo a extremo desde una PC.

### Problemas relevantes durante la migración

Durante el paso de la v1 a la v2 aparecieron problemas de configuración que se resolvieron mediante comprobaciones por equipo y por capa:

1. **Pérdida temporal de acceso SSH:** al retirar un enlace físico de la v1, una PC quedó sin gateway para alcanzar la IP de gestión del otro switch. Se continuó la configuración por consola hasta completar Router-on-a-Stick.
2. **Comando aplicado en el equipo equivocado:** se intentó configurar una interfaz del router desde la consola de un switch. Se identificó el dispositivo mediante el hostname y el prompt.
3. **Trunk entre switches no operativo:** un extremo estaba configurado como trunk y el otro no. Se configuraron ambos lados con las mismas VLAN permitidas.
4. **Dirección IPv6 duplicada:** una dirección permanecía en la interfaz física de la v1 y se intentó reutilizar en una subinterfaz. Se eliminó la configuración anterior antes de asignarla nuevamente.
5. **Falla de DHCP en Soporte:** faltaban las VLAN locales y la asignación de los puertos de acceso en SW-Access-02. Se corrigieron las VLAN, los puertos y la SVI de gestión.
6. **IP de gestión duplicada:** ambos switches habían quedado con la misma dirección. Se asignaron direcciones distintas y se limpió la caché ARP de R1.

## Versión 1 — Topología inicial basada en ITN

La v1 se conserva en [practicacisco.pkt](packet-tracer/practicacisco.pkt) como referencia de la primera etapa del proyecto.

### Topología y direccionamiento

![Topología inicial en Cisco Packet Tracer](docs/img/01-topologia-packet-tracer.png)

La red inicial utilizaba tres interfaces físicas de R1, una por segmento:

| Sector | Subred IPv4 | Máscara | Gateway |
|---|---|---|---|
| Administración | 192.168.1.0/25 | 255.255.255.128 | 192.168.1.1 |
| Soporte | 192.168.1.128/26 | 255.255.255.192 | 192.168.1.129 |
| Servidores | 192.168.1.192/27 | 255.255.255.224 | 192.168.1.193 |

### Funcionalidades trabajadas

- VLSM para dividir el bloque 192.168.1.0/24 según los sectores de la oficina.
- Dual-Stack IPv4/IPv6, con DHCPv4 y SLAAC.
- SSHv2 para administrar los switches.
- Port Security con aprendizaje sticky y acción shutdown ante una MAC no autorizada.
- Servidor interno con DNS y HTTP.

### Evidencias de la v1

En esta versión se comprobó el estado de las interfaces y las rutas de R1, la asignación IPv4/IPv6 en los clientes, la conectividad entre subredes, el acceso SSH a los switches y la resolución del nombre interno `www.ider.com`.

También se probó Port Security conectando una PC distinta en un puerto protegido: el switch registró la violación y dejó el puerto en estado err-disabled.

![Prueba de violación de Port Security](docs/img/02-port-security-violacion.png)

![Puerto en estado err-disabled por Port Security](docs/img/03-port-security-err-disabled.png)

![Configuración IPv4 e IPv6 del cliente de Soporte](docs/img/04-ipconfig-pc-soporte-1.png)

## Próximos pasos de la v2

### Pendiente para completar las verificaciones

- [ ] Probar el acceso a `www.ider.com` desde el navegador de una PC.
- [ ] Verificar SLAAC y conectividad IPv6 desde las PCs de las VLAN de usuarios.
- [ ] Revisar y volver a probar Port Security en los puertos de acceso de la v2.
- [ ] Restringir el acceso SSH mediante ACL en las líneas VTY para permitir únicamente los orígenes de administración autorizados.

### Ampliaciones posteriores

- [ ] Apagar puertos no utilizados y asignarlos a una VLAN de parking.
- [ ] Evaluar DHCP Snooping según las funciones disponibles en Packet Tracer.
- [ ] Incorporar EtherChannel/LACP y comprobar el comportamiento de Rapid-PVST+.
- [ ] Guardar salidas de comandos de verificación: `show interfaces trunk`, `show vlan brief`, `show ip interface brief`, `show ipv6 interface brief` y `show spanning-tree`.

## Tecnologías y conceptos

Cisco Packet Tracer, Cisco IOS, ISR 4331, Catalyst 2960-24TT, VLSM, IPv4, IPv6, DHCPv4, SLAAC, VLAN 802.1Q, trunking, Router-on-a-Stick, routing inter-VLAN, SSHv2, SVI de gestión, Port Security, STP, DNS y HTTP.

## Archivos

- [Topología v1](packet-tracer/practicacisco.pkt)
- [Topología v2](packet-tracer/red-corporativa-v2.pkt)
- [Capturas y diagramas](docs/img/)

## Sobre el proyecto

Soy Técnico Universitario en Redes Informáticas y Telecomunicaciones (Universidad Siglo 21) y estudiante de la Licenciatura en Informática. Desde 2020 realizo soporte técnico independiente a usuarios, con tareas de hardware, software, Windows y conectividad. Este laboratorio forma parte de mi práctica personal para profundizar en redes e infraestructura.

[LinkedIn](https://www.linkedin.com/in/idersalto)

---

Las direcciones IP y configuraciones de este repositorio pertenecen a un entorno de simulación en Cisco Packet Tracer. No representan una red productiva.
