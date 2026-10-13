# Spec 001 — Revisión práctica de Machine Learning

| | |
|---|---|
| **Estado** | Revisada por QA; sin dudas abiertas |
| **Fecha** | 2026-10-12 |
| **Fuentes** | Enunciado del notebook (bloques 1–3), constitución del proyecto, entrevista de aclaración, revisión QA (Anexo A) |
| **Alcance de este documento** | El QUÉ y el POR QUÉ. Herramientas, estructura interna y nombres de archivos se resuelven en el plan. |

En todo el documento, **«el notebook»** es el sistema al que se aplican los requisitos.

---

## 1. Contexto y objetivo

### Contexto

La práctica es un repaso de fundamentos de Machine Learning: se comparan métodos sobre los mismos datasets y se explican los resultados. No requiere conocimientos de Recuperación de Información.

El enunciado permite usar IA generativa para producir código, pero exige que quien lo entrega pueda **explicar las decisiones y los resultados**. Por eso esta spec no se centra en obtener la mejor métrica, sino en que cada resultado llegue acompañado de una predicción previa, un protocolo sin fugas de información y una interpretación que no afirme más de lo que los datos permiten.

### Objetivo

Que quien ejecute el notebook desde cero obtenga una comparación **honesta y reproducible** de:

1. modelos de clasificación sobre Iris,
2. modelos de regresión sobre Diabetes,
3. el efecto de la regularización en Ridge y Lasso,

con cada experimento precedido por una predicción y seguido por una interpretación basada en los resultados obtenidos.

### Por qué importa

Los riesgos reales de esta práctica son conclusiones sin respaldo: ajustar con los datos de prueba, declarar ganador a un modelo por una diferencia de un solo ejemplo, o tomar un coeficiente predictivo por una causa. La spec existe para hacer esos fallos detectables.

### Decisiones confirmadas

D1–D7 son decisiones del autor, tomadas en la entrevista de aclaración (que no está en el repositorio). D8–D12 provienen de la revisión QA de esta spec (Anexo A).

| ID | Decisión |
|---|---|
| D1 | Partición 60/20/20 (estratificada por clase en Iris), máximo 4 configuraciones por modelo en el ajuste (contando la inicial; excepción única en D8) y una única semilla fija. |
| D2 | Si en prueba hay menos de dos instancias mal clasificadas, el análisis se completa con errores de validación de los modelos ya congelados (elegidos según CA-14.2), rotulando de qué partición y de qué modelo proviene cada uno. |
| D3 | La intensidad de regularización con mejor error de validación en el barrido es la que se congela para Ridge y Lasso en la evaluación de prueba. |
| D4 | Los avisos (p. ej. no convergencia) no se ocultan; si se corrigen, la corrección cuenta dentro del límite de configuraciones y se decide solo con validación. |
| D5 | La predicción inicial no se reescribe tras ver resultados y siempre se contrasta explícitamente en la interpretación. |
| D6 | «Modelo más sencillo de programar» se responde con un criterio objetivo declarado en la celda de predicción, antes de implementar. |
| D7 | Toda la implementación se edita únicamente en el notebook de la práctica; no se crean otros archivos de código. |
| D8 | Una **configuración** es una combinación distinta de hiperparámetros (incluidos `max_iter` y `solver`); la semilla no cuenta. La inicial la elige y declara el autor antes de entrenar y cuenta dentro de las 4. La única excepción al límite es el barrido de alpha de Ridge y Lasso (RF-21). |
| D9 | **Congelar** es fijar la configuración y el modelo ajustado únicamente con entrenamiento; no se reentrena con validación antes de prueba. |
| D10 | La métrica que decide en validación es **F1 macro** (clasificación) y **RMSE** (regresión); accuracy y MAE son apoyo y primer desempate. Si persiste el empate, gana la configuración probada primero. |
| D11 | Un modelo es **no distinguible del mejor observado** si queda a ≤ 1 error de prueba de él (clasificación) o a ≤ 5 % de su RMSE de prueba (regresión; el porcentaje se calcula sobre el RMSE del mejor). Cada modelo se compara contra el mejor, no entre pares, para evitar empates encadenados. Es un criterio práctico, no una prueba de significancia. |
| D12 | Hay tres experimentos (E1–E3). La regularización es parte de E3, como en el enunciado. |

---

## 2. Usuarios

| Usuario | Qué necesita |
|---|---|
| **Estudiante-autor** | Conoce ML a nivel de curso previo, no necesariamente Recuperación de Información. Necesita un guion que lo obligue a predecir, comparar con criterio y explicar con sus propias palabras. |
| **Revisor** (docente o par) | Necesita ejecutar el notebook desde cero, obtener los mismos números y comprobar sin ambigüedad que la partición de prueba no se usó para ajustar y que las respuestas se apoyan en los resultados. |

---

## 3. Historias de usuario

| ID | Historia | Requisitos |
|---|---|---|
| HU-1 | Como estudiante, quiero entender qué datos tengo y cómo los voy a dividir antes de modelar, para saber qué estoy prediciendo y no contaminar la evaluación. | RF-2, RF-8 |
| HU-2 | Como estudiante, quiero comparar siete clasificadores bajo el mismo protocolo, para explicar en qué se diferencian y no solo cuál puntúa más. | RF-9 a RF-13, RF-15 |
| HU-3 | Como estudiante, quiero examinar errores concretos de clasificación, para distinguir lo que puedo afirmar de lo que solo es hipótesis. | RF-14 |
| HU-4 | Como estudiante, quiero comparar ocho regresores con métricas complementarias y diagnósticos de residuos, para entender cómo y dónde falla cada uno. | RF-16 a RF-20, RF-22 |
| HU-5 | Como estudiante, quiero ver cómo cambia el comportamiento de Ridge y Lasso al aumentar la penalización dentro de la comparación de regresión (E3), para relacionar regularización con sobreajuste. | RF-21, RF-22 |
| HU-6 | Como estudiante, quiero registrar una predicción antes de cada experimento y contrastarla después, para detectar dónde mi intuición falla. | RF-1, RF-6 |
| HU-7 | Como revisor, quiero reejecutar todo desde cero y obtener los mismos resultados, con la prueba usada una sola vez y los avisos visibles, para confiar en las conclusiones. | RF-2 a RF-5, RF-7, RNF-1 |

