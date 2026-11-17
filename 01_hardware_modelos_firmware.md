# Cisco/Linksys RV082 (10/100 8-Port VPN Router, Dual WAN): hardware, modelos/versiones y firmware

> Documento de referencia autosuficiente para consultar sin internet. Investigado el 2 de noviembre de 2026.
> **Niveles de confianza:**
> - **[ALTA]**: documento oficial de Cisco/Linksys (datasheet, admin guide, release notes, aviso EoL o advisory) leído directamente.
> - **[MEDIA]**: wiki técnica (DeviWiki/WikiDevi), review de SmallNetBuilder o varias fuentes comunitarias que coinciden.
> - **[BAJA]**: un solo post de foro, sitio de terceros o inferencia propia.
>
> Cuando las fuentes se contradicen, se muestran ambas versiones.

---

## 0. Resumen rápido

| Aspecto | Dato |
|---|---|
| Nombre comercial | Linksys "10/100 8-Port VPN Router" (RV082), luego "Cisco Small Business RV082 Dual WAN VPN Router" (también "Cisco RV082 10/100 8-port VPN router") |
| Puertos | 8 LAN 10/100, 1 Internet (WAN1) 10/100 y 1 "DMZ/Internet" 10/100, que sirve como WAN2 o como DMZ |
| Versiones de hardware | **v1.x** (Intel IXP425, Linksys), **v2** (Intel IXP425, cambia el chip de switch) y **v3** (Cavium Octeon CN5020, Cisco) |
| Firmware final | v1/v2: **2.0.2.01-tm** (15-jun-2011). v3: **4.2.3.14** (17-jun-2020) |
| Valores por defecto | `192.168.1.1`, usuario `admin` y clave `admin`, DHCP 192.168.1.100 a .149, máscara 255.255.255.0 |
| Reset | Pulsar 1 s reinicia. Mantener **30 s** restaura los valores de fábrica |
| Rendimiento oficial | NAT 200 Mbps, IPsec 97 Mbps, 100 túneles IPsec, 50 QuickVPN y 5 PPTP |
| Fin de vida | Anuncio EoL **4-feb-2016** (EOL10819); fin de venta 5-may-2016; último día de soporte **31-may-2021** según el aviso (5-may-2021 según otra página de Cisco) |
| Reemplazo | Cisco dijo "no replacement available" y recomendó migrar al **RV325** (que a su vez está EoL, con reemplazo RV345) |
| Seguridad | Varias vulnerabilidades críticas posteriores a 2020 (RCE y bypass de autenticación) que **Cisco no corregirá**. No conviene exponer la interfaz web a Internet |

---

## 1. Historia y marca

### 1.1 Linksys → Cisco Small Business
- **Linksys** (división de Cisco desde 2003) lanzó el RV082 como "Linksys 10/100 8-Port VPN Router, Model RV082". Network Computing publicó su lanzamiento el **9 de enero de 2004**. La review de SmallNetBuilder probó el firmware **1.0.11 del 16-ene-2004**, así que el lanzamiento fue entre fines de 2003 y principios de 2004. **[MEDIA]** Un artículo de TechCentral de abril de 2005 todavía lo trata como novedad en algunos mercados.
- Precio en calle en la época de la primera review: unos **275 USD**. **[MEDIA]**
- Hacia 2007-2008 Cisco movió los productos "business" de Linksys a la marca **Cisco Small Business**. Cisco da como "Series Release Date" de la serie RV el **25-nov-2008** **[ALTA]**. Desde entonces el equipo se vende como "Cisco RV082 Dual WAN VPN Router". Las unidades de transición llevan el logo "Linksys, a Division of Cisco" o "Linksys by Cisco".
- Cuando Cisco vendió Linksys a Belkin (2013), la familia RV0xx se quedó en Cisco. Algunos revendedores muestran "BELKIN RV082-EU" por error de catálogo, pero no es un producto Belkin. **[BAJA/inferencia]**
- **Números de parte (SKU)** según el datasheet de Cisco **[ALTA]**:

| SKU | Mercado |
|---|---|
| RV082 | EE.UU., Canadá, Brasil y Argentina |
| RV082-EU | Europa y Asia |
| RV082-UK | Reino Unido y Singapur |
| RV082-AU | Australia y Nueva Zelanda |
| RV082-CN | China |
| RV082-RF | Reacondicionado (aparece en brokers; no figura en el datasheet) **[BAJA]** |

### 1.2 Versiones de hardware (v1, v2, v3)

| Versión | Época aprox. | Marca típica | SoC / CPU | RAM | Flash | Switch / PHY | Firmware |
|---|---|---|---|---|---|---|---|
| **v1 / v1.x** | 2003/2004 a ~2008 | Linksys (luego "Linksys by Cisco") | Intel XScale **IXP425**. DeviWiki indica **533 MHz**; SmallNetBuilder dice **266 MHz** ("IXP425-266") | **32 MB** (2 × Mira P2V28S40BTP) | **16 MB** (Intel TE28F128J3C150, NOR) | Infineon/ADMtek **ADM6996L** (y "ADM6999" según DeviWiki) | 1.0.x → 1.3.x → 2.0.0.19-tm → **2.0.2.01-tm** |
| **v2** | ~2008 a 2010 | Linksys/Cisco | Intel IXP425 (mismo diseño base) | 32 MB **[MEDIA]** | 16 MB **[MEDIA]** | **ADM6996I** (las release notes oficiales confirman el cambio L → I) | Mínimo práctico 2.0.0.19-tm; final **2.0.2.01-tm** |
| **v3** | desde ~ago-2010 | Cisco Small Business | **Cavium Octeon CN5020** (familia CN50XX), **2 núcleos a 300 MHz**, MIPS64 | **256 MB** | **32 MB** | Realtek **RTL8309G** + **RTL8306SD** | Solo **4.x** (4.0.0.07 → **4.2.3.14**) |

Notas y contradicciones:
- **Velocidad del IXP425 en v1/v2:** DeviWiki y la ficha de SpeedGuide dicen 533 MHz. SmallNetBuilder (review de la v3) dice que el RV016/RV082 original usaba un "Intel IXP425-266". El RV042 v1 sí figura con IXP425 a 266 MHz en el wiki de DD-WRT. **No se pudo confirmar** cuál es la cifra correcta para el RV082. La página *System Summary* del firmware 1.x/2.x muestra CPU, DRAM y Flash y sirve para verificarlo en una unidad concreta. **[MEDIA, contradictorio]**
- Un usuario del foro de Cisco afirmó que su RV082 V1 mostraba **128 MB** de RAM en la consola web. Esto contradice los 32 MB de las demás fuentes y queda como **[BAJA]**.
- Otros chips del v1 según DeviWiki **[MEDIA]**: RTC **Ricoh RS5C372A**, supervisor de voltaje TI, lógica 74LVC04A/74LVC374A y país de fabricación Taiwán. La Admin Guide de Linksys tiene un apéndice "Battery Replacement" porque el v1 lleva una **pila de litio CR2032** para el reloj. Dura de 1 a 2 años; cuando se agota, el router pierde la hora salvo que use NTP. Linksys pedía no cambiarla uno mismo (riesgo de explosión si se reemplaza mal). **[ALTA, guía Linksys]**
- La **v3** según SmallNetBuilder: la flash se duplica (16 → 32 MB) y la RAM se multiplica por 8 (32 → 256 MB) frente al RV082 original. El `/proc/cpuinfo` de una v3 muestra "Cavium Networks Octeon CN50XX V0.1", system type "CN3010_EVB_HS5" (foro OpenWrt). **[MEDIA]**
- La v3 **perdió la función "dual-image"** (dos imágenes de firmware) que tenían el RV082 y el RV016 v1/v2. Según las release notes 4.0.2.08: "Hardware Version 3 … supports all the features of earlier versions… except for the dual-image feature of RV082/RV016". **[MEDIA, cita de release notes vista en un snippet]**
- La v3 cambió la interfaz web y la forma de manejar múltiples subredes LAN: en un RV082 v3 con NAT, las subredes LAN de otro router requieren una regla de acceso. Así lo indican las release notes 4.2.1.02. **[ALTA]**

