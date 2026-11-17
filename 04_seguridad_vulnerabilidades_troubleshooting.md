# Cisco/Linksys RV082: seguridad, vulnerabilidades, problemas conocidos y troubleshooting

> Documento de referencia autosuficiente (para consultar sin internet).
> Investigación hecha el 2026-11-02. Fuentes: Cisco PSIRT, NVD (API 2.0), catálogo CISA KEV (versión 2026.10.30), release notes oficiales de Cisco (recuperadas de cisco.com y de Wayback Machine), la *Administration Guide* RV0xx (78-19576-01 B0, 2011-2012), Sekoia, Censys, XLab y foros de Cisco Community.
> Convenciones: **[VERIFICADO]** = dato tomado de la fuente primaria (Cisco/NVD/CISA). **[DUDOSO]** = fuente secundaria, anecdótica o contradictoria. **[INFERENCIA]** = conclusión mía a partir de datos verificados; no la afirma ninguna fuente.

---

## 0. Resumen ejecutivo

1. **El RV082 es un equipo sin soporte.** Cisco anunció el EoL el 4-feb-2016. El fin de venta y el fin de mantenimiento de software fueron el 5-may-2016, y el último día de soporte de hardware fue el 31-may-2021. Cisco dejó de publicar parches de seguridad para la familia RV016/RV042/RV042G/RV082. Desde 2021 todas las advisories dicen: *"Cisco has not released and will not release software updates"*.
2. **Hay dos CVE de la familia con explotación real confirmada por CISA (KEV):**
   - **CVE-2019-15271**: RCE como root vía la web UI, autenticada. Entró en KEV el 2022-06-08. Corregida en firmware 4.2.3.10 (solo para RV042/RV042G según la advisory; ver §2.2).
   - **CVE-2023-20118**: inyección de comandos vía la web UI, sin parche. Entró en KEV el 2025-03-03. La explotaron las botnets **PolarEdge** (2025) y **ViciousTrap** (2025).
3. **CVE-2023-20025 (CVSS 9.0 Cisco / 9.8 NVD)** es un bypass de autenticación **sin credenciales** en la web UI del RV016/RV042/RV042G/RV082. No tiene parche y existe PoC público. Combinado con cualquiera de las inyecciones de comandos autenticadas (20026/20118/20124…), da root remoto sin credenciales **[INFERENCIA, encadenamiento plausible; Cisco no lo afirma]**.
4. **El último firmware conocido es 4.2.3.14 (junio 2020) y solo vale para hardware V3.** Corrige las CVE de 2019-2020, pero **ninguna** de 2021 en adelante. El hardware V1/V2 está congelado en firmware 1.3.x/2.0.x, aún más antiguo (OpenSSL 0.9.6e en 2008).
5. **La mitigación oficial, a falta de parche, es: Remote Management desactivado + bloqueo de TCP 443 y 60443 en el WAN.** La interfaz LAN **no se puede desactivar** y siempre queda expuesta a cualquier host de la LAN. Recomendación de Cisco y de este documento: **reemplazar el equipo**. Cisco sugería RV325, RV340, Meraki o ISR 1000.
6. **Defaults inseguros documentados:** `admin/admin`; SNMP v1/v2c **habilitado** con comunidades `public`/`private`; DH Group 1 (768 bits) por defecto en IPSec; Aggressive Mode forzado con peer de IP dinámica; PPTP (MS-CHAPv2) disponible; Bonjour activo; regla por defecto WAN→DMZ **permitida**; certificado HTTPS autofirmado.

---

## 1. Contexto de soporte, hardware y firmware (lo relevante para seguridad)

### 1.1 Fechas de ciclo de vida (EoL notice Cisco, EOL10819 / c51-736749) [VERIFICADO]

| Hito | Fecha |
|---|---|
| Anuncio EoL | 4-feb-2016 |
| End-of-Sale | 5-may-2016 |
| End of SW Maintenance Releases | 5-may-2016 |
| Last Ship Date | 3-ago-2016 |
| End of Phone Support (HW) | 5-may-2017 |
| Last Date of Support (HW) | **31-may-2021**. Otra página de Cisco dice 05-May-2021 **[DUDOSO, discrepancia]** |
| Reemplazo sugerido | "No direct replacement"; Cisco sugiere RV325 (también EoL hoy) |

Part numbers: RV082, RV082-AU, RV082-CN, RV082-EU, RV082-UK.
Para comparar: RV042/RV042G tuvieron soporte hasta el 31-ene-2025 (página de la serie RV de Cisco).

### 1.2 Versiones de hardware y ramas de firmware

| HW | CPU / switch | Rama de firmware | Notas de seguridad |
|---|---|---|---|
| V1 / V1.x | Intel IXP (XScale ARM) **[DUDOSO, foro]**; switch ADM6996L | 1.x (p. ej. 1.3.9.8-tm, jul-2008) | 1.3.9.8: OpenSSL actualizado de 0.9.6d a **0.9.6e**, SSLv2 desactivado, fix DoS en puerto UDP 0. |
| V2 | Switch ADM6996I | 2.0.x (2.0.0.19-tm jul-2009; 2.0.2.01-tm jun-2011) | Una vez en 2.x no se puede bajar a 1.x. Último firmware V2: 2.0.2.01-tm. |
| **V3** | **Cavium Octeon CN50XX (MIPS64)** **[DUDOSO, OpenWrt forum]**; 32 MB flash / 256 MB RAM **[DUDOSO]** | 4.x (4.0.0.7 abr-2011 … **4.2.3.14 jun-2020**) | Imagen común "RV0xx" para RV016/RV042/RV082 V3. El firmware y el archivo de configuración **no son compatibles** con HW anterior. V3 pierde la función *dual-image* que tenían RV082/RV016 V1/V2. |

Cómo identificar la versión: la etiqueta PID de la parte inferior dice "V03" en hardware V3 (según Cisco Community).

### 1.3 Componentes open source del firmware (sirven para correlacionar CVE de terceros) [VERIFICADO, documentos OSD de Cisco]

| Componente | 4.2.3.07 (2015-16) | 4.2.3.10 (oct-2019) |
|---|---|---|
| Linux kernel | 2.6.16.26 | 2.6.16.26 |
| OpenSSL | 1.0.1l | **1.0.2s** |
| dnsmasq | 2.40 | **2.80** (sigue siendo vulnerable a DNSpooq, corregido en 2.83) **[INFERENCIA]** |
| libupnp | 1.3.1 | **1.6.21** |
| busybox | 1.2.1 | 1.2.1 |
| net-snmp | 5.1.4.2 | 5.1.4.2 |
| PPP / pptpd | 2.4.3 / 1.3.4 | 2.4.3 / 1.3.4 |
| strongSwan / ipsec-tools | 4.0.4 / 0.6.5 | 4.0.4 / 0.6.5 |
| stunnel | 4.05 | 4.05 |
| samba | 3.0.22 | 3.0.22 |
| BIND | 9.5.0 | 9.5.0 |
| mDNSResponder (Bonjour) | 107.6 | 107.6 |
| u-boot | 1.1.1 | 1.1.1 |
| otros | zebra 0.93a (RIP), radvd 1.6, tftp-hpa 0.48, openldap 2.3.24, krb5 1.3.1, sysklogd 1.5, ntpclient 2000_345 | + msmtp 1.4.21 |

Conclusión **[INFERENCIA]**: aun con el último firmware, kernel, busybox, net-snmp, samba, PPP y stunnel son de 2005-2008. Hay que asumir decenas de CVE de terceros no evaluadas.

---

## 2. Vulnerabilidades: tabla maestra (RV082 y familia RV016/RV042/RV042G/RV082)

Ordenadas por fecha de la advisory de Cisco. "Corregido" = primera versión con fix según Cisco. La columna de CVSS da primero la puntuación de Cisco (PSIRT) y después la de NVD cuando difieren.

