# People Analytics — Hospital Universitari Nexeus
¿Podemos anticipar qué empleados están en riesgo de irse?

---

## ¿Cuál era el problema de negocio?

La rotación de personal es un desafío habitual en el sector sanitario, con impacto directo en la estabilidad de los equipos y en la continuidad asistencial.

En este caso, asumí el rol de analista de Recursos Humanos en el Hospital Universitari Nexeus —un hospital ficticio— donde el departamento de RRHH percibía una posible mayor rotación en el servicio de Urgencias.

Esta hipótesis es coherente con la realidad del sector: Urgencias es un área especialmente exigente por la carga de trabajo, la presión asistencial y la rotación de turnos. Aun así, la percepción no es suficiente; es necesario contrastarla con datos objetivos.

El análisis se estructuró en tres preguntas, cada una construida sobre la anterior:

- ¿La rotación global del hospital está en niveles preocupantes?
- ¿El problema se concentra en Urgencias o afecta a más departamentos?
- Y si no hay un problema en urgencias, ¿Podemos identificar, con los datos disponibles, qué empleados activos tienen mayor riesgo de causar baja voluntaria?

---

## ¿Qué datos usé y de dónde salieron?

Dataset simulado con IA (Claude) con información típica de empleados de un hospital. Las variables cubren seis dimensiones: desde el perfil del empleado hasta señales de riesgo de rotación.

**300 empleados · 24 variables · Período 2018–2026**

| Dimensión | Variables |
|---|---|
| Identificación | `id_empleado` |
| Perfil demográfico | `edad` `genero` `nivel_educativo` |
| Posición en la empresa | `departamento` `categoria_profesional` `seniority` `banda_salarial` `es_jefe` `manager_id` |
| Condiciones del puesto | `tipo_contrato` `modalidad` `turno` |
| Trayectoria | `fecha_ingreso` `fecha_salida` `antiguedad_meses` `estado` `motivo_salida` |
| Señales de riesgo | `horas_extra_mensuales` `absentismo_dias_ano` `puntuacion_desempeno` `training_completado_pct` `satisfaccion_onboarding` `engagement_score` |

---

## ¿Qué herramienta usé y por qué?

**Excel.**

- Es la herramienta de análisis de datos más extendida en el entorno empresarial, lo que la hace especialmente relevante en un contexto de RRHH.
- Permite explorar y analizar datos de forma eficiente sin necesidad de programación.

**¿Cómo organicé el Excel?**

- `datos_crudos` — CSV original. Nunca se modifica.
- `hoja_trabajo` — Copia operativa donde se limpia, transforma y analiza.
- `diccionario_datos` — Definición de cada variable: nombre, tipo, valores posibles y descripción.

> **Colorear encabezados por tipo de dato** — facilita la lectura de un vistazo:
> - Texto/ID: azul
> - Categórico: azul claro
> - Fecha: verde
> - Numérico: naranja
> - Calculado: gris

---

## El proceso analítico de principio a fin

### BLOQUE 1 — Exploración y limpieza

---

#### Paso 1 — Importación y tipado inicial

¿Por qué importar algunas columnas como texto?

- **IDs:** Excel los interpreta como números y puede hacer sumas o eliminar ceros iniciales. Un ID no es un número — es una etiqueta única.
- **Fechas:** Excel las interpreta según la configuración regional del sistema y puede convertir fechas incorrectamente sin avisar. Importar como texto y convertir manualmente garantiza que ves exactamente lo que hay en el fichero.

---

#### Paso 2 — Convertir a tabla y exploración inicial

1. Convertir a tabla para que los filtros sean estables y no se descuadren al añadir o eliminar filas.
2. Revisión columna por columna para entender qué contiene cada variable y detectar dónde puede haber problemas antes de tocar nada.

---

#### Paso 3 — Duplicados

Cada fila representa un empleado. Si un mismo empleado aparece dos veces, cualquier recuento estará inflado.

Se encontraron 4 duplicados (EMP-015, EMP-042, EMP-078, EMP-130).

---

#### Paso 4 — Nulos

Un nulo puede significar dos cosas muy distintas: un error de registro o información válida. Eliminarlos sin entender su significado destruye datos reales.

