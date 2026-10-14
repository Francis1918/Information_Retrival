# Spec 002 — Conceptos y salidas del MLP en `01_revml.ipynb`

| | |
|---|---|
| **Estado** | Propuesta; pendiente de aprobación |
| **Fecha** | 2026-10-13 |
| **Prioridad** | Alta (RF-01 a RF-04) |
| **Alcance de este documento** | El QUÉ y el POR QUÉ. Es autocontenido: todo lo necesario para implementar y verificar los requisitos está en este archivo. No remite a otros documentos. |

En todo el documento, **«el notebook»** es `01_revml.ipynb`, el sistema al que se aplican los requisitos.

---

## 1. Contexto y objetivo

### Contexto

El notebook ya compara siete clasificadores sobre Iris (experimento E2) y ocho regresores sobre Diabetes (experimento E3). Su introducción enuncia como objetivos distinguir clustering, recomendación y asociación, e interpretar softmax y las salidas del MLP, pero ningún apartado los desarrolla. Esta spec los añade sin modificar nada de lo que ya existe.

### Objetivo

Que quien lea el notebook pueda: (1) distinguir los cinco tipos de problemas con un ejemplo de cada uno; (2) explicar qué es softmax y comprobar que produce las probabilidades del MLP clasificador; (3) leer las probabilidades que el MLP entrega sobre Iris; (4) explicar por qué el MLP regresor entrega un número continuo y qué significa en Diabetes.

### Por qué importa

Los riesgos reales son: tomar una probabilidad por certeza, leer la salida de un regresor como si fuera una clase o una probabilidad, mezclar la escala interna estandarizada con la escala original, y recortar o ignorar predicciones fuera del rango observado. La spec existe para que esos fallos sean detectables.

### Términos

| Término | Significado en esta spec |
|---|---|
| **Puntuación** | Cada número real que la capa de salida produce antes de softmax (también llamado logit). |
| **Probabilidad máxima** | La mayor de las probabilidades que el MLP clasificador asigna a una instancia. |
| **Clase predicha** | La clase que devuelve `predict()` del estimador. |
| **Salida interna** | Valor que produce la red del MLP regresor en la escala estandarizada del objetivo. |
| **Rango observado** | Intervalo `[mínimo, máximo]` del objetivo de Diabetes en la partición de entrenamiento. |
| **Salida entregada** | Lo que el sistema le devuelve a quien lo usa (una etiqueta, un número, grupos, una lista ordenada o reglas). |
| **Índice original** | Posición de la instancia en el conjunto completo (0 a 149 en Iris, 0 a 441 en Diabetes), la misma que usan las tablas existentes del notebook. |
| **Cifra de resultado** | Todo número que procede de ejecutar el código del notebook (métricas, probabilidades, conteos, rangos, índices). Los números de definición o de ejemplo conceptual (la fórmula, «0.9», «cinco tipos») no lo son. |
| **Predicción falsable** | Afirmación que fija de antemano un valor, un umbral, un conteo o una comparación (mayor, menor, igual) entre magnitudes de una tabla nombrada, y que el resultado puede contradecir. No cuentan las propiedades que garantiza softmax (suma 1, valores en [0, 1], clase predicha = clase de mayor probabilidad). |

---

## 2. Estado de partida

Hechos del notebook que esta spec **no modifica** y sobre los que se apoya. Un revisor puede comprobarlos abriendo el notebook.

| ID | Hecho |
|---|---|
| EP-1 | El notebook tiene 20 celdas: una de título, siete de soporte (S0 a S6), tres de enunciado (B1 a B3) y tres experimentos E1 a E3, cada uno con tres celdas consecutivas: predicción (Markdown), implementación (código) e interpretación (Markdown). |
| EP-2 | Hay una única semilla, definida en S1. Iris se parte 90/30/30 (entrenamiento/validación/prueba, estratificada) y Diabetes 264/89/89 (sin estratificar). |
| EP-3 | E2 deja congelado un MLP clasificador (`Pipeline` con `StandardScaler` y `MLPClassifier`) y E3 deja congelado un MLP regresor (`TransformedTargetRegressor` con `StandardScaler` sobre el objetivo, que envuelve un `Pipeline` con `StandardScaler` y `MLPRegressor`). Ambos están ajustados solo con entrenamiento. |
| EP-4 | La partición de prueba ya se evaluó una vez por modelo; esos resultados no se recalculan. |
| EP-5 | Los `import` están solo en S1 e incluyen `numpy as np`, `pandas as pd`, `matplotlib.pyplot as plt`, `Pipeline`, `StandardScaler` y `TransformedTargetRegressor`. |
| EP-6 | Las huellas de reproducibilidad que el notebook imprime al final de E1, E2 y E3 son `689602a2376f3020`, `db54a3c0be16c4a4` y `762c72766081749c`. |