### 1.3 Cómo identificar la versión de hardware
1. **Etiqueta inferior** (código de barras, junto al número de serie): el campo **PID VID** muestra, por ejemplo, **`RV082 V03`** en una v3. El mismo campo aparece en la *System Summary* del firmware 4.x, que lista "Serial Number, Firmware version, **PID VID**, MD5 Checksum…". **[ALTA para la v3]**
2. Según la convención general de Linksys, si no hay número de versión junto al modelo, la unidad es **v1**. **[MEDIA, regla genérica de Linksys]**
3. **Por firmware:** si acepta 4.x y la interfaz es la "nueva" de Cisco, es v3. Si corre 1.x o 2.x, es v1 o v2. En un v1/v2 que no es v3, el intento de cargar 4.x "no hace nada" o falla. Hay casos reportados de unidades **brickeadas** al flashear 4.0.0.7 en hardware v1/v2 (DSLReports). **[MEDIA]**
4. **Por la caja:** SmallNetBuilder señala que la caja de la v3 **no indica** la versión. Solo la etiqueta de serie la identifica. **[MEDIA]**
5. **Por la marca:** las v1 llevan logo Linksys y algunos listados las describen como carcasa "plateada". Las v3 son Cisco Small Business. **No se encontró una fuente fiable** que asocie color o logo con v1/v2 de forma inequívoca. **[BAJA]**
6. **Prefijos de número de serie:** **no se encontró** documentación que asocie prefijos de serie a versiones de hardware.

---

## 2. Especificaciones físicas y eléctricas

### 2.1 Tabla general

| Parámetro | Valor | Fuente / confianza |
|---|---|---|
| Dimensiones (An × Al × Pr) | **279,4 × 44,45 × 241,3 mm** (11,00 × 1,75 × 9,50 in) | Datasheet Cisco y Admin Guide **[ALTA]** |
| Formato | Metálico, **1U** para rack de 19", con orejas (brackets) incluidas | Admin Guide/QSG **[ALTA]** |
| Peso | **1,475 kg (3,25 lb)** según Cisco; 1,47 kg (52 oz) según la guía Linksys v1; "3,3 lbs" en la comparativa Cisco | **[ALTA]** |
| Alimentación | **Fuente interna**, AC **100-240 V, 50-60 Hz**, cable de red IEC (inlet C14, cable C13) | Datasheet/Admin Guide **[ALTA]** |
| Consumo / corriente | v1: entrada 100-240 VAC **0,4 A**, salida interna **3,3 V DC 3 A** (guía Linksys). v3: **0,8 A** (DeviWiki) | **[ALTA]** v1, **[MEDIA]** v3 |
| Ventilador | No mencionado en ninguna fuente. Los RV082 son de refrigeración pasiva **[BAJA, no confirmado]** | La garantía Cisco menciona "fans and power supplies" de forma genérica |
| Temperatura de operación | **0 a 40 °C** (32-104 °F) | **[ALTA]** |
| Temperatura de almacenamiento | 0 a 70 °C (32-158 °F) | **[ALTA]** |
| Humedad de operación | 10 % a 85 % sin condensación | **[ALTA]** |
| Humedad de almacenamiento | 5 % a 90 % sin condensación | **[ALTA]** |
| Certificaciones | Datasheet 2011/2013: **FCC clase B, CE clase A, cUL, Anatel, MEPS, CCC, PSB, CB, C-Tick, KCC**. Admin Guide: FCC Class B, CE Class A. Guía Linksys v1: FCC Class B, **CE Class B** | **[ALTA]**, aunque la clase CE difiere según el documento o la versión |
| Cableado | Categoría 5 Ethernet | **[ALTA]** |
| Estándares | IEEE 802.3, 802.3u | **[ALTA]** |
| Ranura de seguridad | Ranura tipo Kensington en el lateral | Admin Guide **[ALTA]** |
| MTBF | **No publicado.** No se encontró ninguna cifra oficial ni de terceros | — |
| Garantía (Cisco) | *Limited lifetime hardware warranty* con reemplazo de vuelta a fábrica, 1 año para ventiladores y fuentes, 12 meses de soporte telefónico gratuito y bug fixes durante la garantía. Tras el EoL, el soporte terminó en 2021 | Datasheet **[ALTA]** |
| Sistema operativo | "Linux" (Admin Guide, especificaciones) | **[ALTA]** |

### 2.2 Montaje
- **Escritorio:** superficie plana, sin nada encima y con buena ventilación.
- **Pared:** el equipo tiene dos ranuras en la base, separadas **94 mm** (RV082/RV016). Tornillos sugeridos: cabeza de 6,5-7 mm y longitud de 16,5-18,5 mm. Hay que dejar una separación de 1 a 1,2 mm entre la cabeza del tornillo y la pared. Los orificios de disipación deben quedar mirando hacia los lados. La tornillería no viene incluida. **[ALTA]**
- **Rack de 19":** ocupa 1 RU (44,45 mm). Se atornilla un bracket por lado con los tornillos suministrados; los tornillos para el rack no vienen incluidos. No conviene sobrecargar el circuito eléctrico del rack. **[ALTA]**

### 2.3 Panel frontal y trasero
- **Frontal:** LEDs (System, DIAG, Internet, DMZ/Internet, DMZ Mode, 1-8), **8 puertos LAN**, puerto **Internet**, puerto **DMZ/Internet** y botón **Reset**. En el RV082 y el RV016 el reset está en el frente, cerca de los puertos Internet y DMZ; en el RV042 está atrás. **[ALTA]**
- **Trasero:** conector de alimentación AC. **[ALTA]**

### 2.4 LEDs y su significado

| LED | Color | Estado → significado |
|---|---|---|
| **System** | Verde | Fijo: equipo encendido. Parpadeando: corriendo un test de diagnóstico |
| **DIAG** | Rojo (guía Linksys) o naranja/ámbar (foros y docs Cisco) | Encendido: el router se está preparando (arranque, no listo). Apagado: listo para usar. **Parpadeo lento**: modo de recuperación/bootloader (ver §6.5). Encendido durante un reset de fábrica de 30 s o mientras graba la flash tras un TFTP |
| **Internet** | Verde | Fijo: dispositivo conectado al puerto Internet. Parpadeando: actividad |
| **DMZ/Internet** | Verde | Fijo: dispositivo conectado (módem como WAN2, o switch/servidor como DMZ). Parpadeando: actividad |
| **DMZ Mode** | Verde | Encendido: el puerto DMZ/Internet está en modo **DMZ**. Apagado: está como **segunda WAN** |
| **1-8** | Verde | Fijo: enlace en ese puerto LAN. Parpadeando: tráfico |

Fuentes: Admin Guide Cisco RV0xx, QSG RV0xx y User Guide Linksys RV082 **[ALTA]**. Durante un reinicio por botón (1 s), todos los LEDs salvo System se apagan, y el equipo está listo cuando vuelve el LED System (artículo Cisco smb288) **[ALTA]**. Un DIAG rojo **parpadeando durante el arranque** tras una actualización fallida indica firmware dañado (blog socc200, en un RV042 v3) **[BAJA/MEDIA]**.

### 2.5 Botón Reset

| Acción | Tiempo | Efecto | Fuente |
|---|---|---|---|
| Reinicio (*restart*) | **~1 segundo** con la punta de un bolígrafo o clip | Reinicia como apagar y encender; conserva la configuración guardada | Admin Guide, QSG, User Guide Linksys y smb288 **[ALTA]** |
| Reset de fábrica | Mantener **30 segundos** | Borra toda la configuración (incluidas la contraseña y los port forwards) y vuelve a admin/admin y 192.168.1.1. El DIAG se enciende en naranja durante el proceso | Las mismas **[ALTA]** |
| Si el reset de 30 s no funciona | — | Desenchufar y volver a enchufar **manteniendo** Reset presionado | smb288 **[ALTA]** |
| Modo recuperación (v3) | Mantener Reset **mientras se conecta la corriente**, de ~5 a 15+ s, hasta que DIAG **parpadee lento** | Arranca en bootloader. Responde ping en 192.168.1.1 y acepta firmware por **TFTP** | Foros Cisco y blog socc200 **[MEDIA]** |

