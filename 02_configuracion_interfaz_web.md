# Cisco/Linksys RV082 (10/100 8-Port VPN Router, Dual WAN): configuración e interfaz web

> Documento de referencia para consultarlo **sin internet**. Recorre la interfaz web (Web-based Utility / Configuration Utility) del RV082 menú por menú, con sus campos, valores por defecto, límites y procedimientos.
> Los nombres de menús, botones y campos se dejan **en inglés, tal como aparecen en la interfaz**.
> Marcas usadas: **[DUDOSO]** = dato contradictorio entre fuentes o no confirmado; **[v1/v2]** = solo firmware Linksys antiguo (1.x/2.x); **[v3/4.x]** = solo firmware Cisco 4.x (hardware v3).
> La VPN solo se menciona aquí; se trata a fondo en otro documento.

---

## 0. Contexto: versiones de hardware y firmware (importa para saber qué menús verás)

| Generación | Marca en la carcasa | Firmware | Hardware (según fuentes) | Notas de la interfaz |
|---|---|---|---|---|
| v1 | Linksys "10/100 8-Port VPN Router" | 1.x (ej. 1.3.x) y luego 2.x | CPU Intel IXP425 (266 MHz), 16 MB flash, 32 MB RAM; switch ADM6996L | Pestañas "LAN Management", "Setting Backup" (archivo `RV082.exp`), botones "Save Settings / Cancel Changes". Hasta 50 túneles VPN. |
| v2 | Linksys / Cisco | 2.x (la última citada en la comunidad: **2.0.0.19-tm**) | Igual que v1 pero con switch ADM6996I | La release note de 2.0.0.19 dice que soporta HW v1.x y v2, y que en HW v2 **no se puede bajar de 2.x a 1.x**. |
| v3 | Cisco Small Business RV082 | **4.x** (4.0.x, 4.1.x, 4.2.x; se citan 4.2.1.02, 4.2.2.08, 4.2.3.03, 4.2.3.06, 4.2.3.10, 4.2.3.14) | CPU Cavium CN5020 a 300 MHz, 32 MB flash, 256 MB RAM, switches Realtek RTL8309G + RTL8306SD | Menú "Port Management", "Backup and Restore" (`Startup.config`/`Mirror.config`), IPv6 dual-stack, Bonjour, 100 túneles IPsec. **El firmware 4.x solo funciona en HW v3**: no es retrocompatible. |

- Cómo identificar la versión de hardware: según la comunidad Cisco, por la etiqueta de la parte inferior (en algunos casos el código "A00" se asoció a v1). No hay un método oficial documentado en la interfaz. **[DUDOSO]**
- El System Summary de **v1/v2** muestra CPU, DRAM y Flash. El de **v3** muestra Serial Number, Firmware version, PID VID, MD5 Checksum, etc. (ver §3). Es una pista práctica para saber qué generación tienes.
- La guía de administración oficial de referencia es la **Cisco Small Business RV0xx Series Routers Administration Guide** (78-19576-01 B0, 2011-2013), común a RV042, RV042G, RV082 y RV016. Hay guías Linksys anteriores ("10/100 8-Port VPN Router User Guide") para v1/v2.

---

## 1. Acceso a la interfaz web

### 1.1 Valores por defecto de fábrica (acceso)

| Parámetro | Valor por defecto |
|---|---|
| IP LAN del router | **192.168.1.1** |
| Máscara LAN | **255.255.255.0** |
| Usuario | **admin** (minúsculas) |
| Contraseña | **admin** (minúsculas) |
| Rango DHCP | **192.168.1.100 a 192.168.1.149** (50 direcciones) |
| Servidor DHCP | Habilitado |
| Firewall | Habilitado (SPI, DoS y Block WAN Request activados) |
| Remote Management (gestión por WAN) | **Deshabilitado** |
| HTTPS | Habilitado |
| Modo de trabajo | Gateway |
| Puerto DMZ/Internet | Ver §4.1.6 (la guía Cisco dice "DMZ is enabled by default" en Edit DMZ Connection) **[DUDOSO]** |

### 1.2 Cómo entrar
1. Conecta un PC a un puerto LAN numerado (1 a 8). El PC recibe por DHCP una IP 192.168.1.x. También puedes poner al PC una IP fija en 192.168.1.x con puerta de enlace 192.168.1.1.
2. Abre el navegador y ve a `http://192.168.1.1`. La guía 4.x cita Internet Explorer 6 o superior, Firefox o Safari; la guía Linksys cita IE o Netscape. Con HTTPS habilitado también responde `https://192.168.1.1` (el certificado es autofirmado y el navegador avisará).
3. Introduce **admin / admin** y pulsa **Login** (en v1/v2 aparece un diálogo de autenticación con botón **OK**).
4. Aparece la página **System Summary**.

### 1.3 Comportamiento general de la interfaz
- **Navegación:** árbol de navegación en el panel izquierdo (v3) o pestañas superiores (v1/v2). Al pulsar un módulo se despliegan sus subpáginas.
- **Guardar:** nada se aplica hasta pulsar **Save** (v3) o **Save Settings** (v1/v2). Si cambias de página sin guardar, **los cambios se pierden**. **Cancel** / **Cancel Changes** descarta.
- **Listas editables:** casi todas las tablas (forwarding, rutas, servicios, IP estáticas...) siguen el mismo patrón: rellenar campos → **Add to list**. Para modificar: clic en la entrada → editar → **Update**. **Add New** limpia los campos sin modificar nada. **Delete** borra (Shift+clic selecciona un bloque, Ctrl+clic selecciona varias). Al final hay que pulsar **Save** en la página.
- **Ventanas emergentes:** Service Management, Show Unknown MAC Addresses, View System Log, tablas, etc. se abren en pop-ups. Hay que permitirlos en el navegador.
- **Help:** enlace arriba a la derecha que abre la ayuda de la página actual.
- **Site Map [v1/v2]:** enlace a la derecha en todas las pantallas con enlaces a todas las pestañas.
- **Logout:** enlace arriba a la derecha (v3) o pestaña Logout (v1/v2; pide confirmación).
- **Service Management (común a varias páginas):** ventana para crear servicios propios: **Service Name**, **Protocol** (TCP, UDP; en 4.x también aparece "IPv6"/"IP" según el artículo), **Port Range**. Según la guía, admite **hasta 30 servicios** definidos por el usuario. Los servicios predefinidos no se pueden borrar ni modificar. La lista de servicios es compartida por Forwarding, UPnP, Access Rules, Bandwidth Management y Protocol Binding. **[DUDOSO: es razonable que sea una lista global, pero la guía la describe página por página]**

---

## 2. Mapa completo de menús

### 2.1 Firmware Cisco 4.x (hardware v3)
```
System Summary
Setup
  ├─ Network            (Host/Domain, IP Mode, LAN, Multiple Subnet, WAN Setting, DMZ Setting)
  ├─ Password
  ├─ Time
  ├─ DMZ Host
  ├─ Forwarding         (Port Range Forwarding, Port Triggering)
  ├─ UPnP
  ├─ One-to-One NAT
  ├─ MAC Address Clone
  ├─ Dynamic DNS
  ├─ Advanced Routing   (Dynamic Routing RIP / RIPng, Static Routing)
  └─ IPv6 Transition    (6to4; solo con Dual-Stack)
DHCP
  ├─ DHCP Setup         (Server/Relay, Dynamic IP, Static IP, DNS, WINS, DNS Local Database)
  ├─ DHCP Status
  └─ Router Advertisement (IPv6)
System Management
  ├─ Dual WAN
  ├─ Bandwidth Management
  ├─ SNMP
  ├─ Discovery-Bonjour
  ├─ Diagnostic
  ├─ Factory Default
  ├─ Firmware Upgrade
  ├─ Restart
  └─ Backup and Restore
Port Management
  ├─ Port Setup
  └─ Port Status
Firewall
  ├─ General
  ├─ Access Rules
  └─ Content Filter
Cisco ProtectLink Web     (Global Settings, Web Protection, License; si está disponible)
VPN                       (Summary, Gateway to Gateway, Client to Gateway, VPN Client Access, VPN Passthrough, PPTP Server)
Log
  ├─ System Log
  └─ System Statistics
Wizard                    (Basic Setup, Access Rule Setup)
Support / Help
Logout
```

### 2.2 Firmware Linksys 1.x/2.x (hardware v1/v2)
Según la guía Linksys hay **11 pestañas principales**: **System Summary, Setup, DHCP, System Management, LAN Management** (renombrada **Port Management** en la guía v2), **Firewall, VPN, Log, Wizard, Support, Logout**.
- Setup: Network, Password, Time, DMZ Host, Forwarding, UPnP, One-to-One NAT, MAC Clone, DDNS, Advanced Routing.
- DHCP: Setup, Status.
- System Management: Dual-WAN, SNMP, Diagnostic, Factory Default, Firmware Upgrade, Restart, **Setting Backup**.
- LAN/Port Management: LAN Setup / Port Setup, LAN Status / Port Status.
- Firewall: General, Access Rules, Content Filter.
- VPN: Summary, Gateway to Gateway, Client to Gateway, VPN Client Access, VPN Pass Through, PPTP Server.
- Log: System Log, System Statistics.
- Algunas versiones de firmware 1.3.x/2.x añadieron la pestaña **ProtectLink** (Trend Micro) y **Bandwidth Management**. **[DUDOSO: la guía Linksys revisada no describe Bandwidth Management, pero sí Protocol Binding; las funciones exactas dependen de la versión de firmware]**

---

## 3. System Summary

Página de solo lectura que aparece al entrar. Muchos elementos son hipervínculos a la página de configuración correspondiente.

### 3.1 [v3/4.x] Campos
**System Information**
- **Serial Number**: número de serie.
- **Firmware version**: versión instalada.
- **PID VID**: identificador de producto y versión de hardware.
- **MD5 Checksum**: suma de verificación del firmware.
- **LAN IP / Subnet mask**: IP LAN actual.
- **Working Mode**: Gateway o Router.
- **LAN**: con Dual-Stack activado muestra IPv4/máscara e IPv6/prefijo.
- **System Up time**: tiempo encendido (días, horas, minutos).

**Cisco ProtectLink Web**: botones **Go buy**, **Register**, **Activate** (no disponible en RV042G). Si no aparecen, la guía sugiere actualizar el firmware.

**Configuration**: botón **Setup Wizard** (Basic Setup Wizard y Access Rule Setup Wizard).

**Port Statistics**: tabla con **Port ID**, **Interface** (LAN, WAN1, WAN2, DMZ) y **Status** (Disabled en rojo, Enabled en negro, Connected en verde). Al pulsar el estado se abre la ventana **Port Information**:
- Type (10Base-T/100Base-TX), Interface, Link Status (Up/Down), Port Activity (Port Enabled/Disabled/Connected), Priority (High/Normal), Speed Status (10/100 Mbps), Duplex Status (Half/Full), Auto negotiation (On/Off), VLAN.
- Receive/Transmit Packet Count, Receive/Transmit Packet Byte Count, Packet Error Count. Botones Refresh y Close.

**WAN Status** (pestañas IPv4/IPv6; IPv6 solo con Dual-Stack):
- Por WAN: IP Address, Default Gateway, DNS, Dynamic DNS (Disabled/Enabled).
- Botones **Release / Renew** si la WAN obtiene la IP automáticamente (DHCP); **Connect / Disconnect** si es PPPoE o PPTP.
- DMZ: IP Address y **DMZ Host** (IP privada del DMZ host; por defecto Disabled).

**Firewall Setting Status**: SPI, DoS, Block WAN Request, Remote Management (On en verde / Off en rojo) y **Access Rule** (número de reglas definidas).

