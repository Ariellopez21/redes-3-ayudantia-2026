## PAT (NAT con sobrecarga) - configuración  
  
Escenario:  
- Red interna: `192.168.0.0/16`  
- Red externa: `209.165.200.224/27`  
  
---  
  
### Definición de ACL estándar  
  
Se identifican las direcciones internas que serán traducidas.  
  
```cli  
access-list 10 permit 192.168.0.0 0.0.255.255
```

- Permite toda la red interna.
- Estas direcciones serán elegibles para PAT.


---

### Configuración de interfaz interna

```cli
interface g0/1  
 ip address 192.168.0.1 255.255.0.0  
 ip nat inside
```

- Define el punto de entrada del tráfico interno.

---

### Configuración de interfaz externa

Se utiliza la IP de la interfaz externa como dirección pública para realizar la sobrecarga.

```cli
interface s0/1/0  
 ip address 209.165.200.225 255.255.255.224  
 ip nat outside
```

- La IP configurada será utilizada para representar a múltiples hosts internos.


---

### Activación de PAT (sobrecarga)

```cli
ip nat inside source list 10 interface [int_public]  overload
```

- **list 10**: ACL que define las direcciones internas.
- **[int_public]**: IP pública utilizada.
- **overload**: habilita PAT (multiplexación por puertos).

> [!info] Comportamiento  
> Múltiples hosts internos compartirán una sola dirección IP pública, diferenciándose por **números de puerto**.

---

### Verificación

```cli
show ip nat translations
```

- Muestra las traducciones activas (incluyendo puertos).

```cli
show ip nat statistics
```

- Muestra estadísticas de uso de NAT/PAT.

---

### Prueba recomendada

- Generar tráfico desde varios hosts internos (por ejemplo, `ping` hacia internet).
- Verificar que:
    - Todos usan la misma IP pública.
    - Se diferencian por puertos en la tabla NAT.