Algunos foros mencionan 10 s para el reset, pero toda la documentación oficial dice **30 s**.

Alternativas por software: *System Management > Restart* (Cisco recomienda este método porque envía el log antes de reiniciar) y *System Management > Factory Default*. **[ALTA]**

---

## 3. Puertos, switch y conectividad

- **8 × LAN 10/100 RJ-45** (switch integrado full-duplex). Desde fábrica todos están en la **VLAN 1**. En la v3 se puede segmentar por VLAN basada en puerto: cada puerto pertenece a una sola VLAN, no hay etiquetado 802.1Q y los puertos WAN/DMZ no pueden ir en VLAN (SmallNetBuilder). Auto-negociación activa por defecto. **[ALTA/MEDIA]**
- **1 × Internet (WAN1) 10/100 RJ-45.**
- **1 × DMZ/Internet 10/100 RJ-45.** Funciona como **WAN2** (dual WAN con *load balancing* o *Smart Link Backup*/failover) o como **puerto DMZ** físico. El LED "DMZ Mode" indica el modo.
- **Todos los puertos son Fast Ethernet (100 Mbps máx.)**, por lo que el router nunca supera unos 95 Mbps por enlace WAN. **[ALTA]**
- **Chips de switch:** v1 ADM6996L, v2 ADM6996I, v3 RTL8309G + RTL8306SD. **[ALTA/MEDIA]**
- No tiene USB, Wi-Fi, módem integrado, puerto de consola externo ni ranuras de expansión. **[ALTA]**

---

## 4. Rendimiento y capacidades

### 4.1 Cifras oficiales (datasheet Cisco C78-501227-05, 07/2013, y Admin Guide RV0xx) **[ALTA]**

| Métrica | Valor oficial |
|---|---|
| NAT throughput | **200 Mbps** (agregado; cada puerto es 10/100) |
| IPsec VPN throughput | **97 Mbps** |
| Túneles IPsec site-to-site (gateway-to-gateway) | **100** |
| Túneles QuickVPN (cliente-a-gateway con el cliente Cisco QuickVPN) | **50** usuarios |
| Túneles PPTP (servidor PPTP integrado) | **5** clientes |
| Cifrado | DES, 3DES, AES-128, AES-192 y AES-256 |
| Autenticación | MD5, SHA1 |
| IKE / NAT-T / DPD | IKE, IPsec NAT-Traversal (gateway-to-gateway y client-to-gateway), Dead Peer Detection, split DNS y VPN backup |
| VPN pass-through | PPTP, L2TP, IPsec |
| Access rules (firewall) | Hasta **50** reglas |
| Port forwarding | Hasta **30** entradas |
| Port triggering | Hasta **30** entradas |
| Sesiones concurrentes | **No publicado** por Cisco (la comparativa Cisco deja la fila vacía para el RV082) |
| QoS | Por servicio (rate control o prioridad, con 3 niveles) y por puerto LAN |
| Firewall | SPI; protección DoS (SYN flood, ping of death, LAND, IP spoofing, Smurf); alerta por email |
| Bloqueo | Java, cookies, ActiveX y proxy HTTP; filtrado de URL estático por dominio o palabra clave. En v1/v2 también filtrado dinámico opcional con **Cisco ProtectLink Web** (Trend Micro) |
| Gestión | HTTP/HTTPS, SNMP v1/v2c, Bonjour, Syslog, alertas por email y monitor de túneles VPN |
| Routing | Estático y RIP v1/v2. Modos *Gateway* (NAT) y *Router* |
| WAN | DHCP, IP estática, PPPoE, PPTP y transparent bridge; DDNS (DynDNS, 3322); protocol binding a una WAN concreta |
| Autenticación externa | RADIUS (según la comparativa Cisco) **[MEDIA]** |
| IPv6 | Solo en firmware **4.x (v3)**: Dual-Stack IP, DHCPv6-PD, 6to4 y RA. SmallNetBuilder indica que requiere la 4.0.3.03 o superior **[MEDIA]** |

### 4.2 Discrepancias históricas
- En la época Linksys v1 (2004), SmallNetBuilder describe el RV082 como endpoint de **50 túneles IPsec**, con solo **DES/3DES**, MD5/SHA1 y DH groups 1, 2 y 5. La especificación de la guía Linksys v1 dice "DES and 3DES encryption for IPSec". **AES se añadió por firmware** en algún punto de la rama 1.x; la versión exacta no está confirmada. El datasheet Cisco posterior (2011/2013) dice **100 túneles** y AES-256. **[MEDIA]**
- La guía Linksys habla de "up to 50 remote office or traveling users". CDW habla de "100 off-site connections". **[MEDIA/BAJA]**

### 4.3 Mediciones independientes

| Fuente | HW/FW | WAN→LAN | LAN→WAN | Simultáneo | IPsec |
|---|---|---|---|---|---|
| SmallNetBuilder (2004) | v1, FW 1.0.11 | 19,8 Mbps | 22,2 Mbps | — | Igualó a NetScreen 5XP (13 Mbps) y 5GT (20 Mbps) sin ser cuello de botella |
| SmallNetBuilder (~2010/2011) | **v3**, FW 4.0.3.03-tm | **93 Mbps** | **94 Mbps** | **149 Mbps** | QuickVPN 3DES/MD5: **~69 Mbps** |

- En la v3, el enrutamiento llega prácticamente a *wire speed* de 100 Mbps. El RV042 v3 no alcanzó el mismo throughput simultáneo (91 Mbps).
- En el test de conexiones simultáneas de SmallNetBuilder, el RV082/RV016 v3 con aquel firmware se detuvo tras "unas pocas docenas" de conexiones UDP, mientras que el RV042 v3 con firmware anterior completó el test. No se encontraron datos oficiales de sesiones. **[MEDIA]**
- La v1 rendía mucho menos en ruteo (~20 Mbps) con el firmware de 2004. Firmwares 1.x posteriores probablemente mejoraron, pero **no hay mediciones publicadas**. **[MEDIA]**
- La v3 usa la aceleración criptográfica del Cavium. El datasheet habla de "hardware encryption". La review v1 menciona un "VPN co-processor" sin nombrarlo (el IXP425 tiene motor criptográfico en algunas variantes). **[MEDIA/BAJA]**

---

## 5. Familia RV0xx y reemplazos

### 5.1 Comparativa de la familia RV0xx (cifras oficiales Cisco salvo indicación)

| Modelo | LAN | WAN | NAT | IPsec thr. | Túneles IPsec | QuickVPN | PPTP | Alimentación | Tamaño / peso | HW v3 (CPU/RAM/Flash/switch) |
|---|---|---|---|---|---|---|---|---|---|---|
| **RV042** | 4 × 10/100 | 1 + 1 DMZ/WAN (10/100) | 100 Mbps | 59 Mbps | 50 | 50 | 5 | Externa 12 V DC 1 A | 130 × 38,5 × 200 mm; 0,576 kg | Cavium CN5010 300 MHz / 128 MB / 32 MB / RTL8309G (v1: IXP425 266 MHz y 32 MB, según el wiki de DD-WRT) |
| **RV042G** | 4 × GbE | 1 + 1 DMZ/WAN (GbE) | 800 Mbps | 75 Mbps | 50 | 50 | 5 | Externa 12 V 1 A | Igual que el RV042 | Cavium (no verificado) |
| **RV082** | **8 × 10/100** | 1 + 1 DMZ/WAN (10/100) | **200 Mbps** | **97 Mbps** | **100** | **50** | **5** | **Interna AC 100-240 V** | 279 × 44 × 241 mm (1U); 1,475 kg | **Cavium CN5020 2×300 MHz / 256 MB / 32 MB / RTL8309G+RTL8306SD** |
| **RV016** | 8 LAN + 5 configurables LAN/WAN (16 puertos 10/100 en total) | 2 WAN dedicadas + hasta 5 más (máx. **7 WAN**) + 1 DMZ | 200 Mbps | 97 Mbps | 100 | 50 | **10** | Interna AC | Igual que el RV082 (1U) | Cavium CN5020 300 MHz / 256 MB / 32 MB / RTL8329M (v1: IXP425-266 / 64 MB / 16 MB) |