**VPN Setting Status**: Tunnel(s) Used y Tunnel(s) Available.

**Log Setting Status**: Syslog Server y Email Log (On/Off).

### 3.2 [v1/v2] Diferencias
- System Information: Serial Number, Firmware version, **CPU** ("Intel IXP425"), **DRAM**, **Flash**, System Up Time (con hora actual).
- Port Statistics: diagrama de puertos clicable con los mismos colores.
- **Network Setting Status**: LAN IP, WAN1 IP (Release/Renew o Connect/Disconnect), WAN2/DMZ IP, Mode (Gateway/Router), DNS, DDNS (On/Off), DMZ Host.
- VPN Setting Status: además muestra los usuarios conectados por GroupVPN1/GroupVPN2 ("No Group VPN was defined" si no hay).
- Log Setting Status muestra mensajes de estado del correo:
  - "E-mail cannot be sent because you have not specified an outbound SMTP server address."
  - "E-mail settings have been configured." (configurado, pero aún no se ha enviado nada)
  - "E-mail settings have been configured and sent out normally."
  - "E-mail cannot be sent out, probably use incorrect settings."

---

## 4. Setup

### 4.1 Setup > Network

#### 4.1.1 Host Name y Domain Name
- **Host Name** / **Domain Name**: algunos ISP los exigen para identificar al router. Normalmente se pueden dejar como están (en v1/v2, en blanco). También pueden ser necesarios para el correo de alertas (§11.1).

#### 4.1.2 IP Mode [v3/4.x]
- **IPv4 Only** (por defecto) o **Dual-Stack IP** (IPv4 + IPv6). Hay que **guardar** después de cambiar el modo para que aparezcan las pestañas IPv6 en el resto de páginas (LAN, WAN, DMZ, DHCP, Advanced Routing, Access Rules...). La reseña de SmallNetBuilder indica que no requiere reinicio.

#### 4.1.3 LAN Setting
- Pestaña **IPv4**: **Device IP Address** (por defecto 192.168.1.1) y **Subnet Mask** (por defecto 255.255.255.0). También se muestra la **MAC** del router (no editable).
- Pestaña **IPv6** (solo Dual-Stack): **IPv6 Address** (por defecto **fc00::1**) y **Prefix Length** (por defecto **7**, según la guía). Los prefijos IPv6 globales para la LAN se configuran en la WAN (campo LAN IPv6 Address o DHCP-PD).
- Al pulsar **Save**, un pop-up recuerda que tendrás que entrar por la nueva IP (**OK** aplica y **Cancel** aborta). Después hay que renovar la IP del PC.
- Nota de la guía: el rango DHCP sigue a la nueva subred. El ejemplo de la guía tiene una errata ("si eliges 192.168.15.1 los equipos recibirán 192.168.2.x"). **Revisa siempre DHCP > DHCP Setup después de cambiar la IP LAN.** **[DUDOSO: ajuste automático del pool]**

#### 4.1.4 Multiple Subnet (solo IPv4) [v3/4.x; también en algunos 2.x]
- Uso: por defecto el firewall **deniega el acceso LAN si la IP de origen no está en la subred LAN del router**. Multiple Subnet permite que el router dé salida a Internet a otras subredes de la LAN (router como equipo de borde).
- Pasos: marca **Enable Multiple Subnet** → **Add/Edit** → se abre la ventana **Multiple Subnet Management** → introduce **LAN IP Address** y **Subnet Mask** → **Add to list** (repite lo necesario; Update/Add New/Delete para editar) → **OK** → **Save**.
- Ejemplos de la guía: con LAN 192.168.1.1/24, una segunda subred 192.168.2.1/255.255.255.0. Con 192.168.1.1/255.255.255.192 se pueden crear 192.168.2.65, .129 y .193 con /26.
- Límite de subredes: **no documentado**. **[DUDOSO]**
- El DHCP del router sirve solo a la subred principal; las subredes adicionales necesitan IP estática u otro servidor DHCP. **[DUDOSO: inferido, no explícito en la guía]**
- Port Range Forwarding acepta IP destino "within the LAN range or multiple subnet IP range" (artículo Cisco smb346).

#### 4.1.5 WAN Setting (conexión a Internet)
- La tabla muestra cada interfaz (WAN1, WAN2 o DMZ, según cómo esté configurado el puerto DMZ/Internet). Si tienes cambios sin guardar, guarda antes. Luego pulsa el icono **Edit** de la interfaz para abrir **Edit WAN Connection**. En v1/v2 los campos están en la misma página Network.
- [v1/v2] Antes de elegir el tipo de conexión hay que elegir **Dual-WAN / DMZ Setting**: Dual-WAN = el puerto DMZ/Internet actúa como WAN2; DMZ = actúa como DMZ.
- Pestaña IPv6 (solo Dual-Stack): solo admite los tipos Obtain an IP Automatically, Static IP y PPPoE.

**Edit WAN Connection: campos comunes**
- **Interface**: WAN seleccionada (no editable).
- **WAN Connection Type**: uno de los tipos siguientes.
- **MTU**: **Auto** (por defecto, recomendado) o **Manual** con tamaño en bytes. Valor manual por defecto 1500. Para PPPoE suele recomendarse 1492 (la guía Linksys lo aconseja para la mayoría de usuarios DSL).

**Tipos de conexión WAN**

| Tipo | Cuándo usarlo | Campos |
|---|---|---|
| **Obtain an IP automatically** (DHCP) | Cable módem o módem/ONT en modo router que entrega IP por DHCP. Es lo más común. | Casilla opcional **Use the Following DNS Server Addresses** → **DNS Server (Required) 1** y **DNS Server 2** opcional (se usa la primera disponible). MTU. [IPv6] Enable DHCP-PD, LAN IPv6 Address. |
| **Static IP** | El ISP te dio una IP fija. | **Specify WAN IP Address**, **Subnet Mask** (IPv4) / **Prefix Length** (IPv6), **Default Gateway Address**, **DNS Server (Required) 1**, DNS Server 2. MTU. [IPv6] LAN IPv6 Address. |
| **PPPoE** | DSL / fibra con usuario y contraseña. | **Username** y **Password** (máx. **60 caracteres**), **Service Name** (según artículo Cisco), modo **Connect on Demand** con **Max Idle Time** (minutos; por defecto **5**) **o** **Keep Alive** con **Redial Period** (segundos; por defecto **30**). DNS opcional. MTU. [IPv6] DHCP-PD, LAN IPv6 Address. |
| **PPTP** | Lo exigen algunos ISP (la guía cita Europa e Israel). | **Specify WAN IP Address**, **Subnet Mask**, **Default Gateway Address**, **Username/Password** (máx. 60), Connect on Demand (Max Idle Time, 5 min) o Keep Alive (Redial Period, 30 s). MTU. Sin campos DNS. |
| **Transparent Bridge** | Unir dos segmentos de red; la WAN y la LAN quedan en la **misma subred**. **Solo una WAN** puede estar en este modo. | **Specify WAN IP Address**, **Subnet Mask**, **Default Gateway Address**, **DNS Server (Required) 1**, DNS 2, **Internal LAN IP Range** (rango LAN que se puentea). MTU. En la comunidad se reportan fallos si el rango interno choca con la configuración LAN. |
| **Heart Beat Signal** [v1/v2; aparece en la guía Linksys v2] | Servicio **solo de Australia** (BigPond Cable, Telstra). | La guía solo dice que pidas al ISP los datos necesarios; no detalla los campos. Probablemente incluyen usuario, contraseña y servidor Heart Beat. **[DUDOSO]** En las guías 4.x ya no aparece. |

- **Connect on Demand**: útil si pagas por tiempo de conexión; corta tras Max Idle Time y reconecta al haber tráfico.
- **Keep Alive**: mantiene la sesión siempre activa enviando paquetes periódicos. Cada Redial Period comprueba la conexión. La guía Linksys sugiere un Redial Period de 20 s si se cae la PPPoE.
- **Enabled DHCP-PD** (IPv6): pide la delegación de prefijo por DHCPv6. Si está activado, el campo manual **LAN IPv6 Address** se deshabilita, y viceversa.

#### 4.1.6 DMZ Setting (puerto DMZ/Internet)
- En el RV082 el segundo puerto WAN (**DMZ/Internet**) puede funcionar como **WAN2** o como **DMZ**. El LED **DMZ Mode** se enciende si el puerto está en modo DMZ y se apaga si es la segunda WAN.
- **Enable DMZ** (casilla): actívala para usar el puerto como DMZ; desactívala para usarlo como WAN2 (y configura Dual WAN, §6.1). En v1/v2 es el selector Dual-WAN / DMZ Setting.
- Una DMZ "real" requiere **IPs públicas enrutables para cada host DMZ** (pídelas al ISP). Si solo tienes una IP pública, usa **DMZ Host** (§4.4).
- Icono **Edit** → **Edit DMZ Connection**:
  - IPv4: **Subnet** (por defecto; la DMZ en una subred distinta de la WAN: IP y máscara de la DMZ) o **Range** (la DMZ en la misma subred que la WAN: rango de IPs reservado para el puerto DMZ).
  - IPv6: **Specify DMZ IPv6 Address** (sustituir "::") y **Prefix Length** (por defecto **64**).
- Reglas de firewall por defecto relacionadas: LAN→DMZ permitido, DMZ→LAN denegado, WAN→DMZ permitido, DMZ→WAN permitido (§8.2).
- [v1/v2] Si la DMZ está activa, la función **Block WAN Request se deshabilita** (guía Linksys).

### 4.2 Setup > Password
- [v3/4.x] Campos: **Old Password** (por defecto admin), **New Username** (en blanco = mantener), **Confirm New Username**, **New Password** (alfanuméricos y símbolos, **sin espacios**), **Confirm New Password**.
- **Minimum Password Complexity** (casilla **Enable**; **activada por defecto**). Exige:
  - al menos 8 caracteres;
  - distinta del usuario;
  - distinta de la contraseña actual;
  - caracteres de al menos 3 de 4 clases: mayúsculas, minúsculas, números y símbolos.
- **Password Strength Meter**: barra de rojo (inaceptable) a amarillo (aceptable) y verde (fuerte).
- **Password Aging Enforcement**: **Disable** o **Change the password after N Days** (por defecto **180**).
- Tras cambiar usuario o contraseña, la interfaz pide de nuevo las credenciales.
- **La contraseña no se puede recuperar**: si la pierdes, tendrás que hacer un reset de fábrica.
- **Es obligatorio cambiar la contraseña por defecto** para habilitar Remote Management y para deshabilitar el firewall.
- [v1/v2] Campos: Old Password, New Password (**menos de 64 caracteres**, sin espacios), Confirm New Password. El usuario no se puede cambiar (siempre admin). **[DUDOSO]**

### 4.3 Setup > Time
Sirve para fechar los registros (logs) y para aplicar los horarios de Access Rules y Content Filter.
- **Set the local time using Network Time Protocol (NTP) automatically**:
  - **Time Zone**: por defecto **(GMT-08:00) Pacific Time (US & Canada); Tijuana**.
  - **Daylight Saving**: Enabled + **Start Date** y **End Date** en formato **mm.dd** (ej. 6.25 = 25 de junio).
  - **NTP Server**: por defecto **time.nist.gov** (URL o IP).
- **Set the local time Manually**: **Date** en formato **yyyy.mm.dd** y **Hours/Minutes/Seconds** en formato **hh:mm:ss** (24 h). En v1/v2: Hours, Minutes, Seconds, Month, Day, Year.
- Consejo: configura bien la zona horaria antes de crear reglas con horario.

