# ESTADO DEL PROYECTO — RV082 (handoff / resumen de sesión)

> Fecha de la sesión: **2026-11-02**. Este archivo resume qué se hizo y qué falta, para retomar sin releer todo el chat.

---

## 1. Objetivo del usuario

- Construir una **base de conocimiento offline** sobre el router **Cisco/Linksys RV082** en `C:\Users\Usuario\Music\modem` (el usuario dijo que luego me quitaría internet y me haría preguntas serias).
- **Meta práctica final:** que sus **compañeros se puedan conectar desde fuera** a la red del RV082 (y a los equipos conectados).

## 2. Archivos ya creados en esta carpeta

| Archivo | Qué es |
|---|---|
| `README.md` | Índice de la base + datos reales del equipo |
| `01_hardware_modelos_firmware.md` | HW, versiones, LEDs, reset, EoL, firmware, recuperación |
| `02_configuracion_interfaz_web.md` | Web UI menú por menú + 18 procedimientos |
| `03_vpn.md` | IPsec, QuickVPN, PPTP, interoperabilidad, logs |
| `04_seguridad_vulnerabilidades_troubleshooting.md` | CVE, hardening, troubleshooting |
| `manual_vpn_facil.html` | Manual sencillo PPTP+DMZ (quedó **superado** por el plan overlay, ver §5) |
| `ESTADO_DEL_PROYECTO.md` | Este archivo |

## 3. Datos REALES del equipo (leídos en vivo vía `http://192.168.1.1`, `<usuario>`/`<contraseña>`)

- **Modelo:** RV082 · **Serial:** <NUMERO_DE_SERIE>
- **Firmware:** `2.0.0.19-tm` (Feb 2009) → rama **v1/v2** (NO actualizable a 4.x)
- **CPU:** Intel IXP425-533 · **DRAM/Flash:** 32MB / 16MB
- **LAN:** 192.168.1.1 · Modo Gateway · DHCP .100–.149
- **WAN1:** recibió `<IP_WAN_RV082>` de pobre · **Gateway (pobre):** `192.168.100.1`
- Acceso web: **HTTP Basic**, credenciales `<usuario>`/`<contraseña>` (revisar que no sean las de fábrica ⚠️)

## 4. "Pobre" (el módem del ISP) — identificado

- **NO es Cisco.** Es un **ONT de fibra (GPON) genérico** (ODM chino tipo VSOL/C-Data/Realtek), firmware 2020, panel en `http://192.168.100.1` (`login.asp`).
- **ISP:** **<ISP>** (proveedor local). Soporte: **<TELEFONO_SOPORTE_ISP>**.
- **IP pública vista desde internet:** `<IP_PUBLICA>` (parece pública real; **CGNAT aún NO descartado** — falta ver la WAN de pobre por dentro).
- **Contraseña admin del ONT: desconocida.** El ISP la cambió y la ocultó; no entran las claves por defecto habituales. La etiqueta del equipo se borró.

## 5. Problema y DECISIÓN de rumbo

- **Bloqueo para VPN clásica (puertos entrantes):** doble NAT (pobre→linksys) + ONT bloqueado + IP pública sin confirmar. Además el RV082 es viejo: **QuickVPN** no va en Windows moderno y **PPTP** es inseguro + necesita GRE a través de pobre.
- **RUMBO ELEGIDO (en curso): VPN de malla / overlay** tipo Tailscale, que funciona **saliente** y **esquiva pobre y el ONT por completo** (no abre puertos, no necesita IP pública).
  - **Recomendado: NetBird** (gratis 5 usuarios/100 equipos, open source, permite uso de empresa). Alternativa: **ZeroTier** (~10 equipos). Cloudflare Tunnel solo para servicios sueltos.
  - **Arquitectura:** instalar el agente en **un equipo siempre encendido dentro de la LAN** y ponerlo como **subnet router** (anuncia `192.168.1.0/24`); los compañeros instalan el agente y acceden a toda la red (incluido el panel del RV082). Opción B: instalar el agente directo en 1-2 máquinas concretas (sin aparato extra).

## 6. PENDIENTE — lo próximo (esperando respuesta del usuario)

Para armar la guía paso a paso del overlay, el usuario debe responder:
1. ¿A qué quieren llegar los compañeros? ¿**toda la red** o solo **1-2 máquinas**?
2. ¿Tiene un equipo **siempre encendido** en la red (Raspberry Pi / mini PC / NAS / servidor)?
3. ¿**Cuántos compañeros** y con qué SO (Windows/Mac/Android/iPhone)?

Otras vías abiertas (si cambia de idea):
- **Llamar al ISP (<TELEFONO_SOPORTE_ISP>)** y pedir **modo bridge** del ONT → el RV082 recibiría IP pública y se podría usar VPN clásica sin doble NAT. (Es la vía limpia si quieren VPN en el propio RV082.)
- Confirmar CGNAT vs IP pública entrando a pobre y comparando su WAN con `<IP_PUBLICA>`.

## 7. Notas de seguridad / recordatorios

- Cambiar la clave de fábrica del RV082 es lo primero si alguna vez se expone.
- El RV082 está **fuera de soporte desde 2021** con CVE sin parche (ver archivo 04). No exponer su gestión a internet.
- No se usaron (ni se deben usar) backdoors/exploits del ONT; la vía correcta para pobre es el ISP.