- El RV082 y el RV016 comparten chasis (1U, fuente interna) y firmware 4.x. Las cifras de NAT e IPsec del RV016 son las mismas que las del RV082.
- Firmware 4.x: un mismo paquete o *release* cubre el RV016, RV042, RV042G y RV082 v3. Las release notes son conjuntas ("RV0xx"). Cada modelo descarga su propia imagen desde su página de Cisco; **no se confirmó** si las imágenes son idénticas entre modelos. **[MEDIA]**

### 5.2 Fechas de fin de vida de la familia

| Modelo | Aviso EoL | Fin de venta | Fin de SW maint. | Último día de soporte | Reemplazo sugerido por Cisco |
|---|---|---|---|---|---|
| **RV082** | **4-feb-2016** (EOL10819) | **5-may-2016** | 5-may-2016 | **31-may-2021** (aviso). La página de soporte Cisco dice **05-may-2021** | "No replacement available"; migrar al **RV325** |
| RV016 | — (no se encontró el aviso global) | 20-ene-2016 (página "obsolete") | — | 31-ene-2021 | — (el RV016-CN tuvo su propio EoL en 2010, con reemplazo RV042-CN) |
| RV042 / RV042G | 31-oct-2019 (EOL13238) | 30-ene-2020 | 29-ene-2021 | 31-ene-2025 | **RV340** |
| RV320 / RV325 | 31-oct-2019 (EOL13240) | 30-ene-2020 | 29-ene-2021 | 31-ene-2025 según el aviso. La página de soporte dice que el soporte terminó el **31-jul-2021** (30-sep-2021 para las variantes WF) | RV320 → **RV340**; RV325 → **RV345** |
| RV340 / RV345 (y RV160/RV260) | 2021 (avisos del 15-sep-2021 y 28-oct-2021) | 28-oct-2021 (serie) | — | **31-oct-2026** | — (a la fecha de este documento, 2-nov-2026, también fuera de soporte) |

Detalle completo del aviso del RV082 (EOL10819) **[ALTA]**:

| Hito | Fecha |
|---|---|
| End-of-Life Announcement | 4 de febrero de 2016 |
| End-of-Sale | 5 de mayo de 2016 |
| Last Ship Date | 3 de agosto de 2016 |
| End of SW Maintenance Releases | 5 de mayo de 2016 |
| End of Phone Support | 5 de mayo de 2017 |
| Last Date of Support | 31 de mayo de 2021 |
| SKUs afectados | RV082, RV082-AU, RV082-CN, RV082-EU, RV082-UK |

Aunque el "End of SW Maintenance" era mayo de 2016, Cisco publicó después parches de seguridad para el RV082 (4.2.3.08, .09, .10 y .14) hasta junio de 2020. Ver §6.

### 5.3 Los "reemplazos" RV320 y RV325

| | RV320 | RV325 |
|---|---|---|
| Puertos | 4 × GbE LAN + 2 × GbE WAN (1 WAN + 1 WAN/DMZ) | **14 × LAN** (GbE) + 2 × GbE WAN |
| CPU / RAM / Flash | Cavium CN5020 2×300 MHz / 256 MB DDR2 / 64 MB (SmallNetBuilder) | Similar (no verificado) |
| Switch | Broadcom BCM53125M | — |
| NAT throughput | 900 Mbps | 900 Mbps |
| IPsec throughput | 100 Mbps (otra página de Cisco dice 75 Mbps) | 100 Mbps (75 según la misma página) |
| Sesiones concurrentes | 20 000 | 40 000 |
| Túneles | 50 IPsec (site-to-site + client), ~30 Easy VPN, 10 SSL VPN, 10 PPTP (SmallNetBuilder) | Al menos los del RV320 (cifra exacta no verificada) |
| Medido (SmallNetBuilder) | 887 Mbps WAN→LAN; IPsec site-to-site ~39-55 Mbps | — |

Ambos son EoL y tienen vulnerabilidades sin parche (comparten advisories con el RV082). El **RV345** es el sucesor directo del RV325 y el reemplazo natural de un RV082 por cantidad de puertos LAN (16 LAN GbE). Su soporte también terminó el 31-oct-2026. **No se investigó** qué recomienda Cisco hoy en lugar de la serie RV.

---

## 6. Firmware

### 6.1 Arquitectura de software interna

| | v1 / v2 (Intel IXP425) | v3 (Cavium Octeon) |
|---|---|---|
| SO | Linux **2.4.19** con **Jungo OpenRG** (DeviWiki) **[MEDIA]**. Hay comandos OpenRG documentados para el RV042 de la misma generación (`rg_conf_set`, `flash_commit`) **[MEDIA]** | Linux **2.6.16.26** (OSD Cisco de la 4.2.3.x) **[ALTA]** |
| Bootloader | Probablemente **RedBoot**: un hilo de OpenWrt reporta RedBoot por serial en un RV082 rev 2 **[BAJA]** | **U-Boot 1.1.1** (Cisco) **[ALTA]**. El U-Boot de Cisco pasa `MEM=2048` como parámetro de arranque (parche LEDE `170-cisco-hack.patch`) **[MEDIA]** |
| Sistema de archivos | — | CramFS (incluye "CramFS Tools 1.1") **[ALTA, inferido del OSD]** |
| Paquetes clave (OSD 4.2.3.x) | — | busybox 1.2.1, **strongSwan 4.0.4**, **ipsec-tools 0.6.5** (racoon), iptables 1.3.7, iproute2 2.6.10, **OpenSSL 1.0.2s**, **dnsmasq 2.80**, PPP 2.4.3, pptpd 1.3.4, net-snmp 5.1.4.2, mDNSResponder 107.6 (Bonjour), libupnp 1.6.21, radvd 1.6, dhcp 4.1.1-P1, dhcpcd 1.3.18, udhcp 0.9.7, zebra 0.93a (RIP), samba 3.0.22, stunnel 4.05, ntpclient, msmtp 1.4.21, sysklogd 1.5, tftp-hpa 0.48, mtd 20050122, zlib 1.2.3, bind 9.5.0, openldap 2.3.24, krb5 1.3.1, naptd 0.4.2, pim6dd 0.2.1.0, ipupdate 3.0.11b7 **[ALTA]** |

### 6.2 Historial de versiones conocidas

**Rama 1.x/2.x (hardware v1 y v2, IXP425):**

| Versión | Fecha | Notas | Confianza |
|---|---|---|---|
| 1.0.11 | 16-ene-2004 | Firmware de la review de SmallNetBuilder. La actualización conservó la config y el reboot tardó unos 45 s | MEDIA |
| 1.1.x (p. ej. 1.1.6.14) | ~2005-2006 | Las RN de la 2.0.0.19 mencionan configs exportadas "from firmware 1.1.6.14 or older" | ALTA (existencia) |
| 1.3.x (1.3.3.5, 1.3.5…, **1.3.9.8-tm**) | ~2006-2008 | La 1.3.9.8-tm es la base sobre la que se listan los cambios de la 2.0.0.19. Existen RN oficiales `RV082_firmware_v1.3.9.8-tm_RN.pdf`, que no se pudieron leer | ALTA (existencia) |
| **2.0.0.19-tm** | **Julio 2009** | Ver detalle abajo | ALTA |
| **2.0.2.01-tm** | **15-jun-2011** | **Última** versión para v1 y v2 | ALTA |

