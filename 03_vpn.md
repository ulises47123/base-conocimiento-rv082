# 03 — VPN en el Cisco/Linksys RV082 (10/100 8-Port VPN Router, Dual WAN)

> Documento de referencia autosuficiente para consultas **sin acceso a internet**.
> Idioma: español; los nombres de menús, campos y opciones se dejan **en inglés tal como aparecen en la interfaz web**.
>
> **Leyenda de fiabilidad** (se usa en todo el documento):
> - **[AG]** = *Cisco Small Business RV0xx Series Routers Administration Guide* (RV042/RV042G/RV082/RV016), doc. 78-19576-01 B0, © 2011-2012. Fuente oficial principal; texto leído directamente del PDF.
> - **[DS]** = *Data Sheet* oficial Cisco RV082 (C78-501227-05, 07/2013).
> - **[KB]** = artículos de soporte Cisco (cisco.com/c/en/us/support/docs/smb/...), casi todos escritos para firmware **v4.2.2.08** (publicados/revisados dic-2018).
> - **[RN]** = Release Notes oficiales de Cisco QuickVPN.
> - **[3P]** = guía de un tercero (TheGreenBow, Shrew Soft, SonicWall, SmallNetBuilder).
> - **[Com]** = foros (Cisco Community, Netgate, etc.). Anecdótico, no oficial.
> - **[Inf]** = inferencia/criterio técnico general propio, no documentado específicamente para el RV082.
> - **[Dudoso]** = dato no confirmado o fuentes contradictorias.

---

## 0. Resumen ejecutivo (lo esencial en 25 líneas)

| Capacidad | Valor en RV082 | Fuente |
|---|---|---|
| Túneles IPsec (Gateway-to-Gateway + Client-to-Gateway) | **100** ("The router supports up to 100 tunnels") | [AG] [DS] |
| Group VPN (Client-to-Gateway tipo grupo) | **máx. 2 grupos** | [AG] |
| Usuarios/túneles Cisco QuickVPN | **50** (si el router sólo admite 10, actualizar firmware) | [AG] [DS] |
| Servidor PPTP | **5 clientes PPTP** simultáneos (RV016: 10) | [AG] [DS] |
| Rango IP por defecto PPTP | 192.168.1.200 – 192.168.1.204 (el AG trae la errata "92.168.1.204") | [AG] |
| Servidor L2TP / L2TP-over-IPsec | **NO soportado** (sólo *passthrough*) | [AG] [3P SmallNetBuilder] |
| SSL VPN / AnyConnect / Easy VPN / IKEv2 / XAUTH / Mode-Config | **NO** (sólo IKEv1 con PSK; sin ModeCfg) | [AG] [3P Shrew] |
| Cifrado | DES, 3DES, AES-128, AES-192, AES-256 (y NULL en Fase 2) | [AG] |
| Hash/Autenticación | MD5, SHA1 (y NULL en Fase 2) — **no hay SHA-2** | [AG] |
| Grupos DH | **1 (768), 2 (1024), 5 (1536)** únicamente — no hay 14/19/20 | [AG] |
| Autenticación IKE | **sólo Pre-Shared Key** (o Manual keying). No hay certificados X.509 para IPsec | [AG] |
| Lifetimes por defecto | Fase 1 **28800 s**, Fase 2 **3600 s** | [AG] |
| Opciones avanzadas | Aggressive Mode, IPComp, Keep-Alive, AH, NetBIOS Broadcast, NAT-T, DPD, Tunnel Backup, Split DNS | [AG] |
| VPN passthrough | IPSec, PPTP, L2TP — los tres **habilitados por defecto** | [AG] |
| Throughput IPsec | **97 Mbps** (dato de catálogo) / ~70 Mbps 3DES medido por SmallNetBuilder (HW v3, fw 4.0.3.03) | [DS] [3P] |
| Throughput NAT | 200 Mbps | [DS] |
| Multicast por el túnel IPsec | **NO soportado** | [AG] |

**Consecuencia práctica clave:** el RV082 es un equipo IKEv1/PSK con algoritmos antiguos (máximo AES-256 + SHA1 + DH5). Para interoperar con equipos modernos, la combinación "más fuerte común" suele ser **AES-256 / SHA1 / DH Group 5 / PFS grupo 5 / Main Mode / 28800-3600 s**, y el otro extremo debe todavía aceptar SHA1 y DH 5 (o DH 2). [Inf]

---

## 1. Contexto de hardware y firmware

### 1.1 Versiones de hardware
- **RV082 v1 / v2** (era Linksys, CPU Intel IXP425-266): firmware rama **1.x / 2.0.x**; el último para v1/v2 es **2.0.2.01-tm**. Cisco no publica firmware para v1/v2. [Com]
- **RV082 v3** (Cisco, CPU Cavium CN5020 300 MHz, 32 MB flash, 256 MB RAM): firmware rama **4.x** (4.0.x, 4.1.x, 4.2.x). El firmware 4.x **no** es compatible con hardware v1/v2 (la etiqueta inferior del equipo indica "v3"). [Com] [3P SmallNetBuilder]
- Versiones 4.2.x vistas: 4.2.1.02, 4.2.2.08 (base de casi todos los KB Cisco), 4.2.3.03 (2014), 4.2.3.10, **4.2.3.14** (la más alta que aparece en bases de CPE; no he podido leer sus release notes → **[Dudoso]** si es la última).
- El producto está fuera de soporte (EoL). [Inf]

### 1.2 Diferencias relevantes v1/v2 vs v3 [Com] [3P]
- Guías antiguas (p. ej. Shrew Soft probado con firmware 1.3.98-tm) dicen "Split DNS is not supported"; en firmware 4.x **sí** existe Split DNS en Gateway-to-Gateway. [AG] [3P]
- Muchos menús son iguales; el AG (2011-2012) describe la UI de la rama 4.x.
- QuickVPN: con firmware antiguo el router admitía sólo 10 clientes QuickVPN; el AG dice "If the router you have only supports up to ten clients, then upgrade its firmware" (a 50). [AG]

---

## 2. Mapa de menús VPN

| Menú (UI) | Función |
|---|---|
| **VPN > Summary** | Estado de túneles, GroupVPN, clientes VPN conectados; botón **Add** para crear túnel |
| **VPN > Gateway to Gateway** | Túnel site-to-site IPsec |
| **VPN > Client to Gateway** | Túnel IPsec para clientes software de terceros (Tunnel = un usuario; Group VPN = grupo) |
| **VPN > VPN Client Access** | Usuarios **Cisco QuickVPN** + **Certificate Management** |
| **VPN > VPN Passthrough** | IPSec / PPTP / L2TP Passthrough para clientes del LAN |
| **VPN > PPTP Server** | Servidor PPTP (5 usuarios) |
| **System Summary > VPN Setting Status** | Tunnel(s) Used / Tunnel(s) Available |
| **Firewall > General > HTTPS** | Debe estar **habilitado** para que QuickVPN funcione ("If you disable the HTTPS feature, then users cannot connect by using QuickVPN") [AG] |
| **System Management > Dual WAN** | Smart Link Backup / Load Balance (relevante para Tunnel Backup) |
| **Log > System Log** | Syslog, e-mail alert, **View System Log** (incluye "VPN Log") |

> En **Firewall > General**: "**IMPORTANT: This router does not support passing multicast traffic over an IPSec tunnel.**" (Multicast Pass Through sólo afecta tráfico Internet→LAN). [AG]

---

## 3. VPN > Summary [AG]

"The VPN > Summary page displays general information about the router's VPN tunnel settings. **The router supports up to 100 tunnels.**" Si el servidor PPTP está habilitado, el resumen de clientes PPTP aparece en **VPN > PPTP Server**, no aquí.

### 3.1 Sección *Summary*
- **Tunnel(s) Used** — túneles en uso.
- **Tunnels Available** — túneles disponibles.
- **Detail** — ventana con, por túnel: No., Name, Status, Phase 2 Enc/Auth/Grp, Local Group, Remote Group, Remote Gateway (botones Refresh / Close).

### 3.2 Sección *Tunnel Status*
- Encabezado: **Tunnel(s) Enabled** (habilitados) y **Tunnel(s) Defined** (definidos, habilitados o no).
- Columnas:
  - **No.** — ID del túnel.
  - **Name** — nombre descriptivo.
  - **Status** — **Connected** o **Waiting for Connection**.
  - **Phase2 Enc/Auth/Grp** — cifrado Fase 2 (NULL/DES/3DES/AES-128/AES-192/AES-256), autenticación (NULL/MD5/SHA1) y grupo DH (1/2/5). Con *Manual keying* sólo se ven cifrado y autenticación.
  - **Local Group** / **Remote Group** — IP y máscara.
  - **Remote Gateway** — IP del par remoto (si se usó *IP by DNS Resolved*, aquí aparece la IP resuelta).
  - **Tunnel Test** — botón **Connect** (fuerza negociación y actualiza Status); si conectado aparece **Disconnect**.
  - **Configure** — icono **Edit** / **Delete**.
- **Add** → elegir **Gateway to Gateway** o **Client to Gateway**.
- Controles de paginación (Rows per page, Page).

### 3.3 Sección *GroupVPN Status* (si hay Group VPN)
Group Name, Connected Tunnels (usuarios conectados al grupo), Phase2 Enc/Auth/Grp, Local Group, Remote Client, Remote Clients Status (Online/Offline), **Detail List** (Group Name, IP, Connection Time), Tunnel Test, Configure.

### 3.4 Sección *VPN Clients Status* (QuickVPN)
No., Username, Status, Start Time, End Time, Duration, botón **Disconnect** por cliente.

---

## 4. IPsec Gateway-to-Gateway (site-to-site) — campo por campo

Ruta: **VPN > Gateway to Gateway** (o **VPN > Summary > Add > Gateway to Gateway**). Pulsar **Save** antes de salir de la página; lo no guardado se pierde. [AG]

**Reglas generales [AG]:**
- Se configuran **ambos** routers, invirtiendo "local" y "remote".
- "A successful connection requires that **at least one router is identifiable by a static IP address or a Dynamic DNS hostname**. Alternatively, if one router has only a dynamic IP address, you can use any email address as authentication."
- "**The two ends of the tunnel cannot be on the same subnet.**" (p. ej. Sitio A 192.168.1.x, Sitio B 192.168.2.x).
- Todos los parámetros IPsec (y los Advanced) deben ser **idénticos** en ambos extremos.

### 4.1 *Add a New Tunnel*
| Campo | Descripción | Notas |
|---|---|---|
| **Tunnel No** | ID generado automáticamente | Sólo lectura |
| **Tunnel Name** | Nombre de referencia (p. ej. "Chicago Branch") | **No** tiene que coincidir con el del otro extremo |
| **Interface** | Puerto WAN usado por el túnel: **WAN1** o **WAN2** | WAN2 sólo aparece si el puerto DMZ/Internet está configurado como WAN (no como DMZ) [KB] |
| **Enable** | Habilita/deshabilita el túnel; habilitado por defecto | Appendix D: "The Enable check box is unavailable until after you save the configuration" (comportamiento según versión) |

