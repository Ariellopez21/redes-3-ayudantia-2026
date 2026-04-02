
Configuración


network

```cli
router ospf [id-ospf]
network [ipv4-subred] wildcard [wildcard-mask] area [area-id-ospf]
```

interfaz con ospf

```cli
int g0/0
ip ospf [id-ospf] area [area-id-ospf]
```


interfaces pasivas

```cli
router ospf [id-ospf]
passive-interface [int]
end
show ip protocols
```

Loopback

```cli
int lo 1
ip add 1.1.1.1 255.255.255.255
end
show ip protocols | include Router ID
```

Prioridad

```cli
int g0/0
ip ospf priority [0-255]
end
show ip ospf g0/0
```

Configurar internet en un router de borde

```cli
int lo1
ip add 64.100.0.1 255.255.255.252
exit
ip route 0.0.0.0 0.0.0.0 loopback 1
router ospf 10
default-information originate
end
```

# Lineamiento de la clase


## Observaciones

### Paso 1: Ver vecinos OSPF

`show ip ospf neighbor`

👉 Aquí verás:

- Quién es **DR**
- Quién es **BDR**
- Estados:
    - FULL
    - 2WAY

💡 Clave:

- DR y BDR → FULL con todos
- DROTHER → FULL solo con DR/BDR

---

### Paso 2: Ver información del proceso

`show ip protocols`

Observa:

- Router ID
- Redes anunciadas
- Área
- Temporizadores

---

### Paso 3: Ver detalles de OSPF

`show ip ospf`

Aquí puedes identificar:

- Router ID
- Tipo de red: **Broadcast**
- Información de DR/BDR

---

### Paso 4: Ver interfaz OSPF (CLAVE PARA DR/BDR)

`show ip ospf interface g0/0`

Aquí verás:

- DR → IP del DR
- BDR → IP del BDR
- Priority
- Estado de la interfaz

💥 Este comando explica TODO el proceso de elección

---

### Paso 5: Base de datos LSDB

`show ip ospf database`

Observa:

- LSA tipo 1 (Router)
- LSA tipo 2 (Network → generado por DR)

💡 IMPORTANTE:

- Solo existe LSA tipo 2 si hay DR

---

