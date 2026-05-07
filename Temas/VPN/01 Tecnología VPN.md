# Tecnología VPN

> [!NOTE] Qué es un "sitio" en VPN
> Un sitio se refiere a una ubicación física de red completa.
> Ejemplo:
> - Casa matriz
> - sucursal
> - Oficina regional
> - Bodega
> - Planta industrial
> - Campus universitario
> 
> Cada sitio tiene su propia LAN interna, por eso mismo, se hablará de **sitio a sitio**.

---
## Redes privadas virtuales

- Son para proteger el tráfico de red entre sitios y usuarios.
- Su finalidad es crear una conexión de red privada de extremo a extremo.
- Una VPN es "Virtual" porque transporta la información dentro de una red privada; **No hay cable privado entre tu casa y la empresa, pero la VPN hace creer que sí existe**.
- La información se transporta usando una red pública.
-  Una VPN es "Privada" porque el tráfico se encripta para preservar la confidencialidad de los datos mientras se los transporta por la red pública.

> [!important] Los primeros tipos de VPN
> Eran únicamente túneles IP que no incluían autenticación o encriptación de datos.
> Ej: GRE (Generic Routing Encapsulation), no tiene autenticacción y encapsula tráfico IP dentro de un túnel IP paraa crear un enlace virtual P2P.

---
## Beneficios de VPN

- **Encriptación**: VPN ahora admite encriptación, como IPSec (Internet Protocol Security) y las VPN de SSL (Secure Socket Layer) para proteger el tráfico de red entre sitios.
- **Ahorro de costos**: Usando VPN, las organizaciones **reducen sus costos de conectividad** mientras **incrementan simultáneamente el ancho de banda** de la conexión remota. Esto se debe a que una **VPN permite** utilizar **Internet pública** con **cifrado** para **reemplazar** enlaces **WAN privados** dedicados.
- **Seguridad**: Debido a encriptación y autenticación.
- **Escalabilidad**: Facilita la adición de nuevos usuarios sin agregar más infraestructura, pues, solo son protocolos que facilitan conexión a distancia fuera de la empresa.
- **Compatibilidad**: Gracias a las tecnologías WAN actuales; El trabajo remoto es compatible con el estilo de vida actual.


> [!info] Ejemplo de Beneficios de las VPN empresariales
> Antes se usaba solamente WAN privada tradicional; Significa que se tenía que usar un **enlace dedicado del ISP**, esto se le llama "Línea arrendada" y es literalmente decirle al ISP que te **reserve un enlace físico** para tú tráfico exclusivo.
> Digamos que eres una empresa en Santiago (casa matriz) con 2 sucursales (1 en Puq y otro en Pto Montt).
> **Arrendar una línea** implica que le decías al ISP que querías una línea dedicada entre Santiago ↔ Punta Arenas por ejemplo.; Esto es **caro**, y sí querías **otra línea dedicada**, necesitabas **pagar más** para, por ejemplo, Santiago ↔ Pto. Montt.
> Ahora, gracias a las VPN, puedes usar VPN Sitio a Sitio en internet; Significa que, tu misma empresa, solo necesita contratar algo llamado "fibra empresarial" en las sucursales que lo requieran, digamos, en el router de la casa matriz haré 2 enlaces de fibra empresarial para que tenga permitido usar VPN sitio a sitio. Mientras que, en Puq tendré 1 enlace de fibra empresarial, y en Pto. Montt tendré otro. Finalmente, habilito túneles VPN (por ejemplo con IPSec.).
> 
> Beneficios reales:
> - Ya no debo pagar por una línea dedicada exclusiva.
> - Sí tengo una sucursal más importante en otro punto del mundo, digamos que me expandí tanto que ahora tengo una sucursal en méxico, entonces, puedo fácilmente cambiar mi enlace de fibra empresarial que uso entre Santiago ↔ Pto. Montt y dedicarlo a Santiago ↔ México.
> - Solo debo pagar internet y los servicios de "fibra empresarial".
> - Encriptación, Seguridad, Ahorro de costos, Escalabilidad, Compatibilidad.


> [!tip] Analogía simple
> Es como si antes arrendaras un **camión blindado privado** solo para mover tus cajas.
> En cambio, ahora usas la autopista pública + cajas blindadas con candado: Más rápido, simple, barato, y seguro.
> Pues, la seguridad recae en las **cajas blindadas**, puedes tener **múltiples**, en cambio, un camión blindado, si falla no puedes llevar las cajas en otro camión normal.  