---

## 4. Requisitos funcionales

Los criterios de aceptación usan notación EARS en español:
**Ubicuo** («El notebook deberá…»), **Evento** («Cuando…, el notebook deberá…»), **Estado** («Mientras…, el notebook deberá…»), **Opcional** («Donde…, el notebook deberá…») y **No deseado** («Si…, entonces el notebook deberá…»).

### 4.1 Requisitos transversales

RF-1 a RF-7 son transversales: se verifican sobre todos los experimentos y no tienen celda propia (RNF-6).

#### RF-1 — Estructura de cada experimento

**Por qué.** Predecir antes, medir y luego interpretar es lo que convierte la comparación en aprendizaje, y hace revisable cada paso.

- **CA-1.1** El notebook deberá organizar cada experimento (E1 Datos y protocolo, E2 Clasificadores, E3 Regresión y regularización) en tres celdas consecutivas y en este orden: predicción inicial (Markdown), implementación con resultados (código), interpretación (Markdown).
- **CA-1.2** Todo experimento deberá responder a al menos un requisito de esta spec, y cada requisito RF-8 a RF-22 deberá implementarse en un experimento.
- **CA-1.3** Si la interpretación de un experimento no cita resultados obtenidos en su propia implementación, entonces ese experimento se considerará incompleto.
- **CA-1.4** El notebook deberá agrupar al inicio, fuera de los experimentos, una sección de soporte con los imports, la semilla (CA-2.5), las funciones de lógica y sus comprobaciones (RNF-5). Las celdas de soporte no son experimentos y no se rigen por CA-1.1.

#### RF-2 — Particiones y semilla (D1)

**Por qué.** Sin una división declarada de antemano y reproducible, no se puede garantizar que la prueba sea imparcial ni repetir los resultados.

- **CA-2.1** El notebook deberá declarar, antes de entrenar cualquier modelo del bloque, una partición en entrenamiento, validación y prueba en proporción 60/20/20. La partición de Iris se declara en E1 y la de Diabetes en E3. Cuando el tamaño no sea divisible, se acepta el redondeo de la librería.
- **CA-2.2** Donde el objetivo sea una clase (Iris), el notebook deberá conservar en cada partición la proporción de clases del conjunto completo.
- **CA-2.3** El notebook deberá usar la misma partición para todos los modelos de una misma comparación.
- **CA-2.4** Cuando declare una partición, el notebook deberá mostrar el tamaño de cada subconjunto y, en Iris, el conteo de ejemplos por clase en cada uno.
- **CA-2.5** El notebook deberá declarar al inicio una única semilla de la que dependa toda fuente de aleatoriedad.
- **CA-2.6** El notebook no deberá usar particiones internas del entrenamiento en ningún modelo (parada temprana, fracción de validación interna). La convergencia se trata con `max_iter`, que cuenta como configuración (CA-7.2).

#### RF-3 — Ajuste limitado, congelación y prueba única (D1, D8–D10)

**Por qué.** Si la prueba influye en alguna decisión, deja de medir desempeño sobre datos nuevos y el resultado final queda inflado.

- **CA-3.1** Mientras ajusta hiperparámetros, el notebook deberá decidir usando únicamente la partición de validación, con la métrica de D10.
- **CA-3.2** El notebook deberá probar como máximo cuatro configuraciones por modelo, contando la configuración inicial. Una configuración es una combinación distinta de hiperparámetros (incluidos `max_iter` y `solver`); la semilla no cuenta. La única excepción es el barrido de alpha de Ridge y Lasso (RF-21): fuera del barrido, los demás hiperparámetros de ambos siguen limitados a cuatro configuraciones.
- **CA-3.3** El notebook deberá registrar, por modelo, las configuraciones probadas, su resultado en validación (F1 macro y accuracy en clasificación; RMSE y MAE en regresión) y la configuración elegida.
- **CA-3.4** Antes de evaluar en prueba, el notebook deberá dejar registrada la configuración final de cada modelo y ajustarlo únicamente con la partición de entrenamiento, sin reentrenar con validación, y no modificarlos después.
- **CA-3.5** El notebook deberá evaluar cada modelo final sobre prueba una única vez.
- **CA-3.6** El notebook no deberá iniciar una nueva ronda de ajuste a partir de la tabla de prueba.
- **CA-3.7** Si dos configuraciones empatan en validación, entonces el notebook deberá desempatar con un criterio declarado antes de ver los resultados: primero por la métrica de apoyo (accuracy en clasificación, MAE en regresión) y, si persiste, gana la configuración probada primero (la inicial gana a las alternativas; en el barrido, la menor intensidad).
- **CA-3.8** La configuración inicial de cada modelo deberá elegirla y declararla el autor antes de entrenar ese modelo. Una configuración se declara escribiendo explícitamente sus hiperparámetros; un **hiperparámetro fijado explícitamente** es el que el autor escribe en la configuración, aunque coincida con el valor por defecto. La semilla no cuenta.

#### RF-4 — Preprocesamiento sin fuga de información

**Por qué.** Calcular medias, escalas o transformaciones con datos de validación o prueba filtra información del futuro hacia el entrenamiento.

- **CA-4.1** Donde un modelo requiera características estandarizadas o escaladas, el notebook deberá ajustar la transformación solo con entrenamiento y aplicarla después a validación y prueba. Donde el conjunto de datos ya venga transformado por su origen, esa transformación previa no es controlable y deberá declararse como limitación (CA-16.4).
- **CA-4.2** Donde se transforme el objetivo numérico, el notebook deberá ajustar la transformación solo con entrenamiento y revertirla antes de calcular métricas y de presentar gráficos y ejemplos de error, de modo que queden en la escala original del objetivo.
- **CA-4.3** Si no se transforma el objetivo, entonces el notebook deberá declararlo.

#### RF-5 — Resultados generados, no transcritos

**Por qué.** Una tabla escrita a mano puede dejar de corresponder al código sin que nadie lo note.

- **CA-5.1** El notebook deberá generar todas las tablas y figuras a partir de los resultados obtenidos en la ejecución.
- **CA-5.2** El notebook no deberá contener cifras de resultados escritas manualmente en tablas o gráficos.
- **CA-5.3** Las celdas de interpretación podrán citar cifras escritas a mano, siempre que indiquen de qué tabla o figura salen. Se reconoce que quedarían desfasadas si cambia el código; RNF-1 garantiza que no cambian al reejecutar.