---

## 3. Restricciones

| ID | Restricción |
|---|---|
| R-1 | **Alcance cerrado.** Toda la implementación vive en `01_revml.ipynb`. No se crean módulos `.py`, paquetes, scripts auxiliares ni archivos de datos externos. |
| R-2 | **Stack fijo, versiones exactas e inmodificables:** `numpy==2.5.3`, `pandas==3.0.6`, `scikit-learn==1.9.1`, `scipy==1.18.1`, `matplotlib==3.11.2`, `seaborn==0.13.2`, `joblib==1.6.0`, `notebook` e `ipykernel==7.4.0`. Ninguna otra dependencia entra sin justificación escrita y aprobación. |
| R-3 | **Sin imports nuevos.** El código nuevo usa solo los `import` que ya existen en S1 (EP-5) y la biblioteca estándar. |
| R-4 | **Datasets.** Iris para clasificación y Diabetes para regresión, cargados con `load_iris` y `load_diabetes` de scikit-learn, sin descargas. |
| R-5 | **Sin operaciones de archivo.** No aparece `open`, `read_csv`, `to_csv`, `savefig` ni `joblib.dump`. |
| R-6 | **Idioma.** Código, nombres de variables y comentarios técnicos en inglés. Celdas Markdown, explicaciones, rótulos de tablas, títulos y ejes de figuras en español (como cadenas), salvo términos técnicos (softmax, logit, MLP, accuracy, nombres de modelos). Los nombres de clases y de variables van como los entrega la librería. |
| R-7 | **No regresión.** Las 20 celdas existentes conservan su código y su texto sin cambios. |

---

## 4. Estructura de celdas nuevas

Se añaden 10 celdas. El notebook pasa de 20 a 30.

| ID | Tipo | Ubicación | Contenido | Cubre |
|---|---|---|---|---|
| **S7** | Código (soporte) | Justo después de S6 y antes de B1 | Dos funciones y sus `assert`: softmax estable y puntuaciones de salida del MLP clasificador | RF-02 |
| **B4** | Markdown | Justo después de E3.I | Enunciado del Bloque 4: cuatro instrucciones (ES-7) | — |
| **K1** | Markdown | Después de B4 | Tabla de los cinco tipos de problemas (TK.1) | RF-01 |
| **K2** | Markdown | Después de K1 | Softmax: definición, funcionamiento, fórmula e interpretación | RF-02 |
| **E4.P** | Markdown | Después de K2 | Predicciones del MLP clasificador | RF-03, RF-02 |
| **E4.C** | Código | Después de E4.P | Softmax, comparación con `predict_proba()` y probabilidades sobre Iris (T4.1 a T4.5) | RF-03, RF-02 |
| **E4.I** | Markdown | Después de E4.C | Interpretación | RF-03, RF-02 |
| **E5.P** | Markdown | Después de E4.I | Predicciones del MLP regresor | RF-04 |
| **E5.C** | Código | Después de E5.P | Salida del MLP regresor sobre Diabetes (T5.1 a T5.3) | RF-04 |
| **E5.I** | Markdown | Después de E5.C | Interpretación | RF-04 |

**Reglas de estructura**

- **ES-1.** K1 y K2 son celdas conceptuales: Markdown, sin código, sin predicciones y sin cifras de resultado (ver Términos). No son experimentos.
- **ES-2.** E4 y E5 son experimentos: tres celdas consecutivas, en este orden (predicción, implementación, interpretación), con una sola celda de código que no define funciones.
- **ES-3.** La primera línea de cada celda nueva contiene su ID (por ejemplo, `## K1 — …` en Markdown o `# [E4.C] …` en código).
- **ES-4.** El texto de E4.P y E5.P se escribe antes de ejecutar E4.C y E5.C y no se modifica después.
- **ES-5.** Cada interpretación cita, para cada cifra, la tabla de origen (T4.x, T5.x) y marca cada predicción falsable como confirmada o contradicha, indicando por qué, incluso cuando se confirmó.
- **ES-6.** La lógica nueva (funciones) vive solo en S7. Ninguna función contiene `print`, `display` ni `plt.`.
- **ES-7.** B4 es una celda Markdown con el título «4. Conceptos y salidas del MLP» y exactamente cuatro instrucciones numeradas, una por RF-01 a RF-04, redactadas como las de B1 a B3 y sin resultados ni predicciones: (1) comparar los 5 tipos de problemas con una tabla que explique clasificación, regresión, clustering, recomendación y asociación, con un ejemplo de cada uno; (2) explicar la función softmax: definición, funcionamiento, fórmula e interpretación de probabilidades; (3) mostrar con `predict_proba()` las probabilidades estimadas para las tres clases de Iris; (4) explicar por qué el MLP regresor produce una salida numérica continua y cómo se interpreta en Diabetes. Esta spec añade al notebook únicamente estos cuatro contenidos; todas las celdas nuevas de la sección 4 existen para desarrollarlos. Verificación: `R`.