---

## VPN de sitio a sitio

> [!example] VPN sitio a sitio
> ![[Pasted image 20260503180552.png|center|1000]]
> - Se tiene **puertos de enlace VPN**. Los cuales establecen la conexión sitio a sitio.
> - Los puertos de enlace VPN están preconfigurados para establecer un túnel seguro.
> - El **tráfico VPN** solo se **cifra** entre estos **dos dispositivos**.
> - Los usuarios no tienen conocimiento de que se está usando VPN.
> 
> Una frase que se puede usar para comprender el llamado "VPN sitio a sitio", es que gracias a estas VPN, todas las sucursales de una empresa **parece que fueran una sola empresa conectada**. No es una PC, son dos/o más LAN enteras las que están interconectadas
> 
> Es importante recalcar que en redes sitio a sitio, los hosts no saben que existe VPN, pues, los routers con VPN habilitado hacen todo el trabajo.
> 
> Ejemplo de VPN de sitio a sitio, Cadena de supermercados: Supongamos que la oficina central del Super está en Santiago y tienen una sucursal en Puq. En la sucursal de Puq se necesita acceder al sistema de inventario, base de datos, servidor de facturación, en fin... Permitir el acceso por VPN sitio a sitio implica que tenemos que permitir que todos los trabajadores de la sucursal en Puq puedan acceder a la oficina central en Santiago sin necesidad de hacerlo manualmente. Entonces, como todos están conectados por WLAN o Ethernet hacia el router de la sucursal de Puq, cuando el tráfico debe salir a internet, el router de la sucursal de Puq encripta los datos y los tuneliza hacia la oficina central en Santiago, así, nadie en Puq debe estar preocupado de que debe conectarse manualmente a la oficina central en santiago, porque el router de la sucursal ya lo hace por ellos.

---
## VPN de acceso remoto

> [!example] VPN de acceso remoto
> ![[Pasted image 20260503180718.png|center|1000]]
> - Se crea dinámicamente.
> - Establece una conexión segura entre un **cliente** y un **puerto de enlace VPN**.
> - Por ejemplo, VPN SSL de acceso remoto cuando se verifica su información bancaria en línea.
> - Otro ejemplo, Teletrabajo: Estás en tu casa, te conectas a la red de la empresa a través de un cliente VPN. Ahora, puedes acceder a carpetas compartidas, ERP, servidores internos, impresoras remotas; Y tu PC personal entra "como si estuviera adentro".
> - Otro ejemplo, Administración: El admin de la empresa está en un viaje de negocios pero aprovecha para seguir trabajando, por lo tanto, en su notebook se conecta por VPN a la empresa y administra routers, firewalls, incluso servidores, los cuales están todos en la oficina centra de la empresas.

---
## VPN empresariales

> [!example] VPN empresariales
> ![[Pasted image 20260503181407.png|center|500]]
> - Solución común para proteger el tráfico empresarial a través de internet.
> - Se usan tanto VPN de sitio a sitio como de acceso remoto.
> - Incluyen por ejemplo IPSec o SSL.

---

## VPN de ISP

- **MPLS (Multiprotocol Label Switching)**: Es una tecnología de enrutamiento; Se usa para crear rutas virtuales entre sitios. 
- Se utiliza MPLS en capa 2 o capa 3 para crear canales seguros entre los sitios de una empresa.
- Soluciones antiguas de ISP: Frame Relay, VPN ATM (Asynchronous Transfer Mode).

> [!info] Entendiendo MPLS VPN
> Una VPN como las que hemos visto con anterioridad (sitio a sitio y de acceso remoto), como por ejemplo IPSec; Funciona con la siguiente idea "Es como mandar una caja fuerte por una carretera pública".
> En cambio, MPLS, usa etiquetas para identificar a quién le pertenece el paquete, por ejemplo "Este paquete le pertenece al cliente **Empresa X VPN**", entonces, el ISP reenvía el paquete por **rutas virtuales predeterminadas**. **MPLS es un VPN** porque son **privadas**, porque están **encriptadas** y porque son virtuales creando un **túnel**, pero son diferentes a los otros tipos de VPN que vimos (VPN de empresa) porque estos **cambian cómo se logra la privacidad**. Las MPLS crean túneles virtuales que tienen rutas por las que el internet público no está habilitado, y así se logra una seguridad diferente, donde el tráfico es seguro en transporte, mas, no en cifrado, porque no es necesario cifrar.