#### RF-6 — Predicción preservada y contrastada (D5)

**Por qué.** Reescribir una predicción después de ver el resultado elimina justamente lo que se quería aprender.

- **CA-6.1** El notebook deberá registrar la predicción de cada experimento antes de su implementación.
- **CA-6.2** El notebook no deberá modificar el texto de una predicción después de haber ejecutado la implementación de su experimento.
- **CA-6.3** Cuando presente la interpretación, el notebook deberá indicar, para cada predicción falsable, si el resultado la confirmó o la contradijo y por qué, incluso cuando la confirmó. Las predicciones conceptuales (CA-9.2, CA-9.3) se contrastan en texto, indicando si lo observado las respalda, sin esa etiqueta.

#### RF-7 — Avisos y resultados degenerados (D4)

**Por qué.** Un modelo que no convergió o que anuló todos sus coeficientes es información sobre el método; ocultarla falsea la comparación.

- **CA-7.1** Si un modelo emite un aviso (cualquier aviso emitido por scikit-learn al ajustar o evaluar), entonces el notebook deberá dejarlo visible y comentarlo en la interpretación.
- **CA-7.2** Si se corrige un aviso modificando una configuración, entonces el notebook deberá contar esa corrección entre las configuraciones permitidas (CA-3.2) y decidirla solo con validación.
- **CA-7.3** Si un modelo produce un resultado degenerado (todos los coeficientes en cero, o un clasificador que nunca predice una clase), entonces el notebook deberá mostrarlo como resultado, rotularlo «degenerado» y explicarlo, sin excluirlo ni suprimir el aviso.
- **CA-7.4** Si un aviso persiste en la configuración elegida y ya se agotó el límite de CA-3.2, entonces el notebook deberá reportar el resultado tal cual, con el aviso visible y comentado.

---

### 4.2 Bloque 1 — Datos y protocolo experimental (E1)

#### RF-8 — Descripción del dataset Iris

**Por qué.** No se puede interpretar un resultado sin saber qué es una instancia, qué mide cada característica y cómo están distribuidas las clases.

- **CA-8.1** Cuando cargue Iris con la función de carga de scikit-learn, el notebook deberá indicar: qué representa cada instancia; número de instancias y de características; nombre (tal como lo entrega la librería) y significado en español de cada característica; clases y cantidad de ejemplos por clase; y dimensiones de la matriz de características y del vector de etiquetas.
- **CA-8.2** El notebook deberá presentar una tabla con cinco ejemplos elegidos al azar con la semilla única, garantizando que aparezcan las tres clases, que muestre sus características y su clase en forma legible.
- **CA-8.3** El notebook deberá presentar un gráfico de distribución de clases del conjunto completo, con título y ejes legibles.
- **CA-8.4** El notebook deberá registrar, antes de cargar los datos, una predicción sobre el tamaño del dataset y el balance entre clases. Es una decisión del autor: el enunciado no la pide, pero la constitución exige una predicción en cada experimento.
- **CA-8.5** La interpretación deberá indicar si las clases están balanceadas y qué implica eso para elegir métricas.
- **CA-8.6** El notebook deberá declarar en E1 la partición de Iris conforme a RF-2.

---

### 4.3 Bloque 2 — Comparación de clasificadores (E2)

#### RF-9 — Predicciones iniciales

**Por qué.** Obliga a razonar sobre escala, complejidad y azar antes de que los números sesguen la opinión.

- **CA-9.1** El notebook deberá registrar, antes de implementar, una predicción sobre qué modelos serán sensibles a la escala de las características.
- **CA-9.2** El notebook deberá registrar, antes de implementar, una predicción sobre qué modelo producirá fronteras de decisión más simples. Es una predicción conceptual: se contrasta sin medición directa y así se declara (CA-6.3).
- **CA-9.3** El notebook deberá registrar, antes de implementar, una explicación de por qué no puede saberse de antemano cuál modelo ganará.
- **CA-9.4** El notebook deberá declarar, en la celda de predicción y antes de implementar, el criterio objetivo con que responderá «más sencillo de programar» (CA-15.2).

#### RF-10 — Siete clasificadores bajo el mismo protocolo

**Por qué.** Una comparación solo es válida si todos los modelos reciben los mismos datos, la misma partición y el mismo trato de preprocesamiento.

- **CA-10.1** El notebook deberá comparar: Gaussian Naive Bayes, regresión logística multiclase, SVM lineal, árbol de decisión, Random Forest, Gradient Boosting y MLP de clasificación.
- **CA-10.2** El notebook deberá entrenar los siete modelos finales con características estandarizadas.
- **CA-10.3** El notebook deberá aplicar a los siete el mismo protocolo de RF-2 a RF-4, incluida la prohibición de particiones internas (CA-2.6).
- **CA-10.4** Como diagnóstico de la sensibilidad a la escala (CA-9.1), el notebook deberá entrenar cada modelo con su configuración inicial sin estandarizar y reportar su accuracy y su F1 macro solo en validación. El diagnóstico no cuenta como configuración, no cambia la configuración elegida y no usa prueba.

#### RF-11 — Comparación en validación

**Por qué.** La validación es el único lugar donde se permite decidir.

- **CA-11.1** Cuando entrene cada modelo con su configuración inicial, el notebook deberá reportar su accuracy y su F1 macro en validación.
- **CA-11.2** El notebook deberá presentar el registro de configuraciones probadas y la elegida para cada modelo (CA-3.3), decidida con F1 macro y con accuracy como apoyo (CA-3.7).

#### RF-12 — Evaluación final en prueba

**Por qué.** La tabla de prueba describe el desempeño esperado en datos nuevos. No se usa para seleccionar configuraciones ni para reabrir el ajuste; nombrar el mejor observado en ella es informar, no seleccionar (CA-15.1).

- **CA-12.1** Cuando todas las configuraciones estén congeladas, el notebook deberá presentar una tabla de prueba con una fila por modelo y las columnas: modelo, transformación, hiperparámetros principales (los fijados explícitamente en la configuración final, CA-3.8), accuracy de prueba y F1 macro de prueba.
- **CA-12.2** El notebook deberá evaluar los siete modelos finales sobre la misma partición de prueba.
- **CA-12.3** El notebook deberá presentar, aparte de la tabla de CA-12.1, una tabla con el F1 micro y el F1 macro de prueba por modelo, que respalde CA-15.4.

