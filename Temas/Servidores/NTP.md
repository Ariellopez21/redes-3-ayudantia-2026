# NTP — Network Time Protocol

NTP sincroniza el reloj de los dispositivos de red con una fuente de tiempo de referencia, garantizando que todos los equipos compartan la misma hora. Esto es crítico para correlacionar eventos en logs, certificados digitales y protocolos que dependen del tiempo.

> [!INFO] NTP utiliza el **puerto UDP 123**.

## Jerarquía de estratos

NTP organiza las fuentes de tiempo en niveles llamados **estratos**:

|Estrato|Descripción|
|---|---|
|**0**|Fuentes de tiempo de alta precisión (relojes atómicos, GPS). No participan directamente en la red.|
|**1**|Dispositivos conectados directamente al estrato 0. Son el estándar principal de tiempo en la red.|
|**2 y superiores**|Se sincronizan con el estrato anterior. A mayor número, menor precisión.|
|**16**|Indica que el dispositivo **no está sincronizado**.|

> [!TIP] Dispositivos en el mismo nivel de estrato pueden actuar como respaldo entre sí para mantener la sincronización si el servidor principal falla.

## Configuración

Consultar la hora actual del dispositivo:

```
R1# show clock detail
```

Establecer la hora manualmente:

```
R1# clock set 16:01:00 june 04 2026
```

Apuntar al servidor NTP (similar al uso de `ip helper-address`, pero para tiempo):

```
R1(config)# ntp server [ipv4-servidor]
```

## Verificación

|Comando|Descripción|
|---|---|
|`show clock detail`|Hora actual y fuente del reloj|
|`show ntp associations`|Estrato del dispositivo y del servidor NTP de referencia|
|`show ntp status`|Detalles del servicio NTP en el dispositivo|