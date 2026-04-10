---

---
# Comandos

---
Recordando como armar una red con contenidos CCNA II.
Checklist:
- [x] VLAN
- [x] Buenas prácticas (VLAN admin/native/consumo)
- [x] SSH
- [x] InterVLAN Routing
- [x] DHCP
- [ ] EtherChannel
- [ ] OSPF
- [ ] FHRP


> [!todo] Checklist
> Los contenidos marcados fue lo que alcanzamos a repasar en 1 clase completa!

---

## Conectar PC a Switch por SSH

Serie de pasos:
1. Meterse al switch
2. crear una vlan
3. meterse a un puerto de interfaz
4. asignar switchport mode access con acceso a la vlan creada
5. meterse a la vlan como interfaz virtual (SVI)
6. asignar ip a la vlan y encenderla
7. configurar ip def gateway (opcional: si se accede por otra subred o vlan)
8. Configurar ssh (version 2, crypto rsa 1024)
9. Crear usuario local del switch (username y password)
10. ingresar a la línea de consola (vty line)
11. modo de acceso (ssh y login local)
12. permitir contraseña de modo privilegiado (para poder usar modo config por ssh)
13. Asignar ip a PC y conectar por ssh (ip de la vlan, nombre de usuario)

### Paso 1

> [!warning] Fíjate en cambiar los range, interfaces, vlan, y las ip para cada vlan que quieras configurar completamente.

> [!error] Este paso se realiza por cada vlan y rango de interfaces que creas.

> [!success] Los pasos 2, 3 y 4 se realizan solo 1 vez por dispositivo

Crear interfaces:
- VLAN
- switchport en interfaz
- SVI

```cli
vlan 10
name ADMIN2
exit
```

```cli
interface range fastEthernet 0/10-11
switchport mode access
switchport access vlan 10
no shutdown
exit
```

```cli
interface vlan 10
ip address 192.168.10.10 255.255.255.0
no shutdown
exit
```

### Paso 2 (opcional)

ip def-gateway:
- Por si se intenta ssh por otra subred o vlan
enable secret:
- Para poder ingresar al modo exec con privilegios desde ssh

```cli
ip default-gateway 192.168.20.1
```

```cli
enable secret admin
```

### Paso 3

Configurar ssh:
- Habilitar ssh 2 y generar key
- Crear usuario local
- Permitir ingreso ssh

```cli
hostname S2-ADMIN
ip domain-name www.umag.cl
crypto key generate rsa // 1024
```

```cli
ip ssh version 2
username admin secret sudo
line vty 0 4
transport input ssh
login local
exit
```

### Paso 4

Configurar PC:
- IP address
- Mask
- Gateway
- Ingresar a **Telnet / SSH Client**

---
## Configurar VLAN Nativa

> [!warning] Revisa bien que la interfaz que estás configurando sea una troncal y no de acceso.

Para seguridad:
- Crear la vlan (como una que no se usará)
- Asegurarse de separarla siempre cuando se use switchport trunk native

```cli
vlan 99
name VLAN_NATIVA
exit
```

```cli
interface g0/1
switchport mode trunk
switchport trunk native vlan 99
no switchport trunk allowed vlan 99
```

> [!NOTE] VLAN nativa: existe pero no se utiliza activamente
> La VLAN nativa en un enlace trunk 802.1Q es aquella cuyos frames se transmiten sin etiqueta (untagged).
>
> Por motivos de seguridad, se recomienda:
> - Cambiar la VLAN nativa por defecto (VLAN 1) a otra VLAN (ej: VLAN 99)
> - No asignar hosts ni tráfico de usuario a esta VLAN
>
> Esto ayuda a mitigar ataques como VLAN hopping y evita mezclar tráfico no etiquetado con tráfico de datos real.
>
> Aunque la VLAN nativa puede transportar tráfico válido, en buenas prácticas se mantiene sin uso operativo.


---
## Configurar VLAN Administratva

Esta es la VLAN que, tenemos que pensar que será usada por los administradores de red (nosotros). Por lo tanto, es la vlan que permitiremos realizar SSH y otras operaciones de administrador.