#### RF-13 — Matrices de confusión

**Por qué.** Las métricas agregadas esconden entre qué clases se confunde cada modelo.

- **CA-13.1** El notebook deberá presentar la matriz de confusión de prueba de Naive Bayes, de la SVM lineal y del MLP.
- **CA-13.2** El notebook deberá rotular filas y columnas con los nombres de las clases (tal como los entrega la librería) y mostrar conteos.

#### RF-14 — Análisis de dos errores de clasificación (D2)

**Por qué.** Mirar errores concretos enseña qué aporta y qué no aporta un ejemplo aislado como evidencia sobre una causa.

Un **error de clasificación** es una instancia de prueba mal clasificada por al menos un modelo final.

- **CA-14.1** Cuando existan al menos dos errores de clasificación en prueba (instancias distintas), el notebook deberá seleccionar dos de ellos para analizar.
- **CA-14.2** Si hay menos de dos errores en prueba, entonces el notebook deberá completar hasta dos con errores de validación de los modelos ya congelados (CA-3.4). Los modelos se ordenan por número de errores de validación, de mayor a menor (en empate, por el orden de CA-10.1), y aportan instancias distintas de las ya elegidas hasta completar dos. El notebook deberá rotular de qué partición y de qué modelo proviene cada error y declarar que, al ser la partición usada para decidir, esos errores pueden ser optimistas.
- **CA-14.3** Si no hay errores ni en prueba ni en validación, entonces el notebook deberá declararlo y explicar qué limita esa ausencia (por ejemplo, el tamaño de la muestra).
- **CA-14.4** Para cada error, el notebook deberá mostrar el modelo (indicando cuántos modelos más fallaron en esa instancia), la partición, los valores originales de las características, la clase real y la clase predicha.
- **CA-14.5** El notebook no deberá usar los errores analizados para modificar configuraciones.
- **CA-14.6** La interpretación deberá separar explícitamente lo que los datos observados permiten concluir de lo que sería una hipótesis no verificada sobre la causa del error.

#### RF-15 — Interpretación del Bloque 2 (D6, D10, D11)

**Por qué.** Cada pregunta evalúa un concepto distinto: qué es «mejor», qué miden las métricas y qué aporta la complejidad.

- **CA-15.1** *Mejor desempeño.* El mejor observado se define por F1 macro de prueba, con accuracy como apoyo; el notebook deberá señalar si accuracy discrepa. Cada modelo se compara contra el mejor observado: el notebook deberá listar los modelos que queden a ≤ 1 error de prueba de él como no distinguibles del mejor con esa muestra, en lugar de nombrar un ganador sin reservas. El modelo nombrado se rotula «mejor observado, no seleccionado» (CA-3.6).
- **CA-15.2** *Más sencillo de programar.* El notebook deberá aplicar el criterio declarado en la celda de predicción (CA-9.4): menor suma de hiperparámetros fijados explícitamente en la configuración final (CA-3.8) más configuraciones probadas (CA-3.3). La respuesta se justifica con lo observado en el propio notebook.
- **CA-15.3** *Accuracy vs F1 macro.* El notebook deberá comparar el orden de los modelos según ambas métricas. Si coinciden, deberá explicar lo observado sin presumir que el balance de clases lo garantice; si difieren, deberá indicar a qué clases se debe la diferencia.
- **CA-15.4** *Macro vs micro.* El notebook deberá explicar la diferencia entre ambos promedios y relacionarla con la tabla de F1 micro y macro (CA-12.3), incluido que en clasificación multiclase de una etiqueta el F1 micro coincide con accuracy.
- **CA-15.5** *MLP vs modelo lineal.* El notebook deberá explicar qué aporta el MLP frente a la regresión logística y la SVM lineal apoyándose en los resultados de este dataset, y no deberá afirmar una ventaja general si los resultados no la muestran.
- **CA-15.6** El notebook deberá contrastar las predicciones de RF-9 con los resultados (CA-6.3), usando el diagnóstico de CA-10.4 para CA-9.1.
- **CA-15.7** El umbral de «no concluyente» es un criterio práctico, no una prueba de significancia. La interpretación deberá señalar que, con 30 instancias y una partición, incluso diferencias mayores pueden no ser robustas, y no deberá afirmar «mejor» sin «en esta partición».

---

### 4.4 Bloque 3 — Comparación de regresión y regularización (E3)

#### RF-16 — Descripción del dataset Diabetes

**Por qué.** Interpretar un error o un coeficiente requiere saber qué mide el objetivo y en qué escala vienen las variables.

- **CA-16.1** Cuando cargue Diabetes con la función de carga de scikit-learn (variante por defecto), el notebook deberá describir qué representa el objetivo numérico y qué variables contiene el dataset.
- **CA-16.2** El notebook deberá comprobar en la descripción del dataset si las características ya incluyen transformaciones y reportar lo que encuentre.
- **CA-16.3** El notebook no deberá interpretar automáticamente los valores de las características como mediciones en sus unidades originales.
- **CA-16.4** El notebook deberá declarar como limitación de RF-4 que, en la variante por defecto, las características ya vienen centradas y escaladas con estadísticos del conjunto completo (una fuga de origen que no controla), y que por eso se muestran «tal como vienen».

#### RF-17 — Predicción inicial del Bloque 3

**Por qué.** Contrasta la intuición de que más flexibilidad siempre es mejor.

- **CA-17.1** El notebook deberá registrar, antes de implementar y en la misma celda de predicción de RF-21, una predicción justificada sobre si un modelo más flexible produce siempre menor error en datos nuevos, declarando qué modelos considera más flexibles.
- **CA-17.2** Al contrastarla, el notebook deberá reformularla en observable: si el modelo declarado más flexible tiene menor RMSE de prueba que el menos flexible, aclarando que un caso en un dataset y una partición no confirma el universal «siempre».

#### RF-18 — Ocho regresores bajo el mismo protocolo