### 4.4 Setup > DMZ Host
- Expone **un único equipo de la LAN** a Internet (juegos, videoconferencia): todo el tráfico entrante no solicitado va a ese equipo.
- Campo **DMZ Private IP Address**. El valor **0** (por defecto) desactiva el DMZ Host.
- Se puede restringir el acceso con Access Rules. El equipo queda **fuera de la protección del firewall** (guía Linksys).
- Diferencia con el puerto DMZ: DMZ Host usa la IP pública del router (para una sola IP pública); el puerto DMZ usa IPs públicas propias.
- FAQ Linksys: si usas DMZ Host, desactiva las entradas de forwarding que entren en conflicto. El host expuesto **no** comparte "la IP pública" en el sentido de tener una propia: recibe lo que llega a la WAN.

### 4.5 Setup > Forwarding
#### 4.5.1 Port Range Forwarding
- Reenvía puertos/servicios entrantes a un servidor de la LAN. Ejemplo de la guía: puerto 80 → 192.168.1.2.
- Campos: **Service** (desplegable; si no está, usa **Service Management**), **IP Address** (IP LAN del servidor; debe estar en la LAN o en una subred de Multiple Subnet) y **Enable**.
- **Add to list** → **Save**.
- **View** (v3) / **Show Tables** (v1/v2): muestra la tabla de Port Range Forwarding o de Port Triggering (Refresh, Close).
- **Límite: 30 entradas** (datasheet: "Port forwarding: up to 30 entries"; FAQ Linksys: "you can only forward 30 ranges of ports"; teóricamente unas 4.000 sesiones simultáneas en v1).
- Conviene que el servidor tenga una IP fija o una reserva DHCP.
- No hay campo de "puerto externo distinto del interno" (no hay traducción de puerto, solo reenvío del mismo rango). **[DUDOSO: no documentado explícitamente; ninguna fuente muestra un campo de puerto interno distinto]**
- No hay selector de WAN en la entrada de forwarding. **[DUDOSO]**

#### 4.5.2 Port Triggering
- El router vigila el tráfico **saliente** en un rango "trigger". Al detectarlo, abre el rango **entrante** hacia el PC que generó ese tráfico.
- Campos: **Application Name**, **Trigger Port Range** (inicio y fin), **Incoming Port Range** (inicio y fin), **Enable**.
- **Límite: 30 aplicaciones**.

#### 4.5.3 Service Management (desde Forwarding)
- **Service Name**, **Protocol** (TCP/UDP...), **Port Range** → **Add to list** → **OK** (v1/v2: Save Setting → Exit). Hasta **30** servicios definidos por el usuario.

### 4.6 Setup > UPnP
- Permite que las aplicaciones de Windows abran y cierren puertos automáticamente. **Por defecto: No** (deshabilitado). Se recomienda dejarlo desactivado salvo que haga falta.
- **UPnP Function**: Yes/No.
- Tabla de entradas UPnP manuales: **Service**, **Name or IP Address**, **Enable** → Add to list. **Hasta 30** entradas.
- **View** / **Show Tables**: **UPnP Forwarding Table List** (Refresh/Close).

### 4.7 Setup > One-to-One NAT
- Asocia IPs públicas adicionales del ISP a IPs privadas concretas (NAT estático 1:1). Se pueden mapear rangos de igual longitud: primera privada con primera pública, y así sucesivamente.
- Casilla **Enable One-to-One NAT**.
- Campos: **Private Range Begin** (sin incluir la IP LAN del router), **Public Range Begin** (sin incluir la IP WAN del router) y **Range Length** (1 para una sola IP; no puede superar las IPs válidas).
- Límite: **10 entradas** según el artículo Cisco SMB544 (firmware 4.2.2.08); **64 rangos** según las guías Linksys v1/v2. **[DUDOSO: depende del firmware]**
- **El NAT 1:1 no abre el firewall por sí solo**: hay que crear Access Rules (WAN→LAN Allow) hacia la IP privada para permitir el tráfico entrante. La guía Linksys v2 tiene una nota contradictoria al respecto ("will be allowed unless..."); la guía v1 y la lógica del firewall indican que **sin regla no entra**. En la comunidad se recomienda escribir las reglas con la **IP privada** como destino. **[DUDOSO en detalle]**
- Ejemplo de la guía Linksys: el ISP asigna 209.19.28.16-31; .16 es la WAN del router; las privadas 192.168.168.2 a .15 se publican en las públicas correspondientes.

### 4.8 Setup > MAC Address Clone (v1/v2: "MAC Clone")
- Página con la configuración actual. Icono **Edit** por WAN → **Edit MAC Address Clone**:
  - **User Defined WAN MAC Address**: introduce los 12 dígitos de la MAC registrada en el ISP.
  - **MAC Address from this PC**: clona la MAC del PC desde el que estás configurando.
- v1/v2: campos **User Defined WAN1 MAC Address** / **WAN2 MAC Address** en la misma página.

### 4.9 Setup > Dynamic DNS (DDNS)
- Asocia un nombre fijo a una IP WAN dinámica. Proveedores admitidos: **DynDNS.org** y **3322.org** (China). **Por defecto: Disable.**
- Página con el estado por WAN → **Edit** → **Edit Dynamic DNS Setup**:
  - **DDNS Service** (Disable / DynDNS.org / 3322.org).
  - **Username**, **Password**.
  - **Host Name** en **tres campos** (ej. `myhouse` . `dyndns` . `org`).
  - Botón **Register** (enlace al alta en DynDNS).
  - Solo lectura: **Internet IP Address** y **Status**.
- Nota práctica: DynDNS dejó de ser gratuito, y el router no admite otros proveedores (No-IP, DuckDNS...). **[DUDOSO/sin verificar si funciona hoy; dato externo a las guías]**

### 4.10 Setup > Advanced Routing
**Dynamic Routing (pestaña IPv4)**
- **Working Mode**:
  - **Gateway** (por defecto): el router da salida a Internet con NAT.
  - **Router**: el RV082 es un router más dentro de una red donde **otro equipo es el gateway a Internet**. En este modo no hay Internet a través del RV082 si no existe otro gateway. La guía aconseja **deshabilitar el firewall** de este router en ese caso, porque la protección la da el gateway.
- **RIP**: Enabled/Disabled (por defecto **Disabled**).
  - **Receive RIP versions**: None, RIPv1, RIPv2, Both RIP v1 and v2.
  - **Transmit RIP versions**: None, RIPv1, RIPv2 - Broadcast (recomendado), RIPv2 - Multicast.
  - RIPv1 es classful (sin VLSM ni autenticación); RIPv2 lleva máscara y admite autenticación por contraseña (la interfaz no muestra un campo de contraseña RIP). **[DUDOSO]**
- Pestaña IPv6 (Dual-Stack): casilla **RIPng**.

**Static Routing**
- Campos: **Destination IP** (dirección de red, ej. x.x.x.0), **Subnet Mask** (IPv4) / **Prefix Length** (IPv6), **Default Gateway** (siguiente salto: la IP del router que da acceso a esa red), **Hop Count** (**máx. 15**), **Interface** (LAN, WAN1, WAN2/DMZ).
- **Límite: 30 rutas**.
- **View** / **Show Routing Table**: **Routing Table Entry List**.
- Guía Linksys v1/v2: "In order to use static routing, the Router's DHCP settings must be disabled". La guía Cisco 4.x **no** dice esto. **[DUDOSO; en la práctica no suele ser necesario]**

### 4.11 Setup > IPv6 Transition [v3/4.x]
- Con Dual-Stack activado, un **túnel 6to4** queda **habilitado por defecto**. Casilla para activarlo o desactivarlo.
- Prefijos 6to4: `2002:<IP WAN en hexadecimal>::`.
- La guía recomienda: activar los flags RA en DHCP > Router Advertisement, probar desactivando temporalmente el firewall y luego crear Access Rules.

---

## 5. DHCP

### 5.1 DHCP > DHCP Setup (v1/v2: DHCP > Setup)
Pestañas IPv4/IPv6 (IPv6 solo con Dual-Stack). **DHCP Relay solo existe en IPv4.**

**Modo**
- **Enable DHCP Server** (por defecto activo): asigna IPs automáticamente. Según la guía, hasta **50** dispositivos.
- **DHCP Relay** (IPv4): reenvía por unicast las peticiones DHCP de los clientes a otro servidor. Campo obligatorio: **DHCP Server IP Address**.
- Si desactivas ambos, todos los equipos necesitarán IP, máscara, puerta de enlace y DNS estáticos, sin duplicados.

**Dynamic IP** (solo servidor)
- **Client Lease Time**: **5 a 43.200 minutos**; por defecto **1440** (24 h).
- **Range Start** / **Range End**: por defecto **.100 a .149**. La guía indica que el rango admite **hasta 50 IPs** ("Valid values are 100~149") y que no debe incluir la IP del router.
- **[DUDOSO] Límite de 50**: la documentación oficial dice 50. En la comunidad Cisco se afirma que se puede ampliar el pool (hasta 253) cambiando inicio y fin, y el FAQ Linksys dice "The Router will support up to 253 IP addresses". Prueba en tu firmware.

**DNS** (solo servidor)
- **DNS Server 1** y **2**. **0.0.0.0** (por defecto) = entregar el DNS obtenido por la WAN.

**WINS** (IPv4)
- **WINS Server**; 0.0.0.0 por defecto. Se entrega a los clientes DHCP.

**Static IP (reservas / "IP & MAC binding")** (IPv4)
- **Hasta 100** dispositivos en la lista (se puede usar aunque el router no sea el servidor DHCP).
- Método A: **Show unknown MAC addresses** → aparece la **IP & MAC binding list** con los equipos conectados → escribe un **Name**, marca **Enable** (o "seleccionar todos") → **OK** → lee el aviso y espera a que aparezcan en la lista.
- Método B (manual): **Static IP Address** (0.0.0.0 = que el router elija una), **MAC Address** (**sin separadores**, ej. 001122AABBCC), **Name**, **Enable** → **Add to list**.
- Control de acceso (IP-MAC binding):
  - **Block MAC address on the list with wrong IP address**: bloquea a un equipo listado si cambia su IP a otra distinta de la asignada.
  - **Block MAC address not on the list**: bloquea a cualquier equipo que no esté en la lista.
  - Cuidado: con la segunda opción, un PC nuevo (incluido el tuyo) queda sin acceso. Añádelo antes de activarla.

**DNS Local Database**
- El router actúa como DNS local para nombres concretos: **Host Name** (ej. ejemplo.com) + **IP Address** → Add to list. Si el nombre no está en la base local, la consulta se reenvía al DNS de la WAN.
- Requisito: que los clientes usen la **IP LAN del router como DNS**. La guía sugiere configurarlo en el PC. Como alternativa, se puede poner la IP del router como DNS en el DHCP. **[DUDOSO: alternativa no documentada]**
- Nota de la guía: si omites el dominio de nivel superior, Windows añade ".com".

### 5.2 DHCP > DHCP Status (v1/v2: Status)
- Servidor: **DHCP Server** (IP), **Dynamic IP Used**, **Static IP Used**, **DHCP Available**, **Total**.
- **Client Table** (IPv4/IPv6): Client Host Name, IP Address, MAC Address, Client Lease Time y el icono **Delete** (papelera), que libera la concesión y desconecta al cliente. Botón **Refresh**.

### 5.3 DHCP > Router Advertisement (IPv6) [v3/4.x]
- **Enable Router Advertisement** (RADVD).
- **Advertise Mode**: **Unsolicited Multicast** (por defecto), con **Advertisement Interval** de 10 a 1800 s (por defecto **30**), o **Unicast only**.
- **RA Flags**: **Managed** (direcciones por DHCPv6) y **Other** (otros datos por DHCPv6).
- **Router Preference**: High (por defecto), Medium o Low.
- **MTU**: por defecto **1500** (1492 típico en PPPoE).
- **Router Lifetime**: por defecto **3600** s.

