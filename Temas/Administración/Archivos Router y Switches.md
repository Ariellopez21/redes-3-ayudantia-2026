# Mantenimiento de archivos del router y del switch

## Sistema de archivos IOS (IFS)

El Sistema de Archivos de Cisco IOS (IFS) permite navegar entre directorios y crear subdirectorios en memoria flash o en disco, de forma similar a un sistema de archivos Linux.

|Comando|Descripción|
|---|---|
|`show file systems`|Muestra el directorio actual, todos los directorios disponibles, memoria total y libre, tipo de sistema de archivos y permisos.|
|`dir`|Lista los archivos del directorio actual. Por defecto muestra `bootflash:/`.|
|`cd [directorio]`|Cambia al directorio indicado (ej: `cd nvram`).|
|`pwd`|Muestra el directorio actual sin listar su contenido.|

> [!NOTE] Memoria Flash Es una memoria **no volátil** con almacenamiento persistente: retiene los datos aunque el dispositivo se apague. En ella reside la imagen de Cisco IOS que se carga en la RAM al iniciar.

> [!INFO] Esta sección abarca el uso de los sistemas de archivos TFTP, flash y NVRAM.

---

## Copia de seguridad de configuración

### Usando Tera Term

**Guardar:** En Tera Term, ir a `File > Log...` para comenzar a registrar la sesión. Al ejecutar `show running-config` o `show startup-config`, el output se guarda automáticamente en el archivo de log.

**Restaurar:** Ir a `File > Send file...` y seleccionar el archivo de configuración. El contenido se envía como texto plano ejecutado línea por línea, lo que permite reconfigurar el dispositivo desde cero.

### Usando TFTP

TFTP (Trivial File Transfer Protocol) permite guardar y restaurar configuraciones en un servidor de red.

**Guardar configuración en servidor TFTP:**

```
# copy running-config tftp
```

Se solicitará la IP del servidor y el nombre del archivo de destino.

**Restaurar configuración desde servidor TFTP:**

```
# copy tftp startup-config
```

Se solicitará la IP del servidor y el nombre del archivo previamente guardado.

### Usando USB

**Guardar configuración en USB:**

```
# copy running-config usbflash0:/[nombre_archivo]
```

**Restaurar configuración desde USB:**

```
# copy usbflash0:/[nombre_archivo] running-config
```

> [!NOTE] Para conocer el nombre del dispositivo USB conectado, usar `show file systems`. Generalmente aparece como `usbflash0`.

---

## Recuperación de contraseña

El siguiente procedimiento permite recuperar el acceso a un dispositivo Cisco cuando se ha perdido la contraseña.


> [!example] Diagrama de flujo de la recuperación de contraseñas
> ![[Excalidraw/lab_7.excalidraw.md#^frame=sCkvHDYf|center|800]]


### Paso 1 — Ingresar al modo ROMMON

Interrumpir la secuencia de inicio durante el arranque del dispositivo. Si se realizó correctamente, se verá:

```
Readonly ROMMON initialized

monitor: command "boot" aborted due to user interrupt
rommon 1 >
```

### Paso 2 — Cambiar el registro de configuración

El valor `0x2142` indica al dispositivo que ignore el archivo de configuración de inicio durante el próximo arranque:

```
rommon 1 > confreg 0x2142
rommon 2 > reset
```

### Paso 3 — Copiar startup-config en running-config

Una vez reiniciado sin contraseña, cargar la configuración original:

```
# copy startup-config running-config
```

### Paso 4 — Cambiar la contraseña

```
# enable
# configure terminal
(config)# enable secret [nueva_contraseña]
```

### Paso 5 — Restaurar el registro de configuración y guardar

```
(config)# config-register 0x2102
(config)# end
# copy running-config startup-config
```

### Paso 6 — Recargar el dispositivo

```
# reload
```

> [!INFO] El valor `0x2102` es el registro de configuración normal. Restablecerlo es obligatorio; de lo contrario, el dispositivo seguirá ignorando el startup-config en cada arranque.