- **CA-18.1** El notebook deberá comparar: regresión lineal, Ridge, Lasso, kNN de regresión, árbol de regresión, Random Forest de regresión, Gradient Boosting de regresión y MLP de regresión.
- **CA-18.2** Donde el modelo lo requiera, el notebook deberá aplicar escalamiento conforme a RF-4 mediante pipelines con estadísticos de entrenamiento, aunque las características ya vengan escaladas por el proveedor (CA-16.4).
- **CA-18.3** El notebook deberá aplicar a los ocho modelos el protocolo de RF-2 a RF-4, incluida la prohibición de particiones internas (CA-2.6), con una partición 60/20/20 sin estratificar por clase por ser el objetivo numérico, declarada en E3.

#### RF-19 — Evaluación final en prueba (regresión)

- **CA-19.1** Cuando todas las configuraciones estén congeladas, incluidas las intensidades de Ridge y Lasso fijadas por el barrido de RF-21 (que se ejecuta antes, dentro de E3), el notebook deberá presentar una tabla de prueba con una fila por modelo y las columnas: modelo, MAE, RMSE y R².
- **CA-19.2** El notebook deberá evaluar cada modelo sobre prueba una única vez.

#### RF-20 — Diagnóstico de regresión lineal y MLP

**Por qué.** Una métrica agregada no dice dónde ni cómo se equivoca el modelo.

- **CA-20.1** Para regresión lineal y MLP, el notebook deberá presentar un gráfico de valores reales frente a predicciones sobre la partición de prueba.
- **CA-20.2** Para regresión lineal y MLP, el notebook deberá presentar un gráfico de residuos.
- **CA-20.3** Para cada uno de esos dos modelos, el notebook deberá mostrar los dos ejemplos de prueba con mayor error absoluto, con su valor real, su predicción, su error (en escala original del objetivo) y sus características tal como vienen en el dataset.

#### RF-21 — Regularización en Ridge y Lasso (parte de E3; D3, D8)

**Por qué.** Permite ver con datos propios la relación entre penalización, tamaño de coeficientes y sobreajuste.

- **CA-21.1** El notebook deberá registrar, antes de implementar y en la celda de predicción de E3 (junto con RF-17), una predicción sobre cómo cambiarán los coeficientes y los errores al aumentar la intensidad de la penalización.
- **CA-21.2** El notebook deberá comparar al menos 8 intensidades de penalización distribuidas en escala logarítmica, en Ridge y en Lasso, con una rejilla propia por modelo declarada antes de ver resultados, usando validación. La rejilla de cada modelo deberá cubrir desde casi sin penalización hasta, en Lasso, todos los coeficientes en cero. El barrido se ejecuta dentro de la implementación de E3, antes de la evaluación de prueba.
- **CA-21.3** El notebook deberá reportar, por intensidad: la magnitud de los coeficientes (norma L2), la cantidad de coeficientes nulos en Lasso y el error (RMSE) de entrenamiento y de validación.
- **CA-21.4** Cuando determine la intensidad con mejor RMSE de validación, el notebook deberá congelarla, con el modelo ajustado solo con entrenamiento, como la configuración de Ridge y de Lasso para la evaluación de prueba de RF-19. Si el óptimo cae en el borde de la rejilla, deberá declararlo y explicarlo sin extender la rejilla. Si varias intensidades empatan exactamente, gana la menor (CA-3.7).
- **CA-21.5** El notebook no deberá usar la partición de prueba en el barrido.
- **CA-21.6** El notebook deberá presentar la evolución de los coeficientes y de los errores de entrenamiento y validación frente a la intensidad en figuras legibles.

#### RF-22 — Interpretación del Bloque 3 (D10, D11)

**Por qué.** Cierra el ciclo conectando métricas, regularización y límites de lo que un modelo predictivo permite afirmar.

- **CA-22.1** *Mejor desempeño.* El mejor observado se define por RMSE de prueba, con MAE de apoyo; el notebook deberá señalar si MAE cambia el orden, reportar R² y explicar que su orden coincide con el de RMSE en la misma partición. Cada modelo se compara contra el mejor observado: deberá listar los modelos cuyo RMSE de prueba quede a ≤ 5 % del RMSE del mejor como no distinguibles del mejor en esta partición. El modelo nombrado se rotula «mejor observado, no seleccionado» (CA-3.6).
- **CA-22.2** *MAE y RMSE.* El notebook deberá explicar por qué ambas pueden favorecer resultados distintos (cómo cada una trata los errores grandes) y relacionarlo con los resultados. Si el orden coincide, entonces deberá decirlo y explicarlo sin afirmar que difieren.
- **CA-22.3** *R² negativo.* El notebook deberá explicar que R² compara con la predicción trivial del promedio de los datos evaluados, que un R² negativo indica un modelo peor que esa predicción trivial y que predecir el promedio de entrenamiento da un R² ≤ 0 en prueba. Si ningún modelo lo obtuvo, entonces deberá indicarlo y explicar el concepto igualmente.
- **CA-22.4** *Efecto de la regularización.* El notebook deberá describir, con base en el barrido, el efecto de aumentar la penalización sobre la magnitud de coeficientes, los coeficientes nulos y la brecha entre error de entrenamiento y de validación.
- **CA-22.5** *Errores grandes.* El notebook deberá proponer posibles explicaciones para los errores grandes y marcar como hipótesis las que no pueda comprobar con los datos disponibles.
- **CA-22.6** *Causalidad.* El notebook deberá responder si un coeficiente predictivo demuestra una relación causal, distinguiendo asociación de causa, y deberá mencionar al menos un motivo concreto por el que un coeficiente puede no reflejar un efecto individual (por ejemplo, variables correlacionadas entre sí o efecto de la regularización).
- **CA-22.7** El notebook deberá contrastar las predicciones de RF-17 y RF-21 con los resultados (CA-6.3, CA-17.2).

---

## 5. Requisitos no funcionales