- `fecha_salida` — nulo significa que el empleado sigue activo. Se rellena con la fecha actual para calcular la antigüedad con la misma fórmula que las bajas.
- `satisfaccion_onboarding` — encuesta no completada. Se rellena con `encuesta_no_completada` para distinguir "el dato no existe" de "la encuesta no se hizo".
- `motivo_salida` — sin entrevista de salida. Se rellena con `sin_motivo_baja` para distinguir empleados activos de bajas sin registrar.
- `manager_id` — el empleado es su propio manager. Se rellena con `propio_manager` para evitar que las tablas dinámicas ignoren esa celda vacía.
- `antiguedad_meses` — baja sin dato calculado. Se deja vacío — no se inventa un dato, se calcula en el paso siguiente.

---

#### Paso 5 — Errores tipográficos en variables categóricas

Excel trata `"Urgencias"`, `"urgencias"` y `"URGENCIAS"` como tres departamentos distintos. Cualquier agrupación posterior estaría contaminada.

Formato estándar elegido: **minúsculas con guion bajo** — consistencia y compatibilidad con cualquier herramienta posterior.

---

#### Paso 6 — Outliers en variables numéricas

Un outlier puede ser un error tipográfico o un valor real extremo. La decisión nunca es automática — hay que contrastar con la fuente.

- `edad` · 17 — menor de edad, imposible. Se elimina la fila.
- `edad` · 91 — fuera de rango laboral real. Se elimina la fila.
- `horas_extra_mensuales` · 180 — error tipográfico evidente. Se corrige a 18h.

---

#### Paso 7 — Variables calculadas

Las variables calculadas son aquellas que el dataset original no incluye pero que serían de utilidad para el análisis —según las hipótesis planteadas— y recomendables para cualquier proyecto futuro.

- `antiguedad_meses` — el dataset original no incluía la duración de los empleados que causaron baja. Sin este dato, no se puede analizar cuánto tiempo permanece un empleado antes de irse.
- `tramo_antiguedad` — la información temporal por sí sola no ofrece información accionable. Se agrupan en cuatro tramos: `0-12m` (early attrition), `1-3a` (consolidación), `3-6a` (consolidado), `+6a` (senior).
- `abandono_temprano` — marca con sí/no si un empleado causó baja antes de cumplir el primer año.
- `es_salida_voluntaria` — separa fuga de talento (el empleado elige irse) de rotación gestionada (despido, fin de contrato, jubilación). Sin esta distinción, cualquier estrategia estaría mal dirigida.

---

### BLOQUE 2 — Análisis según hipótesis

---

#### Hipótesis A — ¿Existe un problema real?

Se crea la variable `año_fecha_salida` para analizar la evolución de las bajas a lo largo del tiempo.

Como indicador principal se utiliza el Turnover Rate, con un benchmark de referencia del 15 % aplicado al último año con datos completos: 2025.

Resultado: 5,33 % — 16 bajas sobre una plantilla media de 300 empleados. Significativamente por debajo del benchmark, y los años anteriores muestran niveles igualmente reducidos.

Sin embargo, al analizar los motivos de salida, la baja voluntaria es la causa predominante: 40 de las 88 bajas totales, superando al resto de motivos en prácticamente todos los años analizados.

2024 fue el año con mayor volumen de salidas; pero 2025 no parece presentar una situación preocupante.

Conclusión provisional: no hay evidencias de un problema general de rotación. Aun así, el peso de las bajas voluntarias justifica seguir profundizando en el análisis.

---

#### Hipótesis B — ¿Es Urgencias donde se pierde el talento?

La percepción inicial apuntaba a que el departamento de Urgencias podía presentar mayores índices de rotación debido a las características propias del servicio: presión asistencial elevada, trabajo a turnos y una mayor exposición al desgaste profesional.

No obstante, los resultados muestran que no se registró ninguna baja voluntaria en Urgencias durante 2025. Por tanto, la hipótesis inicial no queda respaldada por los datos disponibles.

Conclusión: Urgencias no tiene una problemática actual de rotación. Tampoco se identifican otros departamentos problemáticos — el máximo de bajas voluntarias en cualquier área es de 2, un volumen insuficiente para establecer patrones o señalar focos de riesgo.

---

#### Hipótesis C — ¿Qué empleados activos están en riesgo de irse en 2026?

Las hipótesis A y B descartaron un problema de rotación presente. Esto plantea una pregunta diferente: si no hay problema ahora, ¿cómo evitamos que lo haya? El foco pasa de describir el pasado a anticipar el futuro.