### 4.2 *Local Group Setup* y *Remote Group Setup*

#### Local/Remote Security Gateway Type (cómo se identifica cada gateway en IKE)
| Opción (UI) | Cuándo usarla | Campos | Tipo de ID IKE equivalente [Inf] |
|---|---|---|---|
| **IP Only** | IP WAN estática. La IP WAN local aparece automáticamente | Remoto: elegir **IP Address** (IP fija) o **IP by DNS Resolved** (nombre DNS real del router remoto; la IP resuelta se muestra en VPN > Summary) | ID_IPV4_ADDR = IP WAN |
| **IP + Domain Name (FQDN) Authentication** | IP estática + dominio registrado (p. ej. MyServer.MyDomain.com) | Domain Name. "The domain name can be used only for one tunnel connection." Remoto: también IP Address / IP by DNS Resolved | ID_FQDN |
| **IP + E-mail Addr.(USER FQDN) Authentication** | IP estática, autenticación con un e-mail cualquiera | Email Address (no necesita existir) | ID_USER_FQDN |
| **Dynamic IP + Domain Name (FQDN) Authentication** | IP dinámica con hostname DDNS registrado (DynDNS, etc.) | Domain Name (sólo para un túnel) | ID_FQDN |
| **Dynamic IP + E-mail Addr.(USER FQDN) Authentication** | IP dinámica **sin** DDNS | Email Address cualquiera | ID_USER_FQDN |

- **Advertencia oficial [AG]:** "If both routers have dynamic IP addresses (as with PPPoE connections), **do not choose Dynamic IP + Email Addr. for both gateways**. For the remote gateway, choose IP Address and IP Address by DNS Resolved."
- Si el **Remote** Security Gateway Type es de tipo **Dynamic IP**, **Aggressive Mode es obligatorio** (casilla marcada y bloqueada). [AG]
- Con tipos FQDN/USER FQDN, los IDs se cruzan: el ID *local* de un lado debe ser el ID *remoto* del otro (p. ej. ejemplo SonicWall: Local ID yahoo.com ↔ Peer ID yahoo.com). [3P SonicWall]

#### Local/Remote Security Group Type (qué redes viajan por el túnel = selectores de Fase 2 / proxy-IDs)
| Opción | Descripción |
|---|---|
| **IP Address** (a veces "IP") | Un solo host; introducir su IP |
| **Subnet** (**por defecto**) | Red + máscara (por defecto 255.255.255.0). Ej.: 192.168.1.0 / 255.255.255.0 |
| **IP Range** | **Begin IP** – **End IP** |

- Por túnel sólo hay **un** grupo local y **un** grupo remoto. Para varias subredes remotas no contiguas se suelen crear varios túneles (resultados mixtos al hacerlo hacia el mismo gateway remoto; un hilo 2009 reporta "It won't allow two tunnels to the same destination"). Alternativa: una máscara que englobe todas (p. ej. /16) si el otro extremo acepta el mismo selector. [Com] [Dudoso]
- "Full tunnel" (todo el tráfico de la sucursal por el túnel): Remote Group **Subnet 0.0.0.0 / 0.0.0.0** en la sucursal (y Local Group 0.0.0.0/0 en la central). Reportes de pérdida de Internet en la sucursal o de acceso al propio router mientras el túnel está arriba; no es una función documentada en el AG. [Com] [Dudoso]

### 4.3 *IPSec Setup*

**Keying Mode:**
- **IKE with Preshared Key** — recomendado y **por defecto**.
- **Manual** — sin negociación, claves fijas; "used in small static environments or for troubleshooting purposes".

#### 4.3.1 Campos en modo **Manual** [AG]
| Campo | Valores |
|---|---|
| **Incoming SPI** / **Outgoing SPI** | Hex **100 ~ ffffffff**. Únicos por túnel ("No two tunnels share the same SPI"). Incoming local = Outgoing remoto y viceversa |
| **Encryption** | **DES** (56 bit) o **3DES** (168 bit, recomendado). No hay AES en Manual |
| **Authentication** | **MD5** (128 bit) o **SHA1** (160 bit, recomendado) |
| **Encryption Key** | DES: **16** dígitos hex; 3DES: **40** dígitos hex (si faltan, se rellenan con ceros) |
| **Authentication Key** | MD5: **32** dígitos hex; SHA1: **40** dígitos hex (relleno con ceros) |

#### 4.3.2 Campos en modo **IKE with Preshared Key** [AG]
| Campo | Opciones | Por defecto / recomendación |
|---|---|---|
| **Phase 1 DH Group** | Group 1 - 768 bit, Group 2 - 1024 bit, Group 5 - 1536 bit | **Group 1 por defecto**; Group 5 = más seguro |
| **Phase 1 Encryption** | DES, 3DES, AES-128, AES-192, AES-256 | AG recomienda AES-256. Valor de fábrica aparentemente **DES** (ejemplo "default settings" del Appendix D) [Inf] |
| **Phase 1 Authentication** | MD5, SHA1 | SHA1 recomendado. Fábrica aparentemente **MD5** [Inf] |
| **Phase 1 SA Life Time** | segundos | **28800** |
| **Perfect Forward Secrecy** | casilla | Recomendado. En el ejemplo de fábrica aparece **Enabled** |
| **Phase 2 DH Group** | Group 1 / 2 / 5 | Es el grupo PFS. Group 1 por defecto |
| **Phase 2 Encryption** | (NULL), DES, 3DES, AES-128, AES-192, AES-256 | NULL aparece en VPN Summary y en el KB [KB] |
| **Phase 2 Authentication** | MD5, SHA1, (Null) | SHA1 recomendado |
| **Phase 2 SA Life Time** | segundos | **3600** |
| **Preshared Key** | hasta **30** caracteres de teclado o hex (ej. `My_@123` o `4d795f40313233`) | Igual en ambos extremos. El ejemplo oficial usa una clave con `#` (el `#` es válido) |
| **Minimum Preshared Key Complexity** | casilla **Enable** | Activa el **Preshared Key Strength Meter** (rojo débil → amarillo aceptable → verde fuerte). Tip: >8 caracteres, mayúsculas, minúsculas, números, símbolos `-*^+=` |

> **Rango admitido de los SA Life Time**: no documentado en el AG. Hay configuraciones reales de RV082 con 86400 s en Fase 1 (hilo RV082↔ASA) y un comentario de foro de que la versión Linksys "tops out at 28800". **[Dudoso]** — Si el par usa un valor distinto, la mayoría de equipos negocian el menor, pero algunos (o con PFS/IDs estrictos) fallan; lo más seguro es **igualar exactamente** 28800/3600. [Com]

> **No hay** opción de lifetime por volumen (kilobytes). Con ASA (por defecto 4 608 000 KB en Fase 2) se recomienda revisar/poner `lifetime kilobytes unlimited` en el ASA (ASA ≥ 9.1(2)) si aparecen cortes en el rekey. Que el RV0xx no soporte rekey por volumen es **[Inf/Dudoso]** (no confirmado por Cisco).

### 4.4 *Advanced* (sólo con IKE with Preshared Key; botón **Advanced +** / **Advanced -**) [AG] [KB smb568]
"If you change the Advanced settings on one router, be sure to enter the same settings on the other router."

| Opción | Qué hace | Detalles |
|---|---|---|
| **Aggressive Mode** | Usa Aggressive Mode en Fase 1 (si se desmarca → **Main Mode**) | "If network security is preferred, Main Mode is recommended. If network speed is preferred, Aggressive Mode." Ajustable sólo si el Remote Security Gateway Type es IP Only o IP+; con tipos **Dynamic IP es obligatorio y queda bloqueado**. En Aggressive los IDs viajan en claro |
| **Compress (Support IP Payload Compression Protocol (IP Comp))** | Propone IPComp al iniciar | Si el respondedor lo rechaza no se usa. **Como respondedor siempre acepta compresión aunque no esté marcada**. No aporta seguridad |
| **Keep-Alive** | "enables the router to attempt to automatically re-establish the VPN connection if it is dropped" | Útil para que el túnel se levante solo. Recomendación de foros/SonicWall: activarlo sólo en **un** extremo (típicamente el de IP dinámica) si hay problemas de reconexión [Com] [3P] |
| **AH Hash Algorithm** | Añade AH (protege también la cabecera IP); elegir **MD5** o **SHA1** | Igual en ambos lados. **AH es incompatible con NAT** (no usar si hay NAT en el camino) [Inf] |
| **NetBIOS Broadcast** | Re-difunde broadcasts NetBIOS de un extremo al otro (Network Neighborhood) | Desmarcado por defecto |
| **NAT Traversal** | Activar si el router VPN está **detrás de un NAT** | "The same setting must be used on both ends of the tunnel." Implementa RFC 3947 (UDP 4500) [AG App. E] |
| **Dead Peer Detection (DPD)** + **Interval** | Envía HELLO/ACK periódicos para comprobar el par | Sólo funciona si ambos extremos lo tienen. **Valor por defecto del intervalo no documentado**: foros mencionan 10 s o 30 s **[Dudoso]**. Necesario para Tunnel Backup |
| **Tunnel Backup** | Cuando DPD declara muerto al par, reconstruye el túnel por **otra IP remota** y/o **otra WAN local** | Sólo disponible si DPD está marcado |
| ↳ **Remote Backup IP Address** | IP alternativa del par remoto (o la misma IP WAN ya configurada) | |
| ↳ **Local Interface** | WAN local a usar para el respaldo (elegir la WAN distinta a la del túnel principal) | |
| ↳ **VPN Tunnel Backup Idle Time** | "used when the router boots up. If the primary tunnel is not connected within the specified period, then the backup tunnel is used." | **Por defecto 30 s** [AG] |
| **Split DNS** | Envía consultas DNS de dominios concretos a otro servidor DNS (p. ej. el DNS de la oficina remota) | **DNS1** (obligatorio), **DNS2** (opcional), **Domain Name 1 – Domain Name 4**. Lo que no coincide va al DNS de la WAN |

> En **Client to Gateway** las Advanced son: Aggressive Mode (sólo Tunnel, no Group), Compress, Keep-Alive, AH Hash Algorithm, NetBIOS Broadcast, DPD (sólo Tunnel, no Group), NAT Traversal. **No** hay Tunnel Backup ni Split DNS en Client to Gateway. [AG]

### 4.5 Failover del túnel con Dual WAN (Tunnel Backup + Smart Link Backup) [KB smb3009] [AG]
Aplica a RV042, RV042G, RV082.
1. **VPN > Gateway To Gateway** → editar túnel → **Advanced**.
2. Marcar **Dead Peer Detection Interval** e introducir el intervalo (s).
3. Marcar **Tunnel Backup**.
4. **Remote Backup IP Address**: IP de respaldo del router remoto (p. ej. su WAN2).
5. **Local Interface**: WAN distinta a la del túnel principal.
6. **VPN Tunnel Backup Idle Time**: segundos (defecto 30).
7. **Save**.
8. **System Management > Dual WAN** → modo **Smart Link Backup** → **Primary WAN** = WAN del túnel principal → **Save**.

