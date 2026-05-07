# Tipos de VPN

## VPN de acceso remoto

### Características

- Usa túnel encriptado para conectarse de forma segura a la empresa.
- Los socios pueden tener acceso limitado a servidores, páginas web o archivos según sea necesario.
- Se habilitan dinámicamente por el usuario cuando lo necesita.
- Se puede activar usando IPSec o SSL.

---
## SSL VPNs

- Usan TLS (Transport Layer Security) como versión más nueva de SSL (Secure Socket Layer).
- A veces se usa "SSL/TLS" como sinónimo.
- SSL utiliza llave pública y certificados digitales para autenticar a sus pares.
- IPSec es una mejor opción que SSL VPN cuando la seguridad es un problema.

---

## VPN IPSec


- Se usan para conectar redes a través de otra red no confiable como internet.
- Los usuarios finales envían y reciben tráfico normal TCP/IP sin encriptar. Sin embargo, se usan dispositivos con terminación VPN (puertas de enlace de routers/firewalls con VPN habilitado).
- **ASA**: Cisco Adaptive Security Apliance es un firewall que tiene VPN y prevención de intrusiones.

---

### GRE sobre IPSec


- **Generic Routing Encapsulation** es un protocolo de túnel VPN de sitio a sitio no seguro.
- No encripta por defecto, por lo tanto, su túnel VPN no es seguro.
- Una VPN IPSec estándar (no GRE) solo puede crear tráfico seguro unicast.
- En cambio, GRE puede crear túneles de tráfico multicast y broadcast, y encapsular varios protocolos de capa de red.

---

### Protocolo Pasajero


- Paquete IP encapsulado por GRE.
- **Protocolo Operador** (Carrier Protocol): Paquete GRE que encapsula al original.
- **Protocolo Transporte** (Transport Protocol).

---

### VPN Dinámicas Multipunto


- Todo lo que se hace en lo mencionado con anterioridad es "estático", ya que cada IP asegurada con GRE o IPSec se agrega manualmente.
- Por lo tanto, se necesita un protocolo dinámico.
- **DMVPN** (Dynamic Multipoint VPN) simplifica la configuración del túnel VPN y flexibiliza la conexión a un sitio central con varias sucursales.

---

### Interfaz Virtual del Túnel IPSec


- También existe IPSec VTI (Virtual Tunnel Interface).
- Es capaz de enviar y recibir IP encriptado unicast y multicast, por lo tanto, los protocolos de enrutamiento son compatibles sin tener que configurar túneles GRE.

---

### ISP VPN MPLS


- Diseños tradicionales de WAN: Líneas arrendadas, Frame Relay, ATM.
- Hoy: Gracias a VPN se usa una infraestructura MPLS.
- El tráfico se reenvía a través de la red principal del MPLS (backbone) usando **tags** que se distribuyeron entre los routers principales para identificarse.
