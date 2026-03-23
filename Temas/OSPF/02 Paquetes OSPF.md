# Tipos de paquetes OSPF

Los paquetes de estado de enlace (LSP) son los que vimos en [[01 Características y funciones#Mensajes|Mensajes de protocolo de enrutamiento]].

![[01 Características y funciones#Mensajes]]

**Tipo 1: Paquetes Hello**

- Establece adyacencia a través de **Anuncio de parámetros** entre dos routers.
- Los routers eligen un **Designed Router (DR)**.
- También un **Backup Designed Router (BDP)**.

> [!info] Los enlaces P2P no requieren DR o BDR.

**Tipo 2: Paquete de descripción de base de datos (DBD)**

- Se envía una lista abreviada del LSDB para **sincronizar las LSDB**.
- Los routers que la reciben la usan para compararla con la LSDB local.
- Así se va creando un árbol SPF cada vez más preciso.

> [!tip] LSDB debe ser identica en todos los routers.

**Tipo 3: Paquetes de solicitud de estado de enlace (LSR)**

- Los routers que reciben información pueden **solicitar más info** sobre cualquier entrada de la DBD.

**Tipo 4: Paquetes de actualización de estado de enlace (LSU)**

- Responde LSR.
- Anuncia nueva información **específicamente solicitada**.
- LSUs contiene varios tipos de LSA.

> [!info] ¡Recuerda!
Anuncios de estado de enlace (LSA)

**Tipo 5: Paquetes de acuse de recibo de estado de enlace (LSAck)**

- Responde una LSU.
- El campo de la LSAck está **vacío**.

> [!example] Tipos de LSP
> ![[Excalidraw/modulo_1.excalidraw.md#^frame=5NxbqzCKjGBmKXcSiw5F0|center|800]]


> [!info] Más información sobre paquetes Tipo 4
> - También se usan para reenviar actualizaciones de routing OSPF.
> - Contiene 11 tipos de LSA OSPV2.
> - OSPFv3 cambió el nombre de varias de las LSA y agrega 2 más.


