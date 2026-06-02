# CDP — Cisco Discovery Protocol

CDP es un protocolo propietario de Cisco de **capa 2** que permite descubrir dispositivos vecinos conectados directamente, independientemente del medio físico o del protocolo de red utilizado.

Cada dispositivo Cisco envía mensajes CDP periódicamente a sus vecinos, lo que permite construir un **mapa lógico de la red** sin necesidad de documentación previa: cuántos dispositivos hay, cómo están conectados y qué tipo de equipo es cada uno.

> [!NOTE] CDP es especialmente útil cuando la documentación de red es escasa o inexistente. Combinado con SSH/Telnet, permite recorrer la topología dispositivo por dispositivo.

## Configuración

CDP está **habilitado por defecto** en todos los dispositivos Cisco.

|Comando|Descripción|
|---|---|
|`show cdp`|Verifica el estado global de CDP|
|`cdp run`|Habilita CDP en todas las interfaces|
|`no cdp run`|Deshabilita CDP en todas las interfaces|
|`show cdp neighbors`|Muestra los dispositivos vecinos detectados|
|`show cdp neighbors detail`|Incluye la dirección IP de cada vecino|
|`show cdp interface`|Muestra qué interfaces tienen CDP habilitado|

Para habilitar o deshabilitar CDP en una interfaz específica:

```
(config)# interface g0/1
(config-if)# cdp enable
(config-if)# no cdp enable
```

## Uso práctico

Con `show cdp neighbors` se obtiene información de topología lógica: nombres de dispositivos, plataformas, interfaces locales y remotas. Con `show cdp neighbors detail` se agrega la dirección IP, lo que permite conectarse al vecino mediante SSH o Telnet si se dispone de las credenciales.

Este flujo permite ir descubriendo la red de forma incremental desde un único punto de acceso.