# Syslog

Syslog es un protocolo estándar que permite a los dispositivos de red enviar mensajes de notificación de eventos a un servidor centralizado de recopilación de logs.

> [!INFO] Syslog utiliza el **puerto UDP 514**.

El servicio de registro Syslog permite:

- Recopilar información para monitoreo y resolución de problemas.
- Seleccionar qué tipo de información se captura.
- Especificar los destinos donde se envían los mensajes (consola, buffer interno, servidor externo).

## Niveles de gravedad

Cada mensaje Syslog tiene asignado un **nivel de gravedad** del 0 al 7. A menor número, mayor criticidad.

|Nombre|Nivel|Descripción|
|---|---|---|
|Emergencia|0|El sistema no puede operar.|
|Alerta|1|Se requiere acción inmediata.|
|Crítico|2|Condición crítica.|
|Error|3|Condición de error.|
|Advertencia|4|Condición de advertencia.|
|Notificación|5|Evento normal pero significativo.|
|Informativo|6|Mensaje informativo sin impacto funcional.|
|Depuración|7|Mensajes generados por comandos `debug`.|

> [!DANGER] Los niveles del 0 al 4 representan errores o problemas que afectan la funcionalidad del dispositivo y deben atenderse con prioridad.

## Recursos (Facility)

Además del nivel de gravedad, cada mensaje está asociado a un **recurso** que indica qué componente lo generó. Ejemplos: `IP`, `OSPF`, `SYS`, `IPSec`, `IF`.

Esto permite filtrar mensajes por origen además de por criticidad.

## Formato del mensaje

```
%facility-severity-MNEMONIC: description
```

Ejemplo:

```
%LINK-3-UPDOWN: Interface Port-channel1, changed state to up
```

- **Recurso:** `LINK`
- **Gravedad:** `3` (Error)
- **Mnemónico:** `UPDOWN` (cambio de estado de enlace)

> [!INFO] Los mensajes más comunes son de **cambio de estado de interfaz** y **coincidencias de ACL**.

## Ejemplo de configuración práctica

Un switch puede configurarse para enviar mensajes de depuración (nivel 7) únicamente al buffer interno, accesible solo por CLI, mientras que los mensajes críticos (niveles 0–4) se envían a un servidor Syslog externo para monitoreo centralizado.

## Comandos

Agregar marca de tiempo a los mensajes:

```
(config)# service timestamps log datetime
```

> [!WARNING] Para que el timestamp sea correcto, el dispositivo debe tener la hora configurada manualmente o estar sincronizado con un servidor NTP.