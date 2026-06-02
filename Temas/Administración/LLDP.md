# LLDP — Link Layer Discovery Protocol

LLDP es el equivalente **no propietario** de CDP, definido en el estándar IEEE 802.1AB. Funciona de la misma manera: descubre dispositivos vecinos conectados en el mismo enlace de datos y es compatible con equipos Cisco.

> [!NOTE] LLDP es la alternativa abierta para entornos multifabricante donde CDP no está disponible en todos los dispositivos.

## Diferencia clave respecto a CDP

A diferencia de CDP, LLDP permite controlar de forma **independiente** la transmisión y recepción en cada interfaz, lo que da mayor granularidad en entornos donde se quiere limitar qué información se comparte.

## Configuración

Habilitar o deshabilitar LLDP de forma global en el dispositivo:

```
(config)# lldp run
(config)# no lldp run
```

Controlar transmisión y recepción por interfaz:

```
(config)# interface g0/1
(config-if)# lldp transmit
(config-if)# lldp receive
```

## Verificación

Los comandos de verificación son los mismos que en CDP:

|Comando|Descripción|
|---|---|
|`show lldp`|Estado global de LLDP|
|`show lldp neighbors`|Dispositivos vecinos detectados|
|`show lldp neighbors detail`|Detalle con IP de cada vecino|