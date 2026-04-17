## NAT estático - configuración

---

### Asignación estática de direcciones

- **Dirección local interna (privada)**
- **Dirección global interna (pública)**

Esto implica que un host interno siempre será representado por la **misma IP pública**.

```cli
ip nat inside source static [ipv4_private] [ipv4_public]
```

> [!warning] Consideración importante  
> El argumento `[ipv4_private]` corresponde a un **host específico** dentro de la red interna.  
> No se utiliza para redes completas, sino para asignaciones individuales.

---

### Configuración de interfaces

Para que NAT funcione correctamente, el router debe identificar:

- Qué interfaces pertenecen a la **red interna**
- Qué interfaces pertenecen a la **red externa**

---

#### Interfaz interna

```cli
interface [interface]  
 ip address [ipv4]
 ip nat inside
```

- Se configura la dirección IP del segmento interno.
- Se define la interfaz como **inside**, indicando que:
    - Todo el tráfico que ingrese por aquí será considerado **tráfico interno**.

---

#### Interfaz externa

```cli
interface s0/1/0  
 ip address 209.165.200.1 255.255.255.252  
 ip nat outside
```

- Se configura la dirección IP hacia la red externa.
- Se define la interfaz como **outside**, indicando que:
    - Todo el tráfico que salga por aquí será considerado **tráfico hacia internet**.

---

### Flujo de funcionamiento esperado

1. Un host interno genera tráfico hacia el exterior.
2. El router identifica:
    - Entrada por interfaz **inside**
3. Aplica la regla de NAT estático:
    - Traduce la IP privada a su equivalente pública.
4. Reenvía el tráfico por la interfaz **outside**.

> [!info] Comportamiento clave  
> En NAT estático, la traducción es **permanente**:
> 
> - No depende de sesiones activas
> - Siempre existe en la tabla NAT

---

### Verificación y monitoreo

Para comprobar el funcionamiento de NAT, se utilizan los siguientes comandos:

```cli
show ip nat translations
```

- Muestra la **tabla de traducciones activas**.
- Permite verificar:
    - Correspondencia entre direcciones privadas y públicas.

```cli
show ip nat statistics
```

- Entrega estadísticas del funcionamiento de NAT:
    - Cantidad de traducciones activas
    - Uso de recursos
    - Contadores de paquetes

> [!NOTE] Recomendación  
> Ejecutar estos comandos mientras se genera tráfico (por ejemplo, con `ping`) permite observar cómo se crean y utilizan las traducciones en tiempo real.