---

## 6. System Management

### 6.1 System Management > Dual WAN
Requisito: el puerto DMZ/Internet debe estar en modo WAN (Enable DMZ desmarcado). Si está en DMZ, no se puede configurar Dual WAN.

**Modos** (página principal):
- **Smart Link Backup**: conmutación por error (failover). Solo la WAN principal está activa; la otra entra cuando la principal cae. Se elige la **Primary WAN** (WAN1 o WAN2).
- **Load Balance**: usa las dos WAN a la vez con **Weighted Round Robin** según el ancho de banda declarado. **Las consultas DNS no se balancean.** El balanceo es por conexión/sesión, no por paquete, así que una sola descarga no suma las dos líneas. **[DUDOSO: "por conexión" es el comportamiento típico, no lo dice la guía literalmente; la guía Linksys dice que si se supera el ancho de banda declarado el router usa otra WAN para la siguiente conexión]**

**Tabla de interfaces** → icono **Edit** (Configuration) por WAN → página **Dual WAN Settings**:

1. **Max Bandwidth Provided by ISP** (WAN Bandwidth)
   - **Upstream**: por defecto **512 kbit/s**.
   - **Downstream**: por defecto **512 kbit/s**.
   - Hay que poner los valores reales del ISP: los usa el balanceador (y Bandwidth Management). Si se supera, el router usa la otra WAN para la siguiente conexión.
2. **Network Service Detection** (detección de caída por ping)
   - Casilla Enable.
   - **Retry count**: por defecto **5**.
   - **Retry timeout**: por defecto **30** s (espera entre pings).
   - **When Fail**:
     - **Generate the Error Condition in the System Log**: solo registra en el log, **sin failover**.
     - **Remove the Connection** (el artículo Cisco la llama "Keep System Log and Remove the Connection"): hace **failover** a la otra WAN; cuando la conexión vuelve, el tráfico se restaura.
   - Destinos que se comprueban (casillas): **Default Gateway**, **ISP Host** (IP), **Remote Host** (IP), **DNS Lookup Host** (nombre de host o dominio).
   - Consejo: sin Network Service Detection el router solo detecta la caída del enlace físico o de PPP. Si el módem sigue con enlace pero sin Internet, no habrá failover. Para un failover fiable, activa NSD con un Remote Host fiable (por ejemplo, un DNS público). **[recomendación práctica, no de la guía]**
3. **Protocol Binding** (solo en modo **Load Balance**)
   - Obliga a que cierto tráfico salga por una WAN concreta: **Service** (o All Traffic), **Source IP** (desde... hasta), **Destination IP** (desde... hasta), **Interface** (WAN1/WAN2), **Enable** → Add to list → Save.
   - Usos típicos: VoIP por una línea, correo por la WAN con IP fija, etc.
   - La guía 4.x dice "for Cisco RV016 only", pero el artículo SMB622 (RV042/RV042G/RV082, firmware 4.2.1.02) y los datasheets confirman que el RV082 lo tiene. **[la guía tiene una errata]**
   - Comunidad: el Protocol Binding no se aplica al tráfico VPN. Hay casos reportados de que no vuelve a la WAN original tras el failover. **[anecdótico]**

**[v1 / firmware antiguo] opciones de Load Balance distintas**
- Ancho de banda elegido de listas: Upstream 64K/128K/256K/384K/512K/1024K/1.5M/2M/2.5M o más; Downstream 512K/1024K/1.5M/2M/2.5M o más.
- **Intelligent Balancer (Auto)** (Weighted Round Robin) o **user defined**: "si el ancho de banda upstream/downstream supera 30-90 %, levantar el segundo enlace", y tiempo de inactividad (None/10...60 min) para cerrar el segundo enlace.

### 6.2 System Management > Bandwidth Management (QoS)
- **Bandwidth Provided by ISP** (por WAN): **Upstream** y **Downstream** en kbit/s. Por defecto 512/512. La guía 4.x pone "Downstream default 12 kbit/sec", lo que parece una errata de 512. **[DUDOSO]**
- **Bandwidth Management Type** (se elige uno):
  - **Rate Control**: ancho de banda **mínimo garantizado** y **máximo** por servicio y por IP o rango.
  - **Priority**: prioridad **High** o **Low** por servicio. Los datasheets hablan de "3 priority levels" (posiblemente High, Normal implícito y Low). **[DUDOSO]**
- Campos de cada regla:
  - **Interface** (WAN1/WAN2, casillas).
  - **Service** (o Service Management).
  - **IP** (solo Rate Control): IP o rango interno. El valor por defecto (0 / vacío) = todas las IPs internas.
  - **Direction**: **Upstream** (saliente) o **Downstream** (entrante).
  - **Min. Rate** y **Max. Rate** (kbit/s; solo Rate Control).
  - **Priority** (solo Priority): High o Low.
  - **Enable**.
  - → Add to list (o "Update this application") → **Save**.
- **Límite: 100 servicios/reglas.**
- Ejemplo oficial (VoIP Vonage):
  - Servicios: "Vonage VoIP" (UDP 5060-5070) y "Vonage 2" (UDP 10000-25000).
  - 4 reglas Rate Control: cada servicio en Upstream y en Downstream, con Min 40 kbit/s y Max 80 kbit/s.
- Ejemplo oficial (SMB3060): limitar la IP 192.168.0.26 en WAN1, All Traffic (TCP&UDP 1-65535), Upstream máx. 200 kbit/s y Downstream máx. 4096 kbit/s.
- Además existe la **prioridad por puerto LAN** (Port Management, §7.1) y el datasheet cita "application-based priority on WAN port".

### 6.3 System Management > SNMP
- Soporta **SNMP v1/v2c** (no v3), MIB-II y MIB privadas. Operaciones get/next/set. Envía traps (reinicios, ciclos de energía, eventos de enlace WAN).
- **Enabled SNMP**: **habilitado por defecto** (tanto en la guía 4.x como en la Linksys). Conviene desactivarlo o cambiar las comunidades si no se usa.
- **System Name** (hostname), **System Contact**, **System Location**.
- **Get Community Name**: por defecto **public** (máx. 64 caracteres alfanuméricos).
- **Set Community Name**: por defecto **private** (máx. 64).
- **Trap Community Name**: por defecto **public** (máx. 64).
- **Send SNMP Trap to** (IPv4: IP o dominio del gestor) y **Send SNMP Trap to (For IPv6)** con Dual-Stack.

### 6.4 System Management > Discovery-Bonjour [v3/4.x]
- Casilla **Enable**; **habilitado por defecto**. El router anuncia sus servicios por multicast en la LAN (usado por la herramienta FindIT de Cisco).

### 6.5 System Management > Diagnostic
- **DNS Name Lookup**: campo **Look up the name** (ej. www.cisco.com, **sin "http://"**; si no, da "Address Resolving Failed") → **Go**. Requiere un DNS válido en la WAN.
- **Ping**: **IP Address** → **Go**. Resultado: **Status** (Testing / Test Succeeded / Test Failed), **Packets** (enviados/recibidos/% de pérdida) y **Round Trip Time** (mín./máx./media). Necesita una IP (usa DNS Lookup para obtenerla).
- Método recomendado por la guía: primero haz ping al DNS o a un servidor del ISP; si funciona, a equipos fuera del ISP. Así sabrás si el problema es del ISP.
- No hay traceroute en la interfaz. **[DUDOSO]**

### 6.6 System Management > Factory Default
- Botón **Return to Factory Default Setting** → confirmación **OK** → el router se reinicia con la configuración de fábrica (192.168.1.1, admin/admin).
- Alternativa por hardware: el botón **Reset** (§15.13).

### 6.7 System Management > Firmware Upgrade
- **Browse** / **Choose File** → selecciona el archivo **extraído** → **Firmware Upgrade Right Now** (v3: "Upgrade") → OK.
- **Firmware Download from Web Site**: abre la página de descargas de Cisco (en v1/v2, de Linksys).
- Avisos:
  - Tarda **varios minutos**. Troubleshooting de la guía: "aproximadamente diez minutos".
  - **No apagues, no pulses Reset, no cierres el navegador ni desconectes** durante el proceso.
  - **Instalar una versión anterior (downgrade) restaura los valores de fábrica**: se pierde la configuración.
  - Después del reinicio se pierde la sesión; puede que tengas que renovar la IP del PC.
- Si la actualización falla: repítela. Si el LED **Diag** sigue parpadeando, la imagen está dañada; la guía indica recuperarla con una **utilidad TFTP** (no detalla el procedimiento). **[DUDOSO en procedimiento]**
- [v1/v2] Hay **dos imágenes** (activa y de respaldo). En **Restart** se puede arrancar con **Backup Firmware Version** si la actualización sale mal.
- HW v3 solo admite firmware 4.x; HW v1/v2 solo 1.x/2.x. No se puede cruzar.

### 6.8 System Management > Restart
- [v3] Botón **Restart Router** → OK. Es el método recomendado: **envía el log por correo (si está activado) antes de reiniciar**.
- [v1/v2] Selector **Active Firmware Version** / **Backup Firmware Version** + **Restart Router**.
- Reinicio por hardware: pulsa **Reset** durante **1 segundo**.

### 6.9 System Management > Backup and Restore [v3/4.x]
- El router guarda **dos configuraciones**:
  - **Startup**: la que carga al arrancar.
  - **Mirror**: copia de la última configuración válida. Se actualiza automáticamente con la Startup **tras 24 h de funcionamiento estable** (sin reinicios ni cambios). Cualquier cambio reinicia el contador de 24 h. Si la Startup falla, se usa la Mirror.
- **Backup Configuration File**: **Backup Startup Configuration** (archivo por defecto `Startup.config`) y **Backup Mirror Configuration** (`Mirror.config`). Consejo: ponles la fecha en el nombre.
- **Restore Startup Configuration File**: Browse → archivo `.config` → **Restore** (hasta 1 minuto) → **System Management > Restart > Restart Router**. **No se aplica hasta reiniciar.**
- **Copy Configuration File**:
  - **Copy Startup to Mirror**: inmediato, sin posibilidad de cancelar.
  - **Copy Mirror to Startup**: inmediato y reinicia el router. **Si la Mirror sigue en valores de fábrica, esto equivale a un reset de fábrica.**
- Truco de la guía: antes de hacer cambios arriesgados, pulsa Copy Startup to Mirror. Si algo sale mal, Copy Mirror to Startup lo revierte.
- Comunidad: hay casos de un `startup.config` restaurado en **otra** unidad que la dejó sin responder. Restaura preferentemente en el mismo equipo, con el mismo firmware. **[anecdótico]**
- El archivo de configuración puede contener las credenciales; guárdalo en un lugar seguro. **[DUDOSO: no se sabe si va cifrado]**

### 6.10 [v1/v2] System Management > Setting Backup
- **Import Configuration File**: Browse → **Import** (hasta 1 minuto) → reiniciar para aplicar.
- **Export Configuration File**: **Export** → archivo por defecto **`RV082.exp`**.

---

## 7. Port Management (v1: "LAN Management")