| ID | Requisito | Verificación |
|---|---|---|
| **RNF-1** Reproducibilidad | Al ejecutar el notebook completo desde cero (Restart & Run All) en el mismo entorno (las versiones de `requirements.txt`), deberá terminar sin errores y producir valores numéricos y tablas idénticos entre dos ejecuciones. Las figuras deberán ser equivalentes en los datos graficados, no píxel a píxel. | Reiniciar y ejecutar todo dos veces y comparar. Solo cuenta como válida la ejecución completa desde cero. |
| **RNF-2** Autocontención | La implementación vivirá únicamente en el notebook de la práctica (D7): sin módulos, paquetes ni scripts auxiliares. El notebook no leerá ni escribirá archivos; los datasets se cargarán con `load_iris` y `load_diabetes` de scikit-learn, sin descargas. | Inspección: ningún archivo adicional ni operación de entrada/salida de archivos. |
| **RNF-3** Dependencias | No se añadirá ninguna dependencia fuera del conjunto fijado en la constitución (principio 1), salvo con justificación escrita y aprobación. | Todo `import` pertenece a ese conjunto o a la biblioteca estándar. |
| **RNF-4** Idioma | Explicaciones, predicciones, interpretaciones, encabezados de tablas, títulos y ejes de figuras en español (como cadenas de texto); identificadores y comentarios técnicos de código en inglés; salvo términos técnicos (accuracy, F1 macro, nombres de modelos). Nombres de clases y características como los entrega la librería, con su significado en español. | Revisión del notebook. |
| **RNF-5** Verificabilidad | Cada función de lógica de cálculo tendrá al menos un `assert` dentro del notebook (sin pytest), y esa lógica no mezclará cálculo con presentación de resultados (constitución, principios 3 y 4). | Cada función de lógica tiene su `assert`; ninguna contiene `print`, `display` ni `plt.`. |
| **RNF-6** Trazabilidad | Cada requisito RF-8 a RF-22 se implementa en un experimento (E1–E3) y cada experimento responde a al menos un requisito. RF-1 a RF-7 son transversales y se verifican sobre todos los experimentos (constitución, principio 2). | Matriz requisito ↔ celda. |
| **RNF-7** Legibilidad didáctica | Cada término técnico no trivial se explica al introducirlo; cada figura tiene título y ejes con nombre; cada tabla tiene encabezados descriptivos. | Revisión por alguien sin contexto de Recuperación de Información. |
| **RNF-8** Tiempo de ejecución | La ejecución completa deberá terminar en menos de 5 minutos en un equipo personal. | Medir el tiempo de Restart & Run All. |
| **RNF-9** Explicabilidad | Cada decisión relevante (elección de configuración, criterio de desempate, elección de ejemplos analizados) deberá quedar justificada en el notebook. | Revisión de las celdas de interpretación. |

**Observación (fuera de esta spec).** `requirements.txt` no fija la versión de `notebook`; no afecta a los resultados numéricos y no se modifica aquí.

---

## 6. Casos límite

| # | Situación | Comportamiento esperado | Ref. |
|---|---|---|---|
| CL-1 | Prueba de Iris con menos de dos instancias mal clasificadas por los modelos finales (instancias distintas, contadas sobre la unión de los siete modelos). | Completar con errores de validación de los modelos congelados, elegidos según CA-14.2, rotulados por partición y modelo y marcados como posiblemente optimistas. | CA-14.2 |
| CL-2 | Sin errores en prueba ni en validación. | Declarar que no hay errores que analizar y explicar la limitación. | CA-14.3 |
| CL-3 | Uno o más clasificadores quedan a ≤ 1 error de prueba del mejor observado (o empatan con él). | Listarlos como no distinguibles del mejor; no declarar un ganador firme. | CA-15.1 |
| CL-4 | Accuracy y F1 macro dan el mismo orden. | Explicar lo observado sin presumir que el balance de clases lo garantice; no omitir la comparación. | CA-15.3 |
| CL-5 | Ninguna configuración alternativa mejora a la inicial en validación. | Congelar la inicial; no es un fallo. | CA-3.4 |
| CL-6 | Dos configuraciones empatan en validación. | Aplicar el criterio de desempate declarado de antemano. | CA-3.7 |
| CL-7 | MLP o SVM lineal emiten aviso de no convergencia. | Mostrar el aviso, comentarlo y, si se corrige, contarlo dentro del límite; si persiste con el límite agotado, reportarlo tal cual. | CA-7.1, CA-7.2, CA-7.4 |
| CL-8 | Lasso anula todos los coeficientes a intensidades altas. | Es esperado: la rejilla debe llegar a ese régimen; mostrarlo como resultado y explicarlo. | CA-21.2, CA-7.3 |
| CL-9 | MAE y RMSE ordenan los modelos igual. | Decirlo y explicar el concepto sin inventar una diferencia. | CA-22.2 |
| CL-10 | Ningún modelo tiene R² negativo. | Indicarlo y explicar el concepto igualmente. | CA-22.3 |
| CL-11 | Las características de Diabetes ya vienen transformadas por el proveedor (variante por defecto). | No interpretarlas como unidades originales, ni en los ejemplos de error grande; declarar la fuga de origen. | CA-16.3, CA-16.4, CA-20.3 |
| CL-12 | Se transforma (o no) el objetivo del MLP. | Ajustar solo con entrenamiento y revertir antes de métricas, gráficos y ejemplos; si no se transforma, declararlo. | CA-4.2, CA-4.3 |
| CL-13 | El resultado contradice la predicción inicial. | Conservar la predicción y explicar por qué falló. | CA-6.2, CA-6.3 |
| CL-14 | Se ejecutan celdas fuera de orden o se reejecuta parcialmente. | Solo cuenta como válida la ejecución completa desde cero (Restart & Run All); es una regla de validez, no un comportamiento que el notebook garantice por sí mismo. | RNF-1 |
| CL-15 | La mejor intensidad de Ridge o Lasso cae en el borde de la rejilla. | Declararlo y explicarlo, sin extender la rejilla ni reabrir el ajuste. | CA-21.4 |
| CL-16 | Varias intensidades de Lasso empatan exactamente (p. ej. todas con coeficientes en cero). | Gana la menor; si es el óptimo, el Lasso congelado es constante y se muestra como resultado. | CA-21.4, CA-7.3, CA-3.7 |
| CL-17 | Uno o más regresores quedan a ≤ 5 % del RMSE de prueba del mejor observado. | Listarlos como no distinguibles del mejor en esta partición; no declarar un ganador firme. | CA-22.1 |
| CL-18 | Aviso persistente con el límite agotado, o clasificador que nunca predice una clase. | Reportar tal cual, rotular «degenerado» y comentar; no suprimir el aviso. | CA-7.3, CA-7.4 |

---

## 7. Fuera de alcance