Notas [Inf]:
- El otro extremo debe aceptar el túnel desde la IP de respaldo: si es otro RV0xx, configurar allí también DPD/Tunnel Backup apuntando a nuestra segunda WAN, o usar identidades FQDN/USER FQDN (no ligadas a IP) y *IP by DNS Resolved*/DDNS.
- En modo **Load Balance**, cada túnel sale por la WAN elegida en **Interface**. El AG indica que **Protocol Binding** es "for Cisco RV016 only, when Load Balancer is selected", mientras que el Data Sheet del RV082 lista "Protocols can be bound to a specific WAN port for load balancing" → **[Dudoso]** si el RV082 lo muestra (depende de firmware).
- **Network Service Detection** (Dual WAN): Retry count def. 5, Retry timeout def. 30 s; **When Fail** = *Generate the Error Condition in the System Log* (no hay failover) o *Remove the Connection* (sí hay failover). Para que el failover WAN funcione hay que elegir *Remove the Connection*. [AG]

### 4.6 Procedimiento oficial RV0xx ↔ RV0xx (Appendix D del AG)
1. Conectarse al router del **Sitio A** → **VPN > Gateway to Gateway**.
2. **Tunnel Name**; **Interface** WAN1 o WAN2.
3. **Local Group Setup**: Local Security Gateway Type = **IP Only** (la IP WAN se autodetecta); Local Security Group Type = **Subnet** (red LAN + máscara).
4. **Remote Group Setup**: si B tiene IP fija → **IP Only** + IP WAN de B; si B tiene IP dinámica + DDNS → **Dynamic IP + Domain Name (FQDN) Authentication** + dominio (p. ej. MyBusiness.DynDNS.org). Remote Security Group Type = **Subnet** (LAN de B).
5. **IPSec Setup**: dejar valores por defecto (o los elegidos), idénticos en B.
6. **Preshared Key** (p. ej. 13572468).
7. **Advanced** si se necesita (DPD, NAT-T, Split DNS, NetBIOS) → **Save**.
8. Repetir en **Sitio B** con local/remoto invertidos.
9. **VPN > Summary** → verificar estado (o **Connect**).
10. Ping de PC en A a PC en B y viceversa.

**Ejemplo oficial (IPs estáticas)** [AG]:

| Campo | Router Sitio A | Router Sitio B |
|---|---|---|
| Local Security Gateway Type | IP Only (203.165.200.226 autodetectada) | IP Only |
| Local Security Group | Subnet 192.168.1.0 / 255.255.255.0 | Subnet 192.168.2.0 / 255.255.255.0 |
| Remote Security Gateway Type | IP Only — 209.165.200.238 | IP Only — 203.165.200.226 |
| Remote Security Group | Subnet 192.168.2.0 / 255.255.255.0 | Subnet 192.168.1.0 / 255.255.255.0 |
| Keying Mode | IKE with Preshared Key | igual |
| Phase 1 Encryption / Auth | DES / MD5 | igual |
| Phase 1 SA Life Time | 28800 | igual |
| Perfect Forward Secrecy | Enabled | igual |
| Phase 2 DH Group | Group 1 - 768 bit | igual |
| Phase 2 Encryption / Auth | DES / MD5 | igual |
| Phase 2 SA Life Time | 3600 | igual |
| Preshared Key | `<PSK>` (la guía oficial usa una clave de ejemplo; usá una propia y larga) | igual |
| Advanced | Default settings | igual |

**Ejemplo con IP dinámica en B**: en A, Remote Security Gateway Type = *Dynamic IP + Domain Name (FQDN) Authentication*, Domain Name `cisco.com` (ejemplo); en B, Local Security Gateway Type = *Dynamic IP + Domain Name (FQDN) Authentication*, Domain Name `cisco.com`, Remote = IP Only 203.165.200.226. (Aggressive Mode queda forzado.) [AG]

**Recomendación moderna [Inf]:** sustituir DES/MD5/Group 1 por **AES-256 / SHA1 / Group 5** en ambas fases, PFS activado, DPD activado (mismo intervalo en ambos), Keep-Alive en uno o ambos, NAT-T activado si algún extremo está detrás de NAT.

### 4.7 Topologías (Appendix D) [AG]
- **Hub-and-spoke**: un túnel por spoke en el hub; el tráfico spoke↔spoke pasa por el hub (posible cuello de botella). Para que spoke1 llegue a spoke2 vía hub, los selectores deben incluir esas redes (p. ej. Remote Group del spoke = rango/máscara amplia que incluya las otras sedes y, en el hub, túneles que acepten ese selector). Resultados mixtos en foros. [Com] [Dudoso]
- **Mesh**: túneles entre todos los nodos (más configuración, sin cuello de botella; escalabilidad limitada por CPU).
- Diseño WAN: al menos un sitio con IP fija o DDNS (proveedores citados: dyn.com/dyndns, update.ods.org, dhs.org, 3322.org, no-ip.com). LAN: cada sitio en subred distinta.

### 4.8 IPsec NAT Traversal (Appendix E) [AG]
Escenario oficial: Router A (RV016) detrás de NAT1 (RV042), Router B (RV016) detrás de NAT2 (RV042), WAN privadas 192.168.99.x.
- "Both the IPSec initiator and responder must support the mechanism for detecting the NAT router in the path and changing to a new port, as defined in **RFC 3947**."
- Router A: Local = IP Only (autodetecta su WAN privada), Local Group Subnet; Remote = **IP Only con la IP WAN del router NAT** delante de B; Remote Group Subnet; PSK; **Advanced → NAT Traversal** marcado; Save.
- Router B: igual, con Remote = **IP WAN del router NAT2** y NAT Traversal marcado.
- [Inf] En el NAT delante del **respondedor** hay que reenviar **UDP 500 y UDP 4500** (y, sin NAT-T, protocolo **ESP 50**) a la WAN del RV082, o poner el RV082 en DMZ/bridge del módem. Ver §10.4.

---

## 5. Client to Gateway (VPN de acceso remoto con clientes IPsec de terceros)

Ruta: **VPN > Client to Gateway** (o Summary > Add > Client to Gateway). Pensado para clientes como **TheGreenBow**, VPN Tracker, Shrew Soft, etc. [AG] [KB smb624]

### 5.1 Tipos
- **Tunnel** — un único usuario remoto (cuenta como un túnel de los 100).
- **Group VPN** — varios usuarios con la **misma Preshared Key** "up to the maximum number of supported tunnels". "**The router supports up to two VPN groups.**" Número de grupo automático (Group No). [AG]

### 5.2 Campos
- **Tunnel Name**, **Interface** (WAN), **Enable**.
- **Local Group Setup**:
  - **Local Security Gateway Type** (mismas 5 opciones que G2G; el KB dice que en Group VPN se omite).
  - **Local Security Group Type**: IP Address / **Subnet** (defecto) / IP Range (el KB cita valores por defecto IP 192.168.1.0, máscara 255.255.255.0, rango 192.168.1.0–192.168.1.254).
  - **Domain Name** / **Email** según el tipo elegido.
- **Remote Client Setup for Single User ("Tunnel")**: IP Only (IP Address o IP by DNS Resolved), IP + Domain Name (FQDN), IP + Email Address (USER FQDN), Dynamic IP + Domain Name (FQDN), Dynamic IP + E-mail Addr. (USER FQDN). Para clientes móviles lo normal es **Dynamic IP + E-mail Addr. (USER FQDN)** o **Dynamic IP + Domain Name** (→ Aggressive forzado).
- **Remote Client Setup for a Group ("Group VPN")**: **Domain Name (FQDN) Authentication**, **Email Address (USER FQDN) Authentication**, **Microsoft XP/2000 VPN Client** (cliente IPsec integrado de Windows XP/2000).
- **IPSec Setup**: idéntico a G2G (Manual sólo para Tunnel; **Manual está deshabilitado en Group VPN** [KB]). Mismos DH 1/2/5, cifrados, hash, 28800/3600, PFS, PSK ≤30, Minimum Preshared Key Complexity.
- **Advanced**: Aggressive Mode (sólo Tunnel; en Group VPN el Aggressive Mode es automático/no editable según Shrew [3P]), Compress, Keep-Alive, AH Hash Algorithm, NetBIOS Broadcast, DPD (sólo Tunnel), NAT Traversal.

### 5.3 Limitaciones importantes
- **No hay XAUTH, ni Mode Config (IKE Config push/pull), ni asignación de IP virtual, ni DHCP over IPsec**: el cliente debe configurarse manualmente con una IP virtual propia que **no** pertenezca ni a su LAN ni a la LAN del RV082. [3P Shrew] [3P TheGreenBow]
- Por lo anterior **no funcionan** los clientes "Cisco IPsec" de iOS/macOS/Android (requieren XAUTH + ModeCfg), ni Cisco VPN Client clásico, ni AnyConnect. [Inf] [Com]
- No hay servidor L2TP/IPsec → el cliente "L2TP/IPsec PSK" de Windows/iOS/Android **no** sirve. [AG] [3P SmallNetBuilder]
- La opción "Microsoft XP/2000 VPN Client" se refiere al **IPsec Security Manager** de Windows XP/2000 (políticas IPsec configuradas con MMC/ipseccmd), no al asistente "VPN" de Windows. No hay guía oficial paso a paso; poco probado en Windows modernos. **[Dudoso]**

### 5.4 Easy VPN
**No existe en la serie RV0xx (RV016/RV042/RV042G/RV082).** El AG no tiene ninguna sección Easy VPN. (Easy VPN existe en otros modelos como RV320/RV325.) [AG] [Inf]

---

## 6. Cisco QuickVPN

### 6.1 Qué es y cómo funciona
- Cliente gratuito **sólo para Windows** (no Mac). "You do not have to configure VPN policies." No permite elegir algoritmos. **No es site-to-site.** No pasa NetBIOS (usar IPs o LMHOSTS). [RN] [App Note OL-25680]
- Flujo de conexión [KB smb2922]:
  1. Conexión **SSL/HTTPS** al router por **TCP 443 o TCP 60443** "depending on the router's VPN configuration"; se comprueba certificado (si no hay, aparece aviso y se puede continuar).
  2. El router autentica usuario/contraseña y se establece un **túnel IPsec** (IKE en **UDP 500**, **UDP 4500** con NAT-T, ESP).
  3. **Prueba de conectividad**: el cliente envía ICMP Echo Request a la **IP LAN del router**; si no hay respuesta → "**Remote Gateway Not Responding**".