| # | CVE(s) | Advisory Cisco (ID) | Fecha | Tipo | Auth | CVSS (Cisco / NVD) | RV082 afectado | Corregido | Exploit / explotación |
|---|---|---|---|---|---|---|---|---|---|
| 1 | CVE-2015-6418 | cisco-sa-20151210-dwvr | 2015-12-10 | Baja entropía en el RNG: se puede predecir el par de claves TLS | No | v2 4.3 | 4.0.0.7, 4.0.2.8 (listadas) | Cisco "released updates" sin decir versión. Las RN 4.2.3.06 listan **CSCus15224** como resuelto | Sin explotación conocida |
| 2 | (sin CVE) | cisco-sa-20191106-rv0x2 ("Issues") | 2019-11-06 | Cert + clave privada estática emitida a **QNO Technology**; hashes de contraseña hardcoded (root, cisco, lldpd) en /etc/shadow; componentes de terceros vulnerables | n/a | n/a | Sí | 4.2.3.10 (RN; la advisory menciona RV042/RV042G) | Investigadores: S. Viehböck y T. Weber (SEC Consult/IoT Inspector) |
| 3 | **CVE-2019-15271** | cisco-sa-20191106-sbrv-cmd-x | 2019-11-06 (rev 1.2 2022-12-15) | RCE root por HTTP (NVD: CWE-502, deserialización; Cisco: falta de validación del payload HTTP) | Sí: credenciales **o token de sesión activo** | 8.8 / 8.8 | < 4.2.3.10 | RV042/RV042G: 4.2.3.10. RV016/RV082 figuran como "end of SW maintenance", sin fix en la advisory, pero las RN 4.2.3.10 listan CSCvq97031 "Critical command execution on the Cisco RV082" como resuelto **[ver nota]** | **CISA KEV 2022-06-08** (vencimiento 2022-06-22). Cisco vio intentos *in the wild* en jun-2022 |
| 4 | CVE-2019-15957 | cisco-sa-20191106-sbr-cominj | 2019-11-06 (NVD publicó 2020-09-23) | Inyección de comandos (un campo concreto de la UI) | Admin | 7.2 / 7.2 | < 4.2.3.10 | RV042/RV042G 4.2.3.10; RV320/325 1.5.1.05; RV016/RV082: no listado | Sin explotación pública conocida |
| 5 | CVE-2019-15990 | cisco-sa-20191120-sbr-rv-infodis | 2019-11-20 | Divulgación de información (autorización incorrecta en peticiones HTTP; en RN: "Weak cookies in web interface", CSCvq76840) | **No** | 5.3 / 5.3 | < 4.2.3.10 | RV042/RV042G 4.2.3.10; RV016/RV082 no listado | Hallazgo interno de Cisco |
| 6 | CVE-2020-3274, -3275, -3276, -3277, -3278, -3279 | cisco-sa-rv-routers-Rj5JRfF8 | 2020-06-17 (rev 1.1 2020-08-03) | Inyección de comandos (CWE-77) | Admin | 7.2 / 7.2 (v2 9.0) | ≤ 4.2.3.10 | **RV016/RV042/RV082: 4.2.3.14**; RV320/325: 1.5.1.11; RV042G sin fix listado | **PoC público** (según Cisco). Investigador: Kai Cheng |
| 7 | CVE-2020-3286 … CVE-2020-3296 (11 CVE) | cisco-sa-rv-routers-stack-vUxHmnNz | 2020-06-17 | Stack overflow → RCE root o DoS (CWE-119) | Admin | 7.2 / 7.2 | ≤ 4.2.3.10 | **RV016/RV042/RV082: 4.2.3.14**; RV042G sin fix listado | **PoC público** (según Cisco). Kai Cheng |
| 8 | CVE-2021-1314 … CVE-2021-1318 (5 CVE) | cisco-sa-rv-command-inject-BY4c5zd | 2021-02-03 | Inyección de comandos como root (CWE-20) | Admin | 7.2 / 7.2 | ≤ 4.2.3.14 (todas) | **Sin fix** (EoL); RV320/325: 1.5.1.13 | Sin explotación conocida. Kai Cheng (IIE-CAS) |
| 9 | CVE-2021-1319 … CVE-2021-1348 (30 CVE) | cisco-sa-rv-overflow-ghZP68yj | 2021-02-03 | Stack overflow (CWE-121) → RCE root o reload (DoS) | Admin | 7.2 / 7.2 | ≤ 4.2.3.14 | **Sin fix** | Sin explotación conocida. Kai Cheng |
| 10 | CVE-2021-40120 | cisco-sa-sbrv-cmdinjection-Z5cWFdK | 2021-11-03 | Inyección de comandos (un campo; CWE-20) | Admin, **solo si Remote Management está activado** | 6.5 / 7.2 | Sí | **Sin fix** | Sin explotación conocida. "leetsun" |
| 11 | **CVE-2023-20025** | cisco-sa-sbr042-multi-vuln-ej76Pke5 | 2023-01-11 | **Bypass de autenticación** en la web UI (validación incorrecta de la entrada en paquetes HTTP; CWE-293) → acceso root | **No** | **9.0** (AV:N/AC:H/PR:N/UI:N/S:C/C:H/I:H/A:H) / **9.8** | Todas las versiones (solo RV016/RV042/RV042G/RV082) | **Sin fix** | **PoC público** (según Cisco). Investigador: Hou Liuyang (Qihoo 360 Netlab) |
| 12 | CVE-2023-20026 | ídem | 2023-01-11 | Inyección de comandos → root (CWE-77) | Admin | 6.5 / 7.2 | Todas | **Sin fix** | **PoC público** (según Cisco) |
| 13 | **CVE-2023-20118** | ídem (rev 2.0 2023-03-14 la añade; rev 2.1 2025-03-07 añade KEV) | 2023-03-14 (NVD publicó 2023-04-13) | Inyección de comandos → root (CWE-77) | Admin según Cisco (PR:H). Sekoia lo describe como no autenticado **[DUDOSO/contradictorio]** | 6.5 / 7.2 | Todas | **Sin fix** | **CISA KEV 2025-03-03** (vencimiento 2025-03-24). Explotada por **PolarEdge** y **ViciousTrap**. Investigador: Wang Jincheng (X1cT34m) |
| 14 | CVE-2023-20124 | cisco-sa-sb-rv01x_rv32x_rce-nzAGWWDD | 2023-04-05 | Inyección de comandos → root (CWE-77) | Admin | 6.5 / 7.2 | Todas | **Sin fix** | Sin explotación conocida. FeiXincheng y ShaLetian (X1cT34m) |
| 15 | CVE-2023-20137 … CVE-2023-20151 (15 CVE) | cisco-sa-rv-stored-xss-vqz7gC8W | 2023-04-05 | XSS (CWE-79) | No (requiere que la víctima haga clic) | 6.1 / 6.1 | Todas | **Sin fix** | Sin explotación conocida. "Leetsun" |
| 16 | CVE-2024-20362 | cisco-sa-sbiz-rv-xss-OQeRTup | 2024-04-03 | XSS reflejado (CWE-80) | No (UI:R) | 6.1 / — | Todas | **Sin fix** | Sin explotación conocida. "Leetsun" |

**Total CVE asociadas al RV082 en NVD:** 79-80. Son 72 por búsqueda de texto "RV082" más 7-8 por CPE (CVE-2015-6418, 2019-15271, 2019-15957, 2019-15990, 2021-40120, etc.).