### 7.1 Port Management > Port Setup (v1: LAN Setup, "Basic Per Port Config.")
Tabla por puerto:
- **Port ID** e **Interface** (LAN/WAN/DMZ), solo lectura.
- **Disable**: desactiva el puerto (por defecto todos activos).
- **Priority** (solo LAN): **High** o **Normal** (por defecto **Normal**). Es la QoS por puerto, útil para un teléfono IP o una videoconferencia.
- **Speed**: **10M** o **100M**. Solo se puede cambiar si **Auto Neg** está desactivado.
- **Duplex**: **Half** o **Full**. Solo se puede cambiar si **Auto Neg** está desactivado.
- **Auto Neg.**: **Enable** (por defecto activado).
- **VLAN** (solo LAN; v2/v3): todos los puertos en **VLAN 1** por defecto. El número de VLAN disponibles es igual al de puertos LAN: **8 en el RV082**.
  - **No hay comunicación entre VLANs distintas.**
  - VLAN **por puerto**, sin etiquetado **802.1Q**. Un puerto solo puede estar en una VLAN, y los puertos WAN/DMZ no admiten VLAN.
  - Todas las VLAN comparten la **misma subred IP y el mismo DHCP**: aíslan en capa 2, no crean subredes nuevas. **[DUDOSO pero coherente con el ejemplo SMB3060, donde ambas VLAN están en 192.168.0.x]**
  - Ejemplo de la guía: un switch de invitados en el puerto 4 se pone en VLAN 2 para que no vea servidores ni impresoras de la VLAN 1.

### 7.2 Port Management > Port Status (v1: LAN Status)
- Seleccionas el **Port ID** → Refresh.
- **Summary**: Type, Interface, Link Status, Port Activity, Speed Status (10/100 Mbps), Duplex Status, Auto negotiation, VLAN (y Priority en v1).
- **Statistics**: Port Receive Packet Count, Port Receive Packet Byte Count, Port Transmit Packet Count, Port Transmit Packet Byte Count, Port Packet Error Count.

---

## 8. Firewall

### 8.1 Firewall > General

| Campo | Por defecto | Notas |
|---|---|---|
| **Firewall** (Enable/Disable) | **Enabled** | Desactivarlo también desactiva Access Rules y Content Filter. **Solo se puede desactivar tras cambiar la contraseña por defecto** (si no, salta un aviso que lleva a Setup > Password). [v1/v2] Al desactivarlo, SPI, DoS y Block WAN Request se apagan y Remote Management se **habilita**. |
| **SPI (Stateful Packet Inspection)** | Enabled | Inspección con estado. Requiere el firewall activo. |
| **DoS (Denial of Service)** | Enabled | Protege de SYN Flooding, Smurf, LAND, Ping of Death, IP Spoofing y ataques de reensamblado. Requiere el firewall activo. |
| **Block WAN Request** | Enabled | Descarta peticiones TCP no aceptadas e ICMP desde la WAN: el router **no responde a ping en la WAN**. Requiere el firewall activo. [v1/v2] Se deshabilita si la DMZ está activa. |
| **Remote Management** | **Disabled** | Acceso a la interfaz desde la WAN. Requiere el firewall activo y una contraseña fuerte. Campo **Port**: ver la tabla de §8.1.1. URL: `http://<IP WAN>:<puerto>`, o `https://<IP WAN>:<puerto>` con HTTPS. |
| **HTTPS** | **Enabled** | Sesiones web cifradas. **Si se desactiva, QuickVPN deja de funcionar.** |
| **Multicast Pass Through** | Disabled | Deja pasar multicast IP de Internet a la LAN (IPTV, juegos, multimedia). **No** pasa multicast por túneles IPsec. |
| **MTU** [v1/v2, en esta página] | Enabled, 1500 | En 4.x el MTU se configura por WAN (Auto/Manual). |

**Restrict Web Features** (todas **desactivadas** por defecto):
- **Java**: bloquea applets Java.
- **Cookies**: bloquea todas las cookies.
- **ActiveX**: bloquea controles ActiveX.
- **Access to HTTP Proxy Servers**: bloquea el acceso a proxies HTTP (puertos 80/8080).
- **Don't block Java/ActiveX/Cookies/Proxy to Trusted Domains**: solo aparece si se marcó alguna de las anteriores. Permite esas funciones en la lista de **dominios de confianza** (Add to list / Update / Delete).
- Nota: con HTTPS los sitios cifrados no se pueden inspeccionar, así que estos bloqueos tienen poca eficacia hoy. **[observación técnica, no de la guía]**

#### 8.1.1 Puerto de gestión remota y HTTPS: lo que dicen las fuentes

| Fuente | Puerto por defecto de Remote Management |
|---|---|
| Guía Cisco RV0xx (4.x, 2011-2013) | **80** ("8080 is usually used for this purpose") |
| Artículo Cisco SMB5074 (firmware 4.2.3.06) | **443** |
| Guía Linksys v1/v2 | Elegir **80 o 8080** |

- Conclusión: depende de la versión de firmware. Comprueba el campo **Port** en tu equipo. **[DUDOSO]**
- Acceso remoto recomendado: HTTPS activado, puerto no estándar (ej. 8443) **[recomendación]**, contraseña compleja, y si es posible una regla de acceso que limite la IP de origen (verifica que las Access Rules se apliquen al tráfico hacia el propio router: no está documentado **[DUDOSO]**). En LAN, la interfaz responde por HTTP (80) y por HTTPS (443). Reglas implícitas: "HTTP service from the LAN to the router is always allowed".
- QuickVPN usa HTTPS (443) y, según la configuración, el puerto 60443. **[DUDOSO; lo cubre el documento VPN]**

### 8.2 Firewall > Access Rules
**Reglas por defecto** (no editables; **prioridad más baja**):

| Origen → Destino | Acción |
|---|---|
| LAN → WAN | Allow |
| WAN → LAN | **Deny** |
| LAN → DMZ | Allow |
| DMZ → LAN | **Deny** |
| WAN → DMZ | Allow |
| DMZ → WAN | Allow |

**Cuatro reglas implícitas siempre activas que ninguna regla personalizada puede anular:**
1. Servicio **HTTP** desde la LAN al router: siempre permitido (no puedes bloquearte la interfaz desde la LAN con reglas).
2. **DHCP** desde la LAN: siempre permitido.
3. **DNS** desde la LAN: siempre permitido.
4. **Ping** desde la LAN al router: siempre permitido.

**Tabla de reglas** (pestañas IPv4/IPv6):
- Columnas: **Priority** (1 = la más alta; editable con un desplegable), **Enable**, **Action**, **Service**, **Source Interface**, **Source**, **Destination**, **Time**, **Day**, **Edit** y **Delete** (papelera).
- El router asigna una prioridad automáticamente al crear la regla; se puede cambiar después. **Si dos reglas entran en conflicto, gana la de mayor prioridad** (número menor).
- Paginación: **Rows per page** / **Entries Per Page**, **Page** / **Jump to**.
- Botones: **Add New Rule** (o Add), **Restore to Default Rules** (borra todas las reglas personalizadas).
- **Límite: 50 reglas** (datasheet y guía: "Access Rules: Up to 50 entries").

**Campos al añadir o editar una regla**
- **Action**: **Allow** / **Deny**.
- **Service**: desplegable (incluye **All Traffic** y servicios predefinidos o propios) o **Service Management**.
- **Log**: **Log packets match this rule** / **Not log**. Solo aplica si el log está activo; ver Allow/Deny Policies en §11.1.
- **Source Interface**: **LAN**, **WAN1**, **WAN2**, **DMZ** (si la DMZ está activa) o **ANY**.
- **Source IP**: **ANY**, **Single** (una IP) o **Range** (desde-hasta; solo IPv4). En IPv6: **Subnet** (IP/prefijo).
- **Destination IP**: igual que Source IP.
- **Scheduling** (solo IPv4):
  - **Time**: **Always** (por defecto) o **Interval**.
  - **From / To**: hh:mm en formato 24 h. **00:00 a 00:00** = todo el día.
  - **Effective on**: **Everyday** (por defecto) o días concretos (Sun...Sat).
- **Save** → mensaje "Success": OK para añadir otra regla, Cancel para volver a la tabla.
- Alternativa: **Wizard > Access Rule Setup** (§12).

**Consejos y cautelas**
- La guía avisa: "con reglas personalizadas es posible desactivar toda la protección o bloquear todo el acceso a Internet". Antes de experimentar, guarda una copia de la configuración (§6.9).
- Si el router está en modo **Router** o usas **Multiple Subnet**, puede que necesites reglas para las subredes adicionales. **[DUDOSO]**
- Hay reportes de comportamientos inesperados de la prioridad con NAT 1:1 o forwarding: por ejemplo, un Allow de prioridad 1 + un Deny de prioridad 2 para SSH que no filtraba. Prueba siempre desde fuera. **[anecdótico]**

### 8.3 Firewall > Content Filter
**Nota:** si **Cisco ProtectLink** está activado, el Content Filter se desactiva automáticamente.
- **Forbidden Domains**: casilla **Enable Block Forbidden Domains**. Lista de dominios (campo **Add** → Add to list / Update / Delete).
  - Bloquear `yahoo.com` impide `yahoo.com/...` pero **no** `mail.yahoo.com` (otro dominio).
  - La guía Linksys v2 habla de introducir "IP Address(es)" de los sitios.
- **Website Blocking by Keywords**: casilla **Enable Website Blocking By Keywords**. Bloquea cualquier URL que contenga la cadena (ej. "yahoo" bloquea www.yahoo.com, finance.yahoo.com y midominio.com/news/yahoo).
- **Scheduling**: **Always** o **Interval** (From/To hh:mm 24 h; Effective on Everyday o días concretos). En v1/v2: "Always" o "From ... (24-hour) ... days".
- Limitación práctica: el filtrado es por URL/Host HTTP. Con HTTPS solo podría actuar sobre el nombre (SNI/DNS), y es probable que **no funcione con sitios HTTPS**. **[DUDOSO]**
- Límite de entradas: no documentado. **[DUDOSO]**

---

## 9. Cisco ProtectLink Web (Trend Micro)
- Servicio opcional de **filtrado de URL en la nube y reputación web** de Trend Micro con licencia anual. Disponible en RV042, RV082 y RV016 (**no** en RV042G).
- **Fin de venta de licencias: 31 de julio de 2013.** Hoy se considera **inoperativo** (servicio en la nube discontinuado). **[DUDOSO: se infiere del EoS; no hay confirmación explícita del apagado del servicio]**
- Menú: enlaces **Learn more / Free Trial** (30 días), **Register ProtectLink services and obtain an Activation Code (AC)** y **Use the Activation Code (AC) to activate**. Desde el firmware **4.2.2.08** los dos primeros enlaces ya no se muestran. El código de activación también permite **transferir la licencia** a otro router.
- **Global Settings**:
  - **Enable Approved URL list**: hasta **20** URLs siempre accesibles, separadas por ";".
  - **Enable Approved Client list**: hasta **20** IPs o rangos (ej. `10.1.1.1;10.1.1.5` o `10.1.1.0-10.1.1.10`) exentos de filtrado.
- **Web Protection**:
  - **Enable URL Filtering** (categorías y subcategorías para **Business Hours** y **Leisure Hours**; columna Instances Blocked; Reset Counters).
  - **Enable Web Reputation** (nivel High / **Medium**, el recomendado / Low).
  - **Business Hour Setting**: Business Days; Business Times "All day (24 hours)" o mañana/tarde.
  - **URL overflow control**: **Temporarily block URL requests** (por defecto, recomendado) o **Temporarily bypass URL verification**.
- **License**: Update Information, View detailed license online, Status (Activated/Expired), Platform (Gateway Service), License expires on (1 año tras la activación), Renew.
- En la Linksys v1/v2 existía también una pestaña "ProtectLink" en algunos firmwares. **[DUDOSO]**

---