- Puertos a permitir en todo el camino (cliente, router delante del cliente, ISP): **TCP 443, TCP 60443, UDP 500, UDP 4500** (algunos añaden protocolos 50/51 ESP/AH). No usar esos puertos en reglas de port-forwarding del RV082. [Com]
- Estados mostrados al conectar: **Connecting → Provisioning → Activating Policy → Verifying Network**; luego icono verde y ventana de estado (IP remota, hora de inicio, duración). [AG]
- Sólo **un** túnel QuickVPN activo a la vez por PC (se pueden guardar varios perfiles). Cada cuenta permite **una** conexión simultánea. [AG] [KB]
- SmallNetBuilder: QuickVPN "isn't a full, bi-directional IPsec tunnel": equipos del LAN no podían iniciar conexiones hacia el cliente QuickVPN. Throughput medido ~69 Mbps (3DES/MD5) en RV082 v3. [3P]

### 6.2 Versiones de QuickVPN
| Versión | Fecha | Notas |
|---|---|---|
| 1.2.5 – 1.2.8 | ~2007 – ene 2008 | 1.2.8: corrige DDNS en Vista; más log en log.txt |
| 1.2.11 | jul 2008 (RN mar 2009) | Corrige Vista con Wi-Fi; detecta si ICMP o IKE (UDP 500/4500) están bloqueados y lo registra en log.txt |
| **1.4.0.5** | ~feb 2010 | **Primera con Windows 7** y Vista 32/64 bit. Known issue: "Remote Gateway is not responding" con túnel ya levantado |
| **1.4.1.2** | jul 2010 | Corrige DDNS en Win7/Vista; transferencias grandes detrás de NAT; usuario/contraseña de **32 caracteres**; si el servicio IPsec de Windows está deshabilitado intenta habilitarlo |
| **1.4.2.1** | 18 jul 2011 | **Última versión**. Sólo corrige fuente japonesa. Probada en KB Cisco con fw 4.2.2.08 |

Sistemas oficialmente soportados [App Note] [RN]: **Windows XP 32-bit (SP3 o parche KB889527), Vista 32/64 (SP2), Windows 7 32/64.** Producto **End-of-Life**; un empleado Cisco: "compatible with Windows 7 operating system (32-bit and 64-bit editions)". **Windows 8/8.1/10/11: no soportado oficialmente.** [Com]

### 6.3 Requisitos en el PC
- **Derechos de administrador**. [AG] [RN]
- **Windows Firewall habilitado** (en Vista/7 el servicio IPsec se deshabilita si se apaga Windows Firewall; firewalls de terceros que lo desactivan rompen QuickVPN). [RN]
- Servicio **IPSec Policy Agent** activo (msconfig > Services). [KB smb2922] En Win7+ también debe estar activo **IKE and AuthIP IPsec Keying Modules (IKEEXT)** [Inf].
- Regla de firewall entrante **ICMPv4** (Windows Firewall with Advanced Security > Inbound Rules > New Rule > Custom > Protocol type ICMPv4 > Finish). [KB smb2922]
- Vista: SP2. XP: SP3 (bug del firewall de XP SP2 que descarta ICMP).
- Subred del PC/LAN remota **distinta** de la LAN del RV082 (evitar 192.168.1.x en la oficina). [App Note] [KB]

### 6.4 Configuración en el router: VPN > VPN Client Access [AG] [KB smb452]
Configurar primero el router; "you can add up to **50 users**".
- **Username** (no editable después), **New Password**, **Confirm New Password**, **Allow Password Change** (Yes/No), **Active** (casilla) → **Add to list** → **Save**. Editar: seleccionar → **Update**; borrar: **Delete**; limpiar campos: **Add New**.
- Al guardar por primera vez, el router pregunta si quiere **cambiar automáticamente la IP LAN** para evitar conflictos: "If an IP conflict occurs, the QuickVPN client will not connect to the router."
- **Firewall > General > HTTPS** debe estar habilitado (lo está por defecto). [AG]
- KB smb2922: "Enable Remote Management for ports 443 and 60443" — en la práctica, el requisito firme es HTTPS habilitado; habilitar Remote Management no siempre es necesario (contradicciones en foros). **[Dudoso]**

### 6.5 Certificate Management (en VPN Client Access) [AG] [KB smb452]
| Botón | Función |
|---|---|
| **Generate** (Generate New Certificate) | Genera un certificado nuevo que **reemplaza** al existente (pide confirmación OK) |
| **Export for Admin** | Exporta el certificado de administrador **con clave privada** (backup; hacerlo antes de un reset de fábrica para poder reimportarlo) |
| **Export for Client** | Exporta el certificado de cliente (.pem) para instalar en el PC del usuario → evita MITM |
| **Import** (Browse + Import) | Restaura un certificado de administrador guardado |
| **Existing Certificate** | Nombre del certificado actual |

- El certificado de cliente se copia a la carpeta de instalación de QuickVPN: `C:\Program Files\Cisco Small Business\QuickVPN Client\` (en 64 bits normalmente `C:\Program Files (x86)\...` [Inf]).
- **Es opcional**: sin certificado, QuickVPN muestra un aviso de seguridad y se puede continuar.
- KB: para guardar el .pem en el almacén de certificados de Windows habría que convertirlo a .pfx/.p12 (QuickVPN lo lee de su carpeta, no del almacén). [KB]
- Si se pulsa **Generate**, hay que **re-exportar y redistribuir** el certificado de cliente (si no, aviso/fallo de verificación). [Inf]
- **Estos certificados son sólo para QuickVPN**; los túneles IPsec G2G/C2G sólo usan PSK. [AG]

### 6.6 Uso del cliente [AG]
Campos del perfil: **Profile Name**, **User Name**, **Password**, **Server Address** (IP WAN o nombre DDNS del router), **Port for QuickVPN** (**Auto** por defecto; elige 443/60443 según configuración — la lista exacta de opciones del desplegable no está documentada), **Use Remote DNS Server** (resolver nombres del LAN remoto con el DNS que entrega el servidor QuickVPN). Botones Save / Delete / Connect / Disconnect / Change Password (si el admin lo permite: Old Password, New Password, Confirm). El cliente guarda un **log.txt** en su carpeta de instalación. [Com] [RN]

### 6.7 Troubleshooting QuickVPN
| Síntoma | Causas habituales | Acciones |
|---|---|---|
| "**The remote gateway is not responding. Do you want to wait?**" (tras Verifying Network) | Falla el ping ICMP al LAN IP del router: Windows Firewall/antivirus/firewall de terceros bloquea ICMP; servicio IPsec parado; subred del cliente = subred LAN del router; ESP/UDP 4500 bloqueado por el ISP/NAT; Windows 8/10/11 (no soportados) | Activar Windows Firewall + regla ICMPv4 entrante; verificar IPsec Policy Agent/IKEEXT; desinstalar otros clientes VPN y "limpiar" políticas IPsec antiguas; cambiar LAN del router (p. ej. 192.168.10.0/24); probar otra red; si aparece con el túnel ya funcionando, RN dice que puede ignorarse "as long as the traffic continues to flow" |
| Error de login | Usuario/contraseña (sensibles a mayúsculas), cuenta no **Active**, cuenta ya conectada | Revisar VPN Client Access |
| No conecta (no llega a Provisioning) | HTTPS deshabilitado; TCP 443/60443 bloqueados o usados por port-forwarding; Server Address incorrecta | Firewall > General > HTTPS; revisar reglas de reenvío; probar puerto explícito |
| Aviso de certificado | No hay certificado de cliente en la carpeta | Export for Client y copiarlo, o aceptar el aviso |
| Cliente detrás de NAT en Vista+ | Windows no establece SAs con NAT-T hacia un servidor detrás de NAT por defecto | Clave de registro `HKLM\SYSTEM\CurrentControlSet\Services\PolicyAgent\AssumeUDPEncapsulationContextOnSendRule` (DWORD) = **2** y reiniciar (KB Microsoft 926179/944335 citada en RN 1.2.x) [RN] [Inf: el valor 2 es conocimiento general de Microsoft] |
| Win 8/8.1/10/11 | No soportado; instalación puede fallar en 8.1 x64; "Failed to ping VPN router" frecuente | Ejecutar como administrador / compatibilidad Win7 (reportado que NO siempre sirve); usar alternativa: PPTP (§7), Shrew Soft o TheGreenBow con Client to Gateway (§5, §9) — Cisco publica "Alternative VPN Setup for Windows 8" (smb2931) usando el cliente VPN integrado de Windows |

---

## 7. PPTP Server

### 7.1 Configuración [AG]
Ruta: **VPN > PPTP Server**. "enable **up to five** PPTP VPN tunnels" (RV082). PPTP está incluido en Windows; no hace falta instalar cliente. **Deshabilitado por defecto.** "Be aware that security vulnerabilities have been found in this protocol."

| Campo | Descripción |
|---|---|
| **Enable PPTP Server** | Casilla; al marcarla aparecen el resto de campos |
| **IP Address Range: Range Start / Range End** | Direcciones LAN asignadas a los clientes PPTP. **Defecto 192.168.1.200 – 192.168.1.204** (5 IP). Deben estar **fuera del rango DHCP** del router |
| **PPTP Server (usuarios)**: **Username**, **New Password**, **Confirm New Password** → **Add to list** / **Update** / **Delete** / **Add New** | Lista de usuarios PPTP (independiente de la lista QuickVPN) |
| **Connection List** (sólo lectura, **Refresh**) | **Username**, **Remote Address** (IP WAN del cliente), **PPTP IP Address** (IP LAN asignada) |

### 7.2 Cifrado y autenticación
- La UI **no ofrece opciones** de MPPE ni de método de autenticación (no hay casillas de MPPE-40/128, MS-CHAPv2, etc.). Por comportamiento de foros, clientes Windows con MS-CHAPv2 y cifrado MPPE conectan. Cuál exactamente (40/128 bit, "require encryption") **no está documentado → [Dudoso]**. Si un cliente Windows falla con "Require encryption", probar "Optional encryption" para diagnosticar. [Com]
- PPTP con MS-CHAPv2 se considera **criptográficamente roto** (ataques de diccionario/MS-CHAPv2 reducible a DES). Usar sólo si se acepta el riesgo. [Inf]

### 7.3 Cliente Windows
- AG (XP/2000): Network Connections > nueva conexión > "Connect to the network at my workplace" > VPN > host/IP del router > usuario/contraseña. [AG]
- Windows 8/10/11 [KB smb2931 + Inf]: Network and Sharing Center > Set up a new connection or network > Connect to a workplace > Use my Internet connection (VPN) > Internet address = IP/DDNS del router > Destination name > Create. En propiedades de la conexión: **Type of VPN = Point to Point Tunneling Protocol (PPTP)** (el modo "Automatic" también acaba probando PPTP), autenticación **MS-CHAP v2**. El KB smb2931 dice que usuario/contraseña deben coincidir con el usuario VPN del router; para PPTP son los de **VPN > PPTP Server** [Inf].
- Microsoft anunció (2024) la deprecación de PPTP y L2TP en futuras versiones de Windows Server; el cliente PPTP sigue en Windows 10/11 a la fecha de los datos consultados **[Dudoso para versiones futuras]**.

### 7.4 Limitaciones
- 5 sesiones simultáneas máximo (RV082).
- Requiere **TCP 1723 + GRE (protocolo IP 47)** extremo a extremo; si el RV082 está detrás de otro NAT/módem, reenviar ambos (o DMZ/bridge); CGNAT o redes que bloquean GRE impiden PPTP. [Inf]
- **iOS 10+ y macOS 10.12+ eliminaron el cliente PPTP**. [Com] Android eliminó PPTP del cliente integrado en versiones recientes (Android 12+) **[Inf/Dudoso por versión exacta]**.
- No hay split-tunnel configurable desde el router; en Windows se controla con "Use default gateway on remote network" (TCP/IPv4 > Advanced). [Inf]

---

## 8. VPN Passthrough [AG]
Ruta: **VPN > VPN Passthrough**. Permite que clientes VPN **del LAN** del RV082 conecten a servidores VPN en Internet. Cisco recomienda dejarlo activado.

| Opción | Defecto |
|---|---|
| **IPSec Passthrough** | Enabled |
| **PPTP Passthrough** | Enabled |
| **L2TP Passthrough** | Enabled |

Desactivarlos bloquea esos tipos de VPN salientes. Nota [Com]: algunos usuarios reportan interferencias entre reenvío de IPsec hacia el LAN y túneles G2G del propio router (hilo "RV082 IPSec forwarding interferes with gateway 2 gateway VPN", fw 4.1.1.01-sp) — no reenviar UDP 500/4500 a un host interno si el router termina túneles IPsec. [Inf]

---

## 9. Interoperabilidad (con parámetros compatibles)

### 9.1 Reglas de oro de interoperabilidad [Inf, basado en AG + foros]
1. **IKEv1 únicamente**. En el otro equipo forzar *IKEv1* (no IKEv2, no "auto").
2. **Una sola propuesta** en el otro extremo que coincida exactamente: cifrado ∈ {DES, 3DES, AES-128/192/256}, hash ∈ {MD5, SHA1}, DH ∈ {1, 2, 5}. **No** SHA-256, **no** DH 14/19/20, **no** AES-GCM.
3. **PFS**: si el RV082 tiene PFS marcado, el otro debe usar PFS con el **mismo grupo** que *Phase 2 DH Group*; si se desmarca, el otro debe tener PFS **off**. (Desactivar PFS en ambos es un truco de diagnóstico frecuente.) [Com]
4. **Lifetimes**: 28800 (F1) y 3600 (F2) en ambos lados, en segundos; desactivar lifetime por KB/volumen si es posible.
5. **Selectores**: Local/Remote Group del RV082 = red remota/local del otro, con la **misma máscara exacta** (una sola red por túnel; en el otro equipo una sola fase 2 por par de redes).
6. **Main Mode** si ambos tienen IP fija (IDs = IP). **Aggressive** + IDs FQDN/USER FQDN si uno es dinámico.
7. **NAT-T** activado en ambos si hay NAT; DPD en ambos o en ninguno.
8. Excluir el tráfico del túnel del NAT/masquerade en el otro equipo (NAT exemption / no-NAT).

### 9.2 RV082 ↔ RV082 / RV042 / RV042G / RV016
Seguir §4.6. Totalmente compatible; mismos menús. Diferencias: RV042 v3 50 túneles (según SmallNetBuilder) — el AG dice "A Cisco RV0xx Series router supports up to 100 VPN tunnels" **[Dudoso para RV042]**. Problema típico: tras un corte WAN el túnel no se restablece → usar Keep-Alive (en uno o ambos) y/o DPD; un usuario lo resolvió con Keep-Alive en el RV016 y desactivado en el RV042; otros borrando y recreando el túnel en ambos lados. [Com]

### 9.3 RV082 ↔ Cisco ASA (plantilla, ASA 8.4+/9.x) [Inf; no oficial]
Ajustes RV082: Local Group = LAN RV (192.168.20.0/24), Remote Group = LAN ASA (10.10.0.0/24), Remote Gateway IP Only = IP outside ASA; IKE PSK; F1 AES-256/SHA1/Group 2 o 5/28800; PFS on; F2 AES-256/SHA1/Group = el mismo/3600; Main Mode; NAT-T según caso; DPD on.
```
crypto ikev1 enable outside
crypto ikev1 policy 10
 authentication pre-share
 encryption aes-256
 hash sha
 group 5
 lifetime 28800
