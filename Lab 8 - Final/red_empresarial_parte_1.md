# Red Empresarial — Parte 1

## Examen Final Práctico · CCNA III · Packet Tracer

---

## Índice

- [[#1. Información General]]
- [[#2. Tabla de Direccionamiento]]
- [[#3. VLANs]]
- [[#4. Topología y Conexiones Físicas]]
- [[#5. Enrutamiento OSPF]]
- [[#6. Configuración de Servicios]]
    - [[#Servidor DHCPv4]]
    - [[#Servidor DNS]]
    - [[#Servidor FTP]]
    - [[#Servidor Syslog]]
    - [[#Servidor NTP — R1]]
- [[#7. Servidor AAA — Sección Opcional]]
- [[#8. Checklist de Aprobación]]

---

## 1. Información General

Esta práctica consiste en la implementación de una red empresarial en Cisco Packet Tracer, que incluye dispositivos de capa 2 y capa 3, segmentación por VLANs, ruteo dinámico mediante OSPF, y servicios de red (DHCP, DNS, FTP, Syslog y NTP).

Esta primera parte corresponde al **sitio Chile**, la **casa matriz** de la empresa: aquí reside el **Data Center (VLAN 30)** con todos los servidores corporativos que dan servicio al resto de la organización. En la **parte 2** se construye el **sitio España** y en la **parte 3** ambos sitios se interconectan a través de un **backbone internacional**.

La red del sitio Chile se divide en las siguientes zonas:

- **Núcleo (Core L3):** MLS1, MLS2, MLS3 — switches multicapa con enrutamiento habilitado.
- **Acceso L2:** SW1, SW2 — switches de capa 2.
- **Router de borde:** R1 (sitio Chile) — conectado a MLS1, actúa como servidor NTP del sitio.
- **Data Center (VLAN 30):** Todos los servidores de la red residen aquí.
- **VoIP:** Los teléfonos IP y sus PCs asociados se conectan a SW2.

> [!important] La SVI `192.168.99.1` de MLS1 pertenece a la VLAN de administración (VLAN 99). Sin embargo, dado que todos los uplinks de MLS1 son interfaces de capa 3 (`no switchport`), esta SVI no levantará. Esto es un comportamiento esperado en esta topología. MLS1 es gestionable a través de sus IPs de capa 3: `10.255.1.2`, `10.255.1.6` y `10.255.1.10`.

---

## 2. Tabla de Direccionamiento

### Interfaces de capa 3 (Routed / L3)

|Dispositivo|Interfaz|Dirección IP|Máscara|Descripción|
|---|---|---|---|---|
|R1|G0/1|10.255.1.1|255.255.255.252 /30|Enlace R1 → MLS1|
|MLS1|G0/1|10.255.1.2|255.255.255.252 /30|Enlace MLS1 → R1|
|MLS1|Po3|10.255.1.6|255.255.255.252 /30|Enlace L3 MLS1 → MLS2|
|MLS1|Po4|10.255.1.10|255.255.255.252 /30|Enlace L3 MLS1 → MLS3|
|MLS2|Po3|10.255.1.5|255.255.255.252 /30|Enlace L3 MLS2 → MLS1|
|MLS3|Po4|10.255.1.9|255.255.255.252 /30|Enlace L3 MLS3 → MLS1|

### SVIs (Switch Virtual Interfaces)

|Dispositivo|VLAN|Dirección IP|Máscara|
|---|---|---|---|
|MLS1|VLAN 30|192.168.30.1|255.255.255.0 /24|
|MLS1|VLAN 99|192.168.99.1|255.255.255.224 /27|
|MLS2|VLAN 10|192.168.10.1|255.255.255.0 /24|
|MLS2|VLAN 99|192.168.99.33|255.255.255.224 /27|
|MLS3|VLAN 20|192.168.20.1|255.255.255.0 /24|
|MLS3|VLAN 25|192.168.25.1|255.255.255.0 /24|
|MLS3|VLAN 99|192.168.99.65|255.255.255.224 /27|
|SW1|VLAN 99|192.168.99.34|255.255.255.224 /27|
|SW2|VLAN 99|192.168.99.66|255.255.255.224 /27|

### Default Gateways para SW (capa 2)

|Dispositivo|Default Gateway|
|---|---|
|SW1|192.168.99.33|
|SW2|192.168.99.65|

### Servidores en VLAN 30 (192.168.30.0/24)

|Servidor|Dirección IP|Servicio|
|---|---|---|
|SV-Syslog|192.168.30.2|Syslog|
|SV-FTP|192.168.30.3|FTP|
|SV-DNS|192.168.30.4|DNS|
|SV-AAA|192.168.30.5|RADIUS/AAA|
|SV-DHCP|192.168.30.6|DHCPv4|

---

## 3. VLANs

Todos los dispositivos de la red deben tener la base de datos de VLANs completa, independientemente de cuáles SVIs tengan configuradas.

|ID VLAN|Nombre|Uso|
|---|---|---|
|10|operations|Usuarios de operaciones (SW1)|
|20|support|Usuarios de soporte (SW2)|
|25|network-ops|VoIP y datos VoIP (SW2)|
|30|data_center|Servidores y puertos de MLS1|
|57|native|VLAN nativa en todos los trunks|
|99|admin|Gestión SSH de los dispositivos|

---

## 4. Topología y Conexiones Físicas

### Diagrama de conexiones

> [!example] Topología de Casa Matríz en Chile 
> ![[Excalidraw/lab_final_pt.excalidraw.md#^frame=HthJqc0U|center|1800]]

### EtherChannels y tipos de enlace

|EtherChannel|Dispositivos|Puertos físicos|Tipo de enlace|Protocolo|
|---|---|---|---|---|
|Po1|MLS2 ↔ SW1|f0/1-2 en ambos|L2 Trunk|LACP|
|Po2|MLS3 ↔ SW2|f0/1-2 en ambos|L2 Trunk|LACP|
|Po3|MLS1 ↔ MLS2|f0/8-9 en ambos|L3 Routed|LACP|
|Po4|MLS1 ↔ MLS3|f0/18-19 en ambos|L3 Routed|LACP|

> [!info] Todos los EtherChannels usan **LACP en modo active/active**. Los Po3 y Po4 son puertos de capa 3: se les aplica `no switchport` en los puertos físicos antes de asignarlos al channel-group, y el Port-channel resultante recibe directamente una dirección IP.

### Subredes de enlace punto a punto /30

|Subred|Interfaz en extremo 1|IP extremo 1|Interfaz en extremo 2|IP extremo 2|
|---|---|---|---|---|
|10.255.1.0/30|R1 G0/1|10.255.1.1|MLS1 G0/1|10.255.1.2|
|10.255.1.4/30|MLS2 Po3|10.255.1.5|MLS1 Po3|10.255.1.6|
|10.255.1.8/30|MLS3 Po4|10.255.1.9|MLS1 Po4|10.255.1.10|

> [!info] En todos los enlaces punto a punto, **MLS1 siempre tiene la segunda IP disponible** de la subred /30.

---

## 5. Enrutamiento OSPF

Se utiliza **OSPF proceso 1** en todos los dispositivos de capa 3. Todas las redes se anuncian en el **area 0**.

### Router IDs

|Dispositivo|Router ID|
|---|---|
|R1|1.1.1.1|
|MLS1|10.10.10.10|
|MLS2|20.20.20.20|
|MLS3|30.30.30.30|

### Adyacencias (vecinos OSPF)

Las adyacencias OSPF se forman únicamente sobre los enlaces punto a punto /30. No existe adyacencia directa entre MLS2 y MLS3.

|Vecino A|Vecino B|Subred del enlace|
|---|---|---|
|R1|MLS1|10.255.1.0/30|
|MLS1|MLS2|10.255.1.4/30|
|MLS1|MLS3|10.255.1.8/30|

### Redes anunciadas por cada dispositivo

|Dispositivo|Red anunciada|Wildcard|Origen de la red|
|---|---|---|---|
|R1|10.255.1.0|0.0.0.3|Interfaz G0/1|
|MLS1|10.255.1.0|0.0.0.3|Interfaz G0/1 hacia R1|
|MLS1|10.255.1.4|0.0.0.3|Po3 hacia MLS2|
|MLS1|10.255.1.8|0.0.0.3|Po4 hacia MLS3|
|MLS1|192.168.30.0|0.0.0.255|SVI VLAN 30|
|MLS1|192.168.99.0|0.0.0.31|SVI VLAN 99|
|MLS2|10.255.1.4|0.0.0.3|Po3 hacia MLS1|
|MLS2|192.168.10.0|0.0.0.255|SVI VLAN 10|
|MLS2|192.168.99.32|0.0.0.31|SVI VLAN 99|
|MLS3|10.255.1.8|0.0.0.3|Po4 hacia MLS1|
|MLS3|192.168.20.0|0.0.0.255|SVI VLAN 20|
|MLS3|192.168.25.0|0.0.0.255|SVI VLAN 25|
|MLS3|192.168.99.64|0.0.0.31|SVI VLAN 99|

> [!info] Cada dispositivo solo anuncia las redes directamente conectadas a sus propias interfaces. Las rutas hacia redes remotas son aprendidas dinámicamente por OSPF desde los vecinos.

---

## 6. Configuración de Servicios

Todos los servidores se ubican en la **VLAN 30 (Data Center)**, conectados a los puertos de acceso de MLS1 (`f0/20` a `f0/24`), con gateway `192.168.30.1`.

---

### Servidor DHCPv4

**IP del servidor:** `192.168.30.6`

Configurar los pools necesarios para entregar IPv4 dinámica a las VLAN 10 y VLAN 20. Recordar excluir las IPs de gateways y servidores de cada pool.

---

### Servidor DNS

**IP del servidor:** `192.168.30.4`

#### En el servidor (GUI de Packet Tracer)

1. Agregar el siguiente registro tipo **A Record**:
    - **Name:** `examenes-redes.com`
    - **Address:** `192.168.30.4`

#### Verificación desde un PC

Desde la terminal (Command Prompt) de cualquier PC que haya obtenido IP por DHCP:

```
ipconfig /all
```

Confirmar que el campo **DNS Servers** muestre `192.168.30.4`.

Luego verificar resolución de nombres:

```
ping examenes-redes.com
```

Si el servidor DNS está bien configurado y el PC tiene alcance a `192.168.30.4`, el ping debe resolver el nombre y responder desde `192.168.30.4`.

---

### Servidor FTP

#### En el servidor (GUI de Packet Tracer)

1. En **User Setup**, agregar un usuario:
    - **Username:** `ftp_admin`
    - **Password:** `cisco123`
    - Asegurarse de marcar los permisos: **Write**, **Read**, **Delete**, **Rename**, **List**.

#### En todos los switches y MLS (MLS1, MLS2, MLS3, SW1, SW2)

Configurar las credenciales FTP globales del dispositivo:

- Username FTP: `ftp_admin`
- Password FTP: `cisco123`

Esto permite que los dispositivos puedan hacer copias de sus configuraciones hacia el servidor FTP.

#### Verificación desde un PC

Desde la terminal de cualquier PC con alcance a la red 192.168.30.0/24:

```
ftp 192.168.30.X
```

Ingresar las credenciales cuando se soliciten:

```
Username: ftp_admin
Password: cisco123
```

Si la conexión es exitosa, aparecerá el prompt `ftp>`. Para confirmar y salir:

```
ftp> dir
ftp> bye
```

#### Verificación desde un dispositivo de red

Desde el modo privilegiado de cualquier MLS o SW:

```
copy running-config ftp:
```

El dispositivo solicitará la IP del servidor FTP y el nombre del archivo destino. Confirmar que el archivo quede guardado en el servidor.

---

### Servidor Syslog

**IP del servidor:** `192.168.30.2`

#### En el servidor (GUI de Packet Tracer)

1. Ir a la pestaña **Services → SYSLOG**.
2. Activar el servicio: **On**.

#### En todos los switches y MLS (MLS1, MLS2, MLS3, SW1, SW2)

Configurar cada dispositivo para que envíe logs al servidor:

- Dirección del servidor Syslog: `192.168.30.2`
- Nivel de traps: `debugging` (captura todos los niveles de severidad).
- Activar timestamps en los logs con formato fecha y hora incluyendo milisegundos.

#### Verificación

Desde el servidor Syslog en Packet Tracer, hacer clic en la pestaña **Services → SYSLOG** y observar que los mensajes de los dispositivos van apareciendo en la tabla de logs.

Para generar logs de prueba, realizar cualquier cambio de configuración menor en un MLS (por ejemplo, bajar y subir una interfaz no crítica). El evento debe registrarse en el servidor.

Desde un MLS o SW, también verificar que los timestamps estén activos:

```
show logging
```

Los mensajes deben mostrar fecha y hora en cada entrada.

---

### Servidor NTP — R1

**R1 actúa como servidor NTP** para toda la red del **sitio Chile**. No hay un servidor NTP físico separado; el router R1 cumple este rol usando su propia interfaz `G0/1` con IP `10.255.1.1`.

#### En R1

- Configurar el timezone como `CLT -3` (huso de Chile; se usa el de la **Región de Magallanes / Punta Arenas**, que es **UTC-3 fijo** todo el año y evita el cambio de horario de verano).
- Configurar el reloj del sistema con la fecha y hora actual.
- Declarar R1 como **NTP master** con estrato 1.

#### En todos los switches y MLS (MLS1, MLS2, MLS3, SW1, SW2)

Apuntar cada dispositivo al servidor NTP:

- Dirección del servidor NTP: `10.255.1.1`

#### Verificación en cualquier MLS o SW

```
show ntp status
```

Resultado esperado:

```
Clock is synchronized, stratum 2, reference is 10.255.1.1
```

```
show ntp associations
```

Debe aparecer `10.255.1.1` con un asterisco (`*`) indicando que está sincronizado y activo como fuente NTP.

> [!info] Si el asterisco no aparece de inmediato, esperar unos segundos y ejecutar el comando nuevamente. El estrato debe ser 2 (un nivel por debajo del master que es estrato 1).

---

## 7. Servidor AAA — Sección Opcional

> [!warning] Esta sección es de carácter opcional y avanzado La configuración del servidor AAA y la autenticación SSH mediante RADIUS presentan complejidades adicionales en Packet Tracer. Si no logras implementarla correctamente, no afectará la evaluación de los demás puntos.

**Topología:** El servidor AAA (RADIUS) debe estar físicamente ubicado en la **VLAN 30 (Data Center)**, con IP `192.168.30.5`, conectado a uno de los puertos de acceso de MLS1.

> [!tip] Para intentar su configuración, consultar con el ayudante.

---

## 8. Checklist de Aprobación

Utiliza esta lista para verificar que tu implementación está correcta **antes de entregar**. Cada ítem debe poder comprobarse tú mismo en Packet Tracer.

---

### Conectividad general

- [ ] Desde **MLS1**, puedo hacer ping a:
    
    - `10.255.1.1` (R1)
    - `10.255.1.5` (MLS2 via Po3)
    - `10.255.1.9` (MLS3 via Po4)
- [ ] Desde **cualquier dispositivo** de la red, puedo hacer ping a todas las SVIs de VLAN 99, **excepto `192.168.99.1`** (comportamiento esperado):
    
    - `192.168.99.33` (MLS2) ✓
    - `192.168.99.65` (MLS3) ✓
    - `192.168.99.34` (SW1) ✓
    - `192.168.99.66` (SW2) ✓
- [ ] Desde un **PC en VLAN 10**, puedo hacer ping a:
    
    - Su gateway `192.168.10.1`
    - Un PC en VLAN 20 (`192.168.20.X`)
    - El gateway de VLAN 30 `192.168.30.1`
    - La IP del router R1 `10.255.1.1`
- [ ] Desde un **PC en VLAN 20**, puedo hacer ping a:
    
    - Su gateway `192.168.20.1`
    - Un PC en VLAN 10 (`192.168.10.X`)
    - Un Servidor de VLAN 30 `192.168.30.X`

---

### EtherChannels y trunks

- [ ] Al ejecutar `show etherchannel summary` en MLS1, MLS2 y MLS3, todos los Port-channels aparecen en estado **SU** (Switch, in Use) y los puertos físicos en estado **P** (in Port-channel).
    
- [ ] Al ejecutar `show interfaces trunk` en MLS2 y MLS3, los Po1 y Po2 respectivamente aparecen como trunks activos con las VLANs 10, 20, 25, 30 y 99 permitidas y la VLAN 57 como nativa.
    

---

### OSPF

- [ ] Al ejecutar `show ip route` en MLS1, aparecen las rutas hacia:
    
    - `192.168.10.0/24` vía `10.255.1.5` (aprendida de MLS2)
    - `192.168.20.0/24` vía `10.255.1.9` (aprendida de MLS3)
    - `192.168.25.0/24` vía `10.255.1.9` (aprendida de MLS3)
    - `192.168.30.0/24` directamente conectada
    - `192.168.99.32/27` vía MLS2
    - `192.168.99.64/27` vía MLS3
- [ ] Al ejecutar `show ip ospf neighbor` en cualquier MLS, aparecen los vecinos OSPF con estado **FULL**.
    

---

### DHCP

- [ ] Un PC conectado a SW1 (VLAN 10) con configuración **DHCP** obtiene automáticamente:
    
    - Una IP en el rango `192.168.10.0/24`
    - Gateway `192.168.10.1`
    - DNS Server `192.168.30.4`
- [ ] Un PC conectado a SW2 (VLAN 20, puertos `f0/3-f0/10`) con configuración **DHCP** obtiene automáticamente:
    
    - Una IP en el rango `192.168.20.0/24`
    - Gateway `192.168.20.1`
    - DNS Server `192.168.30.4`

---

### DNS

- [ ] Desde cualquier PC que haya obtenido IP por DHCP, al ejecutar `ipconfig /all`, el campo **DNS Servers** muestra `192.168.30.4`.
    
- [ ] Desde cualquier PC con alcance a `192.168.30.4`, el comando `ping examenes-redes.com` resuelve correctamente y llega a `192.168.30.4`.
    

---

### FTP

- [ ] Desde la terminal de cualquier PC con acceso a la red `192.168.30.0/24`, el comando `ftp 192.168.30.X` conecta exitosamente al servidor FTP, acepta las credenciales `ftp_admin / cisco123`, y muestra el prompt `ftp>`.
    
- [ ] El comando `dir` dentro del prompt FTP lista el contenido del servidor sin errores.
    

---

### Syslog

- [ ] Al ejecutar `show logging` en cualquier MLS o SW, los mensajes muestran timestamp con fecha y hora.
    
- [ ] En el servidor Syslog (`Services → SYSLOG`), se observan entradas de log provenientes de los dispositivos configurados.
    

---

### NTP

- [ ] Al ejecutar `show ntp status` en cualquier MLS o SW, el resultado muestra:
    
    - `Clock is synchronized`
    - `stratum 2`
    - `reference is 10.255.1.1`
- [ ] Al ejecutar `show ntp associations` en cualquier MLS o SW, aparece `10.255.1.1` con un asterisco (`*`) al inicio, indicando sincronización activa.