**Enfoque**

Se construye un score de riesgo individual para los empleados activos.
Antes de causar baja voluntaria, un empleado desmotivado tiene comportamientos de desvinculación que los datos ya registran.

**Selección de variables**

Se utilizan dos variables, elegidas por dos criterios: son objetivas (no dependen de encuestas) y están disponibles en tiempo real en cualquier sistema de RRHH.

- `absentismo_dias_ano` — quien empieza a desconectarse lo refleja antes en el absentismo que en una conversación con su manager. Es la señal más temprana y la más difícil de disimular.
- `puntuacion_desempeno` — el deterioro del rendimiento refleja desmotivación sostenida, no un mal día. Un empleado que ha decidido irse mentalmente hace lo mínimo antes de ejecutarlo formalmente.

**Umbrales**

- `absentismo_dias_ano` > 10 días → `alerta_absentismo` = 1
- `puntuacion_desempeno` < 6 → `alerta_desempeno` = 1

**Score y semáforo**

`score_riesgo` = `alerta_absentismo` + `alerta_desempeno`

| Score | Nivel | Acción sugerida |
|---|---|---|
| 0 | Sin riesgo | Seguimiento rutinario |
| 1 | Vigilar | Conversación con el manager |
| 2 | Intervenir | Acción inmediata de RRHH |

**¿Qué nos dice el score hoy? Zoom en Urgencias**

A pesar de que no se identificó ninguna problemática de rotación reciente en Urgencias, el score detecta 5 empleados activos en ese servicio con señales de desvinculación que justifican atención de RRHH. El análisis de su perfil revela lo siguiente:

- **Turno** — 3 de cada 5 tienen turno rotatorio y 2 de cada 5 guardia de 24h. La inestabilidad horaria y la intensidad de jornada aparecen como posibles factores de riesgo.
- **Antigüedad** — el rango va de 2 a 8 años de permanencia en la organización. No se trata de incorporaciones recientes: son empleados consolidados, con mayor experiencia y conocimiento acumulado.
- **Manager** — los 5 empleados reportan a managers distintos. El liderazgo directo no es el factor común.
- **Edad** — no parece determinante.
- **Género** — 4 de las 5 personas en riesgo son mujeres. Este dato no es concluyente sin conocer la composición real de la plantilla de Urgencias: si el servicio es mayoritariamente femenino, el resultado sería proporcional y no indicaría nada. Es una observación que requiere contexto antes de interpretarse.
- **Banda salarial** — solo 1 de las 5 personas tenía una banda salarial baja. No destaca como factor relevante.

> ⚠️ El score tiene valor orientativo. No predice con certeza quién se irá, pero permite adelantar intervenciones antes de que la desvinculación sea irreversible.

---

## Conclusiones

La percepción inicial apuntaba a Urgencias como posible foco de rotación debido a la presión asistencial del servicio. Sin embargo, los datos no respaldan esta hipótesis: en 2025 no se registró ninguna baja voluntaria en Urgencias ni se identifican otros departamentos especialmente problemáticos.

Aun así, el análisis predictivo identifica a cinco profesionales de Urgencias con señales de posible desvinculación. Se trata de empleados consolidados, con experiencia y exposición continuada a turnos exigentes. Aunque el score no predice con certeza quién abandonará la organización, sí permite detectar señales tempranas de desgaste y actuar de forma preventiva.

Dado que la operativa de Urgencias limita la posibilidad de modificar sustancialmente horarios e intensidad asistencial, las medidas deberían centrarse en mejorar la previsibilidad de turnos, reforzar las compensaciones y realizar seguimiento preventivo del desgaste profesional.


---

## Limitaciones del proyecto

**1. Dataset simulado**

El análisis se basa en un dataset sintético generado con IA. Esto implica que puede no reflejar la complejidad de un hospital real.


**2. Limitación de la herramienta**

Excel nos permite crear el score a partir de dos variables y umbrales fijos. Su valor es orientativo: señala dónde mirar, no quién se irá con certeza. Una versión más robusta incorporaría un modelo predictivo, lo que requeriría salir de Excel y trabajar con Python.

## Archivos del repositorio

```
.xlsx    → Excel de los datos
README.md → documento del proceso
```