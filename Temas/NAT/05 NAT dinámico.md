## NAT dinámico - configuración

---

### Definición del pool de direcciones

Se establece un conjunto de direcciones IPv4 públicas que serán utilizadas dinámicamente para realizar traducciones.

```cli
ip nat pool NAT-POOL1 209.165.200.226 209.165.200.240 netmask 255.255.255.224
```

- **NAT-POOL1**: nombre del pool.
- Rango: direcciones disponibles para asignación dinámica.
- Máscara: define el tamaño del bloque de direcciones.

> [!info] Comportamiento  
> A diferencia de NAT estático, las direcciones públicas se asignan **según disponibilidad** dentro del pool.

---

### Definición de ACL estándar

Se utiliza una ACL estándar para identificar qué direcciones internas serán traducidas.

```cli
access-list 10 permit 192.168.0.0 0.0.255.255
```

- La ACL define el **conjunto de direcciones elegibles** para NAT.
- En este caso:
    - Se permite toda la red `192.168.0.0/16`.

> [!note] Rol de la ACL  
> No filtra tráfico, sino que **selecciona qué direcciones serán traducidas**.

---

### Asociación entre ACL y pool

Se vinculan las direcciones internas (definidas en la ACL) con el pool de direcciones públicas.

```cli
ip nat inside source list 10 pool NAT-POOL1
```

- **list 10**: referencia a la ACL.
- **pool NAT-POOL1**: conjunto de direcciones públicas disponibles.

> [!warning] Consideración  
> Si no hay direcciones disponibles en el pool:
> 
> - No se realizará la traducción
> - El tráfico no podrá salir a la red externa

---

### Configuración de interfaces

---

#### Interfaz interna

```cli
interface g0/1  
 ip nat inside
```

- Marca la interfaz como parte de la **red interna**.
- El tráfico que ingresa por aquí será evaluado para NAT.

---

#### Interfaz externa

```cli
interface s0/1/0  
 ip nat outside
```

- Marca la interfaz como parte de la **red externa**.
- El tráfico traducido saldrá por esta interfaz.

---

### Flujo de funcionamiento esperado

1. Un host interno genera tráfico.
2. El router:
    - Verifica si la IP origen coincide con la ACL.
3. Si coincide:
    - Se asigna una IP pública disponible del pool.
4. Se crea una entrada en la tabla NAT.
5. El tráfico es enviado hacia la red externa.

> [!info] Comportamiento dinámico  
> Las traducciones:
> 
> - Se crean bajo demanda
> - Se eliminan tras un período de inactividad

---

### Verificación y monitoreo

```cli
show ip nat translations
```

- Muestra las traducciones activas.

```cli
show ip nat translation verbose
```

- Entrega información detallada:
    - Tiempo de vida
    - Estado de la traducción
    - Uso de puertos

---

### Eliminación de traducciones

```cli
clear ip nat translation *
```

- Elimina todas las entradas de la tabla NAT.

```cli
show ip nat translation
```

- Permite verificar que la tabla ha sido limpiada.

> [!warning] Uso de clear  
> Este comando interrumpe las traducciones activas, por lo que:
> 
> - Puede afectar comunicaciones en curso
> - Se recomienda usar en pruebas o troubleshooting