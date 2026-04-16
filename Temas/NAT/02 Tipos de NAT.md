## Tipos de NAT

---

### NAT estático

- Relación **uno a uno (1:1)** entre:
  - Dirección privada
  - Dirección pública

- Características:
  - La asignación es **fija**.
  - Siempre se usa la **misma IP pública** para un host.

- Uso típico:
  - Servidores o dispositivos que deben ser accesibles desde internet.

> [!warning] Consideración
> Se necesita una IP pública por cada host.

---

### NAT dinámica  
  
- Utiliza un **conjunto de direcciones públicas (NAT Pool)**.  
- Las direcciones se asignan:  
  - **Dinámicamente**  
  - Según disponibilidad (**orden de llegada**).  
  
- Funcionamiento:  
  1. Un host interno solicita salida a internet.  
  2. El router selecciona una IP pública disponible del pool.  
  3. Se crea la traducción temporal.  
  
- Características:  
  - La asignación **no es permanente**.  
  - Cuando la sesión termina:  
    - La IP pública **vuelve al pool**.  
  
> [!info] Diferencia clave con NAT estático  
> - NAT estático: asignación fija    
> - NAT dinámico: asignación temporal reutilizable    
  
> [!warning] Limitación  
> Al igual que NAT estático:  
> - Se requiere una IP pública por sesión simultánea.  
> - Si no hay direcciones disponibles en el pool, **no se pueden establecer nuevas conexiones**.  
  
---  
  
### PAT (Port Address Translation)  
  
- También conocido como:  
  - **NAT con sobrecarga**  
  
- Permite:  
  - Traducir **múltiples direcciones privadas** hacia:  
    - **Una única dirección IPv4 pública** (o un pool reducido).  
  
> [!success] Idea clave  
> La diferenciación entre hosts no se hace solo por IP, sino por:  
> - **Número de puerto**  
  
---  
  
#### Funcionamiento general  
  
- Cada conexión se identifica mediante:  
  - Dirección IP pública  
  - **Puerto de origen**  
  
- Ejemplo:  
  - Red interna: `192.168.32.0/24`  
  - IP pública: `200.223.115.166`  
  
| Host interno        | Traducción PAT       |
| ------------------- | -------------------- |
| 192.168.32.115:1555 | 200.223.115.166:1555 |
| 192.168.32.120:1600 | 200.223.115.166:1600 |
  
- Cada host:  
  - Comparte la misma IP pública  
  - Pero usa **puertos distintos**  
  
---  
  
#### Generación de puertos  
  
- Cuando se inicia una sesión:  
  - **TCP/UDP** → se asigna un **puerto de origen**  
  - **ICMP** → se usa un **ID de consulta**  
  
- Este valor es utilizado por PAT para:  
  - Mantener la unicidad de cada sesión  
  
> [!info] Observación  
> Los puertos de origen suelen estar en rangos **superiores a 1024**.  
  
---  
  
#### Validación de tráfico  
  
- PAT permite el ingreso de tráfico solo si:  
  - Existe una **traducción previa en la tabla NAT**  
  
> [!success] Consecuencia  
> Esto introduce un nivel adicional de:  
> - Control  
> - Seguridad básica  
  
---  
  
### Siguiente puerto disponible en PAT  
  
> [!warning] Problema  
> Puede ocurrir que:  
> - Un puerto ya esté en uso para una misma IP pública.  
  
- Estrategias de solución:  
  
1. **Incremento de puerto**  
   - Si el puerto `1555` está ocupado → se usa `1556`, `1557`, etc.  
  
2. **Uso de múltiples IP públicas**  
   - Si no hay puertos disponibles en una IP:  
     - Se utiliza otra IP del pool  
  
---  
  
### Paquetes sin segmento de capa 4  
  
- Algunos protocolos:  
  - **No utilizan TCP ni UDP**  
  - Por lo tanto:  
    - No tienen **puertos de origen**  
  
- Ejemplo: **ICMP**  
  
- Solución de PAT:  
  - Utiliza el **ID de consulta (Query ID)** como identificador equivalente al puerto.  
  
> [!info] Conclusión  
> PAT adapta su funcionamiento según el protocolo para mantener la capacidad de identificar sesiones únicas.