Cambios de la **2.0.0.19-tm** respecto de la 1.3.9.8-tm (release notes OL-20330-01) **[ALTA]**:
1. Soporta hardware **v2 (ADM6996I)** y **v1.x (ADM6996L)**.
2. **Impide el downgrade de 2.x.x a 1.x.x en hardware v2.**
3. Permite configurar el nombre de usuario administrador (Setup > Password).
4. Corrige: VPN intermitente, registro de logins no autorizados, ProtectLink al importar configs viejas (≤1.1.6.14), verificación del certificado SSL del servidor de licencias ProtectLink, DHCP que no asignaba a todos los clientes, failover dual-WAN con SIP, fechas de horario de verano y Split DNS sobre VPN.
5. Seguridad: puerto origen DNS aleatorizado por NAT (mitigación Kaminsky 2008), límite de solicitudes QuickVPN (anti-DoS), bloqueo de 5 minutos tras 5 errores de contraseña QuickVPN en 1 minuto y **SSLv2 deshabilitado**.
6. Log de aviso cuando la cola de URL de ProtectLink está casi llena.
7. Problemas conocidos: QuickVPN 1.2.8 con Wi-Fi en Vista (corregido en el cliente 1.2.11); Vista requiere el Firewall de Windows activo para QuickVPN; XP SP2 descarta ICMP (KB889527). **El puerto 60443 (QuickVPN) y el 1723 (PPTP) no se pueden usar para gestión remota.**

Cambios de la **2.0.2.01-tm** respecto de la 2.0.0.19-tm (release notes OL-25268-01) **[ALTA]**:
- Si HTTPS está habilitado (Firewall > General), las peticiones HTTP de administración desde la LAN se **redirigen a HTTPS**. La gestión remota usa `https://<IP WAN>:[puerto]`.
- Permite **importar un certificado SSL externo** para gestión remota y QuickVPN. Si la clave está cifrada, pide la contraseña; al exportarlo, la contraseña pasa a ser `password`.
- Motor de filtrado de URL multihilo para Cisco ProtectLink Web.
- Corrige: manejo del certificado SSL de TrendMicro (ProtectLink), timeout de QuickVPN y **deshabilita cifrados SSL de fuerza baja y media** (≤128 bits).
- Problemas conocidos: no se puede entrar con Firefox por HTTPS tras un reset de fábrica (usar IE o borrar caché/cookies); el botón MAC Clone no refleja el estado guardado; "Approved URL" de ProtectLink no funciona bien; la pestaña ProtectLink se ve mal en IE8 y Firefox 4.0.1.

**Rama 4.x (solo hardware v3, Cavium):**

| Versión | Fecha aprox. | Notas | Confianza |
|---|---|---|---|
| **4.0.0.07** (4.0.0.7-tm) | ~ago-2010 (el nombre de archivo incluye 20100819) | Primera versión v3. Cisco: "will only work with the newly released RV042, RV082, and RV016 Version 3 hardware and is not backward compatible". Sale junto con la **Firmware Rescue Utility v4.0.0.7** | MEDIA/ALTA |
| 4.0.2.08 | ~2010-2011 | RN: la v3 tiene todas las funciones de v1/v2 **excepto el dual-image** del RV082/RV016 | MEDIA |
| 4.0.3.03-tm | ~2011 | Usada en la review v3 de SmallNetBuilder; IPv6 a partir de aquí (según SNB) | MEDIA |
| 4.0.4.02-tm | 2011 | Artículo KB de Cisco "RV0xx hardware version 3 firmware version 4.0.4.02-tm" | MEDIA |
| 4.1.0.02-tm | ~2011 | "V3 hardware required" | MEDIA |
| 4.1.1.01 | ~2011-2012 | RN "Multi-WAN Network Security Firewall Firmware 4.1.1.01" | MEDIA |
| **4.2.1.02**(-tm) | **Junio 2012** (KB Cisco del 08-jun-2012) | Añade la opción **NAT-Traversal** en Tunnel > Advanced (CSCtr93536). Corrige el error de rango DHCP (CSCtw78891) y la ayuda de VPN Client-to-Gateway (CSCtx08246). Problemas conocidos: UPnP se anuncia como "Linux Gateway Device"; varios teléfonos SIP con el mismo puerto (CSCty22521); el rango DHCP no se amplía al cambiar la máscara (CSCtx08218); el RV016 no obtiene IPv6 tras reiniciar (CSCtx27745); conflicto de hostname con Bonjour | ALTA |
| 4.2.2.08 | ~17-jun-2013 | Fecha según un issue de GitHub de 2014 (TCP-32764): la v03 con 4.2.2.08 **no** tiene el backdoor TCP/32764. A partir de esta versión, la Admin Guide indica que desaparece un enlace de la interfaz | MEDIA |
| 4.2.3.03 | ~2014 | Fixes de reconexión VPN tras reinicio, desconexiones IPsec y mensajes de log | BAJA (Softpedia) |
| 4.2.3.06 | ~2015 | Documento Open Source del 02-jun-2015 | ALTA (existencia) |
| 4.2.3.07(-tm) | ~2015-2016 | Listada en su momento como "última para v3" (linksysinfo). Hay reportes de cuelgue de la GUI y pérdida de velocidad | BAJA |
| 4.2.3.08 | ~2016 | **CSCvb81527**: el certificado SSL solo soportaba SHA1 → pasa a SHA256 | ALTA |
| 4.2.3.09 | ~sep-2018 (OSD del 25-sep-2018) | **CSCvk56058**: pérdida inesperada de configuración | ALTA |
| 4.2.3.10 | ~oct-2019 (OSD del 23-oct-2019) | **CSCvq34370** (certificados y claves estáticos), **CSCvq34376** (hashes de contraseña estáticos), vulnerabilidades de dnsmasq, OpenSSL y UPnP, cookies débiles en la web (**CSCvq76840**), auth_key hard-coded (CSCvq76771) y ejecución de comandos crítica **en el RV082 (CSCvq97031)** | ALTA (contenido). La publicación para el RV082 tiene datos contradictorios: ver nota |
| **4.2.3.14** | **17-jun-2020** | **Última versión para el RV082.** Corrige múltiples *command injection* y *stack overflow* con ejecución de código (CSCvt29372 … CSCvt29423; **CVE-2020-3286 a CVE-2020-3296**) y evalúa un buffer overflow de pppd (CSCvt39795). Las RN dicen "The Cisco RV016 and RV082 have reached End of Life" | ALTA |

Nota sobre la 4.2.3.10 y la 4.2.3.14:
- El advisory de Cisco del 20-nov-2019 (cisco-sa-20191120-sbr-rv-infodis) da como corregidos solo el RV042 y el RV042G en la 4.2.3.10, y dice que el RV082 y el RV016 "have reached the end of software maintenance".
- Sin embargo, las RN de la 4.2.3.14 incluyen CSCvq97031 ("critical command execution on the Cisco RV082") entre lo resuelto en la 4.2.3.10, y la NVD tiene CPE `rv082_firmware:4.2.3.10`.
- El advisory de junio de 2020 (cisco-sa-rv-routers-stack-vUxHmnNz) lista explícitamente el **RV082 corregido en la 4.2.3.14**.
- Conclusión: **la 4.2.3.14 es la versión final y la recomendada para cualquier RV082 v3**. **[ALTA]**

### 6.3 Compatibilidad firmware ↔ hardware (reglas críticas)
- **v1/v2 → máximo 2.0.2.01-tm.** No se puede pasar a 4.x: la 4.x es para otro SoC (MIPS64 Cavium frente a ARM XScale). Intentarlo puede no hacer nada o **dejar el equipo inservible** (*brick*). **[ALTA/MEDIA]**
- **v3 → solo 4.x.** No existe forma de instalar 1.x/2.x en una v3.
- **v2:** desde la 2.0.0.19-tm, el router **bloquea el downgrade a 1.x**. **[ALTA]**
- **v3:** el downgrade está permitido, pero **restaura la configuración de fábrica** ("If you choose an earlier firmware version, the factory default settings will be used. All custom settings will be lost"). **[ALTA]**
- **Migrar la configuración de v1/v2 a v3:** la configuración 2.x no se puede importar directamente en la v3. Cisco publicó la **"Migration Utility for Cisco RV Series Multi-WAN Routers"**, un ejecutable Windows (`Migration_Utility.exe`). La v1.0.2.2 era la inicial y se reportaban fallos. La **v1.0.2.8** soporta como origen RV082 v2 y v1.x con firmware 2.0.0.x y genera un archivo del tipo `RV082_V3.config`. Limitaciones: la v3 no acepta comillas simples, dobles ni barra invertida en los valores, y el MAC clone se reconfigura a mano. Error típico: "Get ModelName fail". **[MEDIA]**

