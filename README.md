# Trabajo Practico Integrador Organización Empresarial - Sosa Carlos Ezequiel
Módulo centralizado de gestión logística y redistribución de stock para HABiTAR (DILCAR S.A.). TPI de Organización Empresarial, UTN - TUP a Distancia.
# Módulo Centralizado de Gestión Logística y Redistribución de Stock – Habitar / DILCAR S.A.


| | |
|---|---|
| **Tablero ágil** | [GitHub Projects/Tablero Ágil](https://github.com/users/Carlosesosa/projects/1/views/1) |
| **Repositorio** | [Repositorio - Trabajo Practico Integrador Organización Empresarial/Sosa Carlos Ezequiel](https://github.com/Carlosesosa/Trabajo-Practico-Integrador-Organizacion-Empresarial-Sosa-Carlos-Ezequiel) |
| **Docente titular** | Prof. Gabriela Martínez |
| **Docentes adjuntos** | Prof. Carolina Bruno, Prof. Mario Raúl López, Prof. Andrea Ramos |
| **Fecha de entrega** | 1 de noviembre de 2026 |

## Integrantes

- Carlos Ezequiel Sosa

## Descripción del sistema

Habitar (DILCAR S.A.) es una cadena comercial de productos para el hogar con 37 sucursales en Chaco, Corrientes, Formosa, Misiones y Santa Fe. Hoy cada sucursal realiza sus propios pedidos de mercadería según lo que considera necesario, lo que puede generar sobrestock de productos con poca demanda local mientras otra sucursal podría venderlos con mayor facilidad.

El proyecto propone un **módulo centralizado de gestión logística y redistribución de stock** que:

- Centraliza en una base de datos las existencias, ventas y movimientos de mercadería de todas las sucursales.
- Analiza periódicamente la rotación de cada producto y genera **alertas de baja rotación** (regla inicial: 30 días sin ventas, configurable por categoría).
- Busca sucursales con demanda y stock insuficiente, y **recomienda traslados** con la cantidad estimada.
- Deja la decisión final en manos del personal autorizado de Logística (la alerta nunca mueve mercadería de forma automática).
- Registra todo el ciclo del traslado (recomendado, autorizado, en transporte, recibido, con incidencia, rechazado) para garantizar trazabilidad.

**Objetivo general:** diseñar, desarrollar e implementar el módulo, con posibilidad de integración vía API con los sistemas existentes de venta e inventario, para optimizar la rotación de mercadería entre las 37 sucursales, reducir el capital inmovilizado y apoyar la toma de decisiones en datos históricos de demanda.

### Módulos de la solución

1. Gestión de productos y sucursales
2. Consulta y actualización de stock
3. Análisis de ventas y rotación
4. Alertas y recomendaciones
5. Reportes para el área de Logística

### Entidades principales

`PRODUCTO`, `SUCURSAL`, `STOCK`, `VENTA`, `MOVIMIENTO_MERCADERIA`, `ALERTA`, `USUARIO`.

## Modelado del proceso (BPMN)

Diagrama del proceso de redistribución de mercadería entre sucursales, con Pools y Lanes:

![Diagrama BPMN](diagrama%20BPMN.png)

| Pool / Lane | Rol en el proceso |
|---|---|
| **Pool HABITAR (DILCAR S.A.)** | Organización donde se implementa el módulo. Contiene cinco lanes. |
| Lane Sistema (automatizado) | Consulta stock y ventas, genera alertas, busca sucursal candidata, recomienda traslados y actualiza existencias (tareas de servicio). |
| Lane Logística | Revisa la alerta y la recomendación, autoriza o rechaza el traslado y coordina el envío. |
| Lane Sucursal origen | Prepara y despacha la mercadería autorizada. |
| Lane Distribución y Transporte | Flota propia de Habitar: retira la mercadería en origen y la entrega en la sucursal destino. |
| Lane Sucursal destino | Recibe y controla la mercadería, y registra la recepción o la incidencia. |
| **Pool Sistemas de venta e inventario existentes (externo)** | Provee datos de stock y ventas mediante API. |


**Flujo estándar:** el sistema analiza ventas y stock → detecta un producto que supera el plazo sin ventas → genera la alerta → busca una sucursal candidata → recomienda el traslado → Logística revisa y autoriza → la sucursal de origen prepara y despacha y el sistema deja la mercadería en tránsito → Distribución y Transporte retira en origen y entrega en destino → la sucursal destino recibe y controla → se actualiza el stock de ambas sucursales.

**Compuertas de decisión (excepciones):**

| Compuerta | Si la respuesta es "No" |
|---|---|
| ¿Supera el plazo sin ventas? | El producto no genera alerta y se vuelve a evaluar en el siguiente ciclo. |
| ¿Existe sucursal candidata? | La alerta queda registrada sin traslado recomendado, para revisión manual. |
| ¿Logística autoriza el traslado? | El movimiento pasa a estado *Rechazado* y se registra quién lo decidió. |
| ¿Coinciden cantidad enviada y recibida? | Se registra una incidencia (*Con incidencia*) para su revisión. |

## Historias de usuario

Formato: **Como** [usuario], **quiero** [funcionalidad], **para** [beneficio].

### HU-01 – Alertas de baja rotación

**Como** responsable de Logística, **quiero** recibir alertas de los productos que superan el plazo configurado sin registrar ventas en una sucursal, **para** evaluar su traslado antes de que la mercadería quede inmovilizada.

**Criterios de aceptación**
- El sistema analiza ventas y stock de forma periódica y automática.
- Se genera una alerta cuando un producto con stock mayor a cero supera `dias_limite_sin_venta` (valor inicial: 30 días) sin ventas en esa sucursal.
- La alerta muestra producto, sucursal, días sin venta, motivo y fecha de generación.
- Las alertas nuevas quedan en estado *Pendiente* y no se duplican para el mismo producto y sucursal mientras sigan abiertas.

### HU-02 – Revisión y autorización de traslados

**Como** responsable de Logística, **quiero** revisar las recomendaciones de traslado y autorizarlas o rechazarlas, **para** decidir la redistribución considerando compromisos de venta y costos de transporte, sin que el sistema mueva mercadería por sí solo.

**Criterios de aceptación**
- Cada recomendación indica producto, sucursal de origen, sucursal de destino y cantidad sugerida.
- Solo los usuarios con rol *Logística* pueden autorizar o rechazar.
- Al autorizar o rechazar, el sistema registra el usuario, la fecha y el nuevo estado.
- El sistema valida que origen y destino sean distintos y que la cantidad sea mayor que cero y no supere el stock del origen.

### HU-03 – Confirmación de recepción

**Como** encargado de sucursal, **quiero** confirmar la recepción de la mercadería indicando las unidades realmente recibidas, **para** mantener el stock actualizado y dejar registrada cualquier diferencia con lo enviado.

**Criterios de aceptación**
- La sucursal destino puede confirmar solo los traslados en estado *En transporte* dirigidos a ella.
- Al confirmar, se actualiza el stock del destino con la cantidad recibida.
- Si `cantidad_recibida` difiere de `cantidad_enviada`, el traslado pasa a *Con incidencia* para su revisión.
- Queda historial de cada movimiento consultable por Logística.

## Tablero ágil

Las historias se gestionan en el tablero: [GitHub Projects/Tablero Ágil](https://github.com/users/Carlosesosa/projects/1/views/1)

| Historia | Issue | Estado actual |
|---|---|---|
| HU-01 – Alertas de baja rotación | #5 | Por Hacer |
| HU-02 – Revisión y autorización de traslados | #6 | En Progreso |
| HU-03 – Confirmación de recepción | #7 | Listo |
| Documentación y diagrama BPMN | #8 | Listo |

> El estado de cada tarjeta se usa para evidenciar el seguimiento del tablero y refleja el avance de la documentación del proyecto, no del código.

## Indicadores de control y estándares de desempeño

Indicadores para la auditoría técnica del sistema. Los primeros cuatro derivan de las metas de la Entrega N°2.

| Indicador | Fórmula / medición | Estándar (meta) | Frecuencia | Fuente |
|---|---|---|---|---|
| Tiempo de respuesta de consultas de stock | % de consultas por API resueltas en menos de 2 s | ≥ 95 % | Por prueba de carga y mensual | Logs de la API |
| Redistribución de mercadería inmovilizada | Productos con más de 30 días sin ventas redistribuidos ÷ total de productos en esa condición | ≥ 35 % en los primeros 6 meses | Mensual | Tablas ALERTA y MOVIMIENTO_MERCADERIA |
| Capacidad de procesamiento | Registros diarios de transacciones sincronizados sin error | Hasta 100.000 por día | Diaria | Logs de sincronización |
| Cumplimiento del plazo de desarrollo | Semanas reales ÷ semanas planificadas | ≤ 1,0 (16 semanas planificadas) | Semanal | Diagrama de Gantt y tablero |
| Tasa de incidencias en recepción | Traslados *Con incidencia* ÷ traslados recibidos | A definir tras el piloto (referencia inicial: ≤ 5 %) | Mensual | MOVIMIENTO_MERCADERIA |
| Tiempo de gestión de alertas | Tiempo promedio entre generación de la alerta y su resolución | ≤ 3 días hábiles (referencia inicial; a ajustar tras el piloto) | Semanal | ALERTA |
| Trazabilidad de autorizaciones | Traslados autorizados con `id_usuario_autoriza` registrado | 100 % | Mensual | MOVIMIENTO_MERCADERIA |
| Integridad de stock | Registros con `cantidad_disponible` negativa o duplicados de producto + sucursal | 0 | Diaria | Tabla STOCK |

## Flujo de trabajo con Git

**Ramas**
- `main`: versión estable y entregable. El commit inicial se hizo directamente sobre ella; el resto de los cambios se incorpora mediante ramas secundarias y Pull Requests.
- `carga-de-documentos-y-diagrama`: se utilizó para la carga de documentos y corrección del archivo README.
- `actualizacion-documento-final`: rama secundaria utilizada para actualizar los links, la estructura del README y subir la versión final del documento.

**Mensajes de commit**
- `main: initial commit` 
- `main: Redacción archivo README`
- `main: Actualización de links en README.md para proyecto y repositorio`
- `carga-de-documentos-y-diagrama: carga de BPMN.png`
- `carga-de-documentos-y-diagrama: actualizacion de estructura de archivo README`
- `actualizacion-documento-final: se borra archivo docx y se deja el archivo en pdf`

## Pull Request y revisión de pares

| PR | Rama | Que Incorpora |
|---|---|---|
| #9 | carga-de-documentos-y-diagrama → main | Diagrama BPMN, documento consolidado de la Entrega N°4 y actualización del README. |
| #10 | actualizacion-documento-final → main | Links del README al nuevo nombre del repositorio, imagen del BPMN apuntando a main, sección de Pull Request, Issue #8 en Listo, ramas, commits y estructura actualizados, y reemplazo del .docx por el PDF. |
| #11 | main → actualizacion-documento-final | Sincronización de la rama con main para continuar las ediciones pendientes. |
|#12 | actualizacion-documento-final → main | Versión final: PDF consolidado, diagrama BPMN actualizado, indicadores, declaración de IA y README definitivo. |

- **Revisado por (PR #9):** [@Eliasgsosa](https://github.com/Eliasgsosa)
- **Issue vinculado:** [#8](https://github.com/Carlosesosa/Trabajo-Practico-Integrador-Organizacion-Empresarial-Sosa-Carlos-Ezequiel/issues/8)  (`Closes #8`)
- **Qué se revisó:** coincidencia del diagrama BPMN con el flujo y las compuertas descritas, links y estructura del README.
- **Resultado:** aprobado y mergeado.

## Estructura del repositorio

```
.
├── README.md
├── TP Integrador Organización Empresarial Entrega n4 - Sosa Carlos Ezequiel.pdf     Documento consolidado
└── diagrama BPMN.png    Diagrama del proceso
```

## Declaración de Uso de IA

Para la elaboración de esta entrega se utilizó Claude (Anthropic) y ChatGPT, como apoyo en la organización y estructuración del documento, la revisión de la redacción y el formato de presentación. También se empleó como herramienta de consulta para trabajar sobre el diccionario de datos, el diagrama BPMN, diagrama de Gantt, etc.
El caso de estudio, la problemática logística, la propuesta de mejora y el organigrama se basan en las entregas previas y en mi experiencia laboral en la empresa. Como estudiante, asumo la responsabilidad de revisar y verificar el contenido final, procurando mantener su coherencia con los trabajos anteriores y con los contenidos de la asignatura. La responsabilidad por la versión presentada corresponde al estudiante.

### Herramientas utilizadas

| Entrega | Herramienta | Uso |
|---|---|---|
| N°1 y N°2 | ChatGPT | Organizar ideas, mejorar la redacción, estructurar las respuestas y el diagrama de Gantt. |
| N°3 | Claude (Anthropic) | Organizar y dar formato al documento, revisar la redacción y trabajar sobre el diccionario de datos. |
| N°4 | Claude (Anthropic) | Organizar la redacción del README.md, la consolidación del documento y ayuda al generar y corregir el Diagrama BPMN. |

### Prompts utilizados y su justificación

| Prompt | Por qué lo formulé así | Qué resultado obtuve |
|---|---|---|
| Armá un Gantt de 16 semanas con 5 fases, precedencias y roles | Definí yo las fases, el plazo y los roles, y usé la IA solo para organizar la tabla. |  Un cronograma con tareas interdependientes que revisé para chequear que las precedencias fueran coherentes. |
| Ayudame a estructurar el análisis de Habitar como sistema abierto (entradas, procesos, salidas, ambiente) según la TGS | Le pasé la consigna y el contexto real de la empresa para que la respuesta siguiera la teoría vista en la materia y no fuera genérica. | Un esquema ordenado de la organización como sistema abierto, que luego completé con datos de la empresa. |
| Ayudame a modelar en BPMN el proceso de redistribución de mercadería con Pools, Lanes y compuertas de excepción | Indiqué el flujo del algoritmo (alerta, recomendación, autorización, tránsito, recepción) para que el modelo reflejara mi diseño. | Un borrador de diagrama BPMN que revisé y ajusté. |
| Armá indicadores de control para auditar el sistema | Pedí que los primeros indicadores derivaran de las metas cuantitativas de la Entrega N°2, para mantener coherencia. | Una tabla de indicadores con fórmula, estándar y fuente, donde dejé "a definir tras el piloto" en los que no tenían un dato real. |

### Correcciones humanas sobre los resultados de la IA

- La IA tendía a automatizar el traslado, corregí que la alerta **no mueve mercadería** y que la decisión final la toma personal autorizado.
- Hice configurable el plazo de 30 días por categoría de producto.
- La problemática logística surge de mi experiencia laboral en Habitar, la IA no la identificó.
- La referencia comparativa con Cetrogar proviene de información que obtuve a través de un familiar, no de la IA.

### Responsabilidad

Revisé y verifiqué el contenido final de este repositorio y asumo la responsabilidad por la versión presentada. Las herramientas de IA se usaron como apoyo, no como reemplazo del análisis propio.

## Bibliografía

- HABiTAR. (s.f.). [*Quiénes somos*](https://habitar.com.ar/quienes-somos)
- HABiTAR. (s.f.). [*Sucursales*](https://habitar.com.ar/sucursales)
- HABiTAR. (s.f.). [*Cómo comprar online*](https://habitar.com.ar/como-comprar)
- Sosa, C. E. (2026). *Trabajo Práctico Integrador, Entregas N°1, N°2 y N°3*. Organización Empresarial, Tecnicatura Universitaria en Programación, UTN.
