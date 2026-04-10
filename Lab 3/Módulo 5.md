# Conceptos de ACL y configuración

---

En este módulo se verán conceptos a grandes rasgos de **Listas de Control de Acceso** (ACL) y su implementación, usos prácticos y recomendados.

---

## Contenidos

- En esta ocasión, no tuve tiempo de prepararles los contenidos como corresponde 😭 lo siento!!!
- Sin embargo, a continuación están todas las configuraciones que probamos en la clase de este módulo, además, en [[Presentación dinámica|este archivo]] se tiene una serie de dibujos realizados para los contenidos de la clase práctica presente, lo que significa que cada serie de dibujos está relacionada con lo visto en clases.

## ACL estándar
### Control de acceso a red

#### **Solo un host**

```cli
conf t
access-list 10 deny 192.168.1.10
access-list 10 permit any

interface g0/1
ip access-group 10 out
end
```

#### **Una red completa**

Para modificar una ACL se debe borrar `no access-list 10`.

```cli
conf t
no access-list 10

access-list 10 deny 192.168.1.0 0.0.0.255
access-list 10 permit any
end
```


> [!important] Nota importante
> Borramos la ACL porque en la práctica, es más útil comenzar una lista de acceso nueva, en vez de corregirla y dejarla con ambigüedades.


> [!important] Otro dato de relevancia
> Se recomienda utilizar un archivo aparte (bloc de notas, archivo `.txt`, u otros) para anotar todas las reglas del ACL antes de implementarla en el router para evitar equivocarse y solo realizar un copiar y pegar.

### Control de acceso a líneas VTY


#### **ACL estándar para acceso VTY:**

```cli
access-list 20 permit 192.168.1.10
access-list 20 deny any
```

#### **SSH:**

```cli
ip domain-name umag.cl  
  
username admin secret admin  
  
crypto key generate rsa
```

#### **Lineas VTY:**

```cli
line vty 0 4
login local
transport input ssh
access-class 20 in
end
```

## ACL extendida

### Control de acceso específico

#### **Regla HTTP en un host**

```cli
conf t
ip access-list extended BLOQUEO_HTTP
deny tcp host 192.168.1.10 host 192.168.2.10 eq 80
permit ip any any

interface g0/1
ip access-group BLOQUEO_HTTP in
end
```

## Verificaciones ACL

```cli
show access-lists
show ip interface g0/1
show run | section access-list
show run | section vty
```
