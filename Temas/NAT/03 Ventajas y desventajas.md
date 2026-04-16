## Ventajas y desventajas

---

### Ventajas de NAT

- **Privatización de intranets**  
  Permite aislar redes internas del acceso directo desde internet.

- **Conservación del espacio de direcciones IPv4**  
  Reduce la necesidad de direcciones públicas al permitir que múltiples dispositivos compartan una misma IP.

- **Multiplexación a nivel de puertos (PAT)**  
  Permite distinguir múltiples conexiones usando números de puerto, optimizando el uso de una sola dirección pública.

- **Flexibilidad en la asignación de direcciones públicas**  
  - Uso de múltiples **pool de NAT**  
  - Implementación de:
    - Pool de respaldo  
    - Balanceo de carga  

- **Independencia del direccionamiento interno**  
  Facilita la administración de red:
  - Cambios en direcciones públicas (por parte del ISP)  
  - No requieren modificaciones en la red interna  

> [!info] Modularidad
> NAT permite desacoplar el esquema de direccionamiento interno del externo, lo que simplifica cambios y migraciones.

- **Ocultamiento de direcciones internas**  
  Mejora la privacidad al evitar la exposición directa de direcciones IPv4 privadas.

---

### Desventajas de NAT

- **Impacto en el rendimiento**  
  - Cada paquete debe ser procesado y modificado.  
  - Afecta especialmente a aplicaciones en tiempo real como **VoIP**.

- **Sobrecarga de procesamiento**  
  La traducción de direcciones:
  - Se realiza continuamente  
  - Añade latencia en el tránsito de paquetes  

- **Escenarios de múltiples traducciones (NAT anidado)**  
  - Puede existir NAT dentro de NAT  
  - Ejemplo:
    - Red privada del cliente → NAT del ISP → salida a internet  
  - Aumenta la complejidad del tráfico

> [!warning] NAT dentro de NAT
> También conocido como **Carrier-Grade NAT (CGNAT)**, puede dificultar la trazabilidad y el diagnóstico de problemas.

- **Pérdida del principio extremo a extremo**  
  - Las direcciones reales de origen y destino no son visibles  
  - Algunas aplicaciones requieren esta información

- **Limitaciones para ciertas aplicaciones**  
  - Protocolos que dependen de direcciones IP reales pueden fallar  
  - Posibles soluciones:
    - NAT estático  
    - Técnicas adicionales (no cubiertas aquí)

- **Dificultad en el seguimiento del tráfico**  
  - No existe visibilidad directa de comunicación **extremo a extremo**  
  - Complica tareas de monitoreo y auditoría