---

## 5. Requisitos funcionales

Los criterios de aceptación usan notación EARS en español: **Ubicuo** («El notebook deberá…»), **Evento** («Cuando…, el notebook deberá…»), **Estado** («Mientras…, el notebook deberá…»), **Opcional** («Donde…, el notebook deberá…») y **No deseado** («Si…, entonces el notebook deberá…»). Cada criterio indica la celda que lo cubre y cómo se comprueba: **A** = `assert` dentro del notebook; **B** = búsqueda textual en el `.ipynb`; **R** = inspección con lista de verificación (presencia de elementos, sin juicio de calidad).

### RF-01 — Comparación de los cinco tipos de problemas (Alta)

**Por qué.** Distinguir qué entrega cada tipo de problema evita aplicar un modelo a una pregunta que no es la suya.

- **CA-01.1** *(Ubicuo)* El notebook deberá presentar en K1 una tabla (TK.1) con exactamente cinco filas, en este orden: clasificación, regresión, clustering, recomendación y asociación. `[K1 · R]`
- **CA-01.2** *(Ubicuo)* El notebook deberá dotar a TK.1 de exactamente estas columnas: `Tipo de problema` · `Qué hace` · `Datos de entrada` · `Salida entregada` · `Ejemplo`. `[K1 · R]`
- **CA-01.3** *(Ubicuo)* El notebook deberá incluir texto no vacío en cada celda de TK.1, con las cinco celdas de la columna `Ejemplo` distintas entre sí. `[K1 · R]`
- **CA-01.4** *(Ubicuo)* El notebook deberá usar en la columna `Ejemplo` de TK.1 estos cinco ejemplos, en este orden, y rotular «no implementado en este notebook» los tres últimos: clasificación, Iris (predecir la especie de una flor); regresión, Diabetes (predecir la progresión de la enfermedad); clustering, agrupar clientes por hábitos de compra sin etiquetas previas; recomendación, sugerir a un usuario una lista ordenada de películas a partir de las valoraciones de usuarios similares; asociación, descubrir en cestas de compra reglas como «quien compra pan suele comprar mantequilla». `[K1 · R]`
- **CA-01.5** *(Ubicuo)* El notebook deberá mostrar en K1, junto a la tabla, la regla con la que se asigna un problema a una fila, según la **salida entregada** `[K1 · R]`:
  1. una etiqueta de un conjunto finito de clases conocidas → clasificación;
  2. un número continuo → regresión;
  3. grupos de instancias descubiertos sin etiquetas → clustering;
  4. una lista ordenada de ítems sugeridos a un usuario → recomendación;
  5. reglas «si aparecen A, suele aparecer B» entre ítems de transacciones → asociación.
- **CA-01.6** *(No deseado)* Si un ejemplo admite formularse como más de un tipo (por ejemplo, una recomendación vista como clasificación «le gustará / no le gustará»), entonces el notebook deberá asignarlo a la única fila que fija la regla de CA-01.5, mencionar la formulación alternativa en una nota bajo la tabla y no duplicar filas. `[K1 · R]`
- **CA-01.7** *(Ubicuo)* El notebook no deberá incluir código ni cifras de resultado (ver Términos) en K1. `[K1 · B]`
- **CA-01.8** *(Ubicuo)* El notebook deberá usar en la columna `Salida entregada` de TK.1 el término de la regla de CA-01.5 que corresponde a cada fila (*etiqueta de clase*, *número continuo*, *grupos*, *lista ordenada*, *reglas*), redactar cada celda de `Ejemplo` como una frase que nombra la entrada y la salida, y dar al menos 5 palabras a cada celda de `Qué hace` y de `Datos de entrada`. `[K1 · R]`