crypto ipsec ikev1 transform-set TS-RV082 esp-aes-256 esp-sha-hmac
object network LAN-ASA
 subnet 10.10.0.0 255.255.255.0
object network LAN-RV082
 subnet 192.168.20.0 255.255.255.0
access-list VPN-RV082 extended permit ip object LAN-ASA object LAN-RV082
nat (inside,outside) source static LAN-ASA LAN-ASA destination static LAN-RV082 LAN-RV082 no-proxy-arp route-lookup
crypto map OUTSIDE_MAP 20 match address VPN-RV082
crypto map OUTSIDE_MAP 20 set peer <IP_WAN_RV082>
crypto map OUTSIDE_MAP 20 set ikev1 transform-set TS-RV082
crypto map OUTSIDE_MAP 20 set pfs group5
crypto map OUTSIDE_MAP 20 set security-association lifetime seconds 3600
crypto map OUTSIDE_MAP 20 set security-association lifetime kilobytes unlimited
crypto map OUTSIDE_MAP interface outside
tunnel-group <IP_WAN_RV082> type ipsec-l2l
tunnel-group <IP_WAN_RV082> ipsec-attributes
 ikev1 pre-shared-key <PSK>
```
Notas:
- `lifetime kilobytes unlimited` requiere ASA ≥ 9.1(2) [Com]. Hilo RV082↔ASA5510: el túnel "se congela" a las ~8 h (RV082 ve "connected", ASA lo ve caído) con lifetime 86400 en RV y 4 608 000 KB en ASA → igualar lifetimes en segundos y eliminar el de KB; considerar `sysopt connection preserve-vpn-flows`. [Com] **[Dudoso: sin solución confirmada]**
- Versiones recientes de ASA/FTD han **retirado DH 2/5 y algoritmos débiles** en algunas ramas (9.13+/9.15+) → con esas versiones puede que **no sea posible** interoperar con el RV082. **[Dudoso: verificar con la versión concreta]**
- Si el RV082 tiene IP dinámica: en ASA usar `tunnel-group DefaultL2LGroup` o un dynamic crypto map, y en el RV082 Aggressive/ID FQDN; complejo y poco documentado. [Inf]

### 9.4 RV082 ↔ Cisco IOS router (plantilla IKEv1 crypto map) [Inf]
```
crypto isakmp policy 10
 encr aes 256
 hash sha
 authentication pre-share
 group 5
 lifetime 28800
crypto isakmp key <PSK> address <IP_WAN_RV082>
crypto isakmp keepalive 30 5 periodic     ! DPD (opcional; activar DPD también en RV082)
crypto ipsec transform-set TS-RV esp-aes 256 esp-sha-hmac
 mode tunnel
ip access-list extended VPN-RV082
 permit ip 10.10.0.0 0.0.0.255 192.168.20.0 0.0.0.255
crypto map CMAP 10 ipsec-isakmp
 set peer <IP_WAN_RV082>
 set transform-set TS-RV
 set pfs group5
 set security-association lifetime seconds 3600
 match address VPN-RV082
interface <WAN>
 crypto map CMAP
! Excluir del NAT: en la ACL de NAT, "deny ip 10.10.0.0 0.0.0.255 192.168.20.0 0.0.0.255" antes del permit
```

### 9.5 RV082 ↔ pfSense (strongSwan) [Com + Inf]
- **Phase 1**: Key Exchange version **IKEv1**; Internet Protocol IPv4; Interface WAN; Remote Gateway = IP del RV082; Authentication **Mutual PSK**; Negotiation mode **Main** (o Aggressive si el RV082 es dinámico/IDs FQDN); My identifier **My IP address**; Peer identifier **Peer IP address** (o FQDN/User FQDN según §4.2); Encryption **AES 256 bits** + Hash **SHA1** + DH **5 (1536 bit)** o **2 (1024 bit)** (pfSense los marca como débiles pero los permite); Lifetime **28800**; NAT Traversal **Auto**; DPD activado.
- **Phase 2**: Mode **Tunnel IPv4**; Local Network = LAN pfSense; Remote Network = LAN RV082 (misma máscara que Local Group del RV); Protocol **ESP**; Encryption **AES 256**; Hash **SHA1**; PFS key group = el *Phase 2 DH Group* del RV082 (o off si PFS off en RV); Lifetime **3600**.
- Problemas reportados: "Cisco RV042 only uses DH Groups 1, 2, and 5"; un caso pfSense→RV082 con Aggressive/3DES/SHA1/DH2/PFS2 no funcionó y la diferencia con el que sí funcionó fue **PFS** (desactivado en el que funcionaba); otro caso necesitó **reiniciar el RV** y añadir reglas de firewall en pfSense (pestañas LAN e IPsec). [Com]

### 9.6 RV082 ↔ FortiGate (plantilla CLI, FortiOS 6.x/7.x) [Inf]
```
config vpn ipsec phase1-interface
    edit "to-RV082"
        set interface "wan1"
        set ike-version 1
        set mode main
        set peertype any
        set proposal aes256-sha1
        set dhgrp 5
        set remote-gw <IP_WAN_RV082>
        set psksecret <PSK>
        set keylife 28800
        set nattraversal enable
        set dpd on-idle
    next
end
config vpn ipsec phase2-interface
    edit "to-RV082-p2"
        set phase1name "to-RV082"
        set proposal aes256-sha1
        set pfs enable
        set dhgrp 5
        set keylifeseconds 3600
        set src-subnet 10.10.0.0 255.255.255.0
        set dst-subnet 192.168.20.0 255.255.255.0
    next
