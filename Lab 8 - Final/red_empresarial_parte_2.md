# Red Empresarial — Parte 2

## Examen Final Práctico · CCNA III · Packet Tracer

---

## Índice

- [[#1. Información General y Topología]]
- [[#2. Tabla de Direccionamiento]]
- [[#3. VLANs]]
- [[#4. Topología y Conexiones Físicas]]
- [[#5. Enrutamiento OSPF]]
- [[#6. Redundancia — Spanning Tree y HSRP]]
- [[#7. Sucursal — ROAS y Rutas Estáticas]]
- [[#8. Configuración de Servicios]]
    - [[#Servidor DHCPv4]]
    - [[#Servidor DNS]]
    - [[#Servidor FTP]]
    - [[#Servidor Syslog]]
    - [[#Servidor NTP — R2]]
- [[#9. Checklist de Aprobación]]

---

## 1. Información General y Topología

Esta segunda parte amplía la red empresarial hacia el **sitio España**, agregando un núcleo de distribución redundante, un sitio de sucursal con ruteo entre VLANs sobre un único enlace (ROAS) y un esquema de redundancia de primer salto (HSRP). Al igual que en la parte 1, se trabaja con dispositivos de capa 2 y capa 3, segmentación por VLANs, ruteo dinámico OSPF y servicios de red.

> [!info] El **sitio España** es un sitio corporativo remoto que reutiliza los servicios centrales desplegados en la **casa matriz de Chile** (parte 1). En la **parte 3**, ambos sitios se interconectan a través de un backbone internacional mediante BGP, un túnel GRE y NAT.

El sitio España se divide en las siguientes zonas:

- **Núcleo / Distribución (Core L3):** MLS4 y MLS5 — switches multicapa con enrutamiento habilitado y **HSRP** para las VLAN 40, 50, 60 y 99.
- **Acceso L2 (núcleo España):** SW3 (VLAN 40), SW4 (VLAN 50) y SW5 (VLAN 60). Cada switch de acceso tiene **doble uplink** (uno a MLS4 y otro a MLS5).
- **Router de borde España:** R2 — conectado a MLS4, MLS5 y R3; actúa como **servidor NTP** del sitio.
- **Sucursal (branch):** R3 (router con **ROAS**), SW6 y SW7 — VLANs 70, 80 y 90, en el rango `172.16.31.0`.
- **Wireless:** El WLC y el LWAP se conectan a SW5 (`f0/20` y `f0/21`). Su puesta en marcha se aborda en la parte de implementación.

> [!important] **Novedad respecto a la parte 1: ROAS.** En la parte 1 todo el ruteo inter-VLAN se resolvió con SVIs en los multilayer switches. En esta parte, la sucursal usa **Router-on-a-Stick (ROAS)**: un único enlace troncal entre R3 y SW6 transporta las VLAN 70/80/90, y R3 rutea entre ellas mediante **subinterfaces** (`G0/1.70`, `G0/1.80`, `G0/1.90`) con encapsulación `dot1Q`. Es la primera vez que se aplica esta técnica en la construcción de esta red.

> [!important] **Novedad respecto a la parte 1: redundancia HSRP.** MLS4 y MLS5 comparten una **IP virtual (VIP)** por cada VLAN, que actúa como gateway redundante para los hosts. MLS4 es el dispositivo **activo** y MLS5 el **standby**.

> [!example] Topología de Sucursal en España 
> ![[Excalidraw/lab_final_pt.excalidraw.md#^frame=8sOhwe7m4QFmlZhPRjFM_|center|1800]]

---

## 2. Tabla de Direccionamiento

### Interfaces de capa 3 (Routed / L3)

|Dispositivo|Interfaz|Dirección IP|Máscara|Descripción|
|---|---|---|---|---|
|R2|Loopback0|2.2.2.2|255.255.255.255 /32|Router-ID / NTP|
|R2|G0/1|10.255.2.1|255.255.255.252 /30|Enlace R2 → MLS4|
|R2|G0/0|10.255.2.5|255.255.255.252 /30|Enlace R2 → MLS5|
|R2|G0/2|10.255.2.13|255.255.255.252 /30|Enlace R2 → R3|
|MLS4|G0/1|10.255.2.2|255.255.255.252 /30|Enlace MLS4 → R2|
|MLS4|Po5|10.255.2.9|255.255.255.252 /30|Enlace L3 MLS4 → MLS5|
|MLS5|G0/2|10.255.2.6|255.255.255.252 /30|Enlace MLS5 → R2|
|MLS5|Po5|10.255.2.10|255.255.255.252 /30|Enlace L3 MLS5 → MLS4|
|R3|G0/2|10.255.2.14|255.255.255.252 /30|Enlace R3 → R2|

### Subinterfaces ROAS (R3 — sucursal)

|Dispositivo|Subinterfaz|Encapsulación|Dirección IP|Máscara|Rol|
|---|---|---|---|---|---|
|R3|G0/1.70|dot1Q 70|172.16.31.30|255.255.255.224 /27|Gateway VLAN 70|
|R3|G0/1.80|dot1Q 80|172.16.31.62|255.255.255.224 /27|Gateway VLAN 80|
|R3|G0/1.90|dot1Q 90|172.16.31.78|255.255.255.240 /28|Gateway VLAN 90|

### SVIs (Switch Virtual Interfaces)

|Dispositivo|VLAN|Dirección IP|Máscara|
|---|---|---|---|
|MLS4|VLAN 40|192.168.40.2|255.255.255.0 /24|
|MLS4|VLAN 50|192.168.50.2|255.255.255.0 /24|
|MLS4|VLAN 60|192.168.60.2|255.255.255.0 /24|
|MLS4|VLAN 99|192.168.99.130|255.255.255.128 /25|
|MLS5|VLAN 40|192.168.40.3|255.255.255.0 /24|
|MLS5|VLAN 50|192.168.50.3|255.255.255.0 /24|
|MLS5|VLAN 60|192.168.60.3|255.255.255.0 /24|
|MLS5|VLAN 99|192.168.99.131|255.255.255.128 /25|
|SW3|VLAN 99|192.168.99.132|255.255.255.128 /25|
|SW4|VLAN 99|192.168.99.133|255.255.255.128 /25|
|SW5|VLAN 99|192.168.99.134|255.255.255.128 /25|
|SW6|VLAN 90|172.16.31.65|255.255.255.240 /28|
|SW7|VLAN 80|172.16.31.33|255.255.255.224 /27|

### Gateways virtuales HSRP (gateway de los hosts)

|VLAN|VIP (gateway)|Activo|Standby|
|---|---|---|---|
|VLAN 40|192.168.40.1|MLS4|MLS5|
|VLAN 50|192.168.50.1|MLS4|MLS5|
|VLAN 60|192.168.60.1|MLS4|MLS5|
|VLAN 99|192.168.99.129|MLS4|MLS5|

### Default Gateways para SW (capa 2)

|Dispositivo|Default Gateway|Apunta a|
|---|---|---|
|SW3|192.168.99.129|VIP HSRP VLAN 99|
|SW4|192.168.99.129|VIP HSRP VLAN 99|
|SW5|192.168.99.129|VIP HSRP VLAN 99|
|SW6|172.16.31.78|R3 G0/1.90 (VLAN 90)|
|SW7|172.16.31.62|R3 G0/1.80 (VLAN 80)|

---

## 3. VLANs

Todos los dispositivos de cada sitio deben tener la base de datos de VLANs de su sitio, independientemente de cuáles SVIs tengan configuradas.

### VLANs del núcleo España (MLS4, MLS5, SW3, SW4, SW5)

|ID VLAN|Nombre|Subred|Uso|
|---|---|---|---|
|40|software_dev|192.168.40.0/24|R&D / Desarrollo (SW3)|
|50|implementation|192.168.50.0/24|QA / Implementación (SW4)|
|60|sales_fields_ops|192.168.60.0/24|Ventas y Field Ops (SW5)|
|57|native|—|VLAN nativa en todos los trunks|
|99|admin|192.168.99.128/25|Gestión SSH de los dispositivos|

### VLANs de la sucursal (R3, SW6, SW7)

|ID VLAN|Nombre|Subred|Uso|
|---|---|---|---|
|70|accounting_hr|172.16.31.0/27|Contabilidad y RR.HH.|
|80|training_center|172.16.31.32/27|Centro de capacitación|
|90|executive_office|172.16.31.64/28|Oficina ejecutiva|
|57|native|—|VLAN nativa en todos los trunks|

---

## 4. Topología y Conexiones Físicas

### EtherChannels y tipos de enlace

|EtherChannel|Dispositivos|Puertos físicos|Tipo de enlace|Protocolo|
|---|---|---|---|---|
|Po5|MLS4 ↔ MLS5|f0/23-24 en ambos|L3 Routed|LACP|

> [!info] Po5 es un EtherChannel de **capa 3**: se aplica `no switchport` y `speed 100 / duplex full` en los puertos físicos antes de asignarlos al channel-group, y el Port-channel resultante recibe directamente una dirección IP (`10.255.2.9` en MLS4, `10.255.2.10` en MLS5). Sobre este enlace se forma adyacencia OSPF entre MLS4 y MLS5.

### Enlaces troncales (L2)

|Enlace|Puertos|VLANs permitidas|Nativa|
|---|---|---|---|
|MLS4 → SW3|MLS4 f0/10 ↔ SW3 G0/1|40, 50, 60, 99|57|
|MLS4 → SW4|MLS4 f0/11 ↔ SW4 G0/1|40, 50, 60, 99|57|
|MLS4 → SW5|MLS4 f0/12 ↔ SW5 G0/1|40, 50, 60, 99|57|
|MLS5 → SW3|MLS5 f0/13 ↔ SW3 G0/2|40, 50, 60, 99|57|
|MLS5 → SW4|MLS5 f0/14 ↔ SW4 G0/2|40, 50, 60, 99|57|
|MLS5 → SW5|MLS5 f0/15 ↔ SW5 G0/2|40, 50, 60, 99|57|
|R3 → SW6 (ROAS)|R3 G0/1 ↔ SW6 G0/1|70, 80, 90|57|
|SW6 → SW7|SW6 G0/2 ↔ SW7 G0/2|70, 80|57|

> [!info] Cada switch de acceso del núcleo España (SW3/SW4/SW5) tiene **doble uplink sin EtherChannel**: G0/1 sube a MLS4 y G0/2 sube a MLS5. Spanning Tree se encarga de bloquear uno de los dos para evitar bucles (ver sección 6).

### Enlaces ruteados punto a punto /30

|Subred|Interfaz extremo 1|IP extremo 1|Interfaz extremo 2|IP extremo 2|
|---|---|---|---|---|
|10.255.2.0/30|R2 G0/1|10.255.2.1|MLS4 G0/1|10.255.2.2|
|10.255.2.4/30|R2 G0/0|10.255.2.5|MLS5 G0/2|10.255.2.6|
|10.255.2.8/30|MLS4 Po5|10.255.2.9|MLS5 Po5|10.255.2.10|
|10.255.2.12/30|R2 G0/2|10.255.2.13|R3 G0/2|10.255.2.14|

### Puertos de acceso

|Dispositivo|Puertos|VLAN|Notas|
|---|---|---|---|
|SW3|f0/1-9|VLAN 40|PortFast + BPDU Guard|
|SW4|f0/1-9|VLAN 50|PortFast + BPDU Guard|
|SW5|f0/1-9|VLAN 60|PortFast + BPDU Guard|
|SW5|f0/20-21|VLAN 99|WLC (f0/20) y LWAP (f0/21)|
|SW6|f0/20|VLAN 90|PortFast + BPDU Guard|
|SW6|f0/21-24|VLAN 70|PortFast + BPDU Guard|
|SW7|f0/17-20|VLAN 80|PortFast + BPDU Guard|
|SW7|f0/21-24|VLAN 70|PortFast + BPDU Guard|

> [!info] El resto de puertos de acceso de cada switch se dejan en `shutdown`. En SW7, el puerto G0/1 no se utiliza.

---

## 5. Enrutamiento OSPF

Se utiliza **OSPF proceso 1** en todos los dispositivos de capa 3 del núcleo España (R2, MLS4, MLS5). Todas las redes se anuncian en el **area 0**.

> [!warning] **R3 NO participa en OSPF.** La sucursal se integra mediante rutas estáticas + redistribución (ver sección 7). Esto es intencional y agrega un nivel adicional de dificultad a la construcción de la red.

### Router IDs

|Dispositivo|Router ID|
|---|---|
|R2|2.2.2.2|
|MLS4|40.40.40.40|
|MLS5|50.50.50.50|

### Adyacencias (vecinos OSPF)

Las adyacencias OSPF se forman únicamente sobre los enlaces punto a punto /30.

|Vecino A|Vecino B|Subred del enlace|
|---|---|---|
|R2|MLS4|10.255.2.0/30|
|R2|MLS5|10.255.2.4/30|
|MLS4|MLS5|10.255.2.8/30|

### Redes anunciadas por cada dispositivo

|Dispositivo|Red anunciada|Wildcard|Origen de la red|
|---|---|---|---|
|R2|2.2.2.2|0.0.0.0|Loopback0 (passive)|
|R2|10.255.2.0|0.0.0.3|G0/1 hacia MLS4|
|R2|10.255.2.4|0.0.0.3|G0/0 hacia MLS5|
|R2|10.255.2.12|0.0.0.3|G0/2 hacia R3 (passive)|
|R2|_(estáticas)_|—|`redistribute static`|
|MLS4|10.255.2.0|0.0.0.3|G0/1 hacia R2|
|MLS4|10.255.2.8|0.0.0.3|Po5 hacia MLS5|
|MLS4|192.168.40.0|0.0.0.255|SVI VLAN 40 (passive)|
|MLS4|192.168.50.0|0.0.0.255|SVI VLAN 50 (passive)|
|MLS4|192.168.60.0|0.0.0.255|SVI VLAN 60 (passive)|
|MLS4|192.168.99.128|0.0.0.127|SVI VLAN 99 (passive)|
|MLS5|10.255.2.4|0.0.0.3|G0/2 hacia R2|
|MLS5|10.255.2.8|0.0.0.3|Po5 hacia MLS4|
|MLS5|192.168.40.0|0.0.0.255|SVI VLAN 40 (passive)|
|MLS5|192.168.50.0|0.0.0.255|SVI VLAN 50 (passive)|
|MLS5|192.168.60.0|0.0.0.255|SVI VLAN 60 (passive)|
|MLS5|192.168.99.128|0.0.0.127|SVI VLAN 99 (passive)|

> [!info] Las interfaces que no deben formar adyacencia (Loopback de R2, G0/2 de R2 hacia R3, y todas las SVIs de MLS4/MLS5) se declaran como **passive-interface**: anuncian su red pero no envían _hellos_.

---

## 6. Redundancia — Spanning Tree y HSRP

### Spanning Tree (Rapid-PVST) — Root Bridges

Para repartir la carga entre MLS4 y MLS5, se ajustan las prioridades de raíz por VLAN. El dispositivo con menor prioridad numérica es la raíz para esa VLAN.

|VLAN(s)|Root (priority 24576)|Secundario (priority 28672)|
|---|---|---|
|40, 50|MLS4|MLS5|
|60, 99|MLS5|MLS4|

> [!info] Así, el tráfico de VLAN 40 y 50 prefiere a MLS4 como raíz, mientras que VLAN 60 y 99 prefieren a MLS5. Cada switch de acceso (doble uplink) mantiene un uplink en _forwarding_ y el otro en _blocking_ según la VLAN.

### HSRP (First Hop Redundancy)

MLS4 y MLS5 ofrecen una **IP virtual (VIP)** por VLAN, que es el gateway que usan los hosts. Se usa **HSRP versión 2** y el **grupo por defecto** en todas las VLAN.

- **MLS4 = Activo:** prioridad `110` + `preempt` en las cuatro VLAN.
- **MLS5 = Standby:** prioridad por defecto (`100`).

|VLAN|VIP|IP real MLS4|IP real MLS5|
|---|---|---|---|
|VLAN 40|192.168.40.1|192.168.40.2|192.168.40.3|
|VLAN 50|192.168.50.1|192.168.50.2|192.168.50.3|
|VLAN 60|192.168.60.1|192.168.60.2|192.168.60.3|
|VLAN 99|192.168.99.129|192.168.99.130|192.168.99.131|

> [!important] Aunque todas las instancias HSRP usan el **mismo número de grupo** (el grupo por defecto), la **VIP es distinta en cada VLAN** porque debe pertenecer a la subred de su propia SVI. No es posible usar una única VIP para todas las VLAN. El número de grupo es solo un identificador local de la interfaz y sí puede repetirse entre VLANs diferentes.

> [!tip] El `preempt` permite que MLS4 recupere automáticamente el rol activo cuando vuelva a estar disponible tras una caída. Sin `preempt`, MLS4 quedaría como standby aunque tuviera mayor prioridad.

---

## 7. Sucursal — ROAS y Rutas Estáticas

La sucursal (VLAN 70/80/90) cuelga de **R3**, que rutea entre sus VLANs por **ROAS** y se conecta al resto de la red por un único enlace ruteado hacia R2 (`10.255.2.12/30`).

### ROAS en R3

- La interfaz física `G0/1` se levanta sin IP y se troncaliza desde SW6.
- Cada VLAN tiene su **subinterfaz** con `encapsulation dot1Q <vlan>` y la IP que actúa como gateway de esa VLAN (ver tabla de subinterfaces en la sección 2).

### Integración con el resto de la red

> [!warning] **Por qué se usan rutas estáticas en R2.** Como **R3 no corre OSPF**, sus redes conectadas (las VLAN 70/80/90 en `172.16.31.x`) **no se anuncian solas** al dominio OSPF. Esto introduce un trabajo de ruteo manual en ambos sentidos y es deliberadamente la parte más exigente de esta construcción.

El circuito se resuelve en dos mitades:

1. **Salida de la sucursal (R3 → resto de la red):** R3 tiene una **ruta por defecto** hacia R2.
    
    ```
    ip route 0.0.0.0 0.0.0.0 10.255.2.13
    ```
    
    Todo el tráfico que sale de las VLAN 70/80/90 hacia cualquier otro destino se entrega a R2.
    
2. **Retorno hacia la sucursal (red → VLAN 70/80/90):** R2 tiene **rutas estáticas** hacia las tres subredes de la sucursal, apuntando a R3:
    
    ```
    ip route 172.16.31.0  255.255.255.224 10.255.2.14
    ip route 172.16.31.32 255.255.255.224 10.255.2.14
    ip route 172.16.31.64 255.255.255.240 10.255.2.14
    ```
    
    Y para que el resto del núcleo España (MLS4, MLS5) también sepa llegar a la sucursal, R2 **inyecta esas estáticas en OSPF**:
    
    ```
    router ospf 1
     redistribute static subnets
    ```
    

> [!info] Sin las estáticas en R2, un host de la sucursal podría **enviar** un paquete (gracias a su default route), pero R2 **descartaría la respuesta** por no tener ruta de vuelta hacia `172.16.31.x`. La default de R3 resuelve la ida; las estáticas + `redistribute` de R2 resuelven el regreso. Son las dos mitades del mismo camino.

> [!info] El enlace `10.255.2.12/30` (R2 ↔ R3) **sí** está anunciado en OSPF, por lo que la gestión y el NTP de R3 funcionan aunque falten las estáticas. Lo que **no** funciona sin ellas es la conectividad de los **dispositivos finales** detrás de R3.

---

## 8. Configuración de Servicios

Los servidores de la red residen en la **VLAN 30 (Data Center)** del **sitio Chile (casa matriz)** y ya fueron desplegados en la parte 1. En esta parte, los dispositivos del sitio España solo deben **apuntar** a esos servicios.

---

### Servidor DHCPv4

En el sitio España, el DHCP funciona por **relay**: las SVIs de MLS4 y MLS5 reenvían las peticiones DHCP hacia el servidor central.

#### En MLS4 y MLS5

Configurar `ip helper-address 192.168.30.6` en las SVIs de las VLAN **40, 50, 60 y 99**.

> [!warning] **El servidor dedicado DHCPv4 (`192.168.30.6`) se configurará en la parte de implementación.** El relay queda preparado en esta etapa, pero los hosts del sitio España **no obtendrán IP automática** hasta que dicho servidor (y su alcance hacia VLAN 30 vía el enlace inter-sitio) esté operativo. Hasta entonces, asignar direcciones de forma estática para las pruebas.

> [!tip] **Plan de contingencia (opcional):** si en la implementación el servidor central diera problemas, R2 puede asumir temporalmente el rol de **servidor DHCP local** con pools para las VLAN 40/50/60/99. En ese caso, los `ip helper-address` de MLS4/MLS5 deben apuntar a `2.2.2.2` (loopback de R2) en lugar de `192.168.30.6`. El pool de VLAN 99 incluye `option 43 ip 192.168.99.135`, que entrega al LWAP la IP de gestión del WLC para que lo descubra por CAPWAP sin depender de broadcast/DNS.

---

### Servidor DNS

> [!info] El servidor DNS ya está desplegado en la parte 1. En los switches y MLS del sitio España (MLS4, MLS5, SW3, SW4, SW5, SW6, SW7), configurar el cliente DNS **exactamente de la misma forma que en la parte 1**.

---

### Servidor FTP

> [!info] El servidor FTP ya está desplegado en la parte 1. En los switches y MLS del sitio España (MLS4, MLS5, SW3, SW4, SW5, SW6, SW7), configurar las credenciales FTP del dispositivo **exactamente de la misma forma que en la parte 1** (`ftp_admin` / `cisco123`).

---

### Servidor Syslog

> [!info] El servidor Syslog ya está desplegado en la parte 1. En los switches y MLS del sitio España (MLS4, MLS5, SW3, SW4, SW5, SW6, SW7), configurar el envío de logs **exactamente de la misma forma que en la parte 1** (servidor `192.168.30.2`, trap `debugging`, timestamps con fecha/hora/ms).

---

### Servidor NTP — R2

**R2 actúa como servidor NTP del sitio España.** No hay un servidor NTP físico separado; el router R2 cumple este rol, y los dispositivos lo alcanzan a través de su loopback `2.2.2.2` (anunciada en OSPF).

#### En R2

- Configurar el timezone como `CET +1` (**Central European Time**, huso de la España peninsular; UTC+1 en horario estándar).
- Configurar el reloj del sistema con la fecha y hora actual.
- Declarar R2 como **NTP master** con estrato 1.

#### En R3

- Configurar el timezone como `CET +1`.
- Apuntar al servidor NTP `2.2.2.2`.

#### En todos los switches y MLS del sitio España (MLS4, MLS5, SW3, SW4, SW5, SW6, SW7)

Apuntar cada dispositivo al servidor NTP:

- Dirección del servidor NTP: `2.2.2.2`

#### Verificación en cualquier MLS o SW del sitio España

```
show ntp status
```

Resultado esperado:

```
Clock is synchronized, stratum 2, reference is 2.2.2.2
```

```
show ntp associations
```

Debe aparecer `2.2.2.2` con un asterisco (`*`) indicando que está sincronizado y activo como fuente NTP.

> [!info] Si el asterisco no aparece de inmediato, esperar unos segundos y repetir el comando. El estrato debe ser 2 (un nivel por debajo del master, que es estrato 1).

---

## 9. Checklist de Aprobación

Utiliza esta lista para verificar que tu implementación está correcta **antes de entregar**. Cada ítem debe poder comprobarse tú mismo en Packet Tracer.

---

### Conectividad general

- [ ] Desde **R2**, puedo hacer ping a:
    
    - `10.255.2.2` (MLS4) y `10.255.2.6` (MLS5)
    - `10.255.2.14` (R3)
    - `192.168.40.1`, `192.168.50.1`, `192.168.60.1`, `192.168.99.129` (VIPs HSRP)
- [ ] Desde **cualquier dispositivo del núcleo España**, puedo hacer ping a todas las SVIs de VLAN 99:
    
    - `192.168.99.130` (MLS4) · `192.168.99.131` (MLS5)
    - `192.168.99.132` (SW3) · `192.168.99.133` (SW4) · `192.168.99.134` (SW5)
- [ ] Desde un **PC en VLAN 40**, puedo hacer ping a:
    
    - Su gateway `192.168.40.1`
    - Un PC en VLAN 50 (`192.168.50.X`) y VLAN 60 (`192.168.60.X`)

---

### EtherChannel y trunks

- [ ] Al ejecutar `show etherchannel summary` en MLS4 y MLS5, el **Po5** aparece en estado **SU** y sus puertos físicos en estado **P**.
    
- [ ] Al ejecutar `show interfaces trunk` en MLS4 y MLS5, los enlaces hacia SW3/SW4/SW5 aparecen como trunks activos con la VLAN 57 como nativa.
    

---

### OSPF

- [ ] Al ejecutar `show ip ospf neighbor` en R2, MLS4 y MLS5, todos los vecinos aparecen en estado **FULL**.
    
- [ ] Al ejecutar `show ip route ospf` en MLS4, aparecen las redes remotas del sitio aprendidas por OSPF, incluida la `2.2.2.2/32` (loopback de R2).
    

---

### HSRP

- [ ] Al ejecutar `show standby brief` en **MLS4**, aparece como **Active** en las VLAN 40/50/60/99, con `P` (preempt) activado.
    
- [ ] Al ejecutar `show standby brief` en **MLS5**, aparece como **Standby** en las mismas VLAN.
    
- [ ] Al apagar (o desconectar) MLS4, un ping continuo desde un PC del núcleo España hacia su gateway **se mantiene** (MLS5 toma el rol activo). Al reconectar MLS4, este recupera el rol activo gracias al `preempt`.
    

---

### ROAS y sucursal

- [ ] Al ejecutar `show ip route` en R3, aparecen las tres subredes de la sucursal como **conectadas** a las subinterfaces, y una **default route** vía `10.255.2.13`.
    
- [ ] Un **PC en VLAN 70** puede hacer ping a un **PC en VLAN 80** y a uno en **VLAN 90** (ruteo ROAS funcionando).
    
- [ ] Desde un PC del **núcleo España**, puedo hacer ping a un host de la **sucursal** (`172.16.31.X`), confirmando que las rutas estáticas de R2 y el `redistribute static` funcionan en ambos sentidos.
    
- [ ] Al ejecutar `show ip route` en MLS4/MLS5, las redes `172.16.31.0/27`, `172.16.31.32/27` y `172.16.31.64/28` aparecen como rutas **OSPF externas (O E2)**.
    

---

### DHCP

> [!warning] El servidor dedicado DHCPv4 se implementa en la parte de implementación. En esta etapa solo se verifica que el **relay** esté configurado.

- [ ] Al ejecutar `show running-config interface vlan 40` (y 50/60/99) en MLS4 y MLS5, aparece el `ip helper-address 192.168.30.6`.

---

### DNS · FTP · Syslog

- [ ] Estos servicios quedan configurados en los dispositivos del sitio España **igual que en la parte 1**. Verificar con los mismos comandos descritos allí (`ipconfig /all`, `ftp 192.168.30.X`, `show logging`).

---

### NTP

- [ ] Al ejecutar `show ntp status` en cualquier MLS o SW del sitio España, el resultado muestra:
    
    - `Clock is synchronized`
    - `stratum 2`
    - `reference is 2.2.2.2`
- [ ] Al ejecutar `show ntp associations`, aparece `2.2.2.2` con un asterisco (`*`) al inicio, indicando sincronización activa.