### RF-02 — Función softmax (Alta)

**Por qué.** Softmax es el puente entre las puntuaciones de la red y las probabilidades que el MLP clasificador reporta; sin entenderlo, `predict_proba()` es una caja negra.

- **CA-02.1** *(Ubicuo)* El notebook deberá incluir en K2 la **definición**: softmax transforma un vector de K números reales (puntuaciones) en un vector de K probabilidades, cada una en el intervalo (0, 1) y con suma 1. `[K2 · R]`
- **CA-02.2** *(Ubicuo)* El notebook deberá incluir en K2 la **fórmula** `σ(z)ᵢ = e^{zᵢ} / Σⱼ e^{zⱼ}` en notación matemática, con `z`, `K`, `i` y `j` definidos. `[K2 · R]`
- **CA-02.3** *(Ubicuo)* El notebook deberá explicar en K2 el **funcionamiento** en dos pasos numerados (exponencial y normalización) y declarar tres propiedades: conserva el orden de las puntuaciones, las salidas suman 1 y el resultado no cambia si se suma la misma constante a todas las puntuaciones. `[K2 · R]`
- **CA-02.4** *(Ubicuo)* El notebook deberá incluir en K2 la **interpretación**: la salida `pᵢ` es la probabilidad que el modelo estima de que la instancia pertenezca a la clase `i`, entre clases mutuamente excluyentes; no es certeza y no garantiza estar calibrada (una probabilidad de 0.9 no implica acertar el 90 % de las veces). `[K2 · R]`
- **CA-02.5** *(Ubicuo)* El notebook deberá definir en S7 una función propia de softmax numéricamente estable (resta el máximo de las puntuaciones antes de la exponencial) con `assert` que verifiquen: `softmax([0, 0, 0])` = `[1/3, 1/3, 1/3]`; suma 1; conserva el orden; invariancia al sumar una constante; y que `softmax([1000, 1001, 1002])` y `softmax([-1000, -1001, -1002])` son finitos y coinciden con `softmax([0, 1, 2])` y `softmax([0, -1, -2])`. `[S7 · A]`
- **CA-02.6** *(Evento)* Cuando se ejecute E4.C, el notebook deberá mostrar en T4.1 el paso a paso para las puntuaciones `[2.0, 1.0, 0.1]`: una fila por puntuación con las columnas `Puntuación` · `Exponencial` · `Probabilidad`, y la suma de la última columna, calculada con valores sin redondear. `[E4.C · A, R]`
- **CA-02.7** *(Evento)* Cuando se ejecute E4.C, el notebook deberá mostrar en T4.2, para las puntuaciones `[1000, 1001, 1002]`, el resultado de la fórmula sin estabilizar y el de la función de S7, junto con cualquier aviso emitido por la primera, que deberá quedar visible y no suprimido. `[E4.C · A, R]`
- **CA-02.8** *(Evento)* Cuando se ejecute E4.C, el notebook deberá obtener las puntuaciones de la capa de salida del MLP clasificador congelado en E2 sobre las 30 instancias de validación con la función de puntuaciones de S7 (CA-02.10), aplicarles la función de softmax de S7 y mostrar en T4.3: la activación de salida declarada por el estimador, la diferencia máxima absoluta con `predict_proba()` y el resultado de la comprobación `diferencia ≤ 1e-9`. `[E4.C · A]`
- **CA-02.9** *(No deseado)* Si la entrada de la función de S7 contiene valores no finitos (`nan` o `±inf`), entonces el notebook deberá hacer que la función lance `ValueError` en lugar de devolver valores no finitos. `[S7 · A]`
- **CA-02.10** *(Ubicuo)* El notebook deberá definir en S7 una función que calcule las puntuaciones de la capa de salida de un `Pipeline` con `StandardScaler` y `MLPClassifier` ya ajustado: aplica el escalador del `Pipeline` y luego `coefs_`, `intercepts_` y `activation` del MLP, sin acceder a atributos ni métodos con prefijo `_`; soporta las cuatro activaciones que admite `MLPClassifier` (`relu`, `tanh`, `logistic` e `identity`); y lleva `assert` que, para cada una de las cuatro, verifiquen sobre datos sintéticos que el softmax de sus puntuaciones coincide con `predict_proba()` con diferencia ≤ `1e-9`. `[S7 · A, B]`

### RF-03 — Salidas del MLP clasificador (Alta)

**Por qué.** Mostrar las tres probabilidades de cada instancia enseña que el MLP no «decide» una clase: reparte una probabilidad entre todas y `predict()` elige la mayor.

