# Conceptos de Seguridad en Redes

---

En este módulo se abordan los principales conceptos de **seguridad en redes**, comenzando por los distintos **agentes** que participan en este entorno, desde hackers hasta hacktivistas y otros actores relevantes. Posteriormente, se analizan las **amenazas**, **ataques** y **vulnerabilidades** presentes en las distintas **capas de la red** (como TCP/UDP, IP, entre otras). Finalmente, se estudian los principios de confidencialidad, integridad y disponibilidad, junto con diversos **métodos de mitigación**, incluyendo el uso de criptografía.

En esta ocasión, el proceso de aprendizaje será dirigido por ustedes. Cada grupo deberá **investigar** un tema específico relacionado con los **contenidos del módulo** y presentarlo al resto del curso. Se espera que aborden aspectos como vulnerabilidades, amenazas, mecanismos de mitigación u otros elementos relevantes según el tema asignado, adoptando un **enfoque** similar al de un **equipo de investigación** que comparte y enseña sus hallazgos.

---

## Introducción al módulo

Según el programa oficial (Cisco CCNA v7), el Módulo 3 abarca a grandes rasgos nueve temas claves: 

1. _Agentes de amenaza_
2. _Herramientas de ataque y defensa_
3. _Malware_
4. _Ataques de red habituales_
5. _Vulnerabilidades y amenazas en IP_
6. _Vulnerabilidades en TCP/UDP_
7. _Servicios IP vulnerables_
8. _Mejores prácticas de seguridad_ 
9. _Criptografía_

Para facilitar el aprendizaje se agrupan estos temas en tres bloques según la siguiente progresión: 

1. Primero, los **agentes de amenaza**.
2. Luego, los **ataques y vulnerabilidades (capa por capa)**
3. Y finalmente, las **medidas de seguridad y mitigación**. 

Cada bloque se asigna a un grupo de estudiantes (**designado en clases**), que dispondrá de 10–15 minutos para exponer sus temas. A continuación se propone la distribución de temas y el enfoque de cada exposición.

---

## Designación de Temas

###### **Grupo 1**  

Este grupo trabajará con los temas 

1. _Agentes de amenaza_
2. _Ataques de red habituales_ 
3. _Vulnerabilidades y amenazas IP_

> [!todo] Recomendación
> El mayor énfasis debe estar en el **Tema 4 (Ataques de red habituales)**, ya que permite comprender el comportamiento real de los ataques en distintas fases. En cambio, el menor tiempo debe dedicarse al **Tema 1 (Agentes de amenaza)**, utilizándolo más como contexto introductorio.

---

###### **Grupo 2**  

Este grupo trabajará con los temas 

1. _Herramientas de los agentes de amenaza_
2. _Vulnerabilidades TCP y UDP_ 
3. _Mejores prácticas en seguridad de redes_

> [!todo] Recomendación
> El mayor tiempo debe centrarse en el **Tema 6 (Vulnerabilidades TCP y UDP)**, por su relevancia técnica dentro del funcionamiento de la red. El menor énfasis debe estar en el **Tema 8 (Mejores prácticas)**, abordándolo de forma más general como cierre.

---

###### **Grupo 3** 

Este grupo trabajará con los temas 

1. _Malware_ 
2. _Servicios IP_  
3. _Criptografía_ 

> [!todo] Recomendación
> El foco principal debe estar en el **Tema 7 (Servicios IP)**, ya que concentra múltiples ataques clave que afectan directamente el funcionamiento de la red. El menor tiempo debe destinarse al **Tema 3 (Malware)**, tratándolo como una base conceptual inicial.

---

Esta distribución es especialmente útil porque permite que todos los estudiantes tengan una visión global del módulo, combinando en cada grupo elementos de **actores**, **ataques** y **mecanismos de defensa**. Además, al mezclar niveles conceptuales y técnicos en cada exposición, se favorece que las presentaciones sean más dinámicas, evitando que un solo grupo concentre toda la teoría o toda la práctica, y promoviendo una mejor participación y comprensión durante toda la actividad.

---

## Roles

Para dar mayor profundidad y dinamismo a las exposiciones, **cada integrante del grupo** asumirá un **rol específico** que orientará su participación durante las preguntas al grupo expositor.

Por ejemplo, Grupo 1:

- Estudiante A (rol Atacante - Agente de amenaza)
- Estudiante B (rol Defensor - Administrador)
- Estudiante C (rol Auditor - Analista de Seguridad)

Cada estudiante deberá formular preguntas exclusivamente desde la perspectiva de su rol, lo que acota el enfoque de preparación, pero al mismo tiempo exige una escucha activa y crítica durante las exposiciones, centrada en un área determinada.

Los roles sugeridos, pensados para ser versátiles y aplicables a todos los temas del módulo, son los siguientes:

- **Atacante:** enfocado en identificar debilidades, vectores de ataque y posibles formas de explotación. Sus preguntas apuntan a cómo vulnerar o comprometer el sistema presentado.
- **Defensor / Administrador:** centrado en la protección, configuración segura y mitigación de riesgos. Sus preguntas se orientan a cómo prevenir, detectar o responder ante amenazas.
- **Auditor / Analista de Seguridad:** orientado a la evaluación, cumplimiento y análisis de riesgos. Sus preguntas buscan justificar decisiones, identificar impactos, evaluar buenas prácticas y detectar posibles omisiones.

---

## Reglas de la actividad

- La actividad se realizará el día **jueves 16 de abril de 2026**, a partir de las **13:00 hrs**, en el **laboratorio de Redes del LabDIC**. En caso de inasistencia el día de la exposición, los involucrados deberá presentar el tema completo en la siguiente clase, instancia en la cual las preguntas serán realizadas por el ayudante del ramo.
- Cada grupo debe exponer entre **10 y 15 minutos**.
- Al finalizar cada exposición, los grupos oyentes deben realizar **al menos 1 pregunta**.
- Se espera un mínimo de **2 preguntas por exposición** en total.
- Todos los grupos conocen previamente los temas de los demás.
- La nota final será **grupal**, considerando todos los criterios evaluados.

---

## Pauta de Evaluación - Exposición en Grupos

| Criterio                | Descripción                                                                             | Puntaje Máximo | Puntaje Obtenido |
| ----------------------- | --------------------------------------------------------------------------------------- | :------------: | :--------------: |
| Manejo del conocimiento | Dominio del tema, claridad conceptual y precisión en la información                     |      30%       |                  |
| Asistencia              | El grupo se presenta completo en la fecha asignada para la realización de la actividad. |       5%       |                  |
| Defensa ante preguntas  | Capacidad de responder correctamente y argumentar ante preguntas                        |      25%       |                  |
| Cumplimiento del tiempo | Exposición dentro del rango de 10 a 15 minutos                                          |       5%       |                  |
| Autoevaluación de pares | Evaluación interna del grupo respecto al desempeño                                      |      10%       |                  |
| Preguntas realizadas    | El grupo realiza al menos 1 pregunta por exposición                                     |      15%       |                  |
| Respeto de roles        | Cumplimiento de los roles asignados durante la actividad                                |      10%       |                  |
| **Total**               | -                                                                                       |    **100%**    |                  |