- Validación cruzada y repeticiones con distintas semillas.
- Búsqueda automática de hiperparámetros.
- Intervalos de confianza y pruebas de significancia estadística. El umbral de «no concluyente» de D11 es un criterio práctico, no una prueba de significancia (CA-15.7).
- Otros datasets distintos de Iris y Diabetes.
- Ingeniería o selección de características.
- Guardar modelos, tablas o figuras fuera del notebook, o cargar datos externos.
- Módulos, paquetes, scripts o dependencias adicionales.
- Interpretación avanzada de modelos (importancia por permutación, valores SHAP, etc.).
- Clustering, recomendación y asociación, y la interpretación de softmax: aparecen entre los objetivos de la introducción del notebook pero no en los bloques 1 a 3 (ver DA-6).
- Cualquier tarea de Recuperación de Información.

---

## 8. Criterios de finalización

La funcionalidad se considera terminada cuando se cumplen **todos** estos puntos:

- [ ] Reiniciar y ejecutar todo desde cero termina sin errores, repite resultados idénticos y tarda menos de 5 minutos (RNF-1, RNF-8).
- [ ] Existen los tres experimentos (E1–E3), cada uno con sus tres celdas en orden (RF-1), y la sección de soporte inicial (CA-1.4).
- [ ] Cada requisito RF-8 a RF-22 tiene su implementación, RF-1 a RF-7 se cumplen en todos los experimentos y cada experimento responde a un requisito (RNF-6).
- [ ] Particiones y semilla declaradas antes de entrenar; configuraciones registradas y congeladas antes de evaluar en prueba; prueba evaluada una sola vez por modelo (RF-2, RF-3).
- [ ] Entregables presentes y generados a partir de resultados:
  - Bloque 1: partición de Iris con conteo por clase, tabla de cinco ejemplos y gráfico de distribución de clases.
  - Bloque 2: tabla de resultados de prueba, tabla aparte de F1 micro y macro, matrices de confusión de Naive Bayes, SVM lineal y MLP, y diagnóstico sin estandarizar en validación.
  - Bloque 3: tabla MAE/RMSE/R²; real vs predicción y residuos de regresión lineal y MLP; dos ejemplos de error grande por modelo; barrido de regularización.
- [ ] Todas las preguntas de interpretación están respondidas con base en los resultados obtenidos y cada predicción está contrastada según CA-6.3 (RF-6, RF-15, RF-22).
- [ ] El análisis de dos errores de clasificación distingue lo que se puede y no se puede concluir (RF-14).
- [ ] Cada función de lógica tiene al menos un `assert` (sin pytest) y ninguna contiene `print`, `display` ni `plt.` (RNF-5).
- [ ] No hay archivos de código adicionales, dependencias nuevas ni operaciones sobre archivos: ningún `open`, `read_csv`, `to_csv`, `savefig` ni `joblib.dump` (RNF-2, RNF-3).
- [ ] El idioma cumple RNF-4, la legibilidad RNF-7 y las decisiones relevantes están justificadas (RNF-9).
- [x] No quedan dudas abiertas: DA-6 se aceptó como fuera de alcance (sección 9).

---

## 9. Dudas

| ID | Duda | Estado |
|---|---|---|
| DA-1 | ¿Las 4 configuraciones incluyen la inicial? | **Resuelta:** sí, la inicial cuenta (D8, CA-3.2). |
| DA-2 | ¿El barrido de ≥ 8 intensidades es excepción al máximo de 4? | **Resuelta:** sí, solo para alpha de Ridge y Lasso (D8, CA-3.2, CA-21.2). |
| DA-3 | Orden entre la tabla de prueba y el barrido de regularización. | **Resuelta:** el barrido es parte de E3 y se ejecuta antes de la evaluación de prueba (D12, CA-19.1, CA-21.2). |
| DA-4 | Predicciones iniciales para E1 y E4. | **Resuelta:** E4 desaparece al fusionarse en E3 (D12); la predicción de E1 se mantiene como decisión del autor (CA-8.4). |
| DA-5 | Procedencia y criterio de los dos errores de clasificación. | **Resuelta:** dos instancias distintas, rotuladas con el modelo y cuántos más fallaron (RF-14). |
| DA-6 | Los objetivos de la introducción del notebook (distinguir clustering, recomendación y asociación; interpretar softmax y las salidas del MLP) no tienen bloque asociado. ¿Pertenecen a esta funcionalidad? | **Resuelta:** no; se aceptó como fuera de alcance (sección 7). |
| DA-7 | Métrica que decide la elección de configuración. | **Resuelta:** F1 macro y RMSE, con accuracy y MAE de apoyo (D10). |
| DA-8 | Tiempo máximo de ejecución aceptable. | **Resuelta:** menos de 5 minutos (RNF-8). |
| DA-9 | Criterio de «errores absolutos grandes» en regresión. | **Resuelta:** los dos mayores de cada modelo sobre prueba (CA-20.3). |

---

## Anexo A — Decisiones de la revisión QA

La revisión QA de esta spec contra la constitución, el enunciado y `requirements.txt` produjo 38 hallazgos, resueltos en ocho rondas, y una segunda pasada sobre la spec editada produjo 3 más (ítems 39–41). Los números de ítem son los del informe QA. La columna «Requisitos» indica dónde se aplicó cada decisión.