- **CA-03.1** *(Evento)* Cuando se ejecute E4.C, el notebook deberá obtener con `predict_proba()` del MLP clasificador congelado en E2 las probabilidades de las tres clases de Iris para las 30 instancias de la partición de validación, sin reentrenar ni reconfigurar el modelo. `[E4.C · A, B]`
- **CA-03.2** *(Ubicuo)* El notebook deberá presentar en T4.4 30 filas, ordenadas por índice original ascendente, con exactamente estas columnas: `Índice original` · `P(setosa)` · `P(versicolor)` · `P(virginica)` · `Suma` · `Clase predicha` · `Probabilidad máxima` · `Clase real` · `¿Acierta?`. Las columnas `P(·)` deberán rotularse con `target_names[c]` para cada `c` de `classes_`, en ese orden, y `Suma` deberá calcularse con valores sin redondear. `[E4.C · A]`
- **CA-03.3** *(Ubicuo)* El notebook deberá verificar con `assert`, sobre valores sin redondear y para las 30 filas: forma `(30, 3)`; `classes_` igual a `[0, 1, …, K−1]` con K igual a la cantidad de nombres de clase; cada probabilidad en `[0, 1]`; `|suma − 1| ≤ 1e-9`; y que la clase de `predict()` sea la de mayor probabilidad. `[E4.C · A]`
- **CA-03.4** *(Ubicuo)* El notebook deberá mostrar en T4.4 las probabilidades con cuatro decimales sin renormalizarlas e incluir bajo la tabla una nota que declare que `Suma` se calcula con valores sin redondear y que la suma de las tres columnas mostradas puede diferir de 1 hasta `1.5e-4` por redondeo. `[E4.C · R]`
- **CA-03.5** *(Ubicuo)* El notebook deberá mostrar en T4.5: cantidad de instancias con probabilidad máxima ≥ 0.90 y < 0.90, cantidad de instancias mal clasificadas con su probabilidad máxima y cantidad de instancias con empate exacto en la probabilidad máxima. `[E4.C · A]`
- **CA-03.6** *(Ubicuo)* El notebook no deberá, en E4.C, usar la partición de prueba, llamar a `fit` o `partial_fit` ni presentar métricas agregadas de desempeño (accuracy, F1). `[E4.C · B]`
- **CA-03.7** *(Ubicuo)* El notebook deberá incluir en E4.P al menos dos predicciones falsables (ver Términos), cada una con la tabla donde se contrastará, registradas antes de ejecutar E4.C (por ejemplo, la cantidad de instancias con probabilidad máxima ≥ 0.90 en T4.5). `[E4.P · R]`
- **CA-03.8** *(Ubicuo)* El notebook deberá incluir en E4.I: por qué cada fila suma 1; distinguir la clase predicha de la probabilidad máxima; comentar la probabilidad máxima de cada instancia mal clasificada (o declarar que no hubo) señalando que una probabilidad alta no garantiza acierto; y contrastar cada predicción de E4.P. `[E4.I · R]`

### RF-04 — Salidas del MLP regresor (Alta)

**Por qué.** Un regresor no entrega probabilidades ni clases: entrega un número en la escala del objetivo, y esa escala debe poder leerse sin confundirla con la interna del modelo.

