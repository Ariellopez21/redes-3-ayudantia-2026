## Características

### NAT: Network Address Translation

---

### Motivo de NAT

- El crecimiento de internet provocó el **agotamiento de direcciones IPv4 públicas**.
- Las direcciones privadas se reservan para **redes internas**, optimizando el uso de direcciones públicas.
- NAT surge como mecanismo para **extender la vida útil de IPv4**, permitiendo que múltiples dispositivos compartan una o pocas direcciones públicas.

---

### Direcciones Privadas

- El estándar **RFC 1918** define los rangos de direcciones IPv4 privadas:
  - `10.0.0.0/8`
  - `172.16.0.0/12`
  - `192.168.0.0/16`

- Estas direcciones están diseñadas para:
  - **Comunicación interna** dentro de organizaciones o redes locales.
  - **No ser enrutadas en internet**.

> [!info] Importante
> Las direcciones IPv4 privadas pueden repetirse entre distintas redes, ya que **no son únicas globalmente**.

- Debido a esto:
  - No es posible comunicarse directamente con internet usando direcciones privadas.
  - Se requiere un mecanismo de **traducción a direcciones públicas**.

- Ese mecanismo es precisamente **NAT**.

---

### ¿Qué es NAT?

- NAT (**Network Address Translation**) es un mecanismo que permite:
  - Traducir direcciones **IPv4 privadas → IPv4 públicas**.
  - Facilitar la comunicación entre redes internas e internet.

- Además, NAT proporciona:
  - **Abstracción de la red interna**.
  - Un nivel adicional de **privacidad y seguridad**, ya que:
    - Las direcciones privadas **no son visibles desde el exterior**.
    - Los dispositivos externos solo ven direcciones públicas.

---

### Funcionalidades

- Un router con NAT puede manejar:
  - **Una o varias direcciones IPv4 públicas**.

- Estas direcciones forman el:
  - **Conjunto de NAT (NAT Pool)**.

- Flujo básico de operación:
  1. Un host interno envía tráfico hacia una red externa.
  2. El router NAT:
     - Reemplaza la **IPv4 privada de origen** por una **IPv4 pública** del pool.
  3. El tráfico es enviado a internet.

- Desde el exterior:
  - Todo el tráfico parece provenir de **direcciones públicas válidas**.
  - Las direcciones privadas **nunca son visibles**.

---

### ¿Cómo funciona NAT?

- NAT opera mediante una **tabla de traducción**.

- Esta tabla contiene 4 campos clave:

| Tipo de dirección | Descripción |
|-------------------|------------|
| **Local interna** | Dirección privada del host dentro de la red |
| **Global interna** | Dirección pública asignada por NAT al host interno |
| **Local externa** | Dirección del destino vista desde la red interna |
| **Global externa** | Dirección real del destino en internet |

> [!NOTE] ¡Nota comparativa!
> Al igual que en [[Módulo 5\|ACL]], los conceptos dependen de la **perspectiva**.
> 
> En NAT:
> - **Interno** se refiere a nuestra red local.
> - **Externo** se refiere a internet u otras redes.
> 
> La interpretación siempre se hace **desde el punto de vista del router NAT**.

---

### Ejemplo de Tabla NAT

> [!example] Uso de Tabla NAT
> ![[Excalidraw/modulo_6.excalidraw.md#^frame=1f15k8fN|center|1000]]
> 
> - Escenario:
> 	- Host origen: `192.168.10.10`
> 	- Servidor destino: `209.165.201.1`
> 
> - Flujo:
>   1. El host resuelve el nombre mediante DNS.
>   2. Envía el paquete hacia su gateway.
>   3. El router NAT:
>      - Genera una entrada en la tabla.
>      - Traduce la dirección antes de reenviar el tráfico. 

| Local interna | Global interna  | Local externa | Global externa |
|--------------|----------------|--------------|----------------|
| 192.168.10.10 | 209.165.200.226 | 209.165.201.1 | 209.165.201.1 |

---

### Terminología NAT

- Las direcciones se clasifican según:
  - **Ubicación**: interna / externa
  - **Visibilidad**: local / global

> [!warning] Perspectiva clave
> Toda la terminología NAT se interpreta **desde el dispositivo que realiza la traducción**.

#### Definiciones

- **Direcciones internas**:
  - Asociadas a dispositivos dentro de la red local.

- **Direcciones externas**:
  - Asociadas a dispositivos fuera de la red local.

- **Direcciones locales**:
  - Vistas dentro de la red interna.

- **Direcciones globales**:
  - Vistas en internet.

#### Combinaciones

- **Local interna**:
  - IPv4 privada del host origen.

- **Global interna**:
  - IPv4 pública asignada por NAT al host interno.

- **Global externa**:
  - IPv4 pública real del destino.

- **Local externa**:
  - Dirección del destino vista desde la red interna.
  - Generalmente coincide con la global externa, pero puede diferir si:
    - El destino también está detrás de otro NAT.

> [!info] Caso especial
> Si ambos extremos usan NAT:
> - Una dirección puede ser **global desde una perspectiva**  
> - pero **interna desde otra**.