end
```
+ ruta estática a 192.168.20.0/24 por la interfaz "to-RV082" y políticas de firewall en ambos sentidos (sin NAT). Usar **un phase2 por cada par de subredes** (el RV082 sólo negocia un par por túnel). [Com: "split... create a phase 2 interface for each remote or local subnet"]. Algunas versiones recientes de FortiOS limitan algoritmos débiles (DH 1/2/5, SHA1) o los ocultan con `strong-crypto` **[Dudoso: depende de versión]**.

### 9.7 RV082 ↔ SonicWall [3P SonicWall KB]
Ejemplo oficial SonicWall ↔ "Linksys VPN Router" (Aggressive Mode):

| Parámetro | SonicWall | Linksys/RV |
|---|---|---|
| Autenticación | IKE using Preshared Secret | IKE with Preshared Key |
| IDs | Local ID yahoo.com / Peer ID google.com (ejemplo) | Local google.com / Remote yahoo.com (cruzados) |
| F1 | DH Group 1, 3DES, SHA1, 28800 | igual |
| F2 | 3DES, SHA1, PFS on Group 1, 28800 | igual |
| Keep Alive | en un solo extremo (el del RV, que tenía WAN DHCP) | Enabled |
| Aggressive | (debe coincidir) | Enabled |

[Inf] Se recomienda subir a AES-256/SHA1/Group 2 o 5 en ambos. Un reporte de foros: RV042G conectó con SonicWall pero un Linksys RV042 antiguo no (posible firmware viejo). [Com]

### 9.8 RV082 ↔ MikroTik RouterOS (plantilla v6.43+/v7) [Inf]
```
/ip ipsec profile add name=rv082 hash-algorithm=sha1 enc-algorithm=aes-256 dh-group=modp1536 lifetime=8h nat-traversal=yes dpd-interval=30s dpd-maximum-failures=5
/ip ipsec peer add name=rv082 address=<IP_WAN_RV082>/32 profile=rv082 exchange-mode=main
/ip ipsec identity add peer=rv082 auth-method=pre-shared-key secret="<PSK>"
/ip ipsec proposal add name=rv082 auth-algorithms=sha1 enc-algorithms=aes-256-cbc pfs-group=modp1536 lifetime=1h
/ip ipsec policy add src-address=10.10.0.0/24 dst-address=192.168.20.0/24 tunnel=yes peer=rv082 proposal=rv082 action=encrypt
/ip firewall nat add chain=srcnat src-address=10.10.0.0/24 dst-address=192.168.20.0/24 action=accept place-before=0
```
Equivalencias DH: Group 1 = modp768, Group 2 = **modp1024** (por defecto en MikroTik), Group 5 = **modp1536**. pfs-group=none si PFS off en el RV. Un caso MikroTik↔Cisco sólo funcionaba cuando el MikroTik iniciaba; asegurar Keep-Alive/DPD en el RV o `level=require` en la política. [Com]

### 9.9 Clientes software (Client to Gateway)

#### TheGreenBow [3P TheGreenBow + KB smb3049]
Guía TheGreenBow (RV082): "the Greenbow IPSEC VPN client will work only with the Linksys RV082 when the RV082 is configured ... through a **CLIENT-TO-GATEWAY** connection of the type **GROUP** (not TUNNEL)".
- **RV082**: VPN > Client to Gateway > **Group VPN**; Remote Client = **E-mail Addr. (USER FQDN)** (cualquier e-mail, no tiene que resolver); IPSec Setup = mismos grupos/cifrados/hash que el cliente; lifetimes por defecto OK; PFS recomendado; Advanced: sólo **Aggressive** (si acaso); **Enable** el Group VPN.
- **Cliente – Phase 1**: Interface = IP de la NIC; Remote Address = IP pública del RV082; Preshared key igual; IKE = F1 del RV; Advanced: Aggressive si el RV lo tiene; "Enter the value 50 for the NAT PORT" (indicación de la guía antigua) **[Dudoso]**; Local ID = el e-mail (USER FQDN).
- **Cliente – Phase 2**: **VPN Client address** = IP virtual en subred distinta de la NIC y de la LAN del RV (ej. NIC 192.168.10.90 → virtual 192.168.100.1); Remote LAN address = red LAN del RV + máscara (tipo Subnet address); ESP y PFS iguales a F2 del RV.
- **Global parameters**: desmarcar **Block non-ciphered connections** para mantener Internet local.
- KB Cisco smb3049 (fw 4.2.1.02) muestra otra variante: túnel con **NAT-T: Forced**, Local/Remote ID tipo IP Address, DH1/DES/MD5/28800 y DH1/DES/MD5/3600 (valores de ejemplo, débiles).

#### Shrew Soft VPN Client (gratuito) [3P Shrew Howto + KB smb3039]
- **RV0xx**: Client to Gateway > **Group VPN**, Interface WAN1, Local Group Subnet (LAN), Remote Client **Domain Name (FQDN)** = `shrew.net` (ejemplo), F1 DH2/AES-256/SHA1/28800, PFS on, F2 DH2/AES-256/SHA1/3600, PSK ≤30; Advanced: Aggressive (automático en Group), **Keep-Alive**, NetBIOS Broadcast (si se necesita), **NAT Traversal**; resto off.
- **Shrew – General**: Remote host = IP/DDNS del RV; Auto Configuration **disabled** (el RV no soporta ModeCfg); Address method = virtual adapter con **dirección asignada** fuera de ambas LAN (ej. 192.168.30.x/24); MTU 1380.
- **Client**: NAT Traversal **enable**, puerto **4500**, keep-alive 15 s (KB: 30 por defecto); IKE fragmentation **disable**.
- **Name Resolution**: DNS/WINS manuales (DNS normalmente la IP LAN del RV).
- **Authentication**: **Mutual PSK**; Local Identity **Fully Qualified Domain Name** = `shrew.net`; Remote Identity **IP Address** con "Use a discovered remote host address"; Credentials = PSK.
- **Phase 1**: Exchange **aggressive**, DH 2, AES 256, SHA1, 28800 s, data limit 0.
- **Phase 2**: esp-aes 256, HMAC SHA1, PFS group 2, compression disabled, 3600 s, data limit 0.
- **Policy**: desmarcar "Obtain Topology Automatically"; Add → Include → red LAN remota (ej. 192.168.0.0/255.255.255.0).
- Error típico: el router espera ID **User FQDN** y el cliente envía IPv4 → "phase 1 ID type mismatch". [Com]

#### Windows nativo
- **QuickVPN** (XP/Vista/7) — §6.
- **PPTP** integrado — §7 (única opción nativa moderna).
- **L2TP/IPsec** — no (sin servidor L2TP).
- **IPsec "Microsoft XP/2000 VPN Client"** (Group VPN) — política IPsec del sistema; no documentado paso a paso **[Dudoso]**.
- **IKEv2 / SSTP** — no.

#### iOS / Android / macOS
| Tipo en el dispositivo | ¿Funciona con RV082? |
|---|---|
| PPTP | Sólo en dispositivos/OS antiguos (iOS ≤ 9, macOS ≤ 10.11, Android antiguos) [Com] |
| L2TP/IPsec | **No** (RV082 no tiene servidor L2TP) |
| "IPsec (Cisco)" / IKEv1 XAUTH PSK | **No** (requiere XAUTH + Mode Config) [Inf] [Com: "nor is the Cisco IPsec VPN client that also comes on the iPhone"] |
| IKEv2 | **No** |
| AnyConnect | **No** ("Cisco AnyConnect SSL VPN client is not compatible with the RV082") [Com] |
| Apps IPsec de terceros con IKEv1 PSK sin ModeCfg (p. ej. TheGreenBow móvil, strongSwan con configuración manual) | Posible en teoría con Client to Gateway; no documentado **[Dudoso]** |

Recomendación Cisco en foros: sustituir por RV340/RV345 (AnyConnect, L2TP, IKEv2) para móviles. [Com]

---

## 10. Troubleshooting VPN

### 10.1 Dónde mirar
- **VPN > Summary**: Status (*Connected* / *Waiting for Connection*), **Tunnel Test > Connect**, Phase2 Enc/Auth/Grp efectivo, Remote Gateway (IP resuelta si DNS).
- **Log > System Log**: habilitar logs; **View System Log** → desplegable para elegir el tipo de log (incluye el **VPN Log**), **Refresh**, **Clear**. Opcional **Syslog** (Enable Syslog + Syslog Server; reiniciar el RV para que tenga efecto) y **E-Mail Alert**. Event logging oficial: "Syslog, email alerts, VPN tunnel status monitor". [AG] [DS]
- **QuickVPN**: `log.txt` en la carpeta de instalación del cliente. [Com] [RN]
- Cliente TheGreenBow: Console; Shrew: Trace utility.

### 10.2 Formato del VPN Log del RV0xx
El motor IKE es de estilo FreeS/WAN/Openswan ("pluto"): estados `STATE_MAIN_I1`, `#n:` = número de estado/SA. **[Inf a partir del formato]** Ejemplos reales:

```
Jan 1 00:00:14 2010 VPN Log (g2gips0) #1: ISAKMP SA established
Jan 1 00:00:14 2010 VPN Log (g2gips0) #2: up-client output: iptables: No chain/target/match by that name
Jan 1 00:00:14 2010 VPN Log (g2gips0) #2: [Tunnel Negotiation Info] Quick Mode Phase 2 SA Established, IPSec Tunnel Connected
```
(`(g2gips0)` = identificador interno de la conexión; el mensaje *up-client output: iptables* aparece incluso en túneles que funcionan → benigno.) [Com Netgate]

Otros mensajes vistos en RV0xx y su interpretación:

| Mensaje (literal o casi) | Fase | Significado | Qué revisar |
|---|---|---|---|
| `[Tunnel Negotiation Info] ... Main Mode ...` / `Initiating Main Mode` | F1 | Inicio negociación | Si no avanza: el par no responde |
| `max number of retransmissions (20) reached STATE_MAIN_I1. No response (or no acceptable response) to our first IKE message` [Com] | F1 | El par **no contesta** al primer paquete IKE | IP/FQDN remota incorrecta, DDNS desactualizado, UDP 500 bloqueado (ISP/módem/firewall), túnel no habilitado en el otro lado, Main vs Aggressive distinto |
| Igual pero `STATE_AGGR_I1` [Com] | F1 | Ídem en Aggressive Mode | Ídem; además IDs/PSK |
| Bloqueado en `STATE_MAIN_I3` [Com] | F1 (msg 5/6) | Ya hubo intercambio DH; fallo en autenticación | **PSK distinta** o **ID** (IP/FQDN/e-mail) no coincide; NAT no declarado |
| `No acceptable Oakley Transform, No Proposal chosen. Please check your SA or preshared key setting` [Com] | F1 | **Propuesta F1 no coincide** | Cifrado/hash/DH de F1 (o PSK); en el cliente aparece como "IKE Phase1 No Response" |
| `... no connection has been authorized ...` (p. ej. "initial Main Mode message received ... but no connection has been authorized with policy=PSK") [Com, Inf] | F1 | El RV recibe IKE de una IP/ID que **no corresponde a ningún túnel** configurado | Remote Security Gateway (IP o tipo), IP pública real del par (¿detrás de NAT?), Main vs Aggressive. En un caso RV082↔RV082 se resolvió igualando **Phase 2 Encryption/Authentication** |
| `NO_PROPOSAL_CHOSEN` tras "ISAKMP SA established" | F2 | **Propuesta F2 no coincide** | Cifrado/hash F2, **PFS y su grupo**, lifetime |
| `INVALID_ID_INFORMATION` | F2 | **Selectores (Local/Remote Group) no coinciden** | Red/máscara exacta en ambos lados; tipo IP vs Subnet (un host vs red); en clientes: "IPV4_ADDR" vs "IPV4_SUBNET" |
| `we require PFS but Quick Mode proposal omits it` [Inf: mensaje Openswan] | F2 | PFS on en el RV y off en el par | Igualar PFS |
| `INVALID_COOKIE` [3P] | — | Un extremo usa una SA que ya no existe | Desconectar/reiniciar el túnel en ambos lados (Tunnel Test > Disconnect/Connect) |
| `PAYLOAD_MALFORMED` / `INVALID_HASH_INFORMATION` [3P/Inf] | F1 | Suele ser **PSK distinta** o algoritmos de F1 incompatibles | PSK (≤30 caracteres, cuidado con espacios/símbolos), cifrado F1 |
| `received remote ID other than expected` [3P] | F1 | El ID remoto recibido no es el esperado | Tipo y valor de ID (FQDN/USER FQDN/IP) cruzados correctamente |
| `[Tunnel Negotiation Fail] Phase 1 SA was destroyed` [Com] | F1 | La SA de F1 se eliminó (fallo o expiración) | Ver mensajes previos |
| `NAT-T: new mapping ...` [Com] | — | El NAT intermedio cambió el puerto/IP del par; a veces el túnel no se recupera | DPD + Keep-Alive; reducir timeout UDP en el NAT o fijar reenvío UDP 4500; reconectar |
| Mensajes de DPD (peer declarado muerto) | — | El par no responde a R-U-THERE | Enlace caído; si Tunnel Backup está activo, conmuta |

### 10.3 Checklist de diagnóstico rápido [Inf, basado en fuentes]
1. ¿Ambos túneles **habilitados** y guardados? ¿Interface (WAN1/WAN2) correcta y con IP?
2. ¿IP pública del par correcta? (Si el par tiene IP dinámica: DDNS actualizado, o usar *IP by DNS Resolved*.)
3. ¿Hay **NAT** delante de algún RV? → NAT Traversal en **ambos**, reenviar UDP 500/4500 al RV (o DMZ/bridge del módem); la "Remote Gateway" debe ser la IP **pública** del NAT; si el par aplica IDs estrictos, usar FQDN/USER FQDN en vez de IP Only (el RV detrás de NAT se identifica con su IP WAN privada).
4. **F1**: modo (Main/Aggressive), cifrado, hash, DH, lifetime, PSK, IDs.
5. **F2**: cifrado, hash, **PFS + grupo**, lifetime, **selectores idénticos** (red/máscara).
6. Subredes LAN **distintas** y sin solapamiento (también con la red local del cliente remoto).
7. Si "Connected" pero **no hay ping**:
   - Firewall del host destino (Windows bloquea ICMP de otras subredes por defecto).
   - **Default gateway** de los equipos del LAN debe ser el RV082 (o ruta estática hacia la LAN remota vía RV082).
   - En el otro equipo: exclusión de NAT y reglas de firewall para el tráfico del túnel.
   - ESP (proto 50) bloqueado sin NAT-T → activar NAT-T.
   - No usar nombres NetBIOS sin NetBIOS Broadcast/WINS/Split DNS.
8. Si cae cada X horas: lifetimes distintos, rekey por KB en el par, DPD con intervalos distintos, Keep-Alive.
9. Si nada funciona: borrar y recrear el túnel en ambos lados; reiniciar ambos routers; actualizar firmware. [Com]

### 10.4 NAT detrás de NAT (doble NAT)
- Escenario típico: módem/router del ISP haciendo NAT → RV082 con WAN privada.
- Opción 1 (preferida): poner el módem en **bridge** (PPPoE en el RV082) o asignar al RV082 la IP pública (IP passthrough). [Inf]
- Opción 2: en el módem, **DMZ host** = IP WAN del RV082, o reenviar **UDP 500**, **UDP 4500** (IPsec/NAT-T), **TCP 443/60443** (QuickVPN), **TCP 1723 + GRE 47** (PPTP). [Inf]
- Activar **NAT Traversal** en el túnel del RV082 y en el par. [AG]
- Si el ISP usa **CGNAT** (IP WAN 100.64.0.0/10 o privada no controlable): el RV082 no puede ser **respondedor**; sólo puede **iniciar** (Keep-Alive activado) hacia un par con IP pública. Para el par, configurar el RV como dinámico (Aggressive + USER FQDN/FQDN). [Inf]

### 10.5 Subredes solapadas
- "The two ends of the tunnel cannot be on the same subnet." [AG] El RV0xx **no tiene NAT dentro del túnel** (no hay opción de "NAT traversal of addresses"/1:1 NAT para VPN en la UI documentada) → la única solución es **renumerar** una de las LAN (p. ej. evitar 192.168.1.0/24 y 192.168.0.0/24, que coinciden con routers domésticos de los teletrabajadores). [AG] [App Note QuickVPN] [Inf]
- Para QuickVPN, el router ofrece cambiar automáticamente la IP LAN al guardar usuarios. [AG]
- Alternativa: hacer el NAT 1:1 en el otro extremo si éste lo soporta (ASA twice-NAT, FortiGate, pfSense "NAT/BINAT" en Phase 2). [Inf]

### 10.6 MTU / fragmentación
- WAN MTU: **Setup > Network** (edición de WAN): **Auto** (recomendado) o **Manual** en bytes. [AG]
- IPsec ESP + NAT-T añade ~50–80 bytes. Síntomas de MTU: ping pequeño funciona, ping grande / RDP / SMB / web se cuelgan. [Inf]
- Prueba desde Windows: `ping -f -l 1372 <IP_remota>` bajando hasta que pase; ajustar MTU WAN Manual (p. ej. 1400–1440, o 1492 con PPPoE) o MSS en el otro extremo. [Inf]
- Shrew Soft recomienda MTU 1380 en su adaptador virtual. [3P]

### 10.7 DPD / Keep-Alive / reconexión
- DPD sólo funciona activado en **ambos** extremos; usar el **mismo intervalo**. [AG]
- Tunnel Backup requiere DPD. [AG]
- Si los túneles no se restablecen tras caídas de WAN: Keep-Alive en el lado iniciador (normalmente el de IP dinámica); algunos usuarios resolvieron desactivando DPD en ambos lados y dejando Keep-Alive. [Com]
- "VPN stops passing traffic but stays connected": típicamente SA desincronizada tras un rekey o NAT-T new mapping → DPD, igualar lifetimes, **Disconnect/Connect** manual. [Com]

---

## 11. Procedimientos paso a paso (casos comunes)

### 11.1 RV082 (oficina central, IP fija) ↔ RV042 (sucursal, IP dinámica con DDNS)
**Central (RV082, LAN 192.168.10.0/24):**
1. VPN > Gateway to Gateway. Tunnel Name "Sucursal1", Interface WAN1.
2. Local: Security Gateway Type **IP Only**; Security Group **Subnet** 192.168.10.0/255.255.255.0.
3. Remote: Security Gateway Type **Dynamic IP + Domain Name (FQDN) Authentication**, Domain Name `sucursal1.ddns.example` (o Dynamic IP + E-mail si no hay DDNS); Security Group **Subnet** 192.168.20.0/255.255.255.0.
4. IPSec: IKE with Preshared Key; F1 Group 5 / AES-256 / SHA1 / 28800; PFS ✓; F2 Group 5 / AES-256 / SHA1 / 3600; PSK fuerte (≤30).
5. Advanced: Aggressive (forzado), NAT Traversal ✓ (si la sucursal está detrás de NAT), DPD ✓ (p. ej. 10–30 s), Keep-Alive opcional. Save.

**Sucursal (RV042, LAN 192.168.20.0/24):**
1. Setup > Dynamic DNS configurado con el hostname.
2. Local: **Dynamic IP + Domain Name (FQDN) Authentication** con el mismo dominio; Subnet 192.168.20.0/24.
3. Remote: **IP Only** → IP Address = IP fija de la central; Subnet 192.168.10.0/24.
4. IPSec idéntico; PSK idéntica.
5. Advanced: Aggressive ✓, NAT-T ✓ (igual que central), DPD ✓ mismo intervalo, **Keep-Alive ✓** (para que la sucursal inicie y reconecte). Save.
6. VPN > Summary > Connect en la sucursal; verificar "Connected" y ping cruzado.

### 11.2 Túnel con failover WAN1→WAN2 (RV082 central con dos ISP)
1. Configurar el túnel principal por **WAN1** (§11.1).
2. Advanced: **DPD** ✓ + intervalo; **Tunnel Backup** ✓; Remote Backup IP Address = IP de respaldo del par (o la misma); Local Interface = **WAN2**; VPN Tunnel Backup Idle Time = 30. Save.
3. System Management > Dual WAN > **Smart Link Backup**, Primary WAN = WAN1; Network Service Detection ✓ con **When Fail = Remove the Connection**. Save.
4. En el par remoto: debe aceptar el túnel cuando llegue desde la IP de WAN2 de la central → usar identidades no ligadas a IP (FQDN/USER FQDN + Aggressive) o un segundo túnel/Remote Backup IP apuntando a WAN2. [Inf]