| Ítems | Hallazgo | Decisión | Requisitos |
|---|---|---|---|
| 1–2 | «Configuración» e «inicial» sin definir | La inicial la elige y declara el autor antes de entrenar; cada combinación distinta de hiperparámetros cuenta como 1; la semilla no; la inicial cuenta dentro de las 4. | D8, CA-3.2, CA-3.8, CA-11.1 |
| 3 | Qué es «congelar» | Configuración y modelo ajustado solo con entrenamiento; sin reentrenar con validación. | D9, CA-3.4, CA-14.2, CA-21.4 |
| 4 | Métrica de decisión y «mejor» | F1 macro y RMSE; accuracy y MAE de apoyo y primer desempate; luego, la configuración probada primero. | D10, CA-3.1, CA-3.7, CA-11.2, CA-15.1, CA-22.1 |
| 5 | Tamaños de partición | Redondeo de la librería; tamaños y conteo de clases por partición en Iris. | CA-2.1, CA-2.4 |
| 6 | Barrido de regularización | Rejilla por modelo, ≥ 8 puntos log-espaciados, con cobertura hasta Lasso todo cero; magnitud = norma L2; error = RMSE. | CA-21.2, CA-21.3, CL-8 |
| 7 | «Un solo ejemplo» | ≤ 1 error de prueba ⇒ no concluyente, sea cual sea la métrica. | D11, CA-15.1, CL-3 |
| 8 | «Más sencillo de programar» | Hiperparámetros fijados + configuraciones probadas, declarado en la celda de predicción. | D6, CA-9.4, CA-15.2 |
| 9–10 | Análisis de errores | Error = instancia mal clasificada por ≥ 1 modelo; dos instancias distintas; características originales; regresión: los dos mayores \|error\| por modelo, en escala original. | RF-14, CA-14.1, CA-14.4, CA-20.3, CL-1 |
| 11 | Términos sin definir | Aviso relevante, modelo lineal y degenerado definidos operativamente; hiperparámetros principales, ver ítem 39. | CA-7.1, CA-7.3, CA-15.5 |
| 12 | Reproducibilidad y tiempo | Resultados idénticos con `requirements.txt`; figuras equivalentes en los datos; < 5 min; CL-14 como regla de validez. | RNF-1, RNF-8, CL-14 |
| 13–14 | E3/E4 circulares; CA-1.3 vs RF-22 | E4 se fusiona en E3: tres experimentos, barrido dentro de E3, predicciones RF-17 y RF-21 en la misma celda. | D12, CA-1.1, RF-21, RF-22, CA-19.1 |
| 15 | Predicciones sin medición | Diagnóstico sin estandarizar en validación para CA-9.1; CA-9.2 queda conceptual. | CA-10.4, CA-9.2, CA-15.6 |
| 16 | D1 vs CA-21.2 | Excepción solo para alpha; los demás hiperparámetros de Ridge y Lasso siguen limitados a 4. | D8, CA-3.2, CA-21.2 |
| 17 | RF-12 vs CA-15.1/CA-22.1 | Distinguir seleccionar de informar: «mejor observado, no seleccionado». | RF-12, CA-15.1, CA-22.1 |
| 18 | Celdas de soporte | Sección de soporte al inicio, fuera de la regla de tres celdas. | CA-1.2, CA-1.4 |
| 19 | F1 micro | Tabla aparte con F1 micro y F1 macro por modelo. | CA-12.3, CA-15.4 |
| 20 | CA-6.3 vs predicciones no falsables | CA-6.3 solo para falsables; CA-9.2 y CA-9.3 por contraste en texto; CA-17.1 reformulada en observable. | CA-6.3, CA-17.1, CA-17.2 |
| 21–22 | Orden RMSE ≡ R²; baseline de R² | «Mejor» por RMSE; R² contra el promedio de los datos evaluados. | CA-22.1, CA-22.3 |
| 23 | Cifras en Markdown | Cifras a mano permitidas en la interpretación con referencia a su tabla o figura. | CA-5.3 |
| 24 | §7 vs CA-15.1 | Umbral rotulado como criterio práctico, no prueba de significancia. | CA-15.7, §7 |
| 25 | Diabetes ya escalado | Variante por defecto, escalado en pipeline con estadísticos de entrenamiento, fuga de origen declarada. | CA-16.4, CA-18.2, CA-4.1, CL-11 |
| 26 + 28 | Borde, empates y no concluyente en regresión | Borde declarado sin extender; empate exacto gana la menor intensidad; ≤ 5 % del RMSE del mejor ⇒ no distinguible del mejor (ver ítem 40). | D11, CA-21.4, CA-22.1, CL-15, CL-16, CL-17 |
| 27 | Particiones internas | Prohibidas; convergencia con `max_iter`. | CA-2.6, CA-10.3, CA-18.3 |
| 29 | Aviso persistente / degenerado | Reportar tal cual, rotular «degenerado», sin suprimir el aviso. | CA-7.3, CA-7.4, CL-18 |
| 30 + 32 | Sesgo de validación; premisa de CL-4 | Rotular sesgo optimista; no presumir que el balance garantice el mismo orden. | CA-14.2, CA-15.3, CL-4 |
| 31 | Cinco ejemplos de Iris | Al azar con la semilla única, con las tres clases; gráfico del conjunto completo. | CA-8.2, CA-8.3 |
| 33 | Trazabilidad vs principio 2 | Matriz requisito → experimento; RF-1 a RF-7 transversales; RF-2 en E1 y E3; RF-7 en HU-7. | RNF-6, CA-1.2, CA-8.6, HU-7 |
| 34 | §8 vs principios 3 y 4 | Criterios explícitos de `assert`, ausencia de `print`/`display`/`plt.` y de operaciones de archivo. | §8, RNF-5 |
| 35 | Principio 1 vs RNF-3 | Alinear con la constitución; `notebook` sin versión anotado como observación. | RNF-3, RNF-1 |
| 36 | Principio 5 vs RNF-2 | Datasets con `load_iris` y `load_diabetes` de scikit-learn, sin descargas. | RNF-2, CA-8.1, CA-16.1 |
| 37 | Principio 6 vs RNF-4 | «Salvo términos técnicos»; rótulos en español como cadenas; nombres de la librería con significado en español. | RNF-4, CA-8.1, CA-13.2 |
| 38 | Procedencia | D1–D7 y CA-8.4 marcados como decisiones del autor; este anexo registra el resto. | §1, Anexo A |
| 39 | «Principales» (CA-12.1) vs «fijados explícitamente» (CA-15.2) | Un solo término: hiperparámetro fijado explícitamente = el que el autor escribe en la configuración, aunque coincida con el valor por defecto. La tabla de prueba y el criterio de sencillez miden lo mismo. | CA-3.8, CA-12.1, CA-15.2 |
| 40 | Umbral ≤ 5 %: denominador y transitividad | El porcentaje se calcula sobre el RMSE del mejor y cada modelo se compara contra el mejor, no entre pares; lo mismo para ≤ 1 error en clasificación. | D11, CA-15.1, CA-22.1, CL-3, CL-17 |
| 41 | Origen de los errores de validación (CA-14.2) | Los modelos se ordenan por número de errores de validación (en empate, orden de CA-10.1) y aportan instancias distintas hasta completar dos; se rotula modelo y partición. | D2, CA-14.2, CL-1 |
| DA-6 | Duda abierta restante | Aceptada como fuera de alcance. | §7, §9 |
