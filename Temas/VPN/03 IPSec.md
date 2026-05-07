Tecnologías IPSec
- IPSec es un estándar IETF (RFC 2401-2412)
- Protege y autentica paquetes IP entre origen y destino; Protege tráfico de capa 4 a 7.
- IPSec proporciona:
	- Confidencialidad (encriptación evita lectura)
	- Integridad (hashing evita modificación)
	- Autenticación de Origen (Uso de certificados para proporcional auth de origen/destino)
	- Diffie-Hellman (Método de protección)
- IPSec es muy flexible, por lo que la mejor forma de entenderlo es considerando que para cada función de seguridad esencial, existen varios métodos/opciones de IPSec que ayudan a potenciarlos.


> [!example] Marco de IPSec y sus funciones de seguridad esenciales.
> ![[Pasted image 20260506142306.png|center|1000]]

| Función IPsec    | Descripción                                                                                                                                                                                                                  |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Protocolo IPsec  | IPsec puede usar AH (Encabezado de autenticación) o ESP (Encapsulation Security Protocol). AH autentica el paquete de Capa 3, mientras que ESP lo encripta. La combinación ESP + AH es poco común debido a problemas de NAT. |
| Confidencialidad | La encriptación garantiza la confidencialidad del paquete de Capa 3. Opciones incluyen DES, 3DES, AES, y SEAL. Sin encriptación también es una opción.                                                                       |
| Integridad       | Asegura que los datos lleguen sin cambios al destino utilizando un algoritmo hash, como MD5 o SHA.                                                                                                                           |
| Autenticación    | IKE (Internet Key Exchange) autentica usuarios y dispositivos para la comunicación independiente. Métodos de autenticación incluyen nombre de usuario y contraseña, contraseña única, datos PSK, y certificados digitales.   |
| Diffie-Hellman   | IPsec utiliza DH para intercambiar llaves públicas y establecer una llave secreta compartida. Hay varios grupos de DH disponibles para elección.                                                                             |