## 10. VPN (solo resumen; ver el documento específico)
- Submenús: **Summary**, **Gateway to Gateway**, **Client to Gateway**, **VPN Client Access** (usuarios QuickVPN y certificados), **VPN Passthrough**, **PPTP Server**.
- Capacidades (HW v3): **100 túneles IPsec**, **50 usuarios QuickVPN**, **5 clientes PPTP**. HW v1/v2: **50 túneles**.
- **VPN Passthrough**: IPSec, PPTP y L2TP Passthrough, **todos habilitados por defecto**.
- **PPTP Server**: deshabilitado por defecto. Rango por defecto 192.168.1.200 a 192.168.1.204 (la guía escribe "92.168.1.204", una errata). Debe quedar fuera del rango DHCP.
- Al guardar VPN Client Access por primera vez, el router puede ofrecer **cambiar la IP LAN** para evitar conflictos con las redes remotas de QuickVPN.
- **QuickVPN requiere HTTPS activado** en Firewall > General.

---

## 11. Log

### 11.1 Log > System Log
**Syslog**
- **Enable Syslog** + **Syslog Server** (nombre o IP). Envía todo el registro, incluidas las IPs de origen y destino y los servicios. **Hay que reiniciar el router para que el cambio surta efecto.**

**E-mail**
- **Enable E-Mail Alert**.
- **Mail Server**: SMTP del ISP (IP o nombre). Puede requerir el Host Name configurado en Setup > Network. **No hay campos de autenticación SMTP ni TLS**, por lo que no funciona con Gmail o Office 365 modernos. **[DUDOSO: no hay campos en la guía; deducción]**
- **Send Email to**: dirección de destino.
- **Log Queue Length**: por defecto **50** entradas.
- **Log Time Threshold**: por defecto **10** minutos.
- El correo se envía cuando se cumple **cualquiera** de los dos umbrales.
- **Email Log Now**: envío inmediato (sirve de prueba).

**Log Setting**
- **Alert Log**: Syn Flooding, IP Spoofing, Win Nuke, Ping of Death, Unauthorized Login Attempt y (4.x) **Output Blocking Event** (eventos de ProtectLink).
- **General Log**: System Error Messages, Deny Policies, Allow Policies, Configuration Changes, Authorized Login. En v1/v2 también Content Filtering y Data Inspection.

**Botones**
- **View System Log**: pop-up con un desplegable para elegir el tipo de log (v1/v2: ALL, System Log, Access Log, Firewall Log, VPN Log), Refresh, Clear y Close. Cada entrada muestra fecha y hora, tipo y mensaje (política, IP origen, MAC...).
- **Outgoing Log Table**: LAN IP, Destination URL/IP, Service/Port.
- **Incoming Log Table**: Source IP, Destination Port.
- **Clear Log Now**: borra el log sin enviarlo.

### 11.2 Log > System Statistics
Por interfaz (LAN, WAN1, WAN2/DMZ): **Device Name** (eth0, eth1...), **Status**, **IP Address**, **MAC Address**, **Subnet Mask**, **Default Gateway**, **DNS**, Received/Sent/Total Packets, Received/Sent/Total Bytes, **Error Packets Received** y **Dropped Packets Received**. Botón Refresh.
(La guía 4.x titula esta sección por error "Viewing the System Log".)

---

## 12. Wizard, Support y Logout

### 12.1 Wizard
- **Basic Setup** → **Launch Now** (pop-up):
  1. (RV082) Elegir si el puerto DMZ/Internet será **Dual WAN** (WAN2) o **DMZ**.
  2. **Host Name** y **Domain Name**.
  3. Tipo de conexión de **WAN1**: Obtain an IP automatically (con "Use DNS Server provided by ISP" por defecto u "Use the Following DNS Server Addresses"), Static IP o PPPoE.
  4. Si hay Dual WAN, lo mismo para **WAN2**.
  5. Pantalla final → **Save Settings**. Botones **Next / Previous / Exit** en cada paso.
- **Access Rule Setup** → **Launch Now**: explicación de las reglas por defecto → Action (Allow/Deny) → Service → Log → Source (interfaz + Any/Single/Range) → Destination (Any/Single/Range) → Scheduling (Always / horario) → **Save Settings**.

### 12.2 Support
- [v1/v2] Botones **On Line Manual** y **Linksys Web Site** (enlaces externos).
- [v3] Enlaces de soporte y ayuda de Cisco. **[DUDOSO: contenido exacto]**

### 12.3 Logout
- Cierra la sesión de gestión y vuelve a la pantalla de login (en v1/v2 pide confirmación).

---

## 13. Tabla consolidada de valores por defecto

| Área | Parámetro | Por defecto |
|---|---|---|
| Acceso | IP / máscara LAN | 192.168.1.1 / 255.255.255.0 |
| Acceso | Usuario / contraseña | admin / admin |
| Setup > Network | IP Mode | IPv4 Only |
| Setup > Network | IPv6 LAN (Dual-Stack) | fc00::1, prefijo 7 (según la guía) |
| Setup > Network | Tipo de WAN | Obtain an IP automatically (DHCP) **[DUDOSO: es el típico de fábrica]** |
| Setup > Network | MTU WAN | Auto (Manual = 1500) |
| Setup > Network | PPPoE/PPTP Max Idle Time | 5 min |
| Setup > Network | PPPoE/PPTP Redial Period | 30 s |
| Setup > Network | DMZ IPv6 prefix | 64 |
| Setup > Password | Minimum Password Complexity | Enabled |
| Setup > Password | Password Aging | 180 días al activar "Change the password after" |
| Setup > Time | Zona horaria | (GMT-08:00) Pacific Time |
| Setup > Time | NTP Server | time.nist.gov |
| Setup > DMZ Host | IP | 0 (deshabilitado) |
| Setup > UPnP | UPnP | No (deshabilitado) |
| Setup > DDNS | Servicio | Disable |
| Setup > Advanced Routing | Working Mode | Gateway |
| Setup > Advanced Routing | RIP | Disabled |
| Setup > IPv6 Transition | 6to4 | Habilitado al activar Dual-Stack |
| DHCP | Servidor DHCP | Enabled |
| DHCP | Rango | .100 a .149 |
| DHCP | Lease | 1440 min (válido de 5 a 43.200) |
| DHCP | DNS / WINS | 0.0.0.0 (usa los de la WAN) |
| DHCP > Router Advertisement | Modo / Intervalo / Preferencia / MTU / Lifetime | Unsolicited Multicast / 30 s / High / 1500 / 3600 s |
| Dual WAN | Upstream / Downstream | 512 / 512 kbit/s |
| Dual WAN | NSD Retry count / timeout | 5 / 30 s |
| Bandwidth Mgmt | Upstream / Downstream | 512 / 512 kbit/s (la guía pone "12" en downstream, errata probable) |
| SNMP | Estado | Enabled |
| SNMP | Get / Set / Trap community | public / private / public |
| Bonjour | Estado | Enabled |
| Port Mgmt | Priority / Auto Neg / VLAN / Disable | Normal / Enabled / VLAN 1 / no |
| Firewall | Firewall, SPI, DoS, Block WAN Request | Enabled |
| Firewall | Remote Management | Disabled (puerto 80 u 443 según el firmware) |
| Firewall | HTTPS | Enabled |
| Firewall | Multicast Pass Through | Disabled |
| Firewall | Java / Cookies / ActiveX / Proxy | No bloqueados |
| Access Rules | Reglas | Las 6 por defecto + 4 implícitas |
| Content Filter | Forbidden Domains / Keywords | Deshabilitados **[DUDOSO]** |
| VPN | IPSec / PPTP / L2TP Passthrough | Enabled |
| VPN | PPTP Server | Disabled; rango 192.168.1.200-204 |
| Log | Log Queue Length / Time Threshold | 50 entradas / 10 min |
| Backup | Nombres de archivo | Startup.config / Mirror.config (v1/v2: RV082.exp) |

## 14. Tabla de límites conocidos

| Elemento | Límite | Fuente / nota |
|---|---|---|
| Reglas de acceso (firewall) | 50 | Guía y datasheet |
| Port Range Forwarding | 30 | Datasheet, FAQ Linksys |
| Port Triggering | 30 | Guía, datasheet |
| Servicios personalizados (Service Management) | 30 | Guía |
| Entradas UPnP | 30 | Guía |
| One-to-One NAT | 10 (Cisco SMB544, 4.2.2.08) / 64 (Linksys v1/v2) | **[DUDOSO]** |
| Rutas estáticas | 30 | Guía |
| Hop Count de una ruta estática | 15 | Guía |
| Reglas de Bandwidth Management | 100 | Guía |
| Reservas IP estáticas (DHCP) | 100 | Guía |
| Pool DHCP | 50 oficial; posiblemente hasta 253 | **[DUDOSO]** |
| Lease DHCP | 5 a 43.200 min | Guía |
| VLANs (por puerto) | 8 (una por puerto LAN) | Guía |
| Multiple Subnet | No documentado | **[DUDOSO]** |
| Túneles IPsec | 100 (HW v3) / 50 (HW v1/v2) | Datasheet / guía Linksys |
| Usuarios QuickVPN | 50 | Datasheet |
| Clientes PPTP | 5 | Datasheet |
| URLs y clientes aprobados en ProtectLink | 20 + 20 | Guía |
| Usuario/contraseña PPPoE y PPTP | 60 caracteres | Guía |
| Comunidades SNMP | 64 caracteres | Guía |
| Contraseña admin (v1/v2) | < 64 caracteres, sin espacios | Guía Linksys |
| Rendimiento NAT / IPsec (HW v3) | 200 Mbps / 97 Mbps (prueba real: unos 93-94 Mbps WAN↔LAN, 149 Mbps simultáneo) | Datasheet / SmallNetBuilder |
| Sesiones simultáneas | "Theoretically 4,000" (v1, FAQ Linksys); no documentado en v3 | **[DUDOSO]** |

## 15. Puertos y protocolos relacionados con la gestión

| Uso | Puerto / protocolo | Nota |
|---|---|---|
| Interfaz web LAN | TCP 80 (HTTP) | Siempre permitido desde la LAN (regla implícita) |
| Interfaz web LAN cifrada | TCP 443 (HTTPS) | Si HTTPS está habilitado (por defecto) |
| Gestión remota WAN | Configurable; por defecto 80 (guía 4.x) o 443 (artículo de 4.2.3.06); en v1/v2, 80 u 8080 | **[DUDOSO, depende del firmware]** |
| SNMP | UDP 161 (get/set) y 162 (traps) | Puertos estándar; la guía no los cita |
| Syslog | UDP 514 | Puerto estándar; la guía no lo cita, y no es configurable en la interfaz **[DUDOSO]** |
| NTP | UDP 123 | Estándar |
| IPsec / NAT-T | UDP 500 / 4500, ESP | Ver documento VPN |
| PPTP | TCP 1723 + GRE | Ver documento VPN |

---

## 16. Procedimientos paso a paso

### 16.1 Cambiar la IP LAN del router (ej. a 192.168.10.1)
1. Entra en **Setup > Network**. En **LAN Setting** (pestaña IPv4) escribe **Device IP Address** = 192.168.10.1 y **Subnet Mask** = 255.255.255.0.
2. **Save** → en el aviso pulsa **OK**.
3. Renueva la IP del PC (desactiva y activa la tarjeta, o `ipconfig /release` y `ipconfig /renew`). Si el PC tenía IP fija, ponle una de 192.168.10.x.
4. Entra en `http://192.168.10.1`.
5. Revisa **DHCP > DHCP Setup**: Range Start/End en la nueva subred, y las reservas estáticas. Revisa también las IPs de destino de Forwarding, DMZ Host, One-to-One NAT, Access Rules, rutas estáticas y túneles VPN (Local Group): **no se actualizan solas**. **[DUDOSO en cuanto a la actualización automática]**
6. Cuándo hacerlo: si el módem del ISP también usa 192.168.1.x (conflicto). La guía propone 10.1.1.1 o 192.168.0.1, o poner el módem en modo bridge.