- **CA-04.1** *(Evento)* Cuando se ejecute E5.C, el notebook deberá obtener con `predict()` del MLP regresor congelado en E3 las predicciones sobre las 89 instancias de validación de Diabetes, en la escala original del objetivo, sin reentrenar ni reconfigurar el modelo. `[E5.C · A, B]`
- **CA-04.2** *(Ubicuo)* El notebook deberá mostrar en T5.1 la activación de la capa de salida del MLP (`identity`), su número de salidas (1) y la media y la desviación estándar del transformador del objetivo (ajustado solo con entrenamiento), con `assert` sobre los dos primeros valores. `[E5.C · A]`
- **CA-04.3** *(Ubicuo)* El notebook deberá mostrar en T5.2 las cinco instancias de validación de menor índice original con las columnas `Índice original` · `Valor real` · `Salida interna (objetivo estandarizado)` · `Predicción (escala original)` · `Error (real − predicción)`. Además deberá verificar con `assert`, para las 89 instancias, que `predicción = salida interna × desviación + media` con diferencia ≤ `1e-9`. `[E5.C · A]`
- **CA-04.4** *(Ubicuo)* El notebook deberá mostrar en T5.3 el rango observado `[mínimo, máximo]` del objetivo en entrenamiento, el mínimo y el máximo de las 89 predicciones, la cantidad de predicciones fuera del rango observado y, para cada una, su índice original y su valor. Las predicciones no se recortan ni se excluyen. `[E5.C · A]`
- **CA-04.5** *(No deseado)* Si alguna predicción cae fuera del rango observado, entonces el notebook deberá señalarla en E5.I como extrapolación, citar su índice y su valor, y no modificar ni el modelo ni la salida. Si ninguna cae fuera, deberá declararlo en E5.I y explicar el concepto igualmente. `[E5.I · R]`
- **CA-04.6** *(Ubicuo)* El notebook deberá incluir en E5.P al menos dos predicciones falsables (ver Términos), cada una con la tabla donde se contrastará, registradas antes de ejecutar E5.C (por ejemplo, la cantidad de predicciones fuera del rango observado en T5.3). `[E5.P · R]`
- **CA-04.7** *(Ubicuo)* El notebook deberá explicar en E5.I, con base en T5.1, por qué la salida es numérica y continua: la capa de salida tiene una sola neurona con activación identidad (sin función que acote o discretice) y la red se entrena minimizando un error cuadrático sobre un valor numérico, por lo que puede devolver cualquier valor real, incluidos valores no observados. `[E5.I · R]`
- **CA-04.8** *(Ubicuo)* El notebook deberá explicar en E5.I cómo se interpreta la salida en Diabetes: es el valor estimado de la medida cuantitativa de progresión de la enfermedad un año después de la línea base, en la escala del objetivo del dataset; no es una probabilidad ni una clase; la diferencia entre dos salidas es una diferencia en puntos de esa medida. `[E5.I · R]`
- **CA-04.9** *(Ubicuo)* El notebook deberá distinguir en E5.I la salida interna (estandarizada) de la predicción en escala original y declarar cuál de las dos reporta. `[E5.I · R]`
- **CA-04.10** *(Ubicuo)* El notebook no deberá usar, en E5.C, la partición de prueba ni llamar a `fit` o `partial_fit`. `[E5.C · B]`

---

## 6. Casos límite

| # | Situación | Comportamiento esperado (resolución) | Ref. |
|---|---|---|---|
| CL-01 | Un problema puede formularse como más de un tipo a la vez (recomendación vista como clasificación o como regresión de valoraciones). | Se asigna según la **salida entregada**, no según la técnica interna: si entrega una lista ordenada de ítems, es recomendación. Una sola fila por tipo; la formulación alternativa va en una nota bajo la tabla. | CA-01.5, CA-01.6 |
| CL-02 | Hay que elegir el ejemplo canónico de cada fila. | Los cinco ejemplos canónicos quedan fijados en CA-01.4. Se eligieron porque cumplen (a) su salida entregada corresponde, según la regla de CA-01.5, a una sola fila; (b) se enuncian en una frase que nombra datos de entrada y salida; y (c) en clasificación y regresión son el dataset del propio notebook. | CA-01.3, CA-01.4, CA-01.5 |
| CL-03 | Puntuaciones muy grandes o muy negativas en softmax. | Con la fórmula sin estabilizar, `e^z` desborda a `inf` por encima de ≈ 709.78 y subdesborda a `0` por debajo de ≈ −745.13 (`float64`), y el cociente produce `nan`. La versión estable resta el máximo, queda finita y suma 1. Una probabilidad de `1.0000` o `0.0000` por saturación numérica no equivale a certeza ni a imposibilidad. El aviso de la fórmula sin estabilizar se muestra, no se oculta. | CA-02.5, CA-02.7 |
| CL-04 | Relación entre softmax y `predict_proba()` del MLP. | En clasificación multiclase (Iris, 3 clases) la capa de salida del MLP usa softmax: `predict_proba()` es el softmax de las puntuaciones de salida. Se verifica con diferencia máxima ≤ `1e-9` y se muestra la activación de salida declarada por el estimador. | CA-02.8 |
| CL-05 | La entrada de softmax contiene `nan` o `±inf`. | La función lanza `ValueError`; no devuelve valores no finitos ni los enmascara. | CA-02.9 |
| CL-06 | Las tres probabilidades no suman 1 por redondeo. | La verificación y la columna `Suma` usan valores sin redondear (`|suma − 1| ≤ 1e-9`). La presentación usa cuatro decimales, la suma de las tres columnas mostradas puede diferir de 1 hasta `1.5e-4` y así se declara. No se renormaliza ni se ajusta ningún valor. | CA-03.3, CA-03.4 |
| CL-07 | Clase predicha frente a probabilidad máxima. | Son magnitudes distintas: la clase es una etiqueta (la que devuelve `predict()`) y la probabilidad máxima es un número. Se muestran en columnas separadas y se verifica que la clase sea la de mayor probabilidad en las 30 filas. | CA-03.2, CA-03.3 |
| CL-08 | Dos clases empatan exactamente en la probabilidad máxima. | Decide `predict()` (la de menor índice de clase) y el caso se cuenta en T4.5 y se comenta en E4.I. Si no hay ninguno, T4.5 muestra 0. | CA-03.5, CA-03.8 |
| CL-09 | Una instancia mal clasificada tiene probabilidad máxima alta. | Se reporta tal cual en T4.4 y T4.5; no se corrige, no se reentrena y E4.I señala que una probabilidad alta no garantiza acierto. | CA-03.8 |
| CL-10 | El objetivo de Diabetes se transforma. | En la variante por defecto el proveedor no transforma el objetivo (rango 25 a 346, comprobable en T5.3); la única transformación es la del propio MLP regresor (estandarización ajustada solo con entrenamiento). La red trabaja en escala estandarizada y la predicción reportada es la salida interna revertida a escala original; la igualdad se verifica con `assert`. | CA-04.2, CA-04.3, CA-04.9 |
| CL-11 | Cómo se interpreta la salida en escala original. | Como el valor estimado de la medida de progresión de la enfermedad, en puntos de esa medida. No es probabilidad ni clase, y no se asigna a unidades clínicas que el dataset no declara. | CA-04.8 |
| CL-12 | Una predicción cae fuera del rango observado. | La salida del MLP regresor no está acotada: es una extrapolación, no un error de ejecución. Se informa índice y valor, no se recorta ni se excluye y no se modifica el modelo. | CA-04.4, CA-04.5 |
| CL-13 | Ninguna predicción cae fuera del rango observado. | T5.3 muestra 0 y E5.I lo declara y explica el concepto igualmente. | CA-04.5 |