**Nota sobre el fix del RV082 en 2019 [contradicción documentada]:** las advisories de nov-2019 (#3, #4, #5) dicen que RV016/RV082 llegaron a *end of software maintenance* y no les asignan versión corregida. Sin embargo:
- Las release notes de la imagen unificada **RV0xx 4.2.3.10** listan CSCvq97031 ("Critical command execution on the Cisco RV082 router") como resuelto.
- Las advisories de jun-2020 (#6, #7) nombran explícitamente **4.2.3.14 como fix para RV016, RV042 y RV082**.
- La release note de 4.2.3.14 (jun-2020) dice que el RV016 y el RV082 "have reached End of Life".
- NVD/CPE registra "RV082 firmware 4.2.3.14".

Conclusión práctica **[INFERENCIA]**: un RV082 **V3** con 4.2.3.14 tiene corregidas las CVE #3-#7 y nada posterior. Conviene verificar en software.cisco.com que el binario 4.2.3.14 sigue publicado para el RV082.

### 2.1 Vulnerabilidades relacionadas que NO afectan (o no consta que afecten) al RV082

| CVE / advisory | Modelos afectados | Estado para RV082 |
|---|---|---|
| CVE-2024-20516 … 20524 (cisco-sa-sb-rv04x_rv32x_vulns-yJ2OSDhV, 2024-10-02; DoS 6.8 y RCE 6.5, admin) | RV042, RV042G, RV320, RV325 | **El RV082 no se menciona** (ni como afectado ni como no afectado). Como comparte código con el RV042, es probable que también sea vulnerable **[INFERENCIA]** |
| DNSpooq, CVE-2020-25681 … 25687 (cisco-sa-dnsmasq-dns-2021-c5mrdf3g, 2021-01-19) | RV042/RV042G (CSCvv83789, "None planned"), RV320/RV325, RV110W, RV130, RV215W… | **El RV082 no figura en la lista**, aunque usa la misma imagen y el mismo dnsmasq (2.80 en 4.2.3.10). Probablemente afectado **[INFERENCIA]** |
| CVE-2020-3431 (XSS) | RV042/RV042G | RV082 no listado |
| CVE-2019-1652 / 1653 (KEV) | RV320/RV325 | No aplica (otra plataforma) |
| CVE-2021-1251/1308/1309 (LLDP), CVE-2021-1472/1473, CVE-2022-20699…20708 (KEV) | RV160/260/34x/13x | No aplica |
| CVE-2017-3882 (UPnP de CVR100W) y CVE-2015-6319 (SQLi de RV220W) | Otros | NVD los asocia por CPE a "cisco rv082". **Es un error de CPE de NVD** |
| CVE-2004-0580 (DHCP/BOOTP info leak de Linksys BEFSR*) | Linksys BEFSR | NVD lo vincula a la CPE "linksys:rv082" **[DUDOSO]**: la descripción no menciona al RV082 |

### 2.2 Detalle de las advisories clave

**cisco-sa-sbr042-multi-vuln-ej76Pke5** (la más grave; v2.1 Final, última actualización 2025-03-07)
- Bugs: CSCwd47551 (20025), CSCwd60199 y CSCwe41652 (20026/20118).
- Las tres vulnerabilidades son independientes: no hace falta explotar una para explotar otra.
- Snort SIDs: **64712, 64713, 64714, 64715**.
- Equipos confirmados no vulnerables: RV160/160W/260/260P/260W/340/340W/345/345P.
- Cisco: *"aware of attempted exploitation in the wild"* desde marzo de 2025.
- Mitigación: ver §5.

**cisco-sa-20191106-sbrv-cmd-x (CVE-2019-15271)**
- Bugs: CSCvq95596 (RV016), CSCvq97028 (RV042/RV042G), CSCvq97031 (RV082).
- Snort 52119-52122.
- Lo reportó Michel Gaschet-Clousier (Orange Cyberdéfense).
- CISA lo clasifica como "Deserialization of Untrusted Data". Acción requerida por KEV: "Apply updates per vendor instructions".

**cisco-sa-20191106-rv0x2 (Issues, sin CVE)**
- CSCvq34370: X.509 y clave privada de QNO Technology. Cisco dice que solo se usaban en pruebas; los equipos en producción generan certificados dinámicamente.
- CSCvq34376: hashes en /etc/shadow para `root`, `cisco` y `lldpd`. Cisco dice que el login web no usa ese archivo y que no conoce ninguna forma de llegar al SO base.
- Las release notes 4.2.3.10 agrupan además: CSCvq3496 (dnsmasq), CSCvq34412 (OpenSSL), CSCvq34415 (UPnP), CSCvq76840 (cookies débiles), CSCvq76771 ("Hard-coded auth_key"), CSCvq95596/97028/97031 (RCE).

**cisco-sa-20151210-dwvr (CVE-2015-6418)**
- Un atacante que recoja muchos handshakes TLS puede predecir el par de claves generado y descifrar sesiones.
- Bugs: CSCus15224 y 6 más.
- Afectados: RV042 4.0.2.8; RV042G 4.2.2.7/4.2.2.8/4.2.1.2; **RV082 4.0.0.7/4.0.2.8**; RV016 4.0.0.7/4.0.2.8/4.0.5.0; SA520/520W/540 2.2.07.
- La release note 4.2.3.06 lo da por resuelto ("Cisco RV042 Deterministic TLS Public Key Vulnerability").
- Recomendación: después de actualizar, **regenerar el certificado** (VPN > VPN Client Access > Generate) **[INFERENCIA]**.

---

## 3. Explotación activa, botnets e indicadores de compromiso (IoC)

### 3.1 CISA KEV (catálogo consultado: versión 2026.10.30, 1766 entradas) [VERIFICADO]

| CVE | Fecha de alta en KEV | Vencimiento federal | Nombre en KEV | Ransomware conocido | Acción requerida |
|---|---|---|---|---|---|
| CVE-2019-15271 | 2022-06-08 | 2022-06-22 | Cisco RV Series Routers Deserialization of Untrusted Data | Unknown | Apply updates per vendor instructions |
| CVE-2023-20118 | 2025-03-03 | 2025-03-24 | Cisco Small Business RV Series Routers Command Injection | Unknown | Apply mitigations… **or discontinue use of the product if mitigations are unavailable** |

### 3.2 PolarEdge (Sekoia, febrero 2025; actividad desde finales de 2023 o incluso jun-2023)
- **Vector:** CVE-2023-20118. Sekoia sitúa el fallo en `/cgi-bin/config_mirror.exp` (función `delete_cert`) y añade que un PoC público sugiere que `export_cert` también está afectada **[según Sekoia]**.
- **Caso 1 (22-31 ene 2025), webshell:** peticiones POST con cabeceras personalizadas `M` y `CMD`. Se sustituye `/usr/local/EasyAccess/www/cgi-bin/userLogin.cgi` por una webshell que exige una clave en la cabecera `PASSHASH` y el comando en `XCMD`.
- **Caso 2 (10 feb 2025), backdoor TLS:** el exploit hace que el router descargue por FTP un script llamado `q`. El script:
  - borra `/tmp/httpd.log`, `/tmp/web.log` y a sí mismo;
  - sale si existe `/tmp/.lock` o si la arquitectura no es **MIPS64**;
  - mata los procesos `ca`, `config.exp`, `process_monitor` y `application`;
  - descarga `t.tar` con busybox `ftpget` desde 119.8.186[.]227;
  - instala `cipher_log` en `/etc/flash/etc/`.
- **Persistencia:** modifica `/etc/flash/etc/cipher.sh`, inyecta comandos en `/tmp/splitDB/BONJOUR` y `/tmp/splitDB/SYSTEM` y llama a saveSettings. Añade reglas de iptables para abrir el puerto de escucha.
- **Backdoor:** ELF MIPS64 basado en Mbed TLS (PolarSSL). Informa a 195.123.212[.]54:58425 de la IP, el puerto, la ruta y la MAC.
- **Hashes SHA-256:**
  - `cipher_log`: `eda7cc5e1781c681afe99bf513fcaf5ae86afbf1d84dfd23aa563b1a043cbba8`
  - webshell: `1ca7262f91d517853a0551b14abb0306c4e3567e41b1e82a018f0aac718e499e`
  - certificado: `80cbc316aa58f8be722fd26b3026f077e61c82398599f9f719eade4bcd98173e`
- **Dominios:** largeroofs[.]top, siotherlentsearsitech[.]shop (entre otros).
- **Escala:** unas 2.017 IP según una búsqueda por certificado; EE. UU. 540. Modelos vistos: Cisco RV042, RV340, RV345, además de ASUS, QNAP y Synology.
- **Propósito:** sin determinar. Censys lo considera compatible con una red ORB (*Operational Relay Box*).
- **Corrección posterior [DUDOSO]:** el 24-sep-2025 Censys reconoció que el certificado usado para atribuir infraestructura aparece en datos de prueba públicos de Mbed TLS 3.4.0. XLab sigue atribuyendo con confianza alta (RPX relay en el puerto 55555).
- **Relevancia para el RV082 [INFERENCIA]:**
  - El script exige MIPS64. Según OpenWrt, el RV082 **V3** usa Cavium Octeon CN50XX (MIPS64), así que la cadena es técnicamente aplicable al RV082 V3.
  - Un RV082 V1 (Intel IXP/ARM) no ejecutaría ese binario concreto, aunque sigue siendo vulnerable a la inyección.
  - La ruta `/usr/local/EasyAccess` podría ser propia del RV320/325 **[no verificado]**.

### 3.3 ViciousTrap (Sekoia, mayo 2025)
- Explota CVE-2023-20118 en RV016/RV042/RV042G/RV082/RV320/RV325.
- Primer avistamiento el 18-mar-2025, desde una sola IP de origen (AS45839, Shinjiru, Malasia).
- ~5.300 dispositivos en 84 países; Macao con ~850. Otra fuente habla de >5.500 equipos de más de 50 marcas **[DUDOSO]**.
- Instala el script **NetGhost**, que redirige tráfico de ciertos puertos a infraestructura del atacante. Funciona como *honeypot* global para capturar exploits de terceros.
- Reutiliza una webshell de PolarEdge, pero Sekoia no ve vínculo entre ambos.
- Atribución tentativa: actor de habla china.

### 3.4 Otros
- CVE-2019-15271: Cisco PSIRT detectó intentos de explotación adicionales en jun-2022, el motivo de su entrada en KEV.
- CVE-2023-20025/20026: Cisco no observó explotación maliciosa al publicar (ene-2023). Qihoo 360 Netlab, que los descubrió, rastrea botnets, lo que sugiere que los encontró analizando tráfico malicioso **[INFERENCIA]**.

### 3.5 Cómo comprobar si un RV082 está comprometido (enfoque defensivo)
El RV0xx **no tiene CLI/SSH/Telnet documentados**, así que no se pueden inspeccionar los archivos directamente. Señales indirectas:
1. Reglas de iptables o redirecciones que no aparecen en la UI: tráfico saliente anómalo visto en un firewall o IDS aguas arriba.
2. Conexiones salientes a 195.123.212[.]54:58425 o 119.8.186[.]227 (FTP), o a los dominios de los IoC.
3. Puertos TCP extra escuchando en el WAN (comparar `nmap -p-` desde fuera con la configuración).
4. El proceso `httpd` reiniciándose o una web UI que responde distinto en `userLogin.cgi`.
5. Respuestas a peticiones con cabeceras `PASSHASH`/`XCMD` **[solo en laboratorio propio]**.
6. Remediación:
   - Factory reset **y** reflash del firmware desde un archivo descargado de Cisco. La persistencia está en `/etc/flash/etc`, así que un reset podría no bastar **[INFERENCIA]**.
   - Cambiar todas las credenciales, PSK de VPN y comunidades SNMP.
   - Mejor aún: **retirar el equipo**.

---

## 4. Debilidades de diseño y configuración (sin CVE propia)

| Área | Hallazgo | Fuente / estado |
|---|---|---|
| Credenciales | `admin` / `admin` por defecto. Un solo usuario administrador (no hay RBAC ni 2FA). La contraseña no se puede recuperar. "Minimum Password Complexity" viene activado por defecto en 4.x (≥8 caracteres, 3 de 4 clases). Caducidad opcional (180 días por defecto si se activa) | Admin Guide [VERIFICADO] |
| Interfaz de gestión LAN | Hay 4 reglas implícitas no anulables: HTTP desde la LAN al router, DHCP, DNS y Ping **siempre permitidos**. La web UI **no se puede desactivar ni filtrar** en el lado LAN | Admin Guide + advisories |
| Remote Management | Desactivado por defecto. Al activarlo, el puerto por defecto es 80 (se suele usar 8080). Con HTTPS activo, la gestión remota pasa a HTTPS (desde 1.3.9.8). El 60443 (QuickVPN) y el 1723 (PPTP) no se pueden usar para gestión remota | Admin Guide, RN 1.3.9.8 y 2.0.0.19 |
| HTTPS / TLS | Certificado **autofirmado**. Historial: SSLv2 desactivado (1.3.9.8 / 2.0.0.19); cifrados de ≤128 bits desactivados (2.0.2.01 y 4.0.3.03); POODLE SSLv3 corregido (CSCur53627) y OpenSSL 1.0.1l (CSCuo18668) en **4.2.3.06**; DH efímero débil (error de Chrome `ERR_SSL_WEAK_EPHEMERAL_DH_KEY`) corregido en **4.2.3.07** **[DUDOSO, foro]**; certificado SHA-1 → SHA-256 en **4.2.3.08** (CSCvb81527). No hay documentación de qué versiones TLS admite 4.2.3.x **[laguna]** | RN [VERIFICADO], foros [DUDOSO] |
| Entropía | CVE-2015-6418: claves TLS predecibles en 4.0.x | §2 |
| Material criptográfico estático | Certificado y clave de QNO y hashes de root/cisco/lldpd en el firmware hasta 4.2.3.10 | §2.2 |
| SNMP | **Habilitado por defecto**. Solo v1/v2c (sin v3). Comunidades por defecto **Get=`public`, Set=`private`**, Trap=`public`. Admite SET. net-snmp 5.1.4.2 | Admin Guide [VERIFICADO]. No consta si responde en el WAN **[laguna]** |
| Bonjour (mDNS) | Habilitado por defecto: anuncia el equipo en la LAN | Admin Guide |
| UPnP | Configurable en Setup > UPnP; Cisco recomienda desactivarlo. libupnp 1.3.1 tenía overflow (CSCue20980, la familia CVE-2012-5958…, corregido en 4.2.2.08); 4.2.3.10 sube a 1.6.21 (CSCvq34415) | RN |
| IPSec (Gateway-to-Gateway y Client-to-Gateway) | **DH Group 1 (768 bits) por defecto** (opciones: 1, 2=1024, 5=1536; no hay 14+). Cifrados DES/3DES/AES-128/192/256; integridad **MD5 o SHA1** (sin SHA-2). PSK de máximo 30 caracteres. **Aggressive Mode obligatorio si el peer remoto es de IP dinámica**, lo que expone el hash del PSK a cracking offline (técnica conocida tipo ike-scan/psk-crack). En Manual Keying, las claves cortas se rellenan con ceros. PFS opcional. NULL disponible en fase 2 | Admin Guide [VERIFICADO]; la técnica de ataque es conocimiento general |
| PPTP | Servidor PPTP de hasta 5 clientes, desactivado por defecto. PPTP/MS-CHAPv2 está criptográficamente roto (conocimiento general). Rango por defecto 192.168.1.200-204. Evaluación de pppd (CVE-2020-8597, CSCvt39795) en 4.2.3.14 | Admin Guide, RN 4.2.3.14 |
| QuickVPN | IPSec con arranque por SSL en TCP 443/60443. Funciona **sin certificado de cliente** (solo muestra un aviso), lo que lo hace susceptible a MITM. Limitado a 5 errores/min con bloqueo de 5 min (desde 2.0.0.19). Necesita Remote Management y HTTPS | Admin Guide, RN 2.0.0.19 |
| Firewall por defecto | LAN→WAN permitido; WAN→LAN denegado; LAN→DMZ permitido; DMZ→LAN denegado; **WAN→DMZ permitido**; DMZ→WAN permitido. SPI y DoS configurables. "Block WAN Request" descarta TCP no aceptado e ICMP en el WAN. No se puede deshabilitar el firewall sin cambiar antes la contraseña por defecto | Admin Guide |
| IPv6 | Al activar Dual-Stack, el **túnel 6to4 se activaba por defecto** (CSCue04572, corregido en 4.2.2.08) | RN |
| Puertos 0/1 | El equipo respondía a sondeos externos en los puertos 0 y 1 (CSCuc32147, corregido en 4.2.2.08) | RN |
| Configuración | Archivos `.config` (Startup y Mirror). No hay documentación de si cifran las contraseñas; en foros dicen que no se pueden decodificar **[DUDOSO]**. Tratarlos como secretos. Al importar una config de otro router se sobrescribe la MAC WAN | RN, foros |
| Logs | Syslog sin TLS (UDP, sysklogd 1.5). Email de alertas por SMTP (autenticación desde 4.2.3.03) | Admin Guide, RN |

---

## 5. Mitigaciones oficiales cuando no hay parche (Cisco, 2023-2024) [VERIFICADO]

Cisco afirma que **no hay workaround que corrija** las vulnerabilidades. Estas medidas solo **reducen la superficie** desde el WAN. Cisco advierte que las probó en laboratorio y que cada cliente debe evaluar su impacto. Con ellas la LAN sigue expuesta.

**A. Desactivar Remote Management**
1. Firewall > General.
2. Desmarcar **Remote Management**.
3. Save.

**B. Bloquear TCP 443 y 60443 en el WAN (RV016/RV042/RV042G/RV082)**
1. Firewall > Access Rules > **Service Management**. Crear el servicio `TCP-60443`: protocolo TCP, puertos 60443-60443, Add to List, OK. El 443 ya existe como "HTTPS".
2. Firewall > Access Rules > **Add**:
   - Action **Deny**
   - Service **HTTPS (TCP 443-443)**
   - Log "Log packets match this rule"
   - Source Interface **WAN1**
   - Source IP **Any**
   - Destination IP **Single**, con la IP WAN en ambos campos
   - Save
3. Repetir con el servicio **TCP-60443**.
4. Si se usa el segundo WAN (DMZ/Internet), crear las dos reglas equivalentes para WAN2 y su IP.

**Efectos secundarios [INFERENCIA]:**
- Bloquear el 443/60443 en el WAN **rompe QuickVPN**, que depende de esos puertos.
- Si la IP WAN es dinámica (DHCP/PPPoE), la regla "Single <IP WAN>" queda obsoleta cuando cambia la IP. Hay que revisarla o usar un rango.
- Con HTTPS activo, el reenvío de 80/443 a un servidor interno no funcionaba en 4.0.x (corregido en 4.0.3.03).

---

## 6. Hardening recomendado (checklist)

> Objetivo realista: **reducir el riesgo mientras se reemplaza el equipo**. Ningún hardening cubre CVE-2023-20025 desde la LAN.

**Prioridad crítica**
1. **Planificar el reemplazo** por un equipo con soporte. El RV082 lleva sin parches desde 2020 y tiene CVE en KEV sin fix.
2. **Remote Management OFF** y **deny 443/60443 en WAN1/WAN2** (§5). Si hace falta administración remota, hacerla **a través de una VPN que termine en otro equipo**, nunca exponiendo la UI.
3. **Firmware 4.2.3.14** si el hardware es V3. Si es V1/V2 (firmware 1.x/2.x), la exposición es aún mayor: priorizar su retirada.
4. Cambiar `admin/admin`: **otro nombre de usuario** y una contraseña larga. Mantener activada la complejidad mínima.
5. **SNMP:** desactivarlo (System Management > SNMP) o, si se necesita, cambiar las comunidades `public`/`private` por cadenas largas y aleatorias y restringir el acceso con reglas.

**Prioridad alta**

6. Desactivar **UPnP** (Setup > UPnP).
7. Desactivar **Bonjour** (System Management > Discovery-Bonjour) si no se usa.
8. Desactivar el **servidor PPTP** (VPN > PPTP Server) y usar IPSec.
9. **IPSec:**
   - DH **Group 5** (el máximo disponible), **AES-256**, **SHA1** (no hay SHA-2), **PFS** activado y **Main Mode**. Evitar peers de IP dinámica que fuerzan Aggressive Mode; en su lugar, usar DDNS en ambos extremos con tipo "IP by DNS resolved" **[verificar compatibilidad]**.
   - PSK aleatorio de 30 caracteres; rotarlo periódicamente.
   - No usar Manual Keying ni NULL.
10. **QuickVPN:** si se usa, exportar e instalar el **certificado de cliente** en los PC para mitigar MITM, y regenerar el certificado del router. Si no se usa, desactivarlo y dejar bloqueados el 443/60443 del WAN.
11. Mantener **Firewall, SPI, DoS y Block WAN Request activados**.
12. **DMZ:** recordar que WAN→DMZ está **permitido por defecto**. Crear reglas explícitas que solo permitan los servicios publicados, o deshabilitar la DMZ si no se usa.
13. IPv6: si no se usa, dejar IP Mode = IPv4. Si se usa Dual-Stack, revisar Setup > IPv6 Transition (6to4).

**Prioridad media**

14. **Segmentar la LAN**: como la web UI es accesible desde toda la LAN, hay que reducir quién puede llegar al router. Opciones: VLAN por puerto (Port Management, hasta 8 VLAN en el RV082) y no conectar equipos no confiables (Wi-Fi de invitados, IoT) a la misma VLAN que gestiona el router **[la eficacia depende de la topología; no está documentado si la UI escucha en todas las VLAN]**.
15. **HTTPS** activado. Gestionar desde una estación dedicada y cerrar sesión siempre: hay XSS/CSRF sin parche (CVE-2023-20137…151, CVE-2024-20362), así que no hay que navegar por otros sitios mientras se está logueado en el router.
16. **Syslog** hacia un servidor externo y **alertas por email** de *Unauthorized Login Attempt*, *Configuration Changes* y *Authorized Login*. Cambiar el servidor de syslog requiere reiniciar.
17. **NTP** correcto (Setup > Time). El servidor por defecto es time.nist.gov y la zona por defecto GMT-08:00 Pacific. Configurar el horario de verano a mano en formato mm.dd. Así los logs quedan bien sellados.
18. **Backups** de Startup.config y Mirror.config, guardados como secretos, y del certificado de administrador (Export for Admin contiene la clave privada).
19. Revisar periódicamente las reglas de acceso, el port forwarding (máximo 30), el port triggering (30) y one-to-one NAT.
20. Desactivar el "Multicast Pass Through" si no se usa (viene desactivado por defecto).
21. En un IDS/IPS aguas arriba: cargar los **Snort SIDs** 52119-52122, 54320-54331, 54333-54342, 57084-57086, 57095, 57096, 61457, 61458 y 64712-64715, y los IoC de §3.2.

---

## 7. Datos para auditor / pentester (enfoque defensivo)

### 7.1 Superficie de servicios esperada

Esta tabla **[INFERENCIA]** sale de las funciones documentadas y los componentes del firmware. **No es un escaneo real**: hay que validarla con nmap en el entorno propio.

| Servicio | Puerto | Lado | Por defecto | Comentario |
|---|---|---|---|---|
| Web UI HTTP | TCP 80 | LAN | Sí | En 2.0.2.01 y en 4.x, con HTTPS activo el HTTP de la LAN redirige a HTTPS. La regla implícita "HTTP from LAN always allowed" no se puede anular |
| Web UI HTTPS | TCP 443 | LAN; WAN si hay Remote Mgmt | LAN sí | Certificado autofirmado. Es el vector de casi todas las CVE |
| QuickVPN (SSL) | TCP 60443 (y 443) | WAN | Expuesto si hay Remote Mgmt y QuickVPN | Cisco lo incluye en la mitigación |
| Gestión remota | TCP 80/8080 o el configurado | WAN | No | — |
| DNS proxy (dnsmasq) | UDP/TCP 53 | LAN | Sí | DNSpooq (ver §2.1) |
| DHCP server | UDP 67 | LAN | Sí | — |
| SNMP | UDP 161 (traps UDP 162 salientes) | LAN (¿WAN?) | **Sí**, `public`/`private` | — |
| mDNS / Bonjour | UDP 5353 | LAN | Sí | — |
| UPnP (SSDP + HTTP) | UDP 1900 + TCP variable | LAN | Según configuración | Anunciaba "Linux Gateway Device" / "Intel Device SPY" (corregido en 4.2.2.08) |
| IKE / NAT-T | UDP 500 / 4500, ESP | WAN | Si hay túneles | — |
| PPTP | TCP 1723 + GRE | WAN | No | — |
| RIP | UDP 520 | LAN/WAN | No | — |
| Telnet / SSH / CLI | — | — | **No existe CLI documentada** | El firmware incluye tftp-hpa, pero no está documentado como servicio expuesto |

### 7.2 Fingerprinting
- Indicadores útiles **[INFERENCIA]**:
  - Página de login de "Cisco Small Business" en 192.168.1.1 (por defecto).
  - Certificado HTTPS autofirmado. En firmware antiguo puede ser SHA-1; desde 4.2.3.08 es SHA-256.
  - Puerto 60443 abierto en el WAN (rasgo distintivo de QuickVPN/RV0xx).
  - SNMP `sysDescr` vía comunidad `public`.
  - UPnP anunciado como "Linux Gateway Device" en firmware antiguo.
- No encontré firmas públicas fiables (cabecera `Server:`, `http-title` de nmap ni consultas de Shodan) específicas del RV082 **[laguna]**.
- Comandos útiles, solo en equipos propios o con autorización:
  - `nmap -sS -sU -p T:80,443,60443,1723,U:53,67,161,500,4500,1900,5353 <ip>`
  - `nmap --script ssl-enum-ciphers,ssl-cert -p 443,60443 <ip>`
  - `snmpwalk -v2c -c public <ip> system`
  - `ike-scan -M <ip>`, para ver si responde y qué propuestas acepta
- **Scanners:** Tenable/Nessus tiene plugins para estas advisories (aparecen los IDs 151459, 173969 y 173978; el mapeo exacto a cada CVE no está verificado). Qualys publicó QIDs para 20025/20026 (enero 2023).
- **Metasploit:** **no encontré módulos** para RV016/RV042/RV082. Los módulos existentes de Cisco RV son para RV320/325 (CVE-2019-1652/1653) y RV34x (CVE-2021-1472/1473, CVE-2022-20699).
- **PoC públicos:** Cisco confirma PoC públicos para CVE-2020-3274…3279, 2020-3286…3296, 2023-20025 y 2023-20026. En GitHub existe `lnversed/CVE-2023-20025` ("Cisco r042 research": proyecto Ghidra + rootfs, sin exploit documentado). cvefeed.io habla de 3 PoC para 20025 **[DUDOSO]**. Sekoia menciona un PoC público para 20118 (`config_mirror.exp`).

### 7.3 Puntos de auditoría (checklist)

| # | Comprobación | Severidad si falla |
|---|---|---|
| 1 | Firmware/HW: ¿V3 con 4.2.3.14? ¿V1/V2? | Crítica (EoL) |
| 2 | Remote Management activado / 443-60443 alcanzables desde Internet | Crítica (CVE-2023-20025, KEV 20118) |
| 3 | Credenciales por defecto (`admin/admin`) | Crítica |
| 4 | SNMP activo con `public`/`private` | Alta |
| 5 | PPTP activo | Alta |
| 6 | IPSec con DH1/DES/MD5, Aggressive Mode o PSK débil | Alta |
| 7 | UPnP activo | Media |
| 8 | DMZ con WAN→DMZ abierto sin reglas | Media/Alta |
| 9 | QuickVPN sin certificado de cliente | Media |
| 10 | Sin syslog externo ni NTP | Media |
| 11 | Hosts no confiables en la misma LAN que la UI | Alta (XSS, bypass desde la LAN) |
| 12 | Backups `.config` sin proteger | Media |

---

## 8. Troubleshooting y problemas conocidos

### 8.1 Olvido de contraseña / reset a fábrica [VERIFICADO: Cisco smb288 y Admin Guide]
- **La contraseña no se puede recuperar.** Las fuentes oficiales y los foros coinciden en que la única vía es el **reset a fábrica**. Los foros dicen que no hay forma conocida de decodificarla desde un backup `.config` **[DUDOSO]**.
- **Botón Reset** (en el RV082 está en el **panel frontal**, junto a los puertos Internet y DMZ):
  - **1 segundo** = reinicio sin perder la configuración. Se apagan todos los LED salvo System; el equipo está listo cuando vuelve System.
  - **30 segundos** = restaurar valores de fábrica. El LED **DIAG se ilumina en naranja** durante el proceso.
  - **Si no funciona:** desconectar la alimentación y volver a conectarla **manteniendo pulsado Reset**.
  - Algunas fuentes no oficiales hablan de 10 s **[DUDOSO]**; la cifra oficial es 30 s. En un foro de 2014 un usuario dice que 60 s no le funcionaron en un RV082 **[DUDOSO]**.
- **Desde la UI:** System Management > Factory Default > "Return to Factory Default Setting" > OK.
- **Valores tras el reset:** LAN 192.168.1.1/24; DHCP 192.168.1.100-149; usuario `admin`, contraseña `admin`. En 4.x hay que acceder por https://192.168.1.1, aunque el HTTP puede redirigir.
- **Antes del reset**, si todavía se puede entrar:
  - exportar Startup.config;
  - exportar el **certificado de administrador** (VPN > VPN Client Access > Export for Admin), que se pierde con el reset;
  - anotar la MAC WAN si se usa clonado.
- **Ojo [INFERENCIA]:** restaurar un backup hecho con la contraseña olvidada vuelve a dejar esa contraseña.
- Si tras el reset Firefox no permite iniciar sesión por HTTPS (problema conocido en 2.0.2.01), hay que borrar las cookies y la caché o usar otro navegador.

### 8.2 Acceso a la web UI con navegadores modernos

| Síntoma | Causa probable | Solución / workaround |
|---|---|---|
| "Su conexión no es privada" / NET::ERR_CERT_AUTHORITY_INVALID | Certificado autofirmado | Aceptar la excepción. Si se quiere, importar un certificado propio (permitido desde 2.0.2.01 en V2). |
| `ERR_SSL_WEAK_EPHEMERAL_DH_KEY` / "SSL received a weak ephemeral Diffie-Hellman key" | Firmware 4.2.3.06 o anterior con DH < 1024 bits | **Actualizar a ≥4.2.3.07**; según foros se corrigió ahí **[DUDOSO]** |
| Certificado SHA-1 rechazado | Firmware < 4.2.3.08 | Actualizar a ≥4.2.3.08 (CSCvb81527) |
| `ERR_SSL_VERSION_OR_CIPHER_MISMATCH` / `SSL_ERROR_UNSUPPORTED_VERSION` / `NO_CYPHER_OVERLAP` | Firmware antiguo (V1/V2 con OpenSSL 0.9.x, que no implementa TLS 1.1/1.2) frente a navegadores que eliminaron TLS 1.0/1.1 (~2020) **[INFERENCIA técnica]** | (a) Actualizar el firmware si el HW es V3. (b) Firefox: `about:config` → `security.tls.version.enable-deprecated=true` o `security.tls.version.min=1`, solo de forma temporal. (c) Navegador antiguo **en una VM aislada**. (d) Acceder por HTTP desde la LAN si HTTPS está desactivado, aunque con HTTPS activo el HTTP redirige. Un usuario de RV042 en 2016 solo pudo entrar activando SSL 3.0 en IE; lo resolvió actualizando a 4.2.3.08 **[DUDOSO]** |
| La UI se ve mal o en blanco | Problemas conocidos con Firefox 4 (4.0.x) y con la pestaña ProtectLink en IE8/Firefox 4.0.1 | Firmware ≥4.0.3.03/4.1.0.02. En IE, usar el modo de compatibilidad |
| Ventanas que no se abren (Service Management, View System Log, tablas de log) | Se abren como pop-ups | Permitir los pop-ups para la IP del router |
| "404 Not Found: server-side includes" tras actualizar a 4.1.0.02 | Bug conocido | Reiniciar el router |
| UI inaccesible tras actualizar y restaurar un backup | CSCtt36172 (corregido en 4.1.1.01) | Actualizar; reset si hace falta |
| UI y VPN sin respuesta tras mucho tiempo con alertas por email activas | CSCuh83563 (corregido en 4.2.3.03) | Actualizar o desactivar el email de logs |
| La IP del módem choca con 192.168.1.x | Mismo rango en ambos lados | Cambiar la IP LAN (p. ej. 192.168.0.1 o 10.1.1.1) o poner el módem en bridge |

**Java:** no encontré ninguna evidencia de que la UI del RV0xx necesite Java. Las opciones "Java/ActiveX/Cookies" de Firewall > General sirven para **bloquear** esos contenidos a los clientes LAN, no para la UI.

### 8.3 Reinicios, cuelgues y bloqueos
**Reportes de la comunidad [DUDOSO, anecdóticos]:**
- Bloqueos aleatorios: la UI deja de responder y el tráfico se ralentiza o se corta; solo lo arregla un power-cycle. Se dan sobre todo en RV082 v3 (también RV042 v3/RV042G) con 4.2.1.02, 4.2.2.08 y 4.2.3.07, con frecuencias de una vez al mes a varias por día.
- Otros reportan un RV082 v3 con 4.1.1.01-sp funcionando 252 días sin reinicio.
- Reinicios cada 20-50 minutos con carga de usuarios, o tras muchas conexiones de clientes VPN (ShrewSoft).
- Bugs corregidos oficialmente que causaban cuelgues o bloqueos:
  - crash al crear un túnel IPSec G2G hacia 0.0.0.0/0 (4.1.1.01);
  - tráfico FTP DoS sobre PPTP que bloqueaba el router (CSCug25381, 4.2.2.08);
  - email de logs que colgaba la UI y la VPN (CSCuh83563, 4.2.3.03);
  - overflow en libupnp (CSCue20980, 4.2.2.08).

**Diagnóstico sugerido:**
1. Actualizar a 4.2.3.14 (V3).
2. Desactivar las funciones no usadas: email de logs, UPnP, ProtectLink, PPTP.
3. Enviar los logs a un syslog externo para ver qué ocurre antes del cuelgue. El log local se pierde al reiniciar, aunque un Restart desde la UI envía el log antes de reiniciar.
4. Vigilar la temperatura y la fuente de alimentación (§8.4).
5. Revisar si coincide con picos de sesiones (P2P, escaneos).
6. Probar con otra unidad cargando la misma configuración: si falla igual, el problema es de configuración o de carga.

### 8.4 Hardware: fuente de alimentación, capacitores, temperatura y puertos
- El RV082 lleva **fuente interna** (cable AC 100-240 V, 50/60 Hz). Rangos de operación: **0-40 °C**, humedad 10-85 % sin condensación. Almacenamiento: 0-70 °C. [VERIFICADO]
- Instalación: no apilar objetos encima; buen flujo de aire; montado en pared, con los orificios de ventilación hacia los lados.
- **Fallos de fuente o capacitores [DUDOSO, foros]:** son el fallo típico de la familia (sobre todo en el RV042 con adaptador externo; se menciona un lote con capacitores defectuosos).
  - En el RV082 se reportan **ruido tipo "zumbido/canto" de una bobina** en la zona de alimentación, asociado a cortes de Internet.
  - Un usuario afirma que usar el segundo WAN aumenta el consumo y provoca crashes (no verificado).
  - Síntomas sugestivos: reinicios aleatorios, LED DIAG o System parpadeando sin terminar de arrancar, bloqueos con carga.
  - Diagnóstico, **solo personal cualificado, hay tensión de red dentro**: medir el rizado y la tensión de salida de la fuente interna e inspeccionar capacitores abombados o con fugas.
- **LED:**
  - DIAG encendido = arrancando; apagado = listo.
  - DIAG **parpadeando sin parar después de una actualización = imagen dañada** → usar TFTP o la *firmware rescue utility* de Cisco.
  - System fijo = encendido; System parpadeando = ejecutando diagnóstico. Si no deja de parpadear, la guía recomienda apagar módem, router y PC y encenderlos en orden: módem, router, PC.
- **Puertos:**
  - Port Management > Port Status muestra el link, la velocidad (10/100), el dúplex, la autonegociación y los contadores de errores. Port Setup permite deshabilitar puertos y forzar velocidad o dúplex. Un **dúplex mal negociado** (10 Mb half) es causa conocida de lentitud.
  - Un foro reporta que una falla en un puerto de un switch 3Com dejó dañado un puerto del RV082 **[DUDOSO]**.
  - Hay un hilo sobre "RV082 v3 sin LED de link en WAN1" **[DUDOSO]**.

### 8.5 Rendimiento (límite de ~100 Mbps)
- Todos los puertos son **10/100** (8 LAN + Internet + DMZ/Internet). Ningún WAN supera los ~94 Mbps útiles (límite físico de Fast Ethernet) **[INFERENCIA estándar]**.
- Especificaciones oficiales (Admin Guide, Apéndice G): **NAT throughput 200 Mbps** (agregado entre ambos WAN) e **IPSec throughput 97 Mbps**. Como referencia, una tienda da para el RV042 100 Mbps NAT y 59 Mbps IPSec **[DUDOSO]**.
- Con SPI, DoS, logging, ProtectLink o QoS el rendimiento real baja. Hay reportes de RV082 con fibra de 30 Mb que solo daban 10 Mb de bajada **[DUDOSO]**.
- **Ajustes a revisar:**
  1. System Management > Dual WAN > Edit: **Max Bandwidth Provided by ISP**. El valor por defecto es **512 kbit/s** de subida y de bajada. Si no se ajusta a la velocidad real, el balanceo y el QoS toman decisiones con datos irreales.
  2. Bandwidth Management (QoS): reglas mal hechas limitan el caudal.
  3. Port Setup: autonegociación y dúplex.
  4. Probar desactivando SPI de forma temporal. Unos reportan mejora y otros no.
  5. Cable y módem.
- Bug relacionado: CSCuc65750 (RV042G limitado a ~150 Mbps HTTP, corregido en 4.2.3.03). No aplica al RV082 por sus puertos 10/100.

### 8.6 Sesiones / conexiones concurrentes
- **No hay cifra oficial de sesiones concurrentes para el RV082** [laguna]. Para el RV042/RV042G, Cisco indica **12.000 conexiones concurrentes** (documento smb316). En un foro se menciona un límite de 72 sesiones por IP que nunca se confirmó **[DUDOSO]**.
- Síntomas de tabla saturada: páginas que no cargan con muchos usuarios o P2P, o reinicios bajo carga.
- Mitigación: limitar el P2P, usar QoS por servicio, revisar si hay malware que escanea la red y considerar el reemplazo.

### 8.7 DNS
- El router hace de **DNS proxy** (dnsmasq). Las consultas DNS **no se balancean** entre WAN.
- Bugs históricos:
  - nslookup de la UI no resolvía la Local DNS Database (4.0.0.7, corregido en 4.0.2.08);
  - fallos con "Use Remote DNS Server" de QuickVPN (corregido en 4.1.0.02);
  - Split DNS sobre VPN (corregido en 2.0.0.19).
- 2.0.0.19 aleatorizó el puerto origen de las consultas DNS (anti-Kaminsky).
- dnsmasq es vulnerable a DNSpooq hasta 2.82; el RV042 está oficialmente afectado y sin fix (§2.1). **Recomendación [INFERENCIA]:** apuntar los clientes a un resolvedor interno confiable o reforzado en lugar del proxy del router.
- Diagnóstico: System Management > Diagnostic > DNS Name Lookup (usa los DNS configurados en Setup > Network).

### 8.8 Dual WAN: el failover no funciona o no hace failback
**Cómo funciona** (Admin Guide y smb2985):
- Hay dos modos: **Smart Link Backup** (WAN primario + respaldo) y **Load Balance** (round robin ponderado).
- La caída se detecta con **Network Service Detection (NSD)**, que es **opcional** (casilla). Sin NSD, el router solo ve la caída del *link* físico.
- Parámetros:
  - Retry count: 5 por defecto.
  - Retry timeout: 30 s por defecto.
  - Destinos a los que hacer ping: Default Gateway, ISP Host, Remote Host y/o DNS Lookup Host.
  - **When Fail**: con "Generate the Error Condition in the System Log" **no hay failover**, solo se registra. Para que conmute hay que elegir **"Remove the Connection"** ("Keep System Log and Remove this Connection").
- Desde **4.2.3.03** el *recovery back timer* (retorno al WAN primario) es configurable.

**Causas típicas de failover fallido:**

| Síntoma | Causa | Solución |
|---|---|---|
| WAN1 sin Internet pero no conmuta | El módem o router del ISP sigue con el link Ethernet arriba y NSD está desactivado o solo hace ping al **Default Gateway** (el módem local, que sigue respondiendo) | Activar NSD y hacer ping a **Remote Host**/ISP Host públicos y estables (p. ej. dos IP de Internet distintas) además del DNS Lookup |
| Solo aparece en el log | When Fail = "Generate the Error Condition…" | Cambiar a "Remove the Connection" |
| No vuelve al WAN1 (failback) o tarda | Comportamiento reportado en Load Balance + Protocol Binding (RV042) **[DUDOSO]**; recovery timer | Ajustar el recovery timer (≥4.2.3.03); probar Smart Link Backup |
| Balanceo desigual | Bandwidth por defecto de 512 kbit/s en ambos WAN | Introducir los valores reales del ISP |
| VPN G2G no se recupera tras el failover | Bugs CSCum48584 (IP by DNS + Tunnel Backup, corregido en 4.2.3.03) y CSCtt29745 (ping tras failback de túnel, 4.1.1.01) | Actualizar; usar DPD + Tunnel Backup |
| SIP se corta en el failover | Bug corregido en 2.0.0.19 (V1/V2); SIP con varios teléfonos (CSCty22521, corregido en 4.2.2.08) | Actualizar; reenviar puertos SIP/RTP |
| En un foro de 2010: al reconectar el cable de WAN1 (aún sin servicio) vuelve a WAN1 y pierde Internet | Failback por estado de link sin verificar el servicio | NSD con hosts remotos **[DUDOSO]** |

### 8.9 Pérdida de configuración
- **CSCvk56058 "Configuration lost unexpectedly"**, corregido en **4.2.3.09**. Si hay pérdidas espontáneas de configuración, actualizar a 4.2.3.14.
- Mecanismo **Startup/Mirror**: la Mirror se copia automáticamente tras **24 h estables** (sin reinicios ni cambios). Si la Startup se corrompe, arranca con la Mirror.
  - **Trampa:** si la Mirror sigue en estado de fábrica, "Copy Mirror to Startup" **resetea el router a fábrica de inmediato**.
- **Downgrade de firmware = vuelve a fábrica** (Admin Guide y smb541). Ejemplo documentado: bajar de 4.0.2.08 a 4.0.0.07 resetea la configuración.
- Las configuraciones **no son compatibles entre V1/V2 y V3**. Cisco publicó una *Migration Utility* (1.0.2.2 y 1.0.2.8) para convertirlas.
- Al importar la configuración de otro router se **sobrescribe la MAC WAN**: corregirla en Setup > MAC Address Clone.
- Un apagón durante el guardado o la actualización puede corromper la configuración. Usar un SAI **[INFERENCIA]**.

### 8.10 Firmware: actualización y problemas
- Ruta: System Management > Firmware Upgrade. Tarda **~10 minutos** según el Apéndice B ("varios minutos" según otras secciones). **No apagar, no pulsar Reset, no cerrar el navegador y no desconectar** durante el proceso.
- Si falla, reintentar. Si el **DIAG parpadea de forma continua**, la imagen está dañada: recuperar con la **utilidad TFTP / firmware rescue utility** de Cisco.
- **Compatibilidad:**
  - el firmware 4.x (archivos `RV0XX-v4.2.3.xx-…-code.bin`) solo funciona en **HW V3**;
  - V2 llega como máximo a 2.0.2.01-tm (2.0.0.19 cubre V1.x y V2);
  - V2 no permite bajar a 1.x;
  - Cisco solo publica descargas para V3; las de V1/V2 no están disponibles **[DUDOSO, foro]**.
- Último firmware V3: **4.2.3.14 (jun-2020)**. Cisco publicó *release notes* incompletas para algunas versiones (p. ej. 4.2.3.08).

**Historial de cambios relevantes (release notes oficiales):**

| Versión | Fecha | Cambios relevantes |
|---|---|---|
| 1.3.9.8-tm (V1) | jul-2008 | ProtectLink; fix DoS UDP puerto 0; OpenSSL 0.9.6e; gestión remota HTTPS-only si HTTPS activo; 50 usuarios QuickVPN; MTU por WAN; SSLv2 off |
| 2.0.0.19-tm (V1.x/V2) | jul-2009 | Soporta V2 (ADM6996I); bloquea el downgrade a 1.x; nombre de admin configurable; fixes de VPN intermitente, log de logins fallidos, DHCP, failover SIP, DST, Split DNS; puerto origen DNS aleatorio; anti-DoS y anti-brute-force de QuickVPN (5 errores/min → bloqueo de 5 min); SSLv2 off. Puertos 60443 y 1723 no usables para gestión remota |
| 2.0.2.01-tm (V2) | jun-2011 | HTTP LAN → HTTPS; importar certificado externo (al exportarlo, la contraseña pasa a ser "password"); fix timeout QuickVPN; quita cifrados ≤128 bits. Problemas conocidos: login con Firefox tras reset, radio de MAC Clone, ProtectLink en IE8 |
| 4.0.0.7 (V3) | abr-2011 | Firmware inicial V3; sin dual-image. Problemas conocidos: rango de port triggering, MAC clone en RV042, la config importada sobrescribe la MAC WAN, nslookup, PPTP en iOS/Mac, **reenvío de 80/443 no funciona con HTTPS activo**, falta ICMP en servicios, ESP wildcard |
| 4.0.2.08 | may-2011 | IPv6 (6to4 activo por defecto); migration utility; fixes de port triggering, MAC clone, nslookup y ProtectLink |
| 4.0.3.03 | 2011 | ICMP en servicios; NAT LAN→DMZ; 1-to-1 NAT; Firefox 4; PPTP iOS; reenvío de 80/443 con HTTPS; **quita cifrados SSL débiles** |
| 4.0.4.02 | 2011 | ProtectLink tras reiniciar. Conocido: "Deny Policies" registra falsos "Policy Violation" |
| 4.1.0.02 | dic-2011 | Fixes de Firefox 4, falsos Policy Violation, NULL en fase 2, Remote DNS de QuickVPN, PPPoE con %. Conocido: 404 tras actualizar (reiniciar), sin NAT-T en Client-to-Gateway modo Tunnel, DHCPv6 tras reiniciar, **Keep-Alive no reconecta tras un power-cycle**, hostname DDNS de 15 caracteres |
| 4.1.1.01 | 2012 | Fixes de ping tras reconectar IPSec, UI inaccesible tras upgrade+restore, SIP, NAT de varias subredes a DMZ, crash con túnel a 0.0.0.0, MTU manual; PPPoE Service Name |
| 4.2.1.02 | jun-2012 | NAT-T en Tunnel avanzado (CSCtr93536); rango DHCP; ayuda de Client-to-Gateway. Conocido: UPnP "Linux Gateway Device", SIP (CSCty22521) |
| 4.2.2.08 | may-2013 | Zonas horarias (CSCug39617); caducidad de contraseña (CSCuf93284); **DoS FTP sobre PPTP** (CSCug25381); **overflow de libupnp** (CSCue20980); respuesta a sondeos en puertos 0/1 (CSCuc32147); IPSec tras corte de luz (CSCuc66316); **6to4 activo por defecto** (CSCue04572); SIP. Conocido: no bloquea HTTP por proxy (CSCug30363); lease DHCP del RV016; Connect-on-Demand de PPPoE mata IPv6 |
| 4.2.3.03 | abr-2014 | Autenticación SMTP en alertas; QuickVPN con NAT-T; **recovery timer de NSD**; SLAAC en WAN; fixes de email de logs que colgaba (CSCuh83563), VPN con varias subredes, DMZ accediendo a port forwarding (CSCuj47134), reconexión de G2G con IP by DNS (CSCum48584). Conocido: el IPSec cae al cortarse QuickVPN (CSCun67103) |
| 4.2.3.06 | may-2015 | **CVE-2015-6418** (CSCus15224), **OpenSSL 1.0.1l** (CSCuo18668), **POODLE** (CSCur53627) |
| 4.2.3.07 | ~2015-16 | (sin RN localizada) DH débil corregido **[DUDOSO]** |
| 4.2.3.08 | ~2016 | **Certificado SHA-1 → SHA-256** (CSCvb81527) |
| 4.2.3.09 | ~2018 | **Pérdida de configuración** (CSCvk56058) |
| 4.2.3.10 | oct/nov-2019 | Certificados, claves y hashes estáticos fuera; dnsmasq 2.80; OpenSSL 1.0.2s; libupnp 1.6.21; cookies débiles; auth_key hardcoded; **RCE CVE-2019-15271** (incluye el bug de RV082 CSCvq97031) |
| **4.2.3.14** | **jun-2020** | CVE-2020-3274…3279 y 3286…3296 (inyección de comandos y stack overflows, CSCvt29372…CSCvt29423); evaluación de pppd (CSCvt39795). **Último firmware** |

### 8.11 Syslog, logs, hora y NTP
- Log > System Log:
  - Syslog: activar e indicar el servidor; **hay que reiniciar** para que aplique.
  - Email: servidor SMTP, destinatario, Log Queue Length (50) y Log Time Threshold (10 min).
- Eventos:
  - **Alert Log**: Syn Flooding, IP Spoofing, Win Nuke, Ping of Death, Unauthorized Login Attempt, Output Blocking Event.
  - **General Log**: System Error, Deny/Allow Policies, Configuration Changes, Authorized Login.
- Visores: View System Log, Outgoing Log Table, Incoming Log Table, Clear Log Now. El log local es volátil.
- Bugs:
  - "Deny Policies" generaba falsos *Policy Violation* (corregido en 4.1.0.02);
  - logs incorrectos al conectar usuarios (CSCuc37893, 4.2.2.08);
  - el filtro "All" no mostraba nada (CSCui26504, 4.2.3.03);
  - log de intentos de login no autorizados (2.0.0.19).
- **Hora:** Setup > Time.
  - Por defecto: NTP con time.nist.gov y zona GMT-08:00 (Pacífico).
  - El DST se configura **a mano** con fechas de inicio y fin en formato mm.dd. No hay reglas automáticas por país, así que hay que actualizarlo si cambian las fechas.
  - Bugs de zonas horarias corregidos: India, Kabul, Mumbai, New Delhi, Kathmandu, Darwin (CSCug39617, 4.2.2.08) y fechas de DST inválidas (2.0.0.19).
  - Si el NTP no sincroniza: comprobar el DNS (System Management > Diagnostic > DNS Lookup de time.nist.gov), probar con una **IP** de NTP y revisar que las reglas de acceso no bloqueen UDP 123 saliente **[INFERENCIA]**.
  - La hora afecta a las reglas con horario, a los filtros de contenido y a los logs.

### 8.12 VPN (resumen de problemas conocidos; el detalle está en el documento de VPN)
- **QuickVPN:**
  - "Remote Gateway Not Responding": falta ICMP. Activar el Firewall de Windows y añadir una regla de entrada ICMPv4.
  - Requiere el servicio "IPSec Policy Agent", Remote Management en 443/60443, subredes local y remota distintas y una sola sesión por usuario.
  - En Vista/7 el Firewall de Windows debe estar **activado**. En XP SP2 hay un problema de ICMP (KB889527).
- **PPTP** no funcionaba con iPhone/iPad/Mac en 4.0.0.7/4.0.2.08 (corregido en 4.0.3.03).
- Keep-Alive no reconectaba tras un power-cycle (4.1.0.02): Disconnect manual en VPN > Summary.
- El IPSec no se restauraba tras un corte de luz (CSCuc66316, corregido en 4.2.2.08).
- No se soporta multicast por IPSec.

### 8.13 IPv6
- DHCPv6 no se reiniciaba tras un reboot (4.1.0.02): cambiar a IPv4, guardar y volver a Dual-Stack.
- RV016 sin IPv6 en el WAN tras reiniciar (CSCtx27745).
- PPPoE Connect-on-Demand mataba el demonio IPv6 (CSCuh13847): usar Keep Alive.

---

## 9. Herramientas de diagnóstico disponibles en el equipo [VERIFICADO, Admin Guide]

| Herramienta | Ruta en la UI | Uso |
|---|---|---|
| **DNS Name Lookup** | System Management > Diagnostic | Resolver un nombre con los DNS del WAN |
| **Ping** | System Management > Diagnostic | Estado, paquetes enviados/recibidos/% de pérdida y RTT mín/máx/medio |
| System Summary | System Summary | Firmware, uptime, estado de WAN y DMZ (IPv4/IPv6, Renew/Release), resumen de firewall y VPN |
| **System Statistics** | Log > System Statistics | Por interfaz (eth0…): IP, MAC, máscara, gateway, DNS, paquetes y bytes RX/TX, **paquetes con error y descartados** |
| **Port Status** | Port Management > Port Status | Link, velocidad, dúplex, autonegociación, VLAN y contadores (incluye errores) por puerto |
| System Log / Incoming / Outgoing Log Table | Log > System Log | Eventos, intentos de login, políticas aplicadas y tráfico entrante/saliente |
| Syslog remoto / Email alerts | Log > System Log | Envío externo (recomendado) |
| SNMP v1/v2c + traps | System Management > SNMP | Monitorización externa. Traps de reinicio, power-cycle y eventos WAN |
| DHCP Status | DHCP > DHCP Status | Concesiones activas |
| VPN Summary / logs | VPN > Summary | Estado de túneles (Connect/Disconnect) y fases |
| Network Service Detection | System Management > Dual WAN > Edit | Monitorización de WAN por ping, con eventos en el log |
| Backup / Restore / Copy Startup↔Mirror | System Management > Backup and Restore | Guardar y restaurar la configuración |
| Restart | System Management > Restart | Reinicio limpio que **envía el log antes de reiniciar** |
| Rescate de firmware | Utilidad TFTP / *firmware rescue utility* de Cisco (en el PC) | Recuperar un equipo con imagen dañada (DIAG parpadeando) |

**No hay** CLI, SSH, Telnet, traceroute, captura de paquetes ni **port mirroring** documentados en la Admin Guide del RV0xx. Para capturar tráfico, usar un switch con SPAN o un TAP delante o detrás del router.

---

## 10. Lagunas y datos dudosos (para tenerlos en cuenta al responder)

1. **Versiones TLS exactas** que acepta la web UI con 4.2.3.x: sin documentación oficial.
2. **Fix del RV082 para CVE-2019-15271/15957/15990:** las advisories de 2019 no lo listan, pero las RN 4.2.3.10/4.2.3.14 y las advisories de 2020 sugieren que la imagen RV0xx 4.2.3.10+ sí lo cubre en HW V3 (§2).
3. **CVE-2023-20118, ¿autenticada o no?** Cisco dice PR:H (admin); Sekoia, "unauthenticated". Encadenada con CVE-2023-20025 sería no autenticada en RV016/RV042/RV042G/RV082 **[INFERENCIA]**.
4. **RV082 en DNSpooq y en las CVE-2024-20516…20524:** Cisco no lo lista, aunque comparte la imagen con el RV042.
5. **Fechas exactas** de 4.2.3.07, 4.2.3.08 y 4.2.3.09: no localicé esas release notes. Las fechas son aproximadas.
6. **CPU del RV082 V1/V3** según fuentes de comunidad (Intel IXP / Cavium Octeon CN50XX).
7. **Límite de sesiones del RV082:** no publicado.
8. **SNMP en el WAN:** no está documentado si escucha en la interfaz WAN.
9. **Banners/fingerprints concretos** (cabecera Server, título HTML): no hay fuentes públicas fiables.
10. **Cómo se almacena la contraseña en los `.config`:** sin documentar.
11. Discrepancia en el **Last Date of Support** (31-may-2021 en la EoL notice frente a 05-may-2021 en otra página de Cisco).
12. Reset por botón: 30 s (oficial) frente a 10 s (no oficial).

---

## 11. Fuentes

### Cisco Security Advisories / PSIRT
- cisco-sa-sbr042-multi-vuln-ej76Pke5 (CVE-2023-20025/20026/20118): https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-sbr042-multi-vuln-ej76Pke5
- cisco-sa-sbiz-rv-xss-OQeRTup (CVE-2024-20362): https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-sbiz-rv-xss-OQeRTup
- cisco-sa-sb-rv01x_rv32x_rce-nzAGWWDD (CVE-2023-20124): https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-sb-rv01x_rv32x_rce-nzAGWWDD
- cisco-sa-rv-stored-xss-vqz7gC8W (CVE-2023-20137…20151): https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-rv-stored-xss-vqz7gC8W
- cisco-sa-sbrv-cmdinjection-Z5cWFdK (CVE-2021-40120): https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-sbrv-cmdinjection-Z5cWFdK
- cisco-sa-rv-command-inject-BY4c5zd (CVE-2021-1314…1318): https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-rv-command-inject-BY4c5zd
- cisco-sa-rv-overflow-ghZP68yj (CVE-2021-1319…1348): https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-rv-overflow-ghZP68yj
- cisco-sa-rv-routers-Rj5JRfF8 (CVE-2020-3274…3279): https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-rv-routers-Rj5JRfF8
- cisco-sa-rv-routers-stack-vUxHmnNz (CVE-2020-3286…3296): https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-rv-routers-stack-vUxHmnNz
- cisco-sa-20191106-sbrv-cmd-x (CVE-2019-15271): https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-20191106-sbrv-cmd-x
- cisco-sa-20191106-sbr-cominj (CVE-2019-15957): https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-20191106-sbr-cominj
- cisco-sa-20191120-sbr-rv-infodis (CVE-2019-15990): https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-20191120-sbr-rv-infodis
- cisco-sa-20191106-rv0x2 (Issues; versión japonesa consultada): https://www.cisco.com/c/ja_jp/support/docs/csa/2019/cisco-sa-20191106-rv0x2.html
- cisco-sa-20151210-dwvr (CVE-2015-6418): https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-20151210-dwvr
- cisco-sa-sb-rv04x_rv32x_vulns-yJ2OSDhV (CVE-2024-20516…20524, RV042/RV32x): https://www.cisco.com/c/en/us/support/docs/csa/cisco-sa-sb-rv04x_rv32x_vulns-yJ2OSDhV.html
- cisco-sa-dnsmasq-dns-2021-c5mrdf3g (DNSpooq): https://sec.cloudapps.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-dnsmasq-dns-2021-c5mrdf3g
- EoL notice RV082: https://www.cisco.com/c/en/us/products/collateral/routers/small-business-rv-series-routers/eos-eol-notice-c51-736749.html

### NVD / CISA
- NVD API (búsqueda por "RV082" y CPE): https://services.nvd.nist.gov/rest/json/cves/2.0?keywordSearch=RV082
- NVD CVE-2023-20025: https://nvd.nist.gov/vuln/detail/CVE-2023-20025 · CVE-2023-20118: https://nvd.nist.gov/vuln/detail/CVE-2023-20118 · CVE-2019-15271: https://nvd.nist.gov/vuln/detail/CVE-2019-15271
- CISA KEV (JSON): https://www.cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json · https://www.cisa.gov/known-exploited-vulnerabilities-catalog?field_cve=CVE-2023-20118

### Documentación y release notes de Cisco
- Administration Guide RV0xx (78-19576): https://www.cisco.com/c/dam/en/us/td/docs/routers/csbr/rv0xx/administration/guide/rv0xx_AG_78-19576.pdf
- Release Notes 4.2.3.14: https://www.cisco.com/c/dam/en/us/td/docs/routers/csbr/rv0xx/release/RV0xx_V_4-2-3-14.pdf
- Release Notes 4.2.3.06: http://www.cisco.com/c/dam/en/us/td/docs/routers/csbr/rv0xx/release/RV0xx_RN_4_2_3_06.pdf (vía web.archive.org)
- Release Notes 4.2.3.03: http://www.cisco.com/c/dam/en/us/td/docs/routers/csbr/rv0xx/release/RV0xx_RN_4-2-3-03.pdf (vía web.archive.org)
- Release Notes 4.2.2.08: http://www.cisco.com/en/US/docs/routers/csbr/rv0xx/release/Rv0xx_RN_v3_42208.pdf (vía web.archive.org)
- Release Notes 4.2.1.02 (FR): http://www.cisco.com/c/dam/en/us/td/docs/routers/csbr/rv0xx/release-FR/rv0xx_rn_4-2-1-02_fr.pdf (vía web.archive.org)
- Release Notes 4.1.1.01 y 4.1.0.02: http://www.cisco.com/c/dam/en/us/td/docs/routers/csbr/rv0xx/release/rv0xx_rn_v4-1-1-01.pdf · .../rv0xx_rn_v4-1-0-02_OL-26196.pdf (vía web.archive.org)
- Release Notes 4.0.0.7, 4.0.2.08, 4.0.3.03, 4.0.4.02: http://www.cisco.com/en/US/docs/routers/csbr/rv0xx/release/rv0xx_rn_v4-0-0-7.pdf · rv0xx_rn_v4-0-2-08.pdf · RV0xx_RN_v4-0-3-3.pdf · RV0xx_RN_v4-0-4-2.pdf (vía web.archive.org)
- Release Notes RV082 1.3.9.8-tm, 2.0.0.19-tm y 2.0.2.01-tm: http://www.cisco.com/en/US/docs/routers/csbr/rv082/release/notes/RV082_firmware_v1.3.9.8-tm_RN.pdf · RV082_RN_v2-0-0-19-tm.pdf · RV082_RN_v2-0-2-01-tm.pdf
- Open Source 4.2.3.07: http://www.cisco.com/c/dam/en/us/td/docs/routers/csbr/rv0xx/release/Cisco_RV0xx_Series_Routers_4_2_3_07.pdf · Open Source 4.2.3.10: https://www.cisco.com/c/dam/en/us/td/docs/routers/csbr/rv0xx/osln/RV0xxRouters42310.pdf
- Lista de release notes RV: https://www.cisco.com/c/en/us/support/routers/small-business-rv-series-routers/products-release-notes-list.html
- smb288 Reboot/Factory Default: https://www.cisco.com/c/en/us/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb288-reboot-and-restore-to-factory-default-settings-on-rv016-rv04.html
- smb541 Firmware Upgrade: https://www.cisco.com/c/en/us/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb541-firmware-upgrade-on-rv016-rv042-rv042g-and-rv082-vpn-routers.html
- smb2985 Smart Link Backup: https://www.cisco.com/c/en/us/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb2985-configure-smart-link-backup-failover-on-rv042-rv042g-and-rv0.html
- smb2922 QuickVPN tips: https://www.cisco.com/c/en/us/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb2922-cisco-quickvpn-installation-tips-for-windows-operating-syste.html
- smb316 (RV042, 12.000 conexiones): https://www.cisco.com/c/en/us/support/docs/smb/routers/cisco-rv-series-small-business-routers/smb316-get-to-know-the-rv042-and-rv042g-vpn-routers.html

### Inteligencia de amenazas / prensa
- Sekoia, PolarEdge: https://www.sekoia.com/blog/polaredge-unveiling-an-uncovered-iot-botnet
- Censys, PolarEdge Adjacent Infrastructure: https://censys.com/blog/a-look-at-polaredge-adjacent-infrastructure
- XLab, RPX Relay PolarEdge: https://blog.xlab.qianxin.com/smoking-gun-uncovered-rpx-relay-at-polaredges-core-exposed/
- The Hacker News, PolarEdge (feb-2025): https://thehackernews.com/2025/02/polaredge-botnet-exploits-cisco-and.html
- The Hacker News, ViciousTrap (may-2025): https://thehackernews.com/2025/05/vicioustrap-uses-cisco-flaw-to-build.html
- TechRadar, ViciousTrap: https://www.techradar.com/pro/security/cisco-security-flaw-exploited-to-build-botnet-of-thousands-of-devices
- SecurityWeek, KEV mar-2025: https://www.securityweek.com/exploitation-long-known-for-most-of-cisas-latest-kev-additions/
- The Hacker News, CVE-2023-20025 (ene-2023): https://thehackernews.com/2023/01/cisco-issues-warning-for-unpatched.html
- The Register (ene-2023): https://www.theregister.com/2023/01/13/cisco_smb_critical_router_flaw_no_fix/
- Security Affairs, XSS EoL (abr-2024): https://securityaffairs.com/161540/security/cisco-eof-routers-xss.html
- CSA Singapur AL-2023-006: https://csa.gov.sg/alerts-and-advisories/alerts/al-2023-006
- Qualys ThreatProtect (20025/20026): https://threatprotect.qualys.com/2023/01/16/cisco-eol-small-business-vpn-routers-multiple-vulnerabilities-cve-2023-20025-cve-2023-20026/
- GitHub lnversed/CVE-2023-20025: https://github.com/lnversed/CVE-2023-20025

### Comunidad (fuentes anecdóticas, marcadas [DUDOSO])
- RV082 restarts itself: https://community.cisco.com/t5/routers-small-business/rv082-restarts-itself/m-p/1451984
- RV082/RV042/RV042G V4 firmware bug: https://community.cisco.com/t5/routers-small-business/rv082-rv042-rv042g-v4-firmware-bug/m-p/2102302
- RV082 Failover Issues: https://community.cisco.com/t5/routers-small-business/rv082-failover-issues/td-p/1483067
- RV042 load balance no failback: https://community.cisco.com/t5/routers-small-business/rv042-load-balance-protocol-binding-not-failing-back/m-p/2163486
- RV042 4.2.3.06 weak ephemeral DH: https://community.cisco.com/t5/routers-small-business/rv042-w-firmware-4-2-3-06-quot-server-has-a-weak-ephemeral/m-p/2730995
- Cannot login to RV042 (SSL 3.0 / 4.2.3.08): https://community.cisco.com/t5/routers-small-business/cannot-login-to-rv042-or-rv042g/m-p/4720644
- RV082 password from backup: https://community.cisco.com/t5/small-business-support-documents/can-rv082-password-be-decode-decrypted-from-my-backup/ta-p/3110157
- RV082 speed limit to WAN: https://community.cisco.com/t5/small-business-routers/rv082-speed-limit-to-wan/td-p/2429908
- RV042 connection limit: https://community.cisco.com/t5/routers-small-business/rv042-connection-limit/td-p/1565754
- RV016/RV042/RV082 HW V3 firmware: https://community.cisco.com/t5/small-business-support-knowledge-base/rv016-rv042-rv082-hardware-v3-firmware-update/ta-p/3125488
- OpenWrt, RV0xx V3 (Octeon Cavium): https://forum.openwrt.org/t/support-for-cisco-rv016-rv042-rv042g-rv082-v3-octeon-cavium/6716
