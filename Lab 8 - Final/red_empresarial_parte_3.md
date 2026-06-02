# Red Empresarial — Parte 3

## Examen Final Práctico · CCNA III · Packet Tracer

---

## Índice

- [[#1. Información General y Topología]]
- [[#2. Tabla de Direccionamiento]]
- [[#3. Topología y Conexiones Físicas]]
- [[#4. Enrutamiento entre Dominios — BGP]]
    - [[#4.1 OSPF vs BGP — qué cambia]]
    - [[#4.2 El backbone internacional y el IXP]]
    - [[#4.3 Sistemas Autónomos (AS) y números de AS]]
    - [[#4.4 Sintaxis CLI explicada]]
    - [[#4.5 Sesiones y redes anunciadas]]
- [[#5. Túnel GRE entre Sitios]]
- [[#6. NAT / PAT en R1 y R2]]
- [[#7. Redes de Cliente Simuladas]]
    - [[#Red Hogareña (HOGAR)]]
    - [[#Red SOHO (pequeña empresa)]]
- [[#8. Notas de Servicios y Alcance]]
- [[#9. Checklist de Aprobación]]
- [[#Imagen referencial red final]]

---

## 1. Información General y Topología

Esta tercera y última parte conecta los dos sitios construidos antes —**Chile (R1)** y **España (R2)**— a través de un **backbone internacional** simulado, e incorpora tres técnicas nuevas respecto a las partes 1 y 2: **enrutamiento BGP** entre operadores, un **túnel GRE** punto a punto entre los routers de borde, y **NAT/PAT** hacia el "exterior". Además se agregan **dos redes de cliente** que cuelgan de un ISP para ilustrar cómo un mismo proveedor entrega direcciones públicas distintas a clientes que internamente usan el mismo direccionamiento privado.

El escenario se divide en:

- **Borde corporativo:** R1 (Chile, AS65001) y R2 (España, AS65002). Cada uno habla BGP con su ISP y termina un extremo del túnel GRE.
- **Backbone de tránsito:** **Movistar** (AS701, operador del lado **Chile**) ↔ **IXP** (AS24115, punto de intercambio neutral) ↔ **Orange** (AS9299, operador del lado **España/Europa**). Tres routers CISCO2911 que solo transportan **direcciones públicas**.
- **Redes de cliente:** una **red hogareña** y una **pequeña empresa SOHO**, ambas colgando de **Movistar**, cada una detrás de su propio módem con NAT.

> [!important] **Novedad respecto a las partes 1 y 2: BGP.** Hasta ahora todo el ruteo fue **OSPF**, que es un protocolo _interior_ (funciona dentro de una misma organización). Para unir redes de **distintas organizaciones** (operadores que no comparten administración) se usa **BGP**, el protocolo que mantiene unida a la Internet real. Es la primera vez que se aplica en esta red, por lo que la sección 4 explica el concepto antes que la configuración.

> [!important] **Novedad: túnel GRE + NAT.** El backbone solo entiende direcciones públicas, pero los dos sitios usan direccionamiento privado (192.168.x / 172.16.x). Para que Chile y España se "vean" sin exponer sus redes internas, se construye un **túnel GRE** que las encapsula. El tráfico hacia la Internet "real" (no inter-sitio) sale traducido por **NAT/PAT**.

> [!info] Esta parte **no agrega VLANs nuevas**. El backbone trabaja con enlaces ruteados (sin conmutación de VLANs) y la única LAN conmutada nueva es la del SOHO, que queda en la **VLAN 1 por defecto**.

> [!note] **Marcas usadas solo con fines educativos.** Los nombres **Movistar** y **Orange** se emplean únicamente para dar realismo a la simulación; no implican afiliación ni respaldo de dichas empresas. Las direcciones "públicas" tampoco son reales: siguen los rangos de documentación **RFC 5737** (`203.0.113.0/24`) y **RFC 6598 / CGNAT** (`100.64.0.0/10`).

> [!example] Topología internacional y redes de cliente ![[Excalidraw/lab_final_pt.excalidraw.md#^frame=Hth1H91a|center|1800]]

---

## 2. Tabla de Direccionamiento

> [!info] **Direcciones de documentación.** Las "públicas" de esta práctica no son reales: `203.0.113.0/24` es el bloque **TEST-NET-3** reservado para documentación (RFC 5737) y `100.64.0.0/10` es el espacio **CGNAT** compartido entre operadores (RFC 6598). Se usan precisamente para simular Internet sin colisionar con direcciones reales.

### Enlaces del backbone (/30)

|Subred|Interfaz extremo 1|IP extremo 1|Interfaz extremo 2|IP extremo 2|Medio|
|---|---|---|---|---|---|
|203.0.113.0/30|R1 G0/0/0|203.0.113.1|Movistar G0/0|203.0.113.2|Eth|
|100.64.1.0/30|Movistar S0/3/0|100.64.1.2|IXP S0/3/0|100.64.1.1|Serial|
|100.64.2.0/30|IXP S0/3/1|100.64.2.1|Orange S0/3/0|100.64.2.2|Serial|
|203.0.113.4/30|Orange G0/0|203.0.113.5|R2 G0/0/0|203.0.113.6|Eth|

### Enlaces a redes de cliente (/30, cuelgan de Movistar)

|Subred|Interfaz Movistar|IP Movistar|Cliente (WAN del módem)|IP cliente|Tipo|
|---|---|---|---|---|---|
|100.64.3.0/30|Movistar G0/1|100.64.3.1|Módem HOGAR|100.64.3.2|CGNAT|
|203.0.113.8/30|Movistar G0/2|203.0.113.9|Módem SOHO|203.0.113.10|Pública|

### Túnel GRE (lógico, extremo a extremo)

|Interfaz|Dispositivo|IP|Máscara|Parámetros|
|---|---|---|---|---|
|Tunnel0|R1|192.168.100.1|255.255.255.252 /30|source G0/0/0, dest 203.0.113.6, MTU 1476|
|Tunnel0|R2|192.168.100.2|255.255.255.252 /30|source G0/0/0, dest 203.0.113.1, MTU 1476|

### LAN de las redes de cliente

|Red|Subred LAN|Gateway (módem)|Asignación de hosts|
|---|---|---|---|
|HOGAR|192.168.1.0/24|192.168.1.1|DHCP del módem (desde 192.168.1.100)|
|SOHO|192.168.1.0/24|192.168.1.1|Estática: SV y PCs en 192.168.1.10–.13|

> [!important] Las **dos** redes de cliente usan **la misma `192.168.1.0/24`** a propósito. No chocan porque cada módem las **traduce con NAT** a su propia dirección pública: lo que viaja por el backbone nunca es `192.168.1.x`, sino la pública del módem. Demostrar esto es el objetivo pedagógico de la sección 7.

---

## 3. Topología y Conexiones Físicas

Movistar, IXP e Orange son routers **CISCO2911**, con interfaces GigabitEthernet `G0/0`–`G0/2` y seriales `S0/3/0`–`S0/3/1`.

### Enlaces Ethernet (extremos del backbone)

|Enlace|Puertos|
|---|---|
|R1 ↔ Movistar|R1 G0/0/0 ↔ Movistar G0/0|
|Orange ↔ R2|Orange G0/0 ↔ R2 G0/0/0|
|Movistar ↔ HOGAR|Movistar G0/1 ↔ módem HOGAR (WAN)|
|Movistar ↔ SOHO|Movistar G0/2 ↔ módem SOHO (WAN)|

### Enlaces seriales (tránsito por el IXP)

|Enlace|Puertos|Extremo DCE (lleva `clock rate`)|
|---|---|---|
|Movistar ↔ IXP|Movistar S0/3/0 ↔ IXP S0/3/0|**IXP** (4000000)|
|IXP ↔ Orange|IXP S0/3/1 ↔ Orange S0/3/0|**IXP** (4000000)|

> [!warning] **El cable serial tiene un extremo DCE y uno DTE.** El `clock rate` se aplica en el extremo **DCE**, que en ambos enlaces es el **IXP**. Si el cable se conecta al revés (DCE del lado del ISP), el enlace queda `down` y las sesiones BGP no levantan. Es el error más común al armar este tramo.

---

## 4. Enrutamiento entre Dominios — BGP

### 4.1 OSPF vs BGP — qué cambia

En las partes 1 y 2 se usó **OSPF**, un **IGP** (_Interior Gateway Protocol_): enruta **dentro** de una sola organización, donde todos los routers son administrados por la misma entidad y "confían" entre sí.

El backbone internacional conecta **organizaciones distintas** (operadores que no comparten administración). Para eso existe **BGP** (_Border Gateway Protocol_), un **EGP** (_Exterior Gateway Protocol_): el protocolo que enruta **entre** organizaciones y que, en el mundo real, mantiene unida a toda la Internet.

> [!info] La buena noticia: la **sintaxis** de BGP se parece a la de OSPF (`router ...`, `network ...`). Lo nuevo es el concepto de **Sistema Autónomo** y el hecho de que los vecinos se declaran **a mano** (BGP no descubre vecinos solo con _hellos_ como OSPF).

### 4.2 El backbone internacional y el IXP

El **backbone** es el conjunto de enlaces de tránsito (los /30 seriales y Ethernet de la sección 2) que transporta el tráfico entre los dos sitios y hacia "Internet". **Por el backbone solo circulan direcciones públicas**; las redes privadas corporativas nunca aparecen ahí (ver sección 5).

El **IXP** (_Internet Exchange Point_) es un punto de intercambio neutral donde **varios operadores se interconectan** para intercambiar tráfico directamente, en lugar de pagar tránsito a terceros. En esta topología el IXP (AS24115) se ubica **en el medio** entre Movistar e Orange y **propaga las rutas BGP** de un operador al otro: es el punto de encuentro de las dos "carreras" del backbone.

### 4.3 Sistemas Autónomos (AS) y números de AS

Un **Sistema Autónomo (AS)** es una red bajo una **única administración**, identificada por un **número de AS (ASN)**. Cada operador y cada gran organización tiene el suyo.

Igual que con las direcciones IP, los ASN se dividen en **públicos** y **privados**:

|ASN en esta red|Dispositivo|Tipo|Significado|
|---|---|---|---|
|**701**|Movistar|Público|ASN registrable que representa a un **operador de tránsito**|
|**24115**|IXP|Público|ASN registrable que representa al **punto de intercambio (IXP)**|
|**9299**|Orange|Público|ASN registrable que representa al **otro operador de tránsito**|
|**65001**|R1 (Chile)|**Privado**|Borde corporativo. Rango privado 64512–65534 (análogo a IP privada)|
|**65002**|R2 (España)|**Privado**|Borde corporativo. Rango privado 64512–65534|

> [!info] **Por qué 701/24115/9299 vs 65001/65002.** Los tres primeros son ASN del rango **público/registrable** (menores a 64512): en el mundo real se asignan formalmente a operadores e IXPs, y aquí se usan para dar realismo. Los dos últimos (65001, 65002) pertenecen al rango **privado** (64512–65534), reservado para uso interno o de laboratorio —exactamente la misma idea que usar `192.168.x.x` en una LAN—. Por eso el borde de cada empresa usa un ASN privado y los operadores usan ASN públicos.

> [!info] **eBGP.** Cuando dos vecinos BGP están en **AS distintos**, la sesión se llama **eBGP** (externa). En esta red **todas** las sesiones cruzan una frontera de AS (R1↔Movistar, Movistar↔IXP, IXP↔Orange, Orange↔R2), así que **todas son eBGP**.

### 4.4 Sintaxis CLI explicada

La configuración de un router BGP tiene esta forma (ejemplo de Movistar, AS701):

```
router bgp 701
 no synchronization
 bgp log-neighbor-changes
 neighbor 203.0.113.1 remote-as 65001
 neighbor 100.64.1.1 remote-as 24115
 network 203.0.113.0 mask 255.255.255.252
```

Línea por línea:

- **`router bgp 701`** → inicia BGP y declara que **este router pertenece al AS 701**. Un router solo puede estar en **un** AS local.
- **`neighbor <IP> remote-as <ASN>`** → declara un **vecino manualmente**. La `<IP>` es la del otro extremo del enlace; `remote-as <ASN>` indica **a qué AS pertenece ese vecino**.
    - Si el `remote-as` del vecino **es distinto** del AS local → sesión **eBGP**.
    - BGP **no descubre vecinos solo**: si no se declara el `neighbor`, no hay sesión. (Diferencia clave con OSPF.)
- **`network <red> mask <máscara>`** → indica qué prefijos **origina/anuncia** este router hacia sus vecinos. Solo se anuncia si existe **exactamente** esa red en la tabla de ruteo.
    - **Solo se anuncian direcciones públicas.** Las redes privadas corporativas **nunca** se ponen aquí.
- **`bgp log-neighbor-changes`** → registra en el log/Syslog cada vez que un **vecino cambia de estado** (sube o cae la sesión). No afecta el ruteo; es **visibilidad operativa** que ayuda muchísimo a diagnosticar (se ve al instante cuándo un peer se cayó).
- **`no synchronization`** → desactiva una regla heredada (que BGP esperara a que el IGP aprendiera el prefijo antes de anunciarlo). En IOS moderno suele venir desactivada por defecto; se deja explícita por compatibilidad.

> [!tip] Mapa mental: `router ospf` ≈ `router bgp` (ambos abren el proceso), y `network` cumple un rol parecido (decir "esto lo anuncio yo"). Lo **único realmente nuevo** es `neighbor ... remote-as`, porque en BGP el vecino se declara a mano.

### 4.5 Sesiones y redes anunciadas

#### Sesiones eBGP (vecinos)

|Sesión|Lado A (IP / AS)|Lado B (IP / AS)|
|---|---|---|
|R1 ↔ Movistar|203.0.113.1 / AS65001|203.0.113.2 / AS701|
|Movistar ↔ IXP|100.64.1.2 / AS701|100.64.1.1 / AS24115|
|IXP ↔ Orange|100.64.2.1 / AS24115|100.64.2.2 / AS9299|
|Orange ↔ R2|203.0.113.5 / AS9299|203.0.113.6 / AS65002|

#### Prefijos anunciados por cada dispositivo

|Dispositivo|Prefijos en `network`|Comentario|
|---|---|---|
|R1|203.0.113.0/30|Solo su /30 público (extremo del túnel)|
|Movistar|203.0.113.0/30 · 100.64.1.0/30 · 100.64.3.0/30 · 203.0.113.8/30|Sus enlaces + los /30 de los clientes|
|IXP|100.64.1.0/30 · 100.64.2.0/30|Sus dos enlaces de tránsito|
|Orange|203.0.113.4/30 · 100.64.2.0/30|Su enlace al cliente R2 + tránsito|
|R2|203.0.113.4/30|Solo su /30 público (extremo del túnel)|

> [!important] **Ningún borde anuncia sus redes internas.** R1 y R2 publican **solo** su /30 público. Las redes `192.168.x`, `172.16.31.x` y `10.255.x` son privadas, se traducen con NAT (sección 6) y **jamás entran a BGP**: así no se "filtra" el direccionamiento corporativo hacia la Internet simulada.

---

## 5. Túnel GRE entre Sitios

### Por qué un túnel

El backbone solo rutea **direcciones públicas**, y los dos sitios usan **direccionamiento privado** que no es ruteable por Internet. Entonces, ¿cómo se hablan Chile y España?

La respuesta es el **túnel GRE**: encapsula cada paquete privado **dentro** de un paquete con direcciones **públicas** (origen/destino = los extremos del túnel, `203.0.113.1` y `203.0.113.6`). El backbone solo ve el "sobre" público y lo entrega; el contenido privado viaja **invisible** para él.

> [!important] **Esto es lo que conecta BGP con el túnel.** Para que el túnel levante, R1 debe **alcanzar** la IP pública de R2 (`203.0.113.6`) y viceversa. Esa alcanzabilidad la provee **BGP**: Orange anuncia `203.0.113.4/30` y Movistar anuncia `203.0.113.0/30`, y el IXP propaga ambas. Es decir: **BGP hace visible el destino del túnel**, y el **túnel** transporta lo privado. Por eso "las redes internas no pasan por el backbone": pasan **encapsuladas**, sin que el backbone conozca ni una sola ruta privada.

### Parámetros del túnel

|Parámetro|R1|R2|
|---|---|---|
|Dirección IP|192.168.100.1 /30|192.168.100.2 /30|
|`tunnel source`|G0/0/0 (203.0.113.1)|G0/0/0 (203.0.113.6)|
|`tunnel destination`|203.0.113.6|203.0.113.1|
|MTU|1476|1476|

> [!info] La MTU se baja a **1476** porque la cabecera GRE (24 bytes) se resta de los 1500 habituales. Si se deja en 1500, paquetes grandes pueden fragmentarse o descartarse.

### Rutas estáticas cruzadas (lo que hace útil al túnel)

Un túnel levantado **no enruta nada por sí solo**: hay que decirle qué tráfico mandar por él. Cada borde lleva **rutas estáticas** hacia las redes del **otro** sitio, apuntando a la IP del túnel del vecino:

```
! En R1 (hacia España) — ejemplo:
ip route 192.168.40.0 255.255.255.0 192.168.100.2
ip route 192.168.50.0 255.255.255.0 192.168.100.2
...

! En R2 (hacia Chile) — ejemplo:
ip route 192.168.10.0 255.255.255.0 192.168.100.1
ip route 192.168.20.0 255.255.255.0 192.168.100.1
...
```

> [!info] Se usa **ruteo estático** (no OSPF) sobre el túnel porque **OSPF sobre GRE no es estable en Packet Tracer**. R1 lista todas las subredes de España; R2 todas las de Chile. (Las subredes de la sucursal `172.16.31.x` ya las alcanza R2 vía R3 desde la parte 2: **no** se vuelven a rutear por el túnel.)

---

## 6. NAT / PAT en R1 y R2

El tráfico que **no** es inter-sitio (el que iría a la "Internet real") sale **traducido** por **PAT** (NAT con sobrecarga) en la interfaz pública de cada borde.

### Marcado de interfaces

|Router|`ip nat inside` (interno)|`ip nat outside` (público)|
|---|---|---|
|R1|G0/1|G0/0/0|
|R2|G0/0, G0/1, G0/2|G0/0/0|

> [!warning] **Punto crítico de corrección.** El `ip nat inside` debe ir en la interfaz **interna real**. En R1 esa interfaz es **G0/1** (enlace a MLS1). Si se marca por error G0/0 (como sugería un esquema antiguo), la traducción no ocurre. En R2 las tres internas (G0/0/G0/1/G0/2) llevan `inside`.

### ACL de NAT (lista 1)

Cada borde usa una **ACL estándar numerada `1`** que enumera **todas sus subredes internas** (las que deben traducirse al salir a Internet). Luego se aplica el PAT:

```
ip nat inside source list 1 interface G0/0/0 overload
```

Y una **ruta por defecto** hacia el ISP, para que el tráfico traducido tenga salida:

|Router|Ruta por defecto|
|---|---|
|R1|`ip route 0.0.0.0 0.0.0.0 203.0.113.2`|
|R2|`ip route 0.0.0.0 0.0.0.0 203.0.113.5`|

> [!important] **Por qué el tráfico inter-sitio NO se NATea.** El tráfico hacia el otro sitio se rutea primero al **Tunnel0** y se encapsula en GRE; cuando sale por `G0/0/0` (la interfaz _outside_) **ya lleva IP pública de origen** (`203.0.113.1`/`.6`), que **no** está en la ACL 1 → no se traduce. Solo el tráfico con **origen privado** que va a Internet matchea la ACL y se PAT-ea. Este detalle es lo que resolvió el problema de "doble NAT" entre los dos sitios.

---

## 7. Redes de Cliente Simuladas

Ambas redes cuelgan de **Movistar** (puertos libres `G0/1` y `G0/2`) y cada una está detrás de su propio **Wireless Router** de Packet Tracer, que actúa como **módem con NAT**.

> [!warning] Usar el **Wireless Router** (no el "Cable/DSL Modem"). En Packet Tracer el módem puro **no hace NAT ni DHCP** (solo pasa cables); el Wireless Router sí trae NAT/PAT y DHCP, que es lo que se necesita para simular "público afuera, privado adentro".

> [!info] **El NAT lo hace el módem, no el ISP.** El ISP solo **asigna y rutea** una dirección pública a la WAN de cada módem; la traducción `192.168.1.0/24 → pública` ocurre **dentro del módem**. Por eso ambas redes pueden usar la misma `192.168.1.0/24` sin chocar.

### Red Hogareña (HOGAR)

Simula la conexión de un usuario doméstico común. Recibe **CGNAT** (`100.64.3.0/30`), tal como hacen los operadores reales con clientes residenciales.

**Enlace al ISP:** módem WAN `100.64.3.2/30` ↔ Movistar G0/1 (`100.64.3.1`).

#### En el módem (GUI del Wireless Router)

- **Internet Setup → Static IP:**
    - IP Address: `100.64.3.2`
    - Subnet Mask: `255.255.255.252`
    - Default Gateway: `100.64.3.1`
- **Network Setup (LAN):** Router IP `192.168.1.1 / 255.255.255.0`, **DHCP Server: Enabled** (inicio en `192.168.1.100`).
- **Wireless:** SSID `HOGAR_WIFI`, seguridad WPA2 Personal.

#### Clientes

|Dispositivo|Conexión|Direccionamiento|
|---|---|---|
|1 PC|Cable (LAN)|DHCP|
|1 Laptop|WLAN (HOGAR_WIFI)|DHCP|
|1 Smartphone|WLAN (HOGAR_WIFI)|DHCP|

### Red SOHO (pequeña empresa)

Simula una oficina pequeña. Recibe una **dirección pública** (`203.0.113.8/30`) y conecta el módem a un **switch** que alimenta a un servidor y tres PCs con **IP estática**.

**Enlace al ISP:** módem WAN `203.0.113.10/30` ↔ Movistar G0/2 (`203.0.113.9`).

#### En el módem (GUI del Wireless Router)

- **Internet Setup → Static IP:**
    - IP Address: `203.0.113.10`
    - Subnet Mask: `255.255.255.252`
    - Default Gateway: `203.0.113.9`
- **Network Setup (LAN):** Router IP `192.168.1.1 / 255.255.255.0`, **DHCP Server: Disabled** (los hosts van con IP estática).
- Puerto **LAN del módem → SW_SOHO**.

#### SW_SOHO

Switch 2960 con todos los puertos en la **VLAN 1 por defecto**. Solo se le asigna el hostname `SW_SOHO`; no requiere más configuración.

#### Hosts SOHO (GUI → Desktop → IP Configuration → Static)

|Host|IP|Máscara|Gateway|
|---|---|---|---|
|SV_SOHO|192.168.1.10|255.255.255.0|192.168.1.1|
|PC1|192.168.1.11|255.255.255.0|192.168.1.1|
|PC2|192.168.1.12|255.255.255.0|192.168.1.1|
|PC3|192.168.1.13|255.255.255.0|192.168.1.1|

---

## 8. Notas de Servicios y Alcance

> [!info] **DNS de los clientes.** El DNS corporativo (`192.168.30.2`) es **privado** y no es alcanzable desde el lado "Internet". Los hosts de HOGAR y SOHO podrán hacer **ping a direcciones públicas**, pero **no resolverán nombres** contra el DNS interno. Como el objetivo de estas redes es demostrar **NAT y direcciones repetidas**, esto es solo cosmético; si se quisiera resolución real habría que agregar un DNS público a la topología.

> [!info] Las redes de cliente **no participan** de BGP, OSPF, GRE ni de los servicios corporativos (Syslog, FTP, NTP, AAA). Son intencionalmente "ajenas" a la empresa: representan suscriptores del ISP.

---

## 9. Checklist de Aprobación

Utiliza esta lista para verificar que tu implementación está correcta **antes de entregar**. Cada ítem debe poder comprobarse tú mismo en Packet Tracer.

---

### BGP

- [ ] Al ejecutar `show ip bgp summary` en **R1**, el vecino `203.0.113.2` aparece con un **número** en la columna de estado/prefijos (sesión _Established_), **no** `Idle` ni `Active`.
    
- [ ] Al ejecutar `show ip bgp summary` en **R2**, el vecino `203.0.113.5` aparece igualmente establecido.
    
- [ ] Al ejecutar `show ip bgp summary` en el **IXP**, aparecen **dos** vecinos establecidos: `100.64.1.2` (Movistar) y `100.64.2.2` (Orange).
    
- [ ] Al ejecutar `show ip bgp` en R1, aparecen aprendidos por BGP los /30 del lado España (p. ej. `203.0.113.4/30`).
    

> [!tip] Si un vecino queda en `Active`/`Idle`, casi siempre el problema es **físico** (serial DCE del IXP sin `clock rate` o cable invertido), **no** el BGP. Verifica primero `show ip interface brief`.

---

### Túnel GRE

- [ ] Al ejecutar `show interfaces tunnel0` en R1 y R2, el túnel aparece **up / line protocol up**.
    
- [ ] Desde **R1**, `ping 192.168.100.2` responde (extremo del túnel en R2), y viceversa.
    
- [ ] Desde un **PC en VLAN 10 (Chile)**, puedo hacer ping a un host de **VLAN 40 (España)** (`192.168.40.X`): el tráfico inter-sitio cruza el túnel.
    

---

### NAT / PAT

- [ ] Al hacer ping desde un host interno hacia una dirección pública y ejecutar `show ip nat translations` en el borde correspondiente, aparecen entradas de traducción con `overload`.
    
- [ ] Al ejecutar `show ip route` en R1/R2, existe la **ruta por defecto** hacia el ISP (`203.0.113.2` en R1, `203.0.113.5` en R2).
    
- [ ] El tráfico **inter-sitio** (por el túnel) **no** genera traducciones NAT (confirmando que solo se NATea lo que va a Internet).
    

---

### Redes de cliente

- [ ] El **PC**, la **laptop** y el **smartphone** de HOGAR obtienen por **DHCP** una IP en `192.168.1.0/24` y gateway `192.168.1.1`.
    
- [ ] Desde la laptop de HOGAR, `ping 100.64.3.1` (su gateway en el ISP) responde, y `ping 203.0.113.10` (pública del SOHO) también: hay alcance "por Internet".
    
- [ ] Los **SV_SOHO, PC1, PC2 y PC3** tienen IP estática `192.168.1.10–.13` y hacen ping entre sí y a su gateway `192.168.1.1`.
    
- [ ] **Demostración clave:** HOGAR y SOHO funcionan **al mismo tiempo** usando ambos `192.168.1.0/24` sin conflicto, porque cada módem traduce a su propia pública (`100.64.3.2` y `203.0.113.10`).
    

---

## Imagen referencial red final

> [!example] Cómo se debería ver la red al finalizar 
> ![[Excalidraw/lab_final_pt.excalidraw.md#^frame=0JyoOOeiGf8JA7n78ZTBf|center|2920]]