---

## 7. Requisitos no funcionales

| ID | Requisito | Verificación |
|---|---|---|
| **RNF-01** Reproducibilidad | Con *Restart & Run All* en el mismo entorno (stack de R-2), el notebook termina sin errores y las tablas T4.1 a T4.5 y T5.1 a T5.3 son idénticas entre dos ejecuciones. Dos ejecuciones son idénticas si cada tabla coincide con valores `float64` sin redondear (`nan` igual a `nan`) y los textos de las tablas coinciden carácter a carácter. No se introduce ninguna fuente de aleatoriedad nueva. | Ejecutar dos veces y comparar. Solo cuenta la ejecución completa desde cero. |
| **RNF-02** Tiempo | La suma de las duraciones de S7, E4.C y E5.C es menor de 10 s y la ejecución completa sigue por debajo de 5 minutos. | Duración por celda registrada por la propia ejecución (nbconvert guarda `iopub.execute_input` y `shell.execute_reply` en los metadatos de cada celda); total medido con *Restart & Run All*. |
| **RNF-03** Autocontención | Se cumplen R-1, R-3 y R-5: ningún archivo adicional, ningún `import` nuevo, ninguna operación de archivo. | B: búsqueda de `open`, `read_csv`, `to_csv`, `savefig`, `joblib.dump`; comparación de los `import` con EP-5. |
| **RNF-04** Verificabilidad | Cada función nueva de lógica tiene al menos un `assert` y ninguna contiene `print`, `display` ni `plt.`. E4.C y E5.C no definen funciones. | B: búsqueda de `print(`, `display(`, `plt.` y `def ` en S7, E4.C y E5.C. |
| **RNF-05** No regresión | Las 20 celdas existentes conservan su código y su texto, y las huellas de EP-6 se repiten idénticas. | Comparar las fuentes antes y después; comparar las huellas. |
| **RNF-06** Idioma | Se cumple R-6. | R: revisión del notebook. |
| **RNF-07** Legibilidad | Cada término técnico no trivial se explica al introducirlo; cada tabla tiene encabezados descriptivos y cada figura, si la hay, título y ejes con nombre. | R: revisión por alguien sin contexto previo del notebook. |

---

## 8. Trazabilidad