```cli
vlan 50
name VLAN_ADMINISTRATIVA
exit
```

```cli
interface vlan 50
ip address 192.168.50.50 255.255.255.0
no shutdown
exit
```

```cli
interface g0/1
switchport mode trunk
switchport trunk allowed vlan 50
```

```cli
ip default-gateway 192.168.50.1
```

> [!NOTE] VLAN administrativa (Management VLAN)
> Buenas prácticas:
> - Usar una VLAN dedicada (no VLAN 1)
> - Restringir el acceso solo a administradores
> - No mezclar tráfico de usuarios con tráfico de gestión


---
## Configurar varias VLAN en un switch

- Crear una vlan
- Ingresar a los puertos que se necesitan cambiar con modo switchport access
- Considerar que el switch **debe conocer todas las vlans** que pasan por ella
- Los trunk aun no entran acá

La configuración está en el [[#Paso 1|Paso 1 de Conectar PC a Switch por SSH]].

## Comunicar diferentes VLANs por switches

- Usar enlaces troncales

```cli
interface g0/1
switchport mode trunk
switchport trunk allowed vlan 10,20
exit
```

---
## InterVLAN Routing

### Router-on-a-stick

En el router es una rutina sencilla por cada vlan que se desea permitir el paso.

Recuerda:
- Conectarte a cada subinterfaz y realizar los siguientes pasos en orden.

```cli
interface g0/1.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0
exit
interface g0/1.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0
exit
interface g0/1
no shutdown
exit
```

En el switch hay que permitir el paso por la interfaz que conectar el switch al router (en un palo)

```cli
interface g0/2
switchport mode trunk
switchport trunk allowed vlan 10,20
no shutdown
exit
```

### Swich de capa 3 con SVIs

**Paso 1**

Crear las vlans necesarias `# vlan [id]`.

**Paso 2**

Crear SVIs

```cli
int vlan 10
 ip address 192.168.10.1 255.255.255.0
 no shutdown
```

Paso 3

Activar el enrutamiento con `# ip routing`.

Paso 4.1.

Sí el Switch de capa 3 se está usando directamente conectado a los PCs se debe asignar puertos de acceso:

```cli
interface fa0/1
 switchport mode access
 switchport access vlan 10
```

Paso 4.2.

Sí el Switch de capa 3 se está usando **como router** conectado a un switch, se debe asignar puerto troncal:

```cli
interface g0/1
 no switchport
 ip address 192.168.1.1 255.255.255.0
```

## DHCP

- Solo en los puertos (interfaces) donde _NO_ se encuentra la red del servidor DHCP.

```cli
R1(config-if)# ip helper-address [ip-dhcpv4-server]
```

## FHRP - HSRP

**Configurar gateway virtual**

```cli
Router(config-if)# standby [id-group] ip [virtual-gateway]
```

**Configurar Routers**

```cli
Router(config-if)# standby [id-group] priority [0-255]
Router(config-if)# standby [id-group] preempt
```

## EtherChannel

**Paso 1. Seleccionar interfaces físicas**

- Use el comando `interface range` para elegir varias interfaces al mismo tiempo.

Ej:
```cli
Switch(config)# interface range fa0/1 - 2
```

**Paso 2. Asignar las interfaces a un grupo EtherChannel**

- Cree el canal con `channel-group [number] mode active`.
    
- El número identifica el grupo (ej. `1`).
    
- El modo `active` indica que usará **LACP**.
    
Ej:
```cli
Switch(config-if-range)# channel-group 1 mode active
Creating a port-channel interface Port-channel 1
Switch(config-if-range)#
```

**Paso 3. Configurar la interfaz lógica Port-Channel**

- Ingrese a la interfaz lógica creada automáticamente (`port-channel [number]`).
    
- Ajuste aquí las configuraciones de capa 2 (acceso o troncal).

Ej:
```cli
Switch(config)# interface port-channel 1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan 10,20,30
```

## OSPF

```cli
router ospf 1
router-id 1.1.1.1
passive-interface [int] // Las que no rutean
network [A.B.C.D] [W.X.Y.Z] area [id]
```