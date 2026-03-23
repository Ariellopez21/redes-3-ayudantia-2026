# Características y funciones

---
**Introducción**

Qué es OSPF: 
- Protocolo de enrutamiento de estado de enlace.
- Fue una alternativa a RIP.
- RIP era problemático. (conteo de saltos)
- OSPF mejor convergencia y escalabilidad

Qué hace:
- Usa el concepto de áreas.
- Se divide el dominio de enrutamiento en áreas para controlar el tráfico de actualización de enrutamiento.

> [!example] Un **enlace** es una **interfaz en un router**
> ![[Excalidraw/modulo_1.excalidraw.md#^frame=un20SkQq|center|700]]

> [!example] Un **vínculo** es también un **segmento de red** que conecta dos routers
> ![[Excalidraw/modulo_1.excalidraw.md#^frame=S79tBetP|700|center]]

OSPF tiene área única y multiárea.

OSPFv2 es para IPv4.

OSPFv3 para ambos.

> [!info] Este módulo está enfocado en OSPFv2 y cubre implementaciones y configuraciones básicas de OSPF de área única.


---


## Componentes de OSPF

- Mensajes
- Estructuras de datos
- Algoritmos

Los mensajes se usan para intercambiar información que se guardan en las estructuras de datos y se procesan a través de algoritmos de enrutamiento.

### Mensajes

Llamados mensajes de protocolo de enrutamiento.

Son cinco tipos de paquetes:
- Hello
- Descripción de la base de datos.
- Solicitud de estado de enlace.
- Actualización de estado de enlace.
- Acuse de recibo de estado de enlace.

> [!example] Tipos de mensajes de protocolo de enrutamiento OSPF
> ![[Excalidraw/modulo_1.excalidraw.md#^frame=IjO8e7CV|center|300]]

### Estructuras de datos

Los mensajes se usan para actualizar tres bases de datos OSPF:
- Adyacencia (tabla de vecinos)
- De estado de enlace (Layer State DataBase) (LSDB) (Tabla de topología)
- De reenvío (tabla de enrutamiento)

#### Adyacencia

- Tabla: de vecinos (**única por cada router**)
- Lista todos los routers vecinos con los que se tiene conexión bidireccional.
- Ver con `show ip ospf neighbor`

#### LSDB

- Tabla: de topología (**identica en todos los routers de la red**)
- Información de todos los routers de la red.
- Ver con `show ip ospf database`

#### Reenvío

- Tabla: de enrutamiento (**única por cada router**)
- Rutas generadas cuando se ejecuta un algoritmo en la LSDB.
- Información de cómo y dónde vienen los paquetes de un router a otro.
- Ver con `show ip route`


> [!example] Utilización de las bases de datos y tablas
> ![[Excalidraw/modulo_1.excalidraw.md#^frame=Thd3SvqG|center]]


### Algoritmo

**Funcionamiento**
- Algoritmo SPF (Short Path First).
- Cada router está en la raíz del árbol.
- Calcula las rutas más corta hacia cada nodo.

A partir de esto, se calcular las mejores rutas.

OSPF coloca las mejores rutas en la **DB de reenvío** y luego crea la **tabla de enrutamiento**.

## Funcionamiento de estado de enlace

Estado de enlace es el proceso para alcanzar la convergencia.

Se usa el **costo** para determinar la mejor ruta.

**Pasos**

1. Adyacencia de vecinos

Deben reconocerse entre sí en la red antes de poder compartir información.

Acá se envían hellos por todas las interfaces con OSPF habilitados.

> [!info] Sí se utilizó `passive int` no se enviará hello.

Sí se detecta vecino, se intenta establecer adyacencia.

2. Anuncios de estado de enlace

Después de la adyacencia de vecinos, los routers intercambian **anuncios de estado de enlace (LSA)**.

Las LSA contienen el estado y costo de cada enlace conectado directamente.

Las LSA se envían saturando a todos los vecinos con OSPF y dichos vecinos también reenvían sus LSA a los demás vecinos cono OSPF.

> [!NOTE] Nota personal
> Entiendo que... En las LSA se envía la información personal del router como bien menciona el estado, costo, quizá las ip, macs, etc.

> [!NOTE] Otra nota
> Los LSA se profundizan en [[02 Paquetes OSPF#Tipos de paquetes OSPF|este archivo]].


3. Crear LSDB

Con toda la información de los LSA enviados, todos los routers trabajan la LSDB.

Así se contruye la **tabla de topología del área**.

4. Algoritmo SPF

A partir de la LSDB, se ejecuta el algoritmo SPF para comenzar a construir la **tabla de topología**.

5. Mejores rutas

Se construye la **tabla de enrutamiento** con las mejores rutas.

> [!important] La ruta que se inserta en la tabla de enrutamiento es a partir del árbol generado a menos que haya una ruta con distancia administrativa menor (como una estática).

## OSPF área única y multiárea

Un área es un grupo de routers que comparten la misma información de estado de enlace en la LSDB.

**Diferencias**

- área única: Todos los routers están en un área; Suele ser el 0.
- multiárea: Se implementan varias áreas, todas conectadas al área troncal (**área 0**); Los routers que interconectan áreas se llaman routers fronterizos de área (ABR).


> [!example] Ejemplo de routers multlárea
> ![[Excalidraw/modulo_1.excalidraw.md#^frame=OOHTa6yV|800|center]]

### Multiárea

Su motivo es sencillo, reducir la carga.

Cada vez que se agrega, modifica, o elimina un enlace, la LSDB cambia y esto implica:
- Ejecutar el algoritmo SPF
- Crear un árbol nuevo
- Actualizar la tabla de routing

Mientras mayor la red, más exigencia en CPU, más tiempo para procesar todo, etc.

Gracias a las áreas, cada área actúa de forma independiente con su propia LSDB. Es decir, la LSDB se ejecuta en su propia área y se actualiza en su propia área, luego, los router ARB existen para **transferir la información** de la LSDB (finalizada) y la tabla de enrutamiento, lo que implica menos procesamiento.

## OSPFv3

Es lo mismo que OSPFv2 pero para IPv4 e IPv6. Con la diferencia importante que OSPFv3 trabaja en la capa de red con IPv6 por lo que OSPFv3 solo se comunica con otros protocolos OSPFv3.

> [!NOTE] Nota personal
> Entiendo que esto significa que un router puede tener habilitados ambos protocolos pero que funcionan de forma independiente:
> - Red IPv4 establece conexión entre routers con OSPFv2
> - Red IPv4 establece conexión entre routers con OSPFv3
> - Red IPv6 establece conexión entre routers con OSPFv3
>
> Pero ninguna se comunica entre sí.

Por eso la siguiente figura mientras que todas las tablas se crean en bases de datos diferentes.

> [!example] Tablas de OSPFv2-v3
> ![[Pasted image 20260319103253.png|center]]