### 6.4 Procedimiento de actualización (web)
1. Descargar el firmware de la página del RV082 en cisco.com (Downloads; "Small Business Router Firmware"). En v1 se usaba linksys.com, y la GUI tiene un botón "Firmware Download from (Linksys/Cisco) Web Site". Descomprimir el archivo si viene en .zip.
2. **Hacer backup de la configuración** (*System Management > Backup and Restore > Backup Startup Configuration*; archivos `Startup.config` y `Mirror.config`).
3. *System Management > Firmware Upgrade* → Browse/Choose File → **"Firmware Upgrade Right Now"** (v1/v2) o **Upgrade** (v3) → OK.
4. Esperar varios minutos. **No apagar, no pulsar reset, no cerrar el navegador y no desconectar.** El tráfico se corta durante el proceso.
5. El equipo reinicia y cierra la sesión. Volver a entrar en `https://192.168.1.1` (puede hacer falta renovar la IP del PC).
6. Verificar la versión en *System Summary* ("Firmware version"; en v3 también "PID VID" y "MD5 Checksum").

La guía Linksys v1 advierte que actualizar **puede borrar la configuración**. Fuentes: Admin Guide, guía Linksys y Cisco smb541. **[ALTA]**

Configuración *startup/mirror* (v3) **[ALTA]**:
- El router copia automáticamente el *startup* al *mirror* tras **24 h estables**, sin reinicios ni cambios.
- Si el *startup* se corrompe, arranca con el *mirror*.
- Los botones "Copy Startup to Mirror" y "Copy Mirror to Startup" actúan al instante. El segundo reinicia el equipo.
- Si el *mirror* sigue en estado de fábrica, copiarlo al *startup* equivale a un reset de fábrica.

### 6.5 Recuperación de firmware dañado

**Hardware v3 (Cavium/U-Boot)** **[MEDIA, varios reportes coincidentes]**:
1. Configurar el PC con **IP estática en 192.168.1.x/24** (p. ej. 192.168.1.10, máscara 255.255.255.0) y desactivar el firewall o permitir TFTP (UDP 69).
2. Conectar el PC a un puerto **LAN**.
3. Desenchufar el router. **Mantener Reset presionado y volver a enchufar**, sosteniendo de ~5 a 15 s hasta que el **DIAG parpadee lento**.
4. Comprobar que `ping 192.168.1.1` responde (bootloader activo).
5. Enviar el firmware por TFTP:
   - **Método oficial:** "RV0xx V3 Firmware Rescue Utility" (v4.0.0.7) de Cisco. Falla con frecuencia con "TFTP FAIL" o "Login Fail".
   - **Alternativa que funcionó en varios casos:** **Tftpd32/Tftpd64** (Ph. Jounin), pestaña *TFTP Client*, Host `192.168.1.1`, archivo = imagen .bin de firmware v3, botón **Put**. También funciona el cliente de Windows: `tftp -i 192.168.1.1 PUT <firmware.bin>`.
6. Al terminar la transferencia, el DIAG queda **fijo** mientras graba la flash (de 3 a 5 minutos) y luego el equipo reinicia solo. No hay que cortar la corriente.

**Hardware v1/v2 (IXP425)** **[BAJA/MEDIA]**:
- Existía una utilidad de recuperación TFTP de Linksys por modelo; para el RV042 v1 se cita `RV042_1.3.7.2_050727.exe`. Hoy es prácticamente inencontrable.
- El procedimiento es análogo: reset mantenido al encender, DIAG parpadeando y TFTP a 192.168.1.1.
- La v1/v2 del RV082 tenía **dual-image**, lo que reduce el riesgo de *brick* porque puede quedar una imagen alternativa. **No se encontró** documentación de cómo seleccionarla manualmente.
- Como último recurso: **consola serial interna** (RedBoot) o JTAG. **No se encontró el pinout** específico del RV082. Como referencia, el Linksys WRV54G (también IXP425) usa 115200 8N1.

**Riesgos generales:**
- Flashear firmware de otra versión de hardware.
- Cortar la corriente durante la escritura.
- Usar firmware de otro modelo (RV042 o RV016) sin confirmar que es el correcto.
- Con un UPS, el riesgo de corte disminuye.

### 6.6 Acceso oculto (telnet/SSH/consola)
- **No oficial.** Cisco dijo en sus foros: "Telnet on RV0xx is not a supported feature. It's there for development and debugging purpose".
- **v1/v2 (OpenRG):** el truco documentado para el RV042 de la misma generación es abrir `http://192.168.1.1/sysinfo123.htm?ConsoleSimulation=1`, que habilita telnet hasta el próximo reinicio. Luego `telnet 192.168.1.1` con las credenciales de la GUI lleva a la CLI OpenRG. El comando `shell` da una shell Linux. Para dejarlo permanente: `rg_conf_set /nk/telnetd_enabled 1`, `flash_commit` y `reboot`; un reset de fábrica lo elimina. Hay reportes de que también funcionaba en RV016/RV082 "al menos con firmwares antiguos". **[MEDIA para el RV042, BAJA para el RV082]**
- **v3:** el mismo truco **no funciona** según los reportes ("Incorrect login" en unidades nuevas). Existen **firmwares modificados no oficiales** (p. ej. una 4.2.3.07 modificada en treshaut.net) con telnet/SSH y root, compilados desde el GPL de Cisco. No se verificaron. **[BAJA]**
- La v3 tiene **serial y JTAG en placa** según DeviWiki, pero sin pinout publicado. **[MEDIA]**
- El RV082 v2 (2.0.2.01) y el v3 (4.2.2.08) **no** son vulnerables al backdoor TCP/32764 de 2014. **[MEDIA]**

### 6.7 OpenWrt / DD-WRT / firmware alternativo
- **OpenWrt:** **no soportado** en ninguna versión.
  - v1/v2 (IXP425): hubo intentos en foros, con RedBoot por serial funcionando en una rev 2. El obstáculo era el manejo del switch ADM6996. Nunca llegó a ser un port oficial. **[BAJA]**
  - v3 (Octeon CN50XX): hay un hilo de OpenWrt "Support for Cisco RV016/RV042/RV042G/RV082 V3 (Octeon Cavium)", sin port funcional. No se recomienda la imagen Octeon genérica porque no tiene particiones definidas. El parche LEDE resuelve el `MEM=2048` del U-Boot de Cisco. El soporte del RTL8306SD existe en OpenWrt; el del RTL8309G se propuso en 2017 para el DIR-632. Último mensaje: febrero de 2022, sin avances. **[MEDIA]**
- **DD-WRT:** **no soportado**. El wiki de DD-WRT tiene una página del RV042 con su hardware, pero no hay build para la familia. **[MEDIA]**
- **Código GPL de Cisco:** se pide a **external-opensource-requests@cisco.com** citando el número de referencia del documento OSD. Para la 4.2.3.x es **78EE117C99-209556498**. Documentos OSD conocidos: "Open Source Used In RV0xxV3 Series Routers 4.2.1.x" (jun-2012), 4.2.3.06 (jun-2015), 4.2.3.09 (sep-2018), 4.2.3.10 (oct-2019) y 4.2.3.x (jun-2020). **[ALTA]**

### 6.8 Seguridad relevante para decidir la versión de firmware
Este es un resumen; otro archivo puede tratar la seguridad en detalle.

| Advisory / CVE | Fecha | Afecta al RV082 | ¿Parche? |
|---|---|---|---|
| cisco-sa-20191106-rv0x2 (certificados y claves estáticos) y cominj | nov-2019 | Sí | Solo 4.2.3.10 o posterior (con ambigüedad para el RV082; ver §6.2) |
| cisco-sa-20191120-sbr-rv-infodis (CSCvq76840) | nov-2019 | Sí (< 4.2.3.10) | RV082: "end of software maintenance" |
| cisco-sa-rv-routers-stack-vUxHmnNz (**CVE-2020-3286…3296**) | 17-jun-2020 | Sí (≤ 4.2.3.10) | **Corregido en 4.2.3.14** |
| cisco-sa-rv-command-inject-BY4c5zd | 2021 | Sí (≤ 4.2.3.14) | **No** ("has not released and will not release") |
| cisco-sa-sbr042-multi-vuln-ej76Pke5 (**CVE-2023-20025** bypass de autenticación crítico y **CVE-2023-20026** RCE) | ene-2023 (actualizado mar-2025) | Sí | **No.** Hay PoC pública |
| cisco-sa-sb-rv01x_rv32x_rce-nzAGWWDD (**CVE-2023-20118**, RCE) | abr-2023 | Sí | **No.** Entró en el catálogo **CISA KEV** (explotación activa) en marzo de 2025 |
| cisco-sa-rv-stored-xss-vqz7gC8W (XSS almacenado) | abr-2023 | Sí | No |
| cisco-sa-sbiz-rv-xss-OQeRTup (**CVE-2024-20362**, XSS) | abr-2024 | Sí | No |

**Mitigaciones que sugiere Cisco** para todas las vulnerabilidades sin parche:
- Deshabilitar **Remote Management** (Firewall > General).
- Bloquear el acceso a los puertos **443 y 60443** desde la WAN; algún advisory menciona también el **442**.
- No exponer la GUI a Internet.

La solución real es reemplazar el equipo.

---

## 7. Valores por defecto (factory defaults)

| Parámetro | Valor | Confianza |
|---|---|---|
| IP LAN | **192.168.1.1** | ALTA |
| Máscara | **255.255.255.0** | ALTA |
| Usuario / contraseña | **admin / admin** (minúsculas). Desde la 2.0.0.19 el usuario es configurable | ALTA |
| Servidor DHCP | Habilitado; rango **192.168.1.100 a 192.168.1.149** (50 direcciones) | ALTA (Admin Guide v3; probablemente igual en v1) |
| Lease DHCP | 1440 min (24 h) | ALTA (v3) |
| DNS/WINS LAN | 0.0.0.0 (dinámico) | ALTA (v3) |
| WAN | Obtener IP automáticamente (DHCP) | MEDIA |
| Modo de trabajo | **Gateway** (NAT activado) | ALTA |
| DMZ/Internet | Habilitado; modo WAN2 o DMZ según el LED "DMZ Mode" | MEDIA |
| Firewall / SPI / DoS | Habilitados | ALTA |
| Block WAN Request (no responder ping en la WAN) | Habilitado (no confirmado en la guía) | BAJA |
| Remote Management | **Deshabilitado**. Al activarlo, el puerto por defecto es 80 (o 443 si se usa HTTPS) | ALTA |
| HTTPS | **Habilitado.** Si se deshabilita, QuickVPN deja de funcionar | ALTA |
| Multicast pass-through | Deshabilitado | ALTA |
| Java/Cookies/ActiveX/Proxy | No bloqueados | ALTA |
| Complejidad de contraseña | Habilitada; caducidad configurable (por defecto 180 días si se activa) | ALTA (v3) |
| Zona horaria | GMT-08:00 Pacífico | ALTA (v3) |
| SNMP | Comunidades `public` (get/trap) y `private` (set) | ALTA (v3) |
| Bonjour | Habilitado | ALTA (v3) |
| Puertos LAN | Todos habilitados, auto-negociación y VLAN 1 | ALTA |
| Puertos de servicio | QuickVPN: **443 y 60443**; PPTP: **1723** | ALTA |

Al iniciar sesión con admin/admin, el firmware v3 obliga o recomienda cambiar la contraseña. Para deshabilitar el firewall, primero exige cambiar la contraseña por defecto. **[ALTA]**

---

## 8. Contenido de la caja

**[ALTA]** (QSG Cisco RV0xx; coincide con los listados Linksys):
- Router RV082
- **Cable de alimentación AC** (no usa adaptador externo)
- Cable Ethernet RJ-45
- **Kit de montaje en rack** (2 brackets y tornillos; solo RV082/RV016)
- Quick Start Guide / Quick Installation Guide
- CD del producto con la User Guide en PDF (y en la época Linksys, QuickVPN)
- Tarjeta de registro (en los paquetes Linksys)

---

## 9. Documentación oficial (nombres y ubicaciones)

| Documento | Identificador / fecha | URL (puede haber cambiado) |
|---|---|---|
| Data Sheet "Cisco RV082 Dual WAN VPN Router" | C78-501227-05 (07/2013); también existe en es/fr/it/de | https://www.cisco.com/c/dam/en/us/products/se/loc/data_sheet_c78-501227.pdf |
| Cisco Small Business RV0xx Series Routers **Administration Guide** (RV016, RV042, RV042G y RV082; firmware 4.x) | 78-19576-01; revisión de 30-may-2013 | https://www.cisco.com/c/dam/en/us/td/docs/routers/csbr/rv0xx/administration/guide/rv0xx_AG.pdf |
| RV0xx Series **Quick Start Guide** | 78-19481 | https://www.cisco.com/c/dam/en/us/td/docs/routers/csbr/rv0xx/quick_start/guide/RV0xx_QSG_78-19481.pdf |
| Linksys "10/100 8-Port VPN Router Model RV082 **User Guide**" (v1/v2, ~105 páginas) | Apéndices: Upgrading Firmware, Physical Setup, **Battery Replacement**, Specifications | Copias en manualsdir.com y manualslib.com |
| Release Notes RV082 2.0.0.19-tm | OL-20330-01, jul-2009 | https://www.cisco.com/c/dam/en/us/td/docs/routers/csbr/rv082/release/notes/RV082_RN_v2-0-0-19-tm.pdf |
| Release Notes RV082 2.0.2.01-tm | OL-25268-01, 15-jun-2011 | https://www.cisco.com/c/dam/en/us/td/docs/routers/csbr/rv082/release/notes/RV082_RN_v2-0-2-01-tm.pdf |
| Release Notes RV082 1.3.9.8-tm | — | …/rv082/release/notes/RV082_firmware_v1.3.9.8-tm_RN.pdf (no accesible en la investigación) |
| Release Notes RV0xx 4.0.2.08 / 4.0.3.3 | — | …/rv0xx/release/rv0xx_rn_v4-0-2-08.pdf, …/RV0xx_RN_v4-0-3-3.pdf |
| Release Notes RV0xx V3 4.2.1.02 (francés, Canadá) | 78-21025-01, jun-2012 | https://www.cisco.com/c/dam/en/us/td/docs/routers/csbr/rv0xx/release-FR/rv0xx_rn_4-2-1-02_fr.pdf |
| Release Notes RV0xx 4.2.3.14 | jun-2020 | https://www.cisco.com/c/dam/en/us/td/docs/routers/csbr/rv0xx/release/RV0xx_V_4-2-3-14.pdf |
| Release Notes Migration Utility 1.0.2.2 / 1.0.2.8 | — | …/rv0xx/release/rv0xxMigrationUtility_rn_v1-0-2-2.pdf (y …1-0-2-8.pdf) |
| Release Notes QuickVPN Client 1.2.11 | 9-mar-2009 | https://www.cisco.com/c/dam/en/us/td/docs/routers/csbr/rv016/release/notes/QuickVPN_Client_v1-2-11_RN.pdf |
| Open Source (GPL) 4.2.3.x | 78EE117C99-209556498, jun-2020 | https://www.cisco.com/c/dam/en/us/td/docs/routers/csbr/rv0xx/osln/RV0xxrouter423x.pdf |
| Aviso EoL RV082 (EOL10819) | 4-feb-2016 | https://www.cisco.com/c/en/us/products/collateral/routers/small-business-rv-series-routers/eos-eol-notice-c51-736749.html |
| Artículos de soporte Cisco | smb288 (reboot/reset), smb541 (firmware upgrade), smb573 (backup/restore), smb2874 (acceso a la GUI), smb452 (QuickVPN) | https://www.cisco.com/c/en/us/support/docs/smb/routers/cisco-rv-series-small-business-routers/… |
| Página de soporte | — | www.cisco.com/go/RV082 (redirige a la serie RV) |

**Cliente QuickVPN:** las versiones conocidas incluyen 1.2.6, 1.2.8, 1.2.11 (2009) y **1.4.2.1** (la última citada). Oficialmente soporta Windows XP de 32 bits, Vista y **Windows 7** (32/64 bits). En Windows 10 hay reportes de "remote gateway is not responding". QuickVPN está declarado **End-of-Life** por Cisco. **[MEDIA]**