### 16.2 Configurar la WAN con PPPoE
1. Entra en **Setup > Network** → **WAN Setting** → **Edit** en WAN1.
2. **WAN Connection Type** = PPPoE. Escribe **Username** y **Password** (y Service Name si el ISP lo pide).
3. Elige **Keep Alive** (Redial Period de 30 s o 20 s) para estar siempre conectado, o **Connect on Demand** (Max Idle Time).
4. MTU: Auto, o Manual 1492 si hay problemas con algunas webs.
5. **Save**. En **System Summary** comprueba la IP WAN; usa **Connect/Disconnect** si hace falta.

### 16.3 Configurar Dual WAN con balanceo de carga (Load Balance)
1. Conecta el módem 1 al puerto **Internet** y el módem 2 al puerto **DMZ/Internet**.
2. En **Setup > Network**, **desmarca Enable DMZ** (v1/v2: Dual-WAN / DMZ Setting = Dual-WAN) y **Save**. Aparece WAN2 en WAN Setting.
3. Configura WAN1 y WAN2 (**Edit** en cada una) con su tipo de conexión. **Save**.
4. Entra en **System Management > Dual WAN** → modo **Load Balance** → **Save**.
5. Pulsa **Edit** en WAN1:
   - **Upstream/Downstream** = velocidades reales contratadas en kbit/s (ej. 10 Mbps = 10240). El peso del balanceo depende de estos valores.
   - **Network Service Detection**: Enable; Retry count 5; Retry timeout 30; **When Fail = Remove the Connection**; marca **Remote Host** con una IP fiable (y/o Default Gateway, DNS Lookup Host).
   - **Save**.
6. Repite en WAN2.
7. (Opcional) **Protocol Binding** en el Edit de la WAN deseada (solo en Load Balance). Ejemplo: Service = SIP, Source = IP del teléfono, Destination = ANY (0.0.0.0) **[DUDOSO: cómo expresar "cualquier destino" en los campos]**, Interface = WAN1, Enable → Add to list → **Save**. Úsalo para servicios que se rompen si cambian de IP pública: banca, VoIP, algunos webmails o VPN.
8. Comprueba en **System Summary** que las dos WAN tienen IP, y en **Log > System Statistics** que hay tráfico en ambas. Prueba a desconectar un módem y mira el log.
- Nota: las sesiones que necesitan la misma IP de origen (HTTPS con sesión, juegos) pueden fallar con el balanceo. Protocol Binding lo mitiga. **[práctica habitual]**

### 16.4 Configurar Dual WAN como respaldo (Smart Link Backup)
1. Haz los pasos 1 a 3 de §16.3.
2. **System Management > Dual WAN** → **Smart Link Backup** → **Primary WAN** = WAN1 → **Save**.
3. **Edit** en WAN1 → activa **Network Service Detection** con **Remove the Connection** y un Remote Host → **Save**. Sin NSD, el failover solo ocurre si se cae el enlace físico o la sesión PPP.

### 16.5 Abrir puertos para un servidor (port forwarding)
1. Dale al servidor una IP fija o una reserva DHCP (§16.10). Ejemplo: 192.168.1.10.
2. **Setup > Forwarding** → **Port Range Forwarding** → **Service**: elige el servicio (HTTP [TCP/80], FTP, etc.). Si no existe: **Service Management** → Service Name "MiApp", Protocol TCP (o UDP; para ambos crea dos servicios **[DUDOSO: puede existir la opción TCP&UDP]**), Port Range 5000-5000 → **Add to list** → **OK**.
3. **IP Address** = 192.168.1.10 → **Enable** → **Add to list** → **Save**.
4. Comprueba que el firewall no lo bloquea: según la documentación, la regla por defecto WAN→LAN es Deny, pero en la práctica las entradas de forwarding permiten el tráfico reenviado sin reglas adicionales. **[DUDOSO; si no funciona, crea una Access Rule: Allow, Service = MiApp, Source Interface = WAN1 (o ANY), Source = ANY, Destination = Single 192.168.1.10]**
5. Prueba desde fuera de tu red (con datos móviles, por ejemplo). Desde dentro de la LAN usando la IP pública puede no funcionar (NAT loopback no documentado). **[DUDOSO]**
6. Si el módem del ISP también hace NAT (doble NAT), haz el reenvío también en el módem o ponlo en modo bridge.

### 16.6 Port Triggering (ej. una aplicación que sale por 6970 y recibe por 7000-7010)
1. **Setup > Forwarding** → **Port Triggering**: Application Name, **Trigger Port Range** 6970-6970, **Incoming Port Range** 7000-7010, **Enable** → **Add to list** → **Save**.

### 16.7 Exponer un equipo completo (DMZ Host)
1. **Setup > DMZ Host** → **DMZ Private IP Address** = IP del equipo → **Save**. Para desactivarlo, pon **0**.

### 16.8 Publicar un servidor con una IP pública dedicada (One-to-One NAT)
1. **Setup > One-to-One NAT** → **Enable One-to-One NAT**.
2. Private Range Begin 192.168.1.20, Public Range Begin 200.x.x.21, Range Length 1 → **Add to list** → **Save**.
3. **Firewall > Access Rules** → **Add**: Allow, el servicio deseado, Source Interface WAN1, Source ANY, Destination Single 192.168.1.20 → **Save**.

### 16.9 Restringir el acceso
**a) Bloquear Internet a un PC concreto**
1. **Firewall > Access Rules** → **Add New Rule**: Action **Deny**, Service **All Traffic**, Log según prefieras, Source Interface **LAN**, Source IP **Single** 192.168.1.50, Destination **ANY**, Time **Always** → **Save**.
2. Para que la IP del PC no cambie, reserva su IP por MAC y activa **Block MAC address on the list with wrong IP address** (§5.1). Así el usuario no puede saltarse la regla cambiando la IP.

**b) Permitir solo navegación web en horario laboral a un rango**
1. Regla 1 (prioridad 1): Allow, HTTP [TCP/80], LAN, Range 192.168.1.100-149, ANY, Interval 08:00-18:00, de lunes a viernes.
2. Regla 2: Allow, HTTPS [TCP/443], mismas condiciones.
3. Regla 3 (prioridad más baja): Deny, All Traffic, LAN, el mismo Range, ANY, Always.
4. Ordena las prioridades: las reglas Allow por encima de la Deny. DNS y DHCP siguen funcionando gracias a las reglas implícitas.

**c) Limitar un puerto reenviado a una IP de origen concreta**
1. Regla A (prioridad 1): **Allow**, servicio (ej. RDP [TCP/3389]), Source Interface WAN1, Source **Single** = IP pública autorizada, Destination Single = servidor.
2. Regla B (prioridad 2): **Deny**, mismo servicio, WAN1, Source **ANY**, Destination = servidor.
3. Prueba desde una IP no autorizada. Hay reportes de que con forwarding o NAT 1:1 las reglas no se comportan como se espera; verifica en tu firmware. **[DUDOSO]**

**d) Bloquear sitios web**: **Firewall > Content Filter** → Enable Block Forbidden Domains (+ dominios) y/o Enable Website Blocking by Keywords (+ palabras) → horario → **Save**. Probablemente no sea efectivo con HTTPS.

**e) Bloquear dispositivos desconocidos**: **DHCP > DHCP Setup** → añade todos los equipos legítimos a la lista Static IP → marca **Block MAC address not on the list** → **Save**.

**f) Aislar invitados**: **Port Management > Port Setup** → pon en **VLAN 2** el puerto donde está el switch o AP de invitados → **Save**. No podrán comunicarse con la VLAN 1 y seguirán teniendo Internet (§16.15).

### 16.10 Reservar una IP por MAC (DHCP estático)
1. **DHCP > DHCP Setup** → **Show unknown MAC addresses** → ponle nombre al equipo y marca **Enable** → **OK** → espera a que aparezca en la lista → **Save**.
2. Alternativa manual: **Static IP Address** + **MAC Address** (sin ":" ni "-") + **Name** + **Enable** → **Add to list** → **Save**.
3. Renueva la IP en el equipo.

### 16.11 Habilitar la gestión remota de forma segura
1. **Setup > Password**: contraseña fuerte (obligatorio) y, si quieres, otro nombre de usuario.
2. **Firewall > General**: Firewall Enabled, **HTTPS Enabled**, **Remote Management Enable**, **Port** = un puerto no estándar (ej. 8443) **[recomendación]** → **Save**.
3. Accede desde fuera con `https://<IP WAN o nombre DDNS>:8443`.
4. Si la IP WAN es dinámica, configura **Setup > Dynamic DNS**.
5. Revisa periódicamente **Log > System Log** (Unauthorized Login Attempt) o activa las alertas por correo o syslog.
- Advertencia: el equipo está fuera de soporte (§17) y su firmware tiene vulnerabilidades conocidas sin parche. **Es preferible no exponer la gestión a Internet** y administrarlo por VPN. **[recomendación]**

### 16.12 Copia de seguridad y restauración
- **v3/4.x**: **System Management > Backup and Restore** → **Backup Startup Configuration** → guarda `Startup.config` (renómbralo con la fecha).
  - Restaurar: **Restore Startup Configuration File** → Browse → **Restore** → **System Management > Restart > Restart Router**.
  - Copia "en caliente" antes de cambios: **Copy Startup to Mirror**. Para deshacer: **Copy Mirror to Startup** (reinicia).
- **v1/v2**: **System Management > Setting Backup** → **Export** (`RV082.exp`). Para restaurar: **Import** → reiniciar.
- Las certificaciones de VPN se exportan aparte (**VPN > VPN Client Access** → Export for Admin) antes de un reset.

### 16.13 Reiniciar y restablecer a fábrica
| Acción | Cómo |
|---|---|
| Reinicio suave (recomendado) | **System Management > Restart > Restart Router** (envía el log antes) |
| Reinicio por botón | Pulsar **Reset** **1 segundo** (con la punta de un bolígrafo). En el RV082 está en el **panel frontal**, cerca de los puertos Internet y DMZ. |
| Fábrica desde la web | **System Management > Factory Default > Return to Factory Default Setting** → OK |
| Fábrica por botón | Mantener **Reset** pulsado **30 segundos** (guía Cisco 4.x y Linksys "up to 30 seconds"). En el FAQ Linksys de "olvidé la contraseña" aparecen **10 segundos**. **[DUDOSO: usa 30 s para asegurar]** |
| Fábrica "suave" | **Copy Mirror to Startup** si la Mirror está en valores de fábrica |
| Downgrade de firmware | También restaura los valores de fábrica |
- Tras el reset: 192.168.1.1, admin/admin, DHCP .100-.149. Si el PC tenía otra IP fija, cámbiala a 192.168.1.x.

### 16.14 Actualizar el firmware
1. Haz una copia de la configuración (§16.12).
2. Descarga el firmware correcto para tu **versión de hardware** (v3 → 4.2.x; v1/v2 → 1.x/2.x) y descomprímelo.
3. Conecta por **cable** desde la LAN (no por WAN ni VPN).
4. **System Management > Firmware Upgrade** → Browse → **Firmware Upgrade Right Now** → espera (hasta unos 10 min) sin tocar nada.
5. Vuelve a entrar y comprueba la versión en **System Summary**. Si fue un downgrade, estará en fábrica: restaura la configuración.