| Requisito | Celdas | Salidas | Comprobación |
|---|---|---|---|
| RF-01 | K1 | TK.1 | R, B |
| RF-02 | K2, S7, E4.C | T4.1, T4.2, T4.3 | R, A |
| RF-03 | E4.P, E4.C, E4.I | T4.4, T4.5 | A, B, R |
| RF-04 | E5.P, E5.C, E5.I | T5.1, T5.2, T5.3 | A, B, R |

Cada requisito nuevo se cubre en celdas nuevas identificadas por su ID; ninguno depende de editar una celda existente.

---

## 9. Fuera de alcance

- Implementar o entrenar modelos de clustering, recomendación o asociación (RF-01 es solo descriptivo).
- Modificar, reentrenar o reconfigurar los MLP congelados en E2 y E3.
- Evaluar de nuevo la partición de prueba o presentar métricas agregadas de desempeño en E4 y E5.
- Calibrar probabilidades, funciones de pérdida, temperatura de softmax y otras activaciones.
- Validación cruzada, búsqueda de hiperparámetros, intervalos de confianza y pruebas de significancia.
- Otros datasets distintos de Iris y Diabetes; clasificación binaria.
- Módulos, paquetes, scripts, archivos de datos o dependencias adicionales.

---

## 10. Criterios de finalización

La funcionalidad se considera terminada cuando se cumplen **todos** estos puntos:

- [ ] El notebook tiene las 10 celdas nuevas con los IDs y el orden de la sección 4 (ES-1 a ES-3 y ES-7).
- [ ] TK.1 cumple CA-01.1 a CA-01.7 y K2 cumple CA-02.1 a CA-02.4.
- [ ] Los `assert` de S7 y de E4.C y E5.C pasan en una ejecución completa desde cero (CA-02.5, CA-02.8, CA-02.9, CA-02.10, CA-03.3, CA-04.2, CA-04.3).
- [ ] T4.1 a T4.5 y T5.1 a T5.3 existen con las columnas exactas y se generan de resultados, no se transcriben.
- [ ] E4.P y E5.P se escribieron antes de ejecutar E4.C y E5.C, y cada predicción falsable está contrastada en E4.I y E5.I.
- [ ] Los casos límite CL-01 a CL-13 tienen el comportamiento de la sección 6.
- [ ] No hay archivos de código adicionales, imports nuevos ni operaciones de archivo, y las 20 celdas existentes y las huellas de EP-6 no cambian (RNF-03, RNF-05).
- [ ] Reiniciar y ejecutar todo desde cero termina sin errores, repite resultados idénticos y tarda menos de 5 minutos (RNF-01, RNF-02).

---

## 11. Decisiones de diseño propuestas (requieren aprobación)

| ID | Decisión | Motivo |
|---|---|---|
| D-01 | RF-03 y RF-04 usan la partición de **validación**, no la de prueba. | La prueba ya se evaluó una única vez por modelo (EP-4); usarla otra vez para mostrar probabilidades podría confundirse con una segunda evaluación. Las salidas ilustran el formato de la red, no su desempeño. |
| D-02 | Contenido en celdas **nuevas** (S7, B4, K1, K2, E4, E5) y no dentro de E2 o E3. | Editar E2 o E3 alteraría predicciones ya registradas y resultados ya ejecutados (ES-4, R-7). |
| D-03 | RF-01 y RF-02 teóricos van en celdas conceptuales (K1, K2) y no en experimentos. | No producen resultados medibles que predecir. La parte ejecutable de RF-02 se verifica en E4.C. |
| D-04 | Los tipos de problema se asignan por **salida entregada**. | Es una regla mecánica de cinco casos mutuamente excluyentes y evita discutir la técnica interna. |
| D-05 | El rango observado es el del objetivo en **entrenamiento**. | Es lo único que el modelo vio; el rango del conjunto completo incluye validación y prueba. |
| D-06 | Softmax se implementa propio y estable, sin importar `scipy`. | Muestra el funcionamiento (RF-02) y respeta R-3. |
| D-07 | Tolerancia de comprobación `1e-9`; presentación a cuatro decimales. | La comprobación numérica es mucho más estricta que lo que se muestra; la nota de CA-03.4 explica la diferencia. |
| D-08 | Las puntuaciones de salida se calculan con una función de S7 que usa solo atributos públicos del estimador. | scikit-learn no expone las puntuaciones por API pública; usar un método privado ataría la spec a una API que puede cambiar sin aviso. |
| D-09 | Los cinco ejemplos de TK.1 se fijan en CA-01.4. | Elimina la elección subjetiva del ejemplo canónico (CL-02). |
