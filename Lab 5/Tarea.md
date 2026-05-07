# Tarea - Lab 5

Implementar la red ilustrada en [[#Parte 1]], escalarla tal como indica [[#Parte 2]] y seguir las especificaciones dadas de cada implementación.

---


> [!info] Enviar
> Solamente el archivo `.pkt` con ayudas visuales usando los `label` que packet tracer trae para introducir texto y especificar interfaces en uso, IPv4 abreviadas, u otras ayudas que considere necesarias.


> [!important] ¡Recuerda!
> [[README#Dónde, cómo y qué enviar|Cómo enviar la tarea]]

---

## Parte 1

> [!example] Implementar NAT dinámico
> ![[Excalidraw/modulo_6.excalidraw.md#^frame=B3iJjCom|center|1000]]
> Especificaciones y recomiendaciones:
> - Debe existir routeo entre todas las redes implementadas (y las que vienen en la [[#Parte 2]]), por lo tanto, *se recomienda* uso de OSPF.
> - Recuerde tener claro las interfaces que utilizará entre cada enlace para no tener confusiones a la hora de implementar comandos como `ip address` o `ip nat inside/outside` en los lugares incorrectos.
> - La [[#Parte 1]] **contará como cubierta/realizada con éxito** sí al hacer ping desde más de 1 host en las subredes `192.168.0.0/16`  hacía el servidor `209.165.200.254`, las ip fueron traducidas con **NAT dinámico**. (Esto *se recomienda* verificarlo con `sh ip nat tr` y `sh ip nat st` desde R2).

## Parte 2

> [!example] Implementar PAT - Escalabilidad de red
> ![[Excalidraw/modulo_6.excalidraw.md#^frame=SkRTiIS7FP0TZ-gVO5e1R|center|1000]]
> Especificaciones y recomendaciones:
> - Implementar la nueva subred que conecta R1 - ISP y que da origen al nuevo servidor agregado.
> - IDEM que [[#Parte 1]], o sea, debe poder hacer ping desde `192.168.0.0/16` con varios hosts al nuevo servidor `209.165.200.246` y estas deben ser traducidas usando **NAT con sobrecarga**.
> - *Se recomienda* verificar de la misma forma que con R2.


> [!info] Observaciones
> - No se acepta uso de **NAT estático** en la entrega de la tarea.
> - Sí agrega una breve explicación del funcionamiento de cada protocolo a evaluar, es posible que tenga asegurada la nota máxima del Laboratorio.