### 16.15 Separar dos grupos de la LAN por VLAN y priorizar uno (ejemplo Cisco SMB3060)
1. **Port Management > Port Setup**: puertos 1-7 con Priority **High** y VLAN 1; puerto 8 con Priority **Normal** y VLAN 8 → **Save**.
2. (Opcional) **Bandwidth Management** → Rate Control: limita la IP del equipo del puerto 8 (ej. Upstream 200 y Downstream 4096 kbit/s) → **Save**.
3. Verifica con **Port Management > Port Status** y con **Diagnostic > Ping**: el ping desde la VLAN 1 al equipo de la VLAN 8 debe fallar.

### 16.16 Dar prioridad a la VoIP (QoS)
1. **System Management > Bandwidth Management**: Upstream/Downstream reales de cada WAN.
2. **Service Management**: crea los servicios SIP (ej. UDP 5060-5070) y RTP (ej. UDP 10000-25000).
3. Elige **Rate Control** (o **Priority**). Añade reglas para cada servicio en Upstream y en Downstream, con Min. Rate garantizado (ej. 40-100 kbit/s por llamada) y Max. Rate → **Enable** → **Add to list** → **Save**.
4. Opcional: **Port Management** → Priority **High** en el puerto del teléfono IP.

### 16.17 Añadir una ruta estática a otra red interna
Ejemplo: red 10.0.0.0/24 accesible a través del router 192.168.1.254.
1. **Setup > Advanced Routing** → Static Routing: Destination IP 10.0.0.0, Subnet Mask 255.255.255.0, Default Gateway 192.168.1.254, Hop Count 1, Interface **LAN** → **Add to list** → **Save**.
2. Comprueba con **View** (Routing Table).
3. Si los equipos de 10.0.0.0/24 deben salir a Internet por el RV082, puede hacer falta **Multiple Subnet** (§4.1.4), porque el firewall deniega los orígenes ajenos a la subred LAN. **[DUDOSO]**

### 16.18 Usar el RV082 como router interno (no gateway)
1. **Setup > Advanced Routing** → Working Mode **Router** (y RIP si lo usa la red) → **Save**.
2. Configura rutas estáticas o RIP. La guía recomienda **deshabilitar el firewall** de este equipo (antes hay que cambiar la contraseña por defecto).

---

## 17. Estado del producto, avisos y discrepancias conocidas

- **Fin de vida:** el RV082 está descontinuado y sin soporte de Cisco. Las licencias de ProtectLink dejaron de venderse el 31/07/2013. No hay firmware nuevo; la última versión citada para HW v3 es 4.2.3.14 (la cita el NVD/CPE). **[DUDOSO: fecha exacta de EoL y versión final no confirmadas]** Existen vulnerabilidades publicadas para la serie RV0xx (el documento de seguridad o troubleshooting, si existe, debería ampliarlo). Recomendación: no exponer la gestión a Internet.
- **Interfaz antigua:** la interfaz depende de pop-ups y de navegadores antiguos (IE6+/Firefox/Safari). En navegadores modernos puede haber problemas con TLS/HTTPS antiguo (TLS 1.0, certificados débiles), y con algunos pop-ups. Si HTTPS falla, usa HTTP desde la LAN. **[DUDOSO / práctica habitual]**
- **Erratas y contradicciones detectadas en la documentación oficial:**
  1. Remote Management, puerto por defecto: 80 (guía) frente a 443 (artículo 4.2.3.06).
  2. Bandwidth Management, downstream por defecto: "12 kbit/sec" (probablemente 512).
  3. Protocol Binding "solo RV016": falso para el RV082 (lo confirman SMB622 y el datasheet).
  4. One-to-One NAT: 10 frente a 64 entradas.
  5. Pool DHCP: 50 frente a 253.
  6. Ejemplo de cambio de LAN: "192.168.15.1 → 192.168.2.x" es incoherente.
  7. Rango PPTP: "92.168.1.204" (errata de 192.168.1.204).
  8. Prefijo IPv6 LAN por defecto: "7" (podría ser 64 o 7, según el firmware). **[DUDOSO]**
  9. Rutas estáticas: la guía Linksys exige desactivar el DHCP; la guía Cisco no.
  10. NAT 1:1 y el firewall: notas contradictorias en la guía Linksys v2.
  11. Botón Reset para volver a fábrica: 30 s frente a 10 s.
  12. La guía 4.x incluye un glosario copiado de la del RV220W (menciona Wi-Fi, que el RV082 no tiene).
- **Sin Wi-Fi:** el RV082 es solo cableado (8 puertos LAN 10/100, 1 Internet y 1 DMZ/Internet, todos Fast Ethernet). No hay Gigabit; el máximo práctico por WAN es unos 94 Mbps.
- **Sin CLI documentada:** el datasheet antiguo de Linksys (RV042) menciona "Management via Web, Telnet, SNMP", pero las guías del RV082 no documentan Telnet ni SSH. **[DUDOSO]**

---

## 18. Fuentes

Documentación oficial principal:
- Cisco Small Business RV0xx Series Routers Administration Guide (78-19576-01 B0, 2011-2013), RV042/RV042G/RV082/RV016: https://www.cisco.com/c/dam/en/us/td/docs/routers/csbr/rv0xx/administration/guide/rv0xx_AG.pdf (copias: https://www3-realm.cisco.com/en/US/docs/routers/csbr/rv0xx/administration/guide/rv0xx_AG_78-19576_310513.pdf y https://www3-realm.cisco.com/en/US/docs/routers/csbr/rv0xx/administration/guide/rv0xx_AG_78-19576.pdf)
- Índice de guías de usuario RV series: https://www.cisco.com/c/en/us/support/routers/small-business-rv-series-routers/products-user-guide-list.html
- Data Sheet Cisco RV082 Dual WAN VPN Router (C78-501227-05, 07/13): https://www.cisco.com/c/dam/en/us/products/se/loc/data_sheet_c78-501227.pdf
- Linksys RV082 10/100 8-Port VPN Router User Guide (v1, firmware 1.x), copia en ManualsDir: https://www.manualsdir.com/manuals/168417/linksys-rv082.html
- Linksys RV082 User Guide (revisión v2, con Heart Beat Signal, Protocol Binding y VLAN), copia en ManualsDir: https://www.manualsdir.com/manuals/166481/linksys-rv082.html
- Datasheet Linksys RV042 (referencia de la familia): https://images10.newegg.com/UploadFilesForNewegg/itemintelligence/Linksys/33-124-160/RV042.pdf

Artículos de soporte de Cisco (RV016/RV042/RV042G/RV082):
- SMB4887, Edit WAN Connection (4.2.2.08): https://www.cisco.com/c/en/us/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb4887-edit-wan-connection-on-rv016-rv042-rv042g-and-rv082-vpn-rout.html
- SMB622, Set Up Dual WAN Connections (4.2.1.02): https://www.cisco.com/c/en/us/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb622-set-up-dual-wan-connections-on-rv042-rv042g-and-rv082-vpn-ro.html
- SMB589, Bandwidth Management (4.2.1.02): https://www.cisco.com/c/en/us/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb589-bandwidth-management-on-rv016-rv042-rv042g-and-rv082-vpn-rou.html
- SMB5074, Configure General Firewall Settings (4.2.3.06): https://www.cisco.com/c/en/us/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb5074-configure-general-firewall-settings-on-the-rv016-rv042-rv042.html
- SMB528, Configuration of an IPv4 Access Rule (4.2.1.02): https://www.cisco.com/c/en/us/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb528-configuration-of-an-ipv4-access-rule-on-rv016-rv042-rv042g-a.html
- SMB573, Backup and Restore Settings (4.2.2.08): https://cisco.com/c/en/us/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb573-backup-and-restore-settings-on-rv016-rv042-rv042g-and-rv082.html
- SMB544, One-to-One NAT (4.2.2.08): https://www.cisco.com/c/en/us/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb544-one-to-one-network-address-translation-nat-on-rv016-rv042-rv.html
- SMB541, Firmware Upgrade: https://www.cisco.com/c/en/us/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb541-firmware-upgrade-on-rv016-rv042-rv042g-and-rv082-vpn-routers.html
- SMB575, Configure Port Settings (4.2.2.08): https://www.cisco.com/c/en/us/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb575-configure-port-settings-on-rv016-rv042-rv042g-and-rv082-vpn.html
- SMB3060, Separate two LAN networks with few public IPs (VLAN + 1:1 NAT + QoS): https://www.cisco.com/c/en/us/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb3060-separate-two-lan-networks-with-few-public-ips-on-rv042-rv042.html
- SMB346, Port Range Forward Configuration (4.2.1.02): https://www.cisco.com/c/en/us/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb346-port-range-forward-configuration-on-the-rv016-rv042-rv042g-a.html
- SMB4250, IPv4 DHCP Server Configuration (4.2.2.08; versión japonesa consultada): https://www.cisco.com/c/ja_jp/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb4250-ipv4-dhcp-server-configuration-on-rv016-rv042-rv042g-and-rv0.html (en inglés: https://www.cisco.com/c/en/us/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb4250-ipv4-dhcp-server-configuration-on-rv016-rv042-rv042g-and-rv0.html)
- SMB4256, IPv4 DHCP Relay: https://cisco.com/c/nl_nl/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb4256-ipv4-dhcp-relay-configuration-on-rv016-rv042-rv042g-and-rv08.pdf
- SMB298, Port Triggering (referenciado): https://cisco.com/c/en/us/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb298-configuration-of-port-triggering-on-the-rv016-rv042-rv042g-a.epub

Comunidad, reseñas y otros:
- Release notes RV082 v2.0.0.19-tm (HW v1/v2): https://www3-realm.cisco.com/en/US/docs/routers/csbr/rv082/release/notes/RV082_RN_v2-0-0-19-tm.pdf
- Comunidad Cisco, firmware HW v3 (4.x solo para v3): https://community.cisco.com/t5/small-business-support-knowledge-base/rv016-rv042-rv082-hardware-v3-firmware-update/tac-p/3125489/highlight/true y https://community.cisco.com/t5/small-business-support-knowledge-base/new-rv0xx-version-3-hardware-firmware-v-4-0-3-03-tm/ta-p/3124721
- Comunidad Cisco, firmware RV082 v1/v2: https://community.cisco.com/t5/routers-small-business/router-linksys-rv082-firmware-update-latest-or-not/m-p/1763360/highlight/true
- Comunidad Cisco, límite de clientes DHCP del RV042: https://community.cisco.com/t5/routers-small-business/rv042-maximum-number-of-dhcp-clients/td-p/1997101
- Comunidad Cisco, WAN2 del RV082 en Transparent Bridge: https://community.cisco.com/t5/routers-small-business/2nd-wan-port-on-rv082-as-bridge/td-p/2552455
- Comunidad Cisco, NAT 1:1 y Access Rules en el RV082 (4.0.0.07): https://community.cisco.com/t5/routers-small-business/rv082-v4-0-0-07-one-to-one-nat-and-access-rules-problem/m-p/1684344
- Comunidad Cisco, Protocol Binding y failback: https://community.cisco.com/t5/routers-small-business/rv042-load-balance-protocol-binding-not-failing-back/m-p/2163484/highlight/true
- Comunidad Cisco, restauración de startup.config: https://supportforums.cisco.com/discussion/13077386/rv082-startupconfig-file-restore-issues
- SmallNetBuilder, "Cisco RV082 and RV016 v3 VPN Routers Reviewed" (hardware v3, rendimiento, límites): https://www.smallnetbuilder.com/lanwan/lanwan-reviews/cisco-rv082-and-rv016-v3-vpn-routers-reviewed/
- CPE/NVD, versiones de firmware 4.2.3.10 y 4.2.3.14: https://cpe.gcve.eu/cpes/652002 y https://cpe.gcve.eu/cpes/573968
