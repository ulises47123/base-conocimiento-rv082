# Base de conocimiento — Router Cisco/Linksys RV082 (Dual WAN VPN)

Base de conocimiento autosuficiente sobre el **RV082 10/100 8-Port VPN Router**, creada para poder consultarse **sin conexión a internet**. Reúne documentación oficial de Cisco/Linksys, artículos de soporte, release notes, advisories de seguridad (NVD/CISA) y reseñas independientes, en español.

## Archivos

| # | Archivo | Contenido |
|---|---------|-----------|
| 1 | [`01_hardware_modelos_firmware.md`](01_hardware_modelos_firmware.md) | Versiones de hardware (v1/v2 Intel IXP425 vs v3 Cavium), especificaciones, LEDs, botón Reset, rendimiento, fin de vida (EoL), historial y recuperación de firmware (TFTP), telnet oculto, valores por defecto |
| 2 | [`02_configuracion_interfaz_web.md`](02_configuracion_interfaz_web.md) | Recorrido menú por menú del web UI: Setup, DHCP, Dual WAN, Bandwidth Management, SNMP, Firewall, Port/VLAN, Log, backup/restore; 18 procedimientos paso a paso |
| 3 | [`03_vpn.md`](03_vpn.md) | IPsec Gateway-to-Gateway y Client-to-Gateway campo a campo, QuickVPN, PPTP, interoperabilidad (ASA, pfSense, FortiGate, MikroTik…), interpretación de logs, troubleshooting |
| 4 | [`04_seguridad_vulnerabilidades_troubleshooting.md`](04_seguridad_vulnerabilidades_troubleshooting.md) | ~80 CVE y 16 advisories, CISA KEV, botnets (PolarEdge/ViciousTrap) e IoC, defaults inseguros, hardening, troubleshooting de hardware/red/firmware |

## Resumen rápido del dispositivo

- **Acceso por defecto:** `http://192.168.1.1`, usuario `admin` / clave `admin`, DHCP 192.168.1.100–.149.
- **Reset:** 1 s reinicia; 30 s restaura valores de fábrica (botón en el panel frontal).
- **Generaciones:** HW v1/v2 (Intel IXP425, 32 MB RAM, 16 MB flash, firmware 1.x/2.x) · HW v3 (Cavium CN5020, 256 MB RAM, 32 MB flash, firmware 4.x). **No se cruzan los firmware.**
- **Fin de vida:** fin de venta 5-may-2016; último día de soporte 31-may-2021. Último firmware v3: **4.2.3.14** (jun-2020).
- **VPN:** 100 túneles IPsec, 2 Group VPN, 50 usuarios QuickVPN, 5 usuarios PPTP.
- ⚠️ **Equipo fuera de soporte** con CVE sin parche (varias explotadas activamente). No exponer la gestión a internet.

## Convención de confianza / fuentes

Dentro de cada archivo los datos llevan marcadores de confianza y de origen: `[ALTA]/[MEDIA]/[BAJA]`, `[DUDOSO]`, y etiquetas de fuente (`[AG]` Admin Guide, `[DS]` Data Sheet, `[KB]` artículo de soporte, `[RN]` release notes, `[3P]` terceros, `[Com]` comunidad). Cada archivo termina con su propia sección de **Fuentes** con URLs.

---

## Datos reales del equipo inspeccionado (2026-11-02)

Lectura directa del **System Summary** de la unidad del usuario (acceso local a `http://192.168.1.1/home.htm`):

| Campo | Valor |
|-------|-------|
| Modelo | RV082 (10/100 8-port VPN Router) |
| Serial Number | <NUMERO_DE_SERIE> |
| Firmware | **2.0.0.19-tm** (compilado Feb 20 2009 15:15:20) |
| CPU | **Intel IXP425-533** |
| DRAM / Flash | 32 MB / 16 MB |
| **Generación** | **Hardware v1/v2** (confirmado por CPU IXP425, firmware 2.x y auth HTTP Basic) |
| LAN IP / Mode | 192.168.1.1 · Gateway |
| DNS dinámico (DDNS) | Off \| Off |
| DMZ Host | Disabled |
| VPN | 0 túneles usados / 100 disponibles · sin Group VPN |
| Log por e-mail | No configurado (sin servidor SMTP) |
| ProtectLink | Presente (módulo Trend Micro) |

**Notas sobre este equipo concreto:**
- El firmware **2.0.0.19-tm (2009)** es muy antiguo y pertenece a la rama v1/v2; **no** puede actualizarse a la rama 4.x (esa es solo para hardware v3).
- Responde con **autenticación HTTP Basic** (no la pantalla de login del firmware 4.x), coherente con la generación v1/v2.
- Si mantiene las **credenciales de fábrica**, cambiarlas es la primera recomendación de hardening (ver archivo 04).
- La hora del sistema iba desfasada respecto a la fecha real; conviene revisar NTP si se usan logs o reglas por horario.