### 11.3 QuickVPN para un usuario Windows 7
1. Router: LAN distinta de 192.168.1.x; Firewall > General > **HTTPS ✓**.
2. VPN > VPN Client Access: usuario, contraseña, Allow Password Change, **Active ✓**, Add to list, **Save** (aceptar o no el cambio automático de LAN IP).
3. (Opcional) Certificate Management > **Export for Client** → copiar `.pem` a `C:\Program Files\Cisco Small Business\QuickVPN Client\`.
4. PC: instalar QuickVPN **1.4.2.1** como administrador; Windows Firewall **ON**; IPSec Policy Agent activo; regla entrante ICMPv4.
5. Perfil: Server Address = IP/DDNS del RV082; Port = Auto; Use Remote DNS Server según necesidad; Connect.
6. Si "Remote gateway is not responding": §6.7.

### 11.4 PPTP para usuarios Windows 10/11 (cuando QuickVPN no funciona)
1. VPN > PPTP Server: **Enable PPTP Server ✓**; Range Start/End fuera del DHCP (p. ej. 192.168.10.200–204); añadir usuarios (máx. 5 conectados); Save.
2. Si hay módem delante: reenviar TCP 1723 + GRE (o DMZ/bridge).
3. Windows: Settings > Network & Internet > VPN > Add VPN: VPN provider Windows (built-in); Server = IP/DDNS; VPN type **Point to Point Tunneling Protocol (PPTP)**; Sign-in info usuario/contraseña. Conectar y comprobar en **Connection List**.
4. Para no enviar todo el tráfico por la VPN: Adaptador > Propiedades IPv4 > Avanzado > desmarcar "Use default gateway on remote network" y añadir ruta a la LAN remota. [Inf]

### 11.5 Cliente IPsec Shrew Soft / TheGreenBow (alternativa a QuickVPN, cifrado fuerte)
Seguir §9.9 (Group VPN, Aggressive, ID FQDN/USER FQDN, IP virtual fuera de ambas LAN, sin ModeCfg, NAT-T ✓, AES-256/SHA1/DH2-5).

### 11.6 RV082 ↔ equipo de otro fabricante (método general)
1. Elegir **un** conjunto de algoritmos: AES-256 / SHA1 / DH5 (o DH2) / PFS mismo grupo / 28800 / 3600, IKEv1 Main Mode, PSK.
2. Configurar el RV082 (Gateway to Gateway, IP Only en ambos lados si son IP fijas).
3. Configurar el otro equipo con **una sola** propuesta F1 y F2 idénticas, IKEv1, selector único = Local/Remote Group del RV082 (invertidos), sin lifetime por KB, exclusión de NAT, reglas de firewall, ruta.
4. NAT-T y DPD iguales.
5. Iniciar desde ambos lados (Tunnel Test > Connect en el RV) y leer logs (§10.2).

---

## 12. Datos sueltos útiles
- Throughput: NAT 200 Mbps; **IPsec 97 Mbps** (catálogo) [DS]. SmallNetBuilder (v3, fw 4.0.3.03-tm): ~70 Mbps 3DES; QuickVPN 69 Mbps (RV082) / 71 (RV016) / 37 (RV042). [3P]
- Puertos: LAN 8×10/100, WAN1 Internet 10/100, puerto **DMZ/Internet** (WAN2 o DMZ). [DS]
- Gestión segura: HTTPS, usuario/contraseña, complejidad de contraseña; SNMP v1/v2c; Bonjour. [DS]
- La rama de firmware 4.x añade IPv6 en WAN/LAN, pero los túneles IPsec documentados son **IPv4**. [AG] [Inf]
- Configuración exportable/importable (System Management > Backup and Restore); antes de un reset exportar también el **certificado de administrador** de QuickVPN (Export for Admin). [AG]
- El nombre de dominio de "IP + Domain Name"/"Dynamic IP + Domain Name" sólo puede usarse en **un** túnel. [AG]

---

## 13. Lagunas conocidas de este documento
- No se pudo leer el contenido completo de foros de Cisco Community (bloqueados); se usaron fragmentos de búsqueda → marcados [Com].
- **Valor por defecto y rango del DPD Interval**: no documentados (foros: 10 s o 30 s).
- **Rango permitido de SA Life Time**: no documentado.
- **Detalles de MPPE/autenticación del servidor PPTP**: no documentados en el AG.
- **Release notes del firmware 4.2.3.x**: no consultadas; no se confirma cuál es la última versión.
- Opciones exactas del desplegable "Port for QuickVPN" (Auto/443/60443): sólo "Auto" confirmado.
- Plantillas ASA/IOS/FortiGate/MikroTik/pfSense: construidas con sintaxis estándar de cada fabricante, **no probadas** contra un RV082; la compatibilidad de algoritmos débiles depende de la versión del otro equipo.

---

## 14. Fuentes

**Oficiales Cisco**
- Cisco Small Business RV0xx Series Routers Administration Guide (78-19576-01 B0): https://www.cisco.com/c/dam/en/us/td/docs/routers/csbr/rv0xx/administration/guide/rv0xx_AG_78-19576.pdf (espejo: https://www3-realm.cisco.com/en/US/docs/routers/csbr/rv0xx/administration/guide/rv0xx_AG_78-19576_310513.pdf)
- Data Sheet Cisco RV082 (C78-501227-05): https://www.cisco.com/c/dam/en/us/products/se/loc/data_sheet_c78-501227.pdf
- SMB3294 – Configuration of Gateway to Gateway VPN on RV016/RV042/RV042G/RV082: https://www.cisco.com/c/en/us/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb3294-configuration-of-gateway-to-gateway-vpn-on-rv016-rv042-rv042.html
- SMB568 – Configure Advanced Settings for Gateway to Gateway VPN: https://www.cisco.com/c/en/us/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb568-configure-advanced-settings-for-gateway-to-gateway-vpn-on-rv.html
- SMB3009 – Configuration of Backup VPN Tunnel (RV042/RV042G/RV082): https://www.cisco.com/c/en/us/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb3009-configuration-of-backup-virtual-private-network-vpn-tunnel-o.html
- SMB624 – Set Up a Remote Access Tunnel (Client to Gateway): https://www.cisco.com/c/en/us/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb624-set-up-a-remote-access-tunnel-client-to-gateway-for-vpn-clie.html
- SMB452 – Manage VPN Users and Configure Quick VPN: https://www.cisco.com/c/en/us/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb452-manage-vpn-users-and-configure-quick-vpn-on-rv016-rv042-rv04.html
- SMB2922 – Cisco QuickVPN Installation Tips for Windows: https://www.cisco.com/c/en/us/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb2922-cisco-quickvpn-installation-tips-for-windows-operating-syste.html
- SMB2931 – Alternative VPN Setup for Windows 8 on RV016/RV042/RV042G/RV082: https://www.cisco.com/c/es_mx/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb2931-alternative-vpn-setup-for-windows-8-on-the-rv016-rv042-rv042.html
- SMB3049 – Configuration of C2G with GreenBow software: https://www.cisco.com/c/en/us/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb3049-configuration-of-c2g-with-greenbow-software-on-rv016-rv042-r.html
- SMB3039 – Configuration of Shrew VPN Client on RV042/RV042G/RV082: https://www.cisco.com/c/en/us/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb3039-configuration-of-shrew-vpn-client-on-rv042-rv042g-and-rv082.html
- Application Note "Easy and Secure Remote Access with Cisco QuickVPN" (OL-25680-01): https://www.cisco.com/en/US/docs/routers/csbr/app_notes/QuickVPN_an_OL-25680.pdf
- Release Notes QuickVPN 1.4.1.2: https://www.cisco.com/c/dam/en/us/td/docs/routers/csbr/quickvpn/release/notes/QuickVPN_RN_v1-4-1-2.pdf
- Release Notes QuickVPN 1.2.11/1.2.8: https://www.cisco.com/c/dam/en/us/td/docs/routers/csbr/rv016/release/notes/QuickVPN_Client_v1-2-11_RN.pdf
- Release Notes QuickVPN 1.4.2.1 (sólo vía resumen de búsqueda): https://www3-realm.cisco.com/en/US/docs/routers/csbr/quickvpn/release/notes/quickvpn_relnote_1.4.2.1.pdf
- ASA – Troubleshoot Common L2L and Remote Access IPsec VPN Issues: https://www.cisco.com/c/en/us/support/docs/security/asa-5500-x-series-next-generation-firewalls/81824-common-ipsec-trouble.html

**Terceros**
- TheGreenBow – Configuration Guide Linksys RV082: https://www.thegreenbow.com/wp-content/uploads/2021/05/tgbvpn_cg_LinksysRV082_en.pdf
- Shrew Soft – Howto Linksys (RV042/RV082): https://www.shrew.net/support/Howto_Linksys
- SonicWall KB – Site to Site VPN between SonicWall and Linksys VPN Router: https://www.sonicwall.com/support/knowledge-base/how-can-i-configure-a-site-to-site-vpn-tunnel-between-a-sonicwall-and-linksys-vpn-router/kA1VN0000000MnE0AU
- SmallNetBuilder – Cisco RV082 and RV016 v3 VPN Routers Reviewed: https://www.smallnetbuilder.com/lanwan/lanwan-reviews/cisco-rv082-and-rv016-v3-vpn-routers-reviewed/
- Netgate Forum – IPSEC with pfSense and Linksys RV082: https://forum.netgate.com/topic/9514/ipsec-with-pfsense-and-linksys-rv082
- Netgate Forum – pfSense 2.2.2 -> Cisco RV042 (logs reales RV042): https://forum.netgate.com/topic/84331/pfsense-2-2-2-cisco-rv042
- Netgate docs – IPsec Phase 1: https://docs.netgate.com/pfsense/en/latest/vpn/ipsec/configure-p1.html
- MikroTik docs – IPsec: https://help.mikrotik.com/docs/spaces/ROS/pages/11993097/IPsec
- Fortinet CLI – vpn ipsec phase1-interface: https://docs.fortinet.com/document/fortigate/6.2.1/cli-reference/286620/vpn-ipsec-phase1-interface
- CPE RV082 firmware 4.2.3.14: https://cpe.gcve.eu/cpes/573969

**Foros Cisco Community (consultados vía extractos de búsqueda)**
- RV082 to ASA 5510 tunnel freezes after 8 hours: https://community.cisco.com/t5/routers-small-business/cisco-rv082-to-asa-5510-tunnel-freezes-after-8-hours/td-p/1937847
- RV082-RV082 no connection: https://community.cisco.com/t5/vpn/rv082-rv082-no-connection/td-p/1711219
- Does QuickVPN 1.4.2.1 support Windows 10: https://community.cisco.com/t5/routers-small-business/does-quickvpn-1-4-2-1-support-windows-10/m-p/3091371
- RV042 VPN issue "Remote gateway is not responding": https://community.cisco.com/t5/routers-small-business/cisco-rv042-vpn-issue-remote-gateway-is-not-responding-quick-vpn/td-p/4106706
- Ports used by QuickVPN: https://community.cisco.com/t5/small-business-routers/ports-used-by-quickvpn/td-p/1794101
- How to configure an RV0XX for VPN from iPhone iOS 10 (no PPTP): https://community.cisco.com/t5/routers-small-business/how-to-configure-an-rv0xx-router-for-vpn-access-from-an-iphone/td-p/2972198
- iOS and Cisco RV082: https://community.cisco.com/t5/small-business-routers/ios-and-cisco-rv082/td-p/2276855
- RV042 v3 not reconnecting dropped VPN tunnels: https://community.cisco.com/t5/routers-small-business/rv042-v3-not-reconnecting-dropped-vpn-tunnels/td-p/2137080
- RV042 no recovery of VPN after NAT-T new mapping: https://community.cisco.com/t5/routers-small-business/rv042-no-recovery-of-vpn-after-t-nat-new-mapping-entry/td-p/3364227
- RV042 Shrew Soft client NAT-T new mapping: https://community.cisco.com/t5/routers-small-business/rv042-shrew-soft-client-nat-t-new-mapping/td-p/2577410
- RV082 IPsec forwarding interferes with G2G VPN: https://community.cisco.com/t5/routers-small-business/rv082-ipsec-forwarding-interferes-with-gateway-2-gateway-vpn/td-p/1879538
- RV082 firmware upgrade (HW v3 requerido): https://community.cisco.com/t5/routers-small-business/rv082-firmware-upgrade/td-p/2700384
- VPN tunnel dropping connection (logs STATE_MAIN_I1): https://community.cisco.com/t5/vpn/vpn-tunnel-dropping-connection/td-p/1966259
- Cisco <> MikroTik site-to-site: https://community.cisco.com/t5/vpn/cisco-lt-gt-mikrotik-site-to-site-ipsec-tunnel/td-p/2257550
- Sophos UTM to RV042 ("No acceptable Oakley Transform"): https://community.sophos.com/utm-firewall/f/vpn-site-to-site-and-remote-access/55423/utm-to-rv042-router-vpn
- RV042 "doesn't allow any IKE tunnels" (LinksysInfo): https://www.linksysinfo.org/index.php?threads/rv042-doesnt-allow-any-ike-tunnels.24141/
- Configure a VPN tunnel that routes all traffic to remote gateway: https://community.cisco.com/t5/small-business-support-knowledge-base/configure-a-vpn-tunnel-that-routes-all-traffic-to-remote-gateway/ta-p/3125787