---

## 10. Lagunas y datos no confirmados
- **MTBF:** no publicado en ninguna fuente encontrada.
- **Sesiones concurrentes máximas:** no publicadas por Cisco.
- **Velocidad exacta del IXP425 en v1/v2** (266 frente a 533 MHz) y RAM de v2: contradictorio o sin confirmar.
- **Fechas exactas** de lanzamiento de v2 y v3. La v3 es de ~2010, deducida de la fecha del firmware 4.0.0.7.
- **Diferencias físicas visibles** (color o logo) entre v1 y v2: sin fuente fiable.
- **Notas de versión completas** de 1.x, 4.0.x, 4.1.x, 4.2.2.08, 4.2.3.03 y 4.2.3.06/07: no accesibles; solo hay fragmentos.
- **Pinout serial y parámetros de consola** del RV082: no encontrados.
- **Nombres exactos** de los archivos .bin de firmware: no confirmados.
- **Recomendación vigente de Cisco** para reemplazar la serie RV tras el fin de soporte de RV340/RV345 (31-oct-2026): no investigada.
- **Tensión interna de la v3:** solo se conoce la corriente de entrada (0,8 A, DeviWiki).

---

## 11. Fuentes

**Oficiales Cisco:**
- Aviso EoL RV082 (EOL10819): https://www.cisco.com/c/en/us/products/collateral/routers/small-business-rv-series-routers/eos-eol-notice-c51-736749.html
- Datasheet RV082 (C78-501227-05): https://www.cisco.com/c/dam/en/us/products/se/loc/data_sheet_c78-501227.pdf
- Admin Guide RV0xx: https://www.cisco.com/c/dam/en/us/td/docs/routers/csbr/rv0xx/administration/guide/rv0xx_AG.pdf (también https://www3-realm.cisco.com/en/US/docs/routers/csbr/rv0xx/administration/guide/rv0xx_AG_78-19576.pdf)
- QSG RV0xx: https://www.cisco.com/c/dam/en/us/td/docs/routers/csbr/rv0xx/quick_start/guide/RV0xx_QSG_78-19481.pdf
- RN 2.0.0.19-tm: https://www.cisco.com/c/dam/en/us/td/docs/routers/csbr/rv082/release/notes/RV082_RN_v2-0-0-19-tm.pdf (copia: https://images10.newegg.com/UploadFilesForNewegg/itemintelligence/cisco/RV082_RN_v2_0_0_19_tm1403836501022.pdf)
- RN 2.0.2.01-tm: https://www.cisco.com/c/dam/en/us/td/docs/routers/csbr/rv082/release/notes/RV082_RN_v2-0-2-01-tm.pdf
- RN 4.2.1.02 (FR): https://www.cisco.com/c/dam/en/us/td/docs/routers/csbr/rv0xx/release-FR/rv0xx_rn_4-2-1-02_fr.pdf
- RN 4.2.3.14: https://www.cisco.com/c/dam/en/us/td/docs/routers/csbr/rv0xx/release/RV0xx_V_4-2-3-14.pdf
- Índice de release notes de la serie RV: https://www.cisco.com/c/en/us/support/routers/small-business-rv-series-routers/products-release-notes-list.html
- OSD 4.2.3.x: https://www.cisco.com/c/dam/en/us/td/docs/routers/csbr/rv0xx/osln/RV0xxrouter423x.pdf
- Página de soporte y estado de la serie RV (modelos retirados): https://www.cisco.com/c/en/us/support/routers/rv082-dual-wan-vpn-router/model.html
- Página "obsolete" del RV016: https://www.cisco.com/c/en/us/obsolete/routers/cisco-rv016-multi-wan-vpn-router.html
- Comparativa de modelos Cisco: https://www.cisco.com/c/en/us/support/smb/product-support/small-business/other-routers-comparison.html
- EoL RV042/RV042G (EOL13238): https://www.cisco.com/c/en/us/products/collateral/routers/small-business-rv-series-routers/eos-eol-notice-c51-743070.html
- EoL RV320/RV325 (EOL13240): https://www.cisco.com/c/en/us/products/collateral/routers/small-business-rv-series-routers/eos-eol-notice-c51-743069.html
- smb288 Reboot/Factory reset: https://www.cisco.com/c/en/us/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb288-reboot-and-restore-to-factory-default-settings-on-rv016-rv04.html
- smb541 Firmware upgrade: https://www.cisco.com/c/en/us/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb541-firmware-upgrade-on-rv016-rv042-rv042g-and-rv082-vpn-routers.html
- Advisory infodis 2019: https://www.cisco.com/c/en/us/support/docs/csa/cisco-sa-20191120-sbr-rv-infodis.html
- Advisory stack overflow 2020: https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-rv-routers-stack-vUxHmnNz
- Advisory command inject 2021: https://www.cisco.com/c/en/us/support/docs/csa/cisco-sa-rv-command-inject-BY4c5zd.html
- Advisory 2023 (CVE-2023-20025/20026): https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-sbr042-multi-vuln-ej76Pke5
- Advisory RCE 2023 (CVE-2023-20118): https://www.cisco.com/c/en/us/support/docs/csa/cisco-sa-sb-rv01x_rv32x_rce-nzAGWWDD.html
- Advisory stored XSS 2023: https://www.cisco.com/c/en/us/support/docs/csa/cisco-sa-rv-stored-xss-vqz7gC8W.html

**Reviews, wikis y comunidad:**
- SmallNetBuilder, review del RV082 v1: https://www.smallnetbuilder.com/lanwan/lanwan-reviews/linksysrv082review/
- SmallNetBuilder, review del RV082/RV016 v3: https://www.smallnetbuilder.com/lanwan/lanwan-reviews/cisco-rv082-and-rv016-v3-vpn-routers-reviewed/
- SmallNetBuilder, review del RV320: https://www.smallnetbuilder.com/lanwan/lanwan-reviews/cisco-rv320-dual-gigabit-wan-vpn-router-reviewed/
- DeviWiki RV082 (Linksys): https://deviwiki.com/wiki/Cisco_RV082_(Linksys)
- DeviWiki RV082 v3: https://deviwiki.com/wiki/Cisco_RV082_v3 (espejo: https://wikidevi.wi-cat.ru/Cisco_RV082_v3)
- Foro OpenWrt, soporte RV0xx V3: https://forum.openwrt.org/t/support-for-cisco-rv016-rv042-rv042g-rv082-v3-octeon-cavium/6716
- Guía de usuario Linksys RV082 (manualsdir): https://www.manualsdir.com/manuals/168417/linksys-rv082.html
- GitHub TCP-32764, issue #95 (RV082 v02/v03): https://github.com/elvanderb/TCP-32764/issues/95
- Telnet en el RV042 (OpenRG): https://www.pbnet.ro/?p=39
- Recuperación de un RV042 v3 por TFTP: http://socc200.blogspot.com/2014/08/rv042-v3-recuperar-despues-de-una.html
- Comunidad Cisco (hilos citados de forma indirecta mediante snippets de búsqueda; el sitio bloqueaba la lectura directa): https://community.cisco.com/t5/routers-small-business/rv082-16-ver-3-vs-ver-4/m-p/1604811 ; https://community.cisco.com/t5/small-business-support-knowledge-base/rv016-rv042-rv082-hardware-v3-firmware-update/ta-p/3125488 ; https://community.cisco.com/t5/small-business-support-knowledge-base/rv0xx-firmware-rescue-utility-v-4-0-0-7/ta-p/3126161 ; https://community.cisco.com/t5/routers-small-business/firmware-for-rv082/td-p/2436366
- DSLReports, advertencia sobre la 4.0.0.7: https://www.dslreports.com/forum/r25758922-WARNING-RV042-RV082-RV016-firmware-4-0-0-7-tm
- NVD/CPE de las versiones de firmware del RV082: https://cpe.gcve.eu/products/33e266db-544d-5293-ad7c-9d028e94ba0b
- Network Computing, lanzamiento del producto (ene-2004): https://networkcomputing.com/switches-routers/linksys-ships-small-business-vpn-router
