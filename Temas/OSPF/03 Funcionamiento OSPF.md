# Estados de funcionamiento de OSPF

Cuando un router se conecta a una red intenta:
- Crear adyacencia de vecinos
- Intercambiar información de enrutamiento
- Calcular las mejores rutas
- Lograr adyacencia

**Estado inactivo**
- Estado `Down`
- Ningún paquete hello recibido
- Transición a Init

**Estado Init**
- Se reciben hellos.
- Los paquetes recibidos contienen en router ID del emisor.
- Transición a Two-Way

**Estado Two-Way**
- Comunicación bidireccional (ambos se conocen)
- En los enlaces de acceso múltiple (los que no son P2P) se elige DR y BDR.
- Transición a ExStart

**Estado ExStart**
- Se ejecuta en redes P2P.
- Los 2 routers deciden quién iniciará el **intercambio de paquetes DBD**.
- Y deciden el número de **secuencia inicial** de paquetes DBD.

**Estado Exchange**
- Los routers intercambian paquetes DBD.
- Sí está **todo actualizado** realiza una transición a **Full**.
- Sí falta información, realiza transición a **Loading**.

**Estado Loading**
- Las LSR y LSU se usan en este estado.
- Se procesan las rutas a través del algoritmo SPF
- Transición a Full.

**Estado Full**
- La LSDB está sincronizada con este router.

---

A continuación se describe cómo se pasa a cada estado de funcionamiento en un router. Y qué se hace en cada uno.

---

## Establecimiento de adyacencia de vecinos

Paso de estado **Down** $\rightarrow$ **Init**

- Se debe habilitar OSPFv2.
- Las interfaces con OSPFv2 habilitado pasarán de **Down** $\rightarrow$ **Init**.
- Se empiezan a enviar **Hello**.

Durante **Init**

- Se debe recibir **Hellos**
- Cada Hello tiene el **Router ID** del router emisor.
- Se debe enviar un **Hello de respuesta**.

Paso de estado **Init** $\rightarrow$ **Two-Way**

- Cuando un router recibe un hello y ve **su propio** router ID pasa de **Init** $\rightarrow$ **Two-Way**

Durante **Two-Way**

- Sí el enlace es P2P, inmediatamente pasamos de **Two-Way** $\rightarrow$ **ExChange**
- Sí el enlace es normal, o sea, por Ethernet común, se debe realizar el proceso de asignación de DR y BDR (**ExStart**).
- El proceso de elección usa: **Valor de prioridad** >  **Router ID más alto**.

> [!example] Visualización simplificada
> ![[Excalidraw/modulo_1.excalidraw.md#^frame=kZJ35D8gG_Sn4ld-s1mx3|center]]
> - Nótese que **ExStart** está en naranjo por ser opcional.
> - Además, la cantidad de paquetes que se envían para la sincronización muy posiblemente sea mayor a uno cada uno.

## Sincronización de bases de datos OSPF

A partir de ahora se sincroniza la **LSDB** y las tablas de **adyacencia** y tablas de **enrutamiento**.

Los primeros tres pasos son para sincronizar las LSDB.

Paso 1: Durante **ExStart**

- Se decide quién envía los paquetes DBD primero (paquetes de Tipo 2)
- Una vez decidido, se pasará de **ExStart** $\rightarrow$ **ExChange**

Paso 2: Durante **ExChange**

- Se intercambian todos los paquetes DBD necesarios (a veces no son los suficientes XD por eso existen los LSR y LSU).
- Los paquetes DBD contienen entradas LSA que aparecen en la LSDB del router que las envía.
- Se tienen muchos datos, tales como:
	- Referencia a una red o enlace.
	- Información del tipo de estado del enlace.
	- Dirección del router que realiza el LSA.
	- Costo de enlace.
	- Número de secuencia.
	- Etc.
- Después de intercambiar una vez la información entre ambos routers, el último router que recibió info decide si le hace falta saber más y pasará de **ExChange** $\rightarrow$ **Loading**

Paso 3: Durante **Loading**

- Se envían todas las LSR necesarias y se responden con todas las LSU necesarias.
- Una vez que las LSDB sean las mismas en ambos routers, existe sincronización y pasará de **Loading** $\rightarrow$ **Full**


> [!example] Visualización simplificada
> ![[Excalidraw/modulo_1.excalidraw.md#^frame=75YxplSI61_KUmeot3jn0|center]]
> - Nótese que a cada tipo de paquete se le asignó un color para ser diferenciado.
> - Además, la cantidad de paquetes que se envían por estado, muy posiblemente, sea mayor a uno cada uno.


> [!tip] ¡Importante!
> Las LSU se envían solo sí se percibe un cambio o cada 30 minutos (predet).

> [!info] El paquete LSAck
> Se usa siempre para acusar recibo en el intercambio de información entre pares, así que siempre debes imaginarte la conversación como un: **A envía** algo a B, **B responde** con un LSAck; **B envía** algo a A, **A responde** con LSAck.

## Designed Router

Como analogía para entender una idea central: En Spanning Tree existen los Root Bridge, que son los switches que actúan como jefes de la distribución completa de switches en un red.

¿Qué diferencia a STP de OSPF?

Primero, obviamente que uno trabaja con Switches y el otro con Routers XD. Segundo, y lo más importante, que STP trabaja con un **Root Bridge que es único en toda la red** y controla toda la subred, mientras que, **los DR y BDR existen por cada segmento de red**.

Qué quiere decir "por cada segmento de red", imaginemos que tenemos la siguiente red:

> [!example] Red 1: Malla
> ![[Excalidraw/modulo_1.excalidraw.md#^frame=gz8oE22ubyKQDGXvwTFx8|800|center]]

Por cada $n$ router, debe haber 
$$n\frac{(n-1)}{2} \text{ adyacencias}$$
Lo que nos da un total de 10 adyacencias $(n=5)$

> [!example] Red 1: Adyacencias
> ![[Excalidraw/modulo_1.excalidraw.md#^frame=8wD980AZCb0kAVw-myRcQ|center|800]]

La cantidad de adyacencia significaría la cantidad de paquetes que se enviarían por router existente (de manera aproximada).

Para evitar esto, al elegir un DR y un BDR, cada paquete se envía hacia ellos 2 y solo el DR reenvía, reduciendo significativamente la carga en red.

Otro caso, sería la siguiente red:

> [!example] Red 2: Estrella extendida
> ![[Excalidraw/modulo_1.excalidraw.md#^frame=m4W3Q-vts6-gPc8u5cEDg|800|center]]

Con esta, sus adyacencias se verían así:

> [!example] Red 2: Adyacencias
> ![[Excalidraw/modulo_1.excalidraw.md#^frame=bftxSFyqu4Hprj_D-v_t1|center|800]]

Ahora, vamos a una red P2P:

> [!example] Red 2: P2P
> ![[Excalidraw/modulo_1.excalidraw.md#^frame=4EHMB74XrHZ3DCPQ1NfB-|center|800]]

Esta posee, claramente una única conexión, en este caso, no hay necesidad de asignar DR y BDR.

Así, si decidiéramos juntar todas las redes en una, tendríamos lo siguiente:

> [!example] Red completa
> ![[Excalidraw/modulo_1.excalidraw.md#^frame=-efMoCCRHhI6KMIbOlLJx|center|1920]]

Considerando los DR y BDR:

> [!example] Red con DRs y BDRs
> ![[Excalidraw/modulo_1.excalidraw.md#^frame=kJ2ih4AO49iDchIw90xLg|center|1920]]

Así es como se evita saturación, se optimiza una red con OSPF, y todo lo demás...