# SNMP — Simple Network Management Protocol

SNMP es un protocolo de **capa de aplicación** que permite monitorear y administrar dispositivos de red de forma centralizada: routers, switches, firewalls, servidores, workstations, entre otros.

Permite:

- Monitorear y administrar el rendimiento de la red.
- Detectar y resolver problemas de red.
- Planificar el crecimiento de la red.

## Arquitectura: Administrador y Agente

La comunicación en SNMP se basa en dos roles:

|Elemento|Descripción|
|---|---|
|**Administrador SNMP**|Sistema central (NMS) que consulta y controla a los agentes.|
|**Agente SNMP**|Módulo de software en el dispositivo administrado que responde al administrador.|
|**MIB**|Base de Información de Administración. Almacena los datos y estadísticas del dispositivo.|

> [!IMPORTANT] Antes de configurar SNMP en un dispositivo, se debe definir la relación administrador–agente.

Los agentes y sus MIB residen en los dispositivos finales configurables (switches, routers, firewalls, etc.). El administrador accede a esta información de forma remota, siempre con autenticación.

### Puertos

- **UDP 161** — El administrador consulta (sondea) a los agentes.
- **UDP 162** — Los agentes envían notificaciones (traps) al administrador.

## NMS — Network Management System

El NMS es la interfaz administrativa del administrador SNMP. Mantiene un monitoreo constante de todos los agentes, actualizando valores periódicamente mediante sondeos. Para reducir la sobrecarga de red, los agentes también pueden enviar **traps** de forma inmediata ante eventos relevantes.

## Operaciones del administrador

|Operación|Descripción|
|---|---|
|`get-request`|Solicita el valor de una o más variables de la MIB del agente.|
|`get-next-request`|Solicita la siguiente variable en la MIB (útil para recorrer tablas).|
|`get-bulk-request`|Solicita múltiples variables en una sola operación, reduciendo el número de intercambios.|
|`get-response`|Respuesta del agente ante cualquier solicitud `get`.|
|`set-request`|Modifica el valor de una variable en la MIB del agente (operación de escritura).|

## Operaciones del agente

- **get an MIB variable:** El agente recupera el valor solicitado de su MIB y lo envía al administrador como respuesta. Es el resultado de procesar un `get-request`.
- **set an MIB variable:** El agente actualiza el valor de una variable en su MIB según la instrucción del administrador. Permite cambiar configuraciones de forma remota.

## Traps

Los traps son **mensajes no solicitados** que el agente envía directamente al administrador para alertar sobre eventos importantes de forma inmediata, sin esperar un sondeo.

Ejemplos de eventos que generan traps:

- Autenticación incorrecta de usuarios.
- Reinicios del dispositivo.
- Cambio de estado de un enlace (activo → inactivo).
- Seguimiento de MACs.
- Cierre de una conexión TCP (three-way handshake).
- Pérdida de conexión con un vecino.

> [!TIP] Los traps reducen el tráfico de red al eliminar la necesidad de sondeos constantes para detectar eventos críticos.

## Versiones de SNMP

|Versión|Característica principal|
|---|---|
|**SNMPv1**|Primera versión. Seguridad básica mediante comunidades en texto plano.|
|**SNMPv2c**|Mejora el rendimiento con `get-bulk`. Sigue usando comunidades en texto plano.|
|**SNMPv3**|La más segura. Implementa autenticación, cifrado y control de acceso (AAA).|

### Modos de seguridad en SNMPv3

|Modo|Descripción|
|---|---|
|**NoAuthNoPriv**|Sin autenticación ni cifrado. Equivalente en seguridad a SNMPv1/v2. Solo para entornos de prueba.|
|**AuthNoPriv**|Con autenticación (verifica la identidad del origen), pero sin cifrado. Los datos viajan en texto plano.|
|**AuthPriv**|Con autenticación y cifrado completo. Es el modo recomendado para producción.|