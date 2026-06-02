# Administración de Imágenes IOS

La administración de imágenes IOS sigue la misma lógica que el manejo de archivos de configuración (ver [[Archivos Router y Switches]]), pero aplicada a las **imágenes del sistema operativo Cisco IOS**.

El objetivo es usar un **servidor TFTP centralizado** para almacenar, respaldar y distribuir las imágenes IOS de los dispositivos de la red.

---

## Respaldo de imagen IOS al servidor TFTP

### Paso 1 — Tener el servidor TFTP disponible y accesible

Verificar conectividad con el servidor antes de iniciar la transferencia.

### Paso 2 — Identificar la imagen IOS en el dispositivo

La imagen IOS generalmente se encuentra en la memoria `flash:`:

```
R1# show flash:
```

### Paso 3 — Copiar la imagen al servidor TFTP

```
R1# copy flash: tftp:
```

El sistema solicitará:

1. La dirección IP (o nombre) del servidor TFTP.
2. El nombre del archivo de origen (la imagen en flash).
3. El nombre del archivo de destino en el servidor.

**Ejemplo con servidor IPv6:**

```
R1# copy flash: tftp:
Address or name of remote host []? 2001:DB8:CAFE:100::99
Source filename []? isr4200-universalk9_ias.16.09.04.SPA.bin
Destination filename [isr4200-universalk9_ias.16.09.04.SPA.bin]?
Accessing tftp://2001:DB8:CAFE:100::99/isr4200-universalk9_ias.16.09.04.SPA.bin...
Loading isr4200-universalk9_ias.16.09.04.SPA.bin
from 2001:DB8:CAFE:100::99 (via GigabitEthernet0/0/0): !!!!!!!!!!!!!!!!!!!!

[OK - 517153193 bytes]
517153193 bytes copied in 868.128 secs (265652 bytes/sec)
```

---

## Restaurar imagen IOS desde el servidor TFTP

Para cargar una imagen IOS desde el servidor TFTP al dispositivo:

```
R1# copy tftp: flash:
```

El sistema solicitará la IP del servidor, el nombre del archivo en el servidor y el nombre de destino en flash.

> [!WARNING] Al reemplazar la imagen IOS en flash, asegúrese de que hay suficiente espacio disponible. Usar `show flash:` para verificar la memoria libre antes de iniciar la transferencia.