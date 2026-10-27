# Tareas — Spec 001: Parte A, corpus pequeño

| | |
|---|---|
| **Estado** | Aprobado por una persona el 2026-10-22 |
| **Fecha** | 2026-10-22 |
| **Entradas** | `constitution.md`, `spec.md` (revisión 2 aprobada), `plan.md` (aprobado) |
| **Qué es este documento** | La descomposición del plan en tareas pequeñas y ordenadas. **No se ha ejecutado ninguna tarea** y no contiene código. |

## Cómo leer este documento

- **IDs.** `T-<n>` es una tarea de este documento. Las pruebas del plan se citan por su nombre (`T-LEC`, `T-NORM`, `T-EXH`…, plan §6.1) y las celdas por su ID (`C01`…`C29`, `MD-N`…, plan §1.3).
- **Tamaño.** Cada tarea está pensada para 20 a 30 minutos. Las tareas que añaden pruebas a una celda compartida (por ejemplo `C07`) lo hacen cada una por separado.
- **Orden.** Las tareas siguen sus dependencias, no el orden de las celdas. Cada celda se inserta en la posición que fija el plan §1.3, dentro de la sección correspondiente del enunciado y antes de las celdas vacías existentes (plan §8).
- **Archivo.** Salvo indicación, «`02_booleansearch.ipynb`» es `notebooks/Tarea2/02_booleansearch.ipynb`. No se crea ningún otro archivo `[P 1]`.
- **Reglas comunes de «Hecho cuando».** Toda tarea que crea una celda L cumple además I-1 (sin `print`, `input`, `open`, `nltk` ni `import`) y I-3 (sin escritura en disco) del plan §1.4. Toda verificación es un `assert` dentro del notebook, sin framework de pruebas `[P 4]`. Código, comentarios y textos mostrados en español `[P 6]`.
- **[EXISTENTE].** El proyecto **no tiene código implementado**: `02_booleansearch.ipynb` solo contiene el enunciado en markdown y celdas vacías (plan §8). Por eso **ninguna tarea lleva la marca [EXISTENTE]**. Ninguna tarea modifica ni borra celdas existentes; todas insertan celdas nuevas.
- **Pendientes del plan.** Las decisiones P1, P2, P3, P8 y P9 se cierran dentro de tareas concretas (T-30, T-14, T-12, T-44). Ver la tabla al final.

## Fases

| Fase | Tareas | Contenido |
|---|---|---|
| A | T-1 a T-4 | Entorno, configuración, tipos y lectura |
| B | T-5 a T-9 | Normalización |
| C | T-10 a T-14 | Colección, inventario y primera salida con datos reales |
| D | T-15 y T-16 | Búsqueda secuencial |
| E | T-17 a T-19 | Medición |
| F | T-20 a T-27 | Índice, fusiones y consultas (con datos sintéticos) |
| G | T-28 a T-33 | Índice y términos de prueba con datos reales |
| H | T-34 a T-42 | Equivalencia, medición e informe |
| I | T-43 a T-45 | Cierre y verificación global |

---

## Fase A — Entorno, configuración, tipos y lectura

- [x] T-1 Instalar las dependencias y la colección Gutenberg de NLTK (paso manual del usuario)
  - RF cubiertos: ninguno directo; prerrequisito de RF-1.1, RF-1.2 y RF-1.4 (spec §1)
  - Depende de: ninguna
  - Archivos involucrados: `requirements.txt` (ya actualizado, plan §8); la colección se instala fuera del repositorio
  - Hecho cuando: `pip install -r requirements.txt` termina sin errores, la descarga manual de la colección (`python -m nltk.downloader gutenberg`) termina sin errores y `import nltk` funciona en el kernel del proyecto (confirmación del usuario). La comprobación automática de los 18 documentos queda en T-12.

- [x] T-2 Celdas C01 (importaciones) y C02 (configuración)
  - RF cubiertos: RF-1.9, RF-10.1, RF-12.3
  - Depende de: ninguna
  - Archivos involucrados: `02_booleansearch.ipynb` (C01, C02, tras la celda markdown «1. Preparación del corpus»)
  - Hecho cuando: C01 y C02 se ejecutan sin error; C02 define una única variable de ruta `RUTA_NLTK_DATA = None` y las constantes `DOCUMENTOS_ESPERADOS = 18`, `CALENTAMIENTOS = 3`, `REPETICIONES = 30`, `UMBRAL_FRECUENTE = 12`, `MAX_POCO_FRECUENTE = 2`, `TOP_DF = 10`, `PRESUPUESTO_SEGUNDOS = 300`, `TERMINOS_PRUEBA` con marcadores (P1 aún abierto) y la marca `T_INICIO`; inspección I-2 (las importaciones están solo en C01 y son de la biblioteca estándar).

- [x] T-3 Celda C03: tipos de datos, excepciones y `formatear_tabla`
  - RF cubiertos: RF-5.2, RF-13.1
  - Depende de: T-2
  - Archivos involucrados: `02_booleansearch.ipynb` (C03)
  - Hecho cuando: C03 se ejecuta sin error y define `Documento`, `Indice`, `ConsultaPrueba`, `ConsultaPreparada`, `Rechazo`, `FilaInventario`, `ProblemaTermino`, `ResultadoMedicion`, `ErrorDeColeccion`, `TerminoInvalido` (con `original` y `cantidad_tokens` o `tipo`, mensajes en español) y `formatear_tabla` según plan §2.1 y §4; I-1 en C03. Las comprobaciones de `formatear_tabla` (prueba T-FMT) se hacen en T-18.

- [x] T-4 Celda C04 (definiciones): `obtener_lector_nltk` y `extraer_textos`, con la prueba T-LEC
  - RF cubiertos: RF-1.1, RF-1.2, RF-1.4, RF-1.5, RF-1.6, RF-1.10, RF-1.12
  - Depende de: T-3
  - Archivos involucrados: `02_booleansearch.ipynb` (C04 para las definiciones; C07 para la prueba T-LEC)
  - Hecho cuando: la prueba T-LEC pasa en C07 con un lector falso: devuelve los ids tal cual, un archivo vacío da `""`, un `UnicodeDecodeError` produce `ErrorDeColeccion` con el nombre del archivo y `obtener_lector_nltk(None, nombre_modulo="modulo_inexistente")` lanza `ErrorDeColeccion`; `obtener_lector_nltk` devuelve también la versión de NLTK; inspección I-2 e I-3 (la importación de NLTK es dinámica, sin `open`, escritura ni `download`).

## Fase B — Normalización

- [x] T-5 Celda C06 (parte 1): `clasificar_caracter`
  - RF cubiertos: RF-2.13
  - Depende de: T-2
  - Archivos involucrados: `02_booleansearch.ipynb` (C06; pruebas en C07)
  - Hecho cuando: en C07 un `assert` por cada una de las 10 clases de RF-2.13 con al menos un carácter representativo (apóstrofos U+0027, U+2018, U+2019 y U+02BC con prioridad sobre las demás clases; letra; dígito; marca combinable; guion o raya y U+2212; espacio incluido el duro; control; formato invisible; puntuación o símbolo; y U+0378 y U+E000 como `no_clasificado`); un argumento que no sea de longitud 1 lanza `ValueError`.

- [x] T-6 Celda C06 (parte 2): `normalizar_texto`, núcleo sin apóstrofos
  - RF cubiertos: RF-2.1, RF-2.3, RF-2.4, RF-2.5, RF-2.6, RF-2.9, RF-2.10, RF-2.12, RF-2.14, RF-2.15, RF-2.16, RF-2.17, RF-2.18, RF-2.20, RF-2.21, RF-2.22
  - Depende de: T-5
  - Archivos involucrados: `02_booleansearch.ipynb` (C06; pruebas en C07)
  - Hecho cuando: la prueba T-NORM pasa en C07 para los ejemplos sin apóstrofos: «Café»→`cafe`; «año»→`ano`; «ß æ ø đ ł» conservadas; «ﬁ»→`fi`; «x²»→`x2`; «ＡＢ»→`ab`; «İSTANBUL»→`istanbul`; «well-known»→`well`,`known`; «a--b»→`a`,`b`; «_x_»→`x`; «co­operate»→`cooperate`; «a​b»→`ab`; «a b»→`a`,`b`; «3.14»→`3`,`14`; «b52»→`b52`; vacío→vacío; «!!!»→vacío; «ΑΣ»→`ασ`; «Ωμέγα»→`ωμεγα`.

- [x] T-7 Celda C06 (parte 3): tratamiento de apóstrofos en `normalizar_texto`
  - RF cubiertos: RF-2.7, RF-2.8, RF-2.18, RF-2.19
  - Depende de: T-6
  - Archivos involucrados: `02_booleansearch.ipynb` (C06; pruebas en C07)
  - Hecho cuando: la prueba T-NORM pasa en C07 para los ejemplos con apóstrofos: «don't»→`dont`; «king’s»→`kings`; «dogs'»→`dogs`; «'tis»→`tis`; «don''t»→`don`,`t`; «5'10»→`510`; y siguen pasando los ejemplos de T-6.

- [x] T-8 Celda C06 (parte 4): `normalizar_termino`
  - RF cubiertos: RF-2.1, RF-2.12, RF-5.1, RF-5.2, RF-5.5, RF-5.7
  - Depende de: T-3, T-7
  - Archivos involucrados: `02_booleansearch.ipynb` (C06; pruebas en C07)
  - Hecho cuando: la prueba T-TERM pasa en C07: `normalizar_termino("Café")`=`cafe`; `" casa "`=`casa`; `"and"`=`and`; `""`, `"!!!"`, `"well-known"`, `"new york"`, `None` y `5` lanzan `TerminoInvalido` con el original y la cantidad de tokens o el tipo; la función reutiliza `normalizar_texto` (una sola configuración, RF-2.1).

- [x] T-9 Prueba diferencial de la normalización (celda C07)
  - RF cubiertos: RF-2.13, RF-2.18, RF-3.2
  - Depende de: T-7
  - Archivos involucrados: `02_booleansearch.ipynb` (C07)
  - Hecho cuando: la prueba T-DIF pasa: `normalizar_texto("a"+c+"b")` coincide con una función de referencia carácter a carácter (definida solo en la celda de pruebas) para todos los puntos de código del BMP y una muestra de los planos 1 y 2; U+0378 y U+E000 dan clase `no_clasificado`.

## Fase C — Colección, inventario y primera salida con datos reales

- [x] T-10 Celda C05: `ordenar_ids`, `validar_cantidad`, `construir_documentos`, `resumir_tamanos`
  - RF cubiertos: RF-1.11, RF-4.2, RF-4.5, RF-4.6, RF-4.7
  - Depende de: T-3, T-7
  - Archivos involucrados: `02_booleansearch.ipynb` (C05; pruebas en C07)
  - Hecho cuando: la prueba T-COL pasa en C07: `ordenar_ids(["b.txt","B.txt","a.txt","a-1.txt","a2.txt","a10.txt"])` = `["B.txt","a-1.txt","a.txt","a10.txt","a2.txt","b.txt"]`; `validar_cantidad` falla con 0, 17 y 19 (mensaje con cantidad hallada y esperada) y pasa con 18; `resumir_tamanos` sobre la colección sintética del plan §2.2 da tokens 5, 4 y 0, caracteres = longitud de cada texto y totales correctos.

- [x] T-11 Celda C08: `describir_tratamiento` e `inventariar_caracteres`
  - RF cubiertos: RF-3.1, RF-3.2, RF-3.3
  - Depende de: T-7
  - Archivos involucrados: `02_booleansearch.ipynb` (C08; pruebas en C07)
  - Hecho cuando: la prueba T-INV pasa sobre la colección sintética: las cuentas suman la longitud total de los textos; «é» aparece con tratamiento «se convierte en «e»»; un apóstrofo aparece como «se une»; U+0378 aparece como `no_clasificado`; el tratamiento sale de `normalizar_texto` (sonda `"a"+c+"b"`, DT15).

- [x] T-12 Ejecutar la lectura real (C04) y la orquestación de la sección 1 (C09)
  - RF cubiertos: RF-1.1, RF-1.10, RF-4.2
  - Depende de: T-1, T-4, T-10, T-11
  - Archivos involucrados: `02_booleansearch.ipynb` (C04 para la llamada real; C09)
  - Hecho cuando: C04 y C09 se ejecutan con la colección real y la comprobación T-REAL1 pasa (`validar_cantidad` con 18 documentos); `textos`, `version_nltk`, `ids` (orden de RF-1.11), `docs`, `tamanos` e `inventario` existen; C09 no imprime nada; queda confirmado que `fileids()`, `raw()` y el comportamiento descrito en plan §3.1 funcionan como se asumió (cierra P3).

- [x] T-13 Celda C10: presentación de la sección 1
  - RF cubiertos: RF-1.12, RF-3.1, RF-3.3, RF-4.1, RF-4.5
  - Depende de: T-3, T-12
  - Archivos involucrados: `02_booleansearch.ipynb` (C10)
  - Hecho cuando: al ejecutar C10 se muestran las versiones de Python y NLTK, el número de documentos, el inventario (U+XXXX, carácter visible o nombre, clase, tratamiento y apariciones, con los `no_clasificado` visibles) y la tabla de tamaños con totales de documentos, caracteres y tokens; inspección de la salida contra RF-1.12, RF-3.3, RF-4.1 y RF-4.5 (plan §10).

- [x] T-14 Celda MD-N: documentación de la normalización
  - RF cubiertos: RF-2.2
  - Depende de: T-13
  - Archivos involucrados: `02_booleansearch.ipynb` (MD-N)
  - Hecho cuando: MD-N contiene la regla por clase de carácter con su motivo, el orden de aplicación (RF-2.18), las colisiones aceptadas (año/ano, it's/its, résumé/resume, números con puntuación, casos límite 3 y 19 a 21 de la spec) y la codificación con la que NLTK entrega la colección junto con lo observado en el inventario de T-13, incluidos posibles caracteres mal decodificados (cierra P2); inspección documentada (plan §6.5).

## Fase D — Búsqueda secuencial

- [x] T-15 Celda C11 (parte 1): `buscar_secuencial`
  - RF cubiertos: RF-5.3, RF-6.1, RF-6.2, RF-6.3, RF-6.4, RF-6.5, RF-6.6
  - Depende de: T-3
  - Archivos involucrados: `02_booleansearch.ipynb` (C11; pruebas en C12)
  - Hecho cuando: la prueba T-SEQ pasa en C12: ids ordenados y sin repetidos; un término repetido en un documento cuenta una vez; un término ausente da `[]`; «cat» no encuentra «cats» (token completo); un documento vacío no aparece; inspección I-5 (la función no menciona `indice`) y lectura de los dos `for` anidados con salida al primer acierto.

- [x] T-16 Celda C11 (parte 2): `terminos_de_consulta`, `evaluar_presencia` y `resolver_secuencial`
  - RF cubiertos: RF-6.6, RF-9.1, RF-9.2, RF-9.3, RF-9.4, RF-9.5, RF-9.7
  - Depende de: T-15
  - Archivos involucrados: `02_booleansearch.ipynb` (C11; pruebas en C12)
  - Hecho cuando: en C12 un `assert` comprueba que la consulta del ejemplo del plan §2.2 (`the AND NOT cafe`, ya preparada) devuelve `["a.txt"]`, que una consulta de un término delega en `buscar_secuencial` y que un término ausente en NOT devuelve todos los documentos; el árbol inválido lanza `ValueError`; I-5. La equivalencia exhaustiva con el índice se comprueba en T-22.

## Fase E — Medición

- [x] T-17 Celda C14 (parte 1): `medir_media_ms`, con la prueba T-MED
  - RF cubiertos: RF-12.1, RF-12.3, RF-12.5, RF-12.8, RF-12.11
  - Depende de: T-2
  - Archivos involucrados: `02_booleansearch.ipynb` (C14; pruebas en C15)
  - Hecho cuando: la prueba T-MED pasa en C15 con un ejecutable contador y un reloj falso: el ejecutable se llama exactamente `3 + 30` veces; las 3 primeras no cuentan; la media sale exacta con incrementos conocidos; `cero=True` cuando todos los tiempos son 0; el recolector de basura queda restaurado también si el ejecutable lanza una excepción; `repeticiones=0` lanza `ValueError`. La prueba no usa el reloj real.

- [x] T-18 Celda C14 (parte 2): `formatear_ms`, con la prueba T-FMT (incluye `formatear_tabla`)
  - RF cubiertos: RF-12.2, RF-12.9
  - Depende de: T-3
  - Archivos involucrados: `02_booleansearch.ipynb` (C14; pruebas en C15)
  - Hecho cuando: la prueba T-FMT pasa en C15: `formatear_ms(0)` = `0.000000000`; un valor con más de 9 decimales se redondea al más cercano; `formatear_tabla` alinea columnas y rechaza filas de otra longitud.

- [x] T-19 Celda MD-M: protocolo de medición
  - RF cubiertos: RF-12.4, RF-12.5, RF-12.6, RF-12.7, RF-12.10
  - Depende de: T-17, T-18
  - Archivos involucrados: `02_booleansearch.ipynb` (MD-M, tras la celda markdown «3. Medición de tiempos de ejecución»)
  - Hecho cuando: MD-M explica qué entra y qué no en el tiempo (entra todo el trabajo de resolver la consulta; no entran carga, normalización, términos ni índice), el orden de medición (consulta por consulta, primero secuencial y luego índice), los 3 calentamientos y las 30 ejecuciones, y que no se mide ni se reporta el tiempo de construcción del índice en la Parte A; inspección documentada (plan §6.5).

## Fase F — Índice, fusiones y consultas con datos sintéticos

- [x] T-20 Celda C16 (parte 1): `construir_indice` y `consultar_indice`
  - RF cubiertos: RF-5.3, RF-7.1, RF-7.2
  - Depende de: T-3
  - Archivos involucrados: `02_booleansearch.ipynb` (C16; pruebas en C17)
  - Hecho cuando: la prueba T-IDX (parte de construcción) pasa en C17: listas ordenadas y sin repetidos; el vocabulario es la unión de los tokens; el universo contiene todos los ids, incluido un documento vacío; consultar un término ausente da `[]` y no crea entradas.

- [x] T-21 Celda C16 (parte 2): `fusionar_and`, `fusionar_or` y `complementar`
  - RF cubiertos: RF-9.1, RF-9.2, RF-9.3
  - Depende de: T-3
  - Archivos involucrados: `02_booleansearch.ipynb` (C16; pruebas en C17)
  - Hecho cuando: la prueba T-FUS pasa en C17: las tres funciones coinciden con el álgebra de conjuntos para todos los pares de subconjuntos ordenados de un universo de 4 ids; inspección I-6 (sin `set` ni operadores de conjuntos en las funciones; solo el oráculo de la prueba).

- [x] T-22 Celda C16 (parte 3): `resolver_indice` y la prueba exhaustiva T-EXH
  - RF cubiertos: RF-9.4, RF-9.5, RF-9.7, RF-9.8
  - Depende de: T-16, T-20, T-21
  - Archivos involucrados: `02_booleansearch.ipynb` (C16; pruebas en C17)
  - Hecho cuando: la prueba T-EXH pasa en C17: para un vocabulario pequeño y todas las consultas de profundidad hasta 2 (términos, NOT, AND y OR), `resolver_secuencial` = `resolver_indice` = un oráculo con conjuntos (definido solo en esa celda), con documentos vacíos y términos ausentes.

- [x] T-23 Celda C16 (parte 4): `df_de`, `mayores_df`, `distribucion_df` y `revisar_invariantes_indice`
  - RF cubiertos: RF-7.1, RF-8.1, RF-8.2, RF-8.3, RF-8.4
  - Depende de: T-20
  - Archivos involucrados: `02_booleansearch.ipynb` (C16; pruebas en C17)
  - Hecho cuando: la prueba T-IDX (parte de inspección) pasa en C17: `mayores_df` desempata por orden alfabético y devuelve cuántos términos comparten la DF máxima; `distribucion_df` incluye los ceros; `revisar_invariantes_indice` detecta, con casos defectuosos construidos a propósito, una lista desordenada, una repetición y una DF alterada, y no devuelve violaciones con un índice correcto.

- [x] T-24 Celda C20 (parte 1): constructores de consultas, `preparar_consulta` y `preparar_consultas`
  - RF cubiertos: RF-5.4, RF-5.6, RF-9.4, RF-9.6, RF-13.10
  - Depende de: T-3, T-8
  - Archivos involucrados: `02_booleansearch.ipynb` (C20; pruebas en C21)
  - Hecho cuando: la prueba T-CON (parte de preparación) pasa en C21: el árbol preparado lleva tokens normalizados; un término inválido en el segundo operando rechaza la consulta completa sin evaluar ninguna parte; `preparar_consultas` separa los rechazos (con el motivo en español) y deja pasar las demás; I-1 y ausencia de cualquier análisis de texto libre de consultas.

- [x] T-25 Celda C20 (parte 2): `texto_consulta` y `tipo_consulta`
  - RF cubiertos: RF-13.5
  - Depende de: T-24
  - Archivos involucrados: `02_booleansearch.ipynb` (C20; pruebas en C21)
  - Hecho cuando: la prueba T-CON (parte de texto y tipo) pasa en C21: `texto_consulta` escribe los términos reales con paréntesis alrededor de los operandos AND y OR; `tipo_consulta` devuelve `término único`, `AND`, `OR`, `NOT`, `a AND NOT b` o `(a OR b) AND NOT c` según corresponda.

- [x] T-26 Celda C20 (parte 3): `generar_consultas_prueba`
  - RF cubiertos: RF-10.3, RF-10.4, RF-13.5, RF-13.6
  - Depende de: T-25
  - Archivos involucrados: `02_booleansearch.ipynb` (C20; pruebas en C21)
  - Hecho cuando: la prueba T-CON (parte de generación) pasa en C21 con 9 términos sintéticos: 33 consultas con 9, 6, 6, 3, 6 y 3 por familia en ese orden (bloque AND completo y luego OR, plan §9 P7); ninguna consulta repite un término; cada término de prueba aparece al menos en un AND u OR; en «(a OR b) AND NOT c» c difiere de a y de b; las categorías de cada consulta están en el orden de sus términos; sin 3 términos por categoría lanza `ValueError`.

- [x] T-27 Celda C20 (parte 4): `clasificar_df` y `revisar_terminos_prueba`
  - RF cubiertos: RF-10.1, RF-10.2, RF-10.5, RF-10.6, RF-10.7
  - Depende de: T-8, T-15, T-20
  - Archivos involucrados: `02_booleansearch.ipynb` (C20; pruebas en C21)
  - Hecho cuando: la prueba T-DF pasa en C21: `clasificar_df` para DF 0, 1, 2, 3, 11, 12 y 18 da `inexistente`, `poco frecuente`, `poco frecuente`, `intermedia`, `intermedia`, `frecuente` y `frecuente`; `revisar_terminos_prueba` detecta, sobre una colección sintética, un término en la categoría equivocada, uno inválido, uno repetido, menos de 3 por categoría y una DF distinta entre mecanismos.

## Fase G — Índice y términos de prueba con datos reales

- [x] T-28 Celda C18: construir el índice real y comprobar sus invariantes
  - RF cubiertos: RF-7.1, RF-8.3, RF-9.3
  - Depende de: T-12, T-20, T-23
  - Archivos involucrados: `02_booleansearch.ipynb` (C18, tras la celda markdown «4. Construcción del índice invertido»)
  - Hecho cuando: C18 se ejecuta con la colección real y la comprobación T-REAL2 pasa (`revisar_invariantes_indice` devuelve lista vacía); `indice.universo` contiene los 18 ids en el orden de RF-1.11; C18 no imprime nada.

- [x] T-29 Celda C19 (parte 1): vocabulario, top 10 de DF y distribución de DF
  - RF cubiertos: RF-8.2, RF-8.4
  - Depende de: T-3, T-23, T-28
  - Archivos involucrados: `02_booleansearch.ipynb` (C19)
  - Hecho cuando: al ejecutar C19 se muestran el tamaño del vocabulario, los 10 términos de mayor DF con desempate alfabético y el número de términos que comparten la DF máxima, y la distribución de DF de 1 a 18; inspección de la salida contra RF-8.2 y RF-8.4 (plan §10).

- [x] T-30 Elegir los 9 términos de prueba con las DF reales y registrarlos en C02 (resuelve P1)
  - RF cubiertos: RF-10.1, RF-10.2, RF-10.5
  - Depende de: T-27, T-29
  - Archivos involucrados: `02_booleansearch.ipynb` (C02, variable `TERMINOS_PRUEBA`; celda de exploración temporal descrita en plan §9 P1, que se elimina al terminar)
  - Hecho cuando: `TERMINOS_PRUEBA` contiene exactamente 3 términos distintos por categoría, todos válidos como un solo token, con DF reales ≥ 12 (frecuentes), 1 o 2 (poco frecuentes) y 0 (inexistentes), ninguno con DF entre 3 y 11; la celda de exploración temporal ya no está en el notebook; el método de exploración (celda temporal o función auxiliar) lo decide el usuario al iniciar la tarea (plan §9 P1); cierra P1.

- [x] T-31 Celda C22 y MD-T: validar y justificar los términos de prueba
  - RF cubiertos: RF-10.1, RF-10.2, RF-10.6, RF-10.7
  - Depende de: T-27, T-28, T-30
  - Archivos involucrados: `02_booleansearch.ipynb` (C22, MD-T)
  - Hecho cuando: la comprobación T-REAL3 pasa (`revisar_terminos_prueba` devuelve lista vacía; si no, el `assert` falla con el término y su DF); la DF calculada con el índice coincide con la calculada con la búsqueda secuencial; MD-T indica para cada término su DF y por qué representa su categoría (inspección, plan §6.5).

- [x] T-32 Celda C19 (parte 2): listas y DF de los términos de prueba
  - RF cubiertos: RF-8.1
  - Depende de: T-29, T-30
  - Archivos involucrados: `02_booleansearch.ipynb` (C19)
  - Hecho cuando: C19 muestra, para cada uno de los 9 términos de prueba, su lista de documentos y su DF, y la DF mostrada es igual a la longitud de su lista (comprobación RF-8.3 de T-REAL2 y T-IDX); inspección de la salida.

- [x] T-33 Celda C13: consultas secuenciales con los 9 términos de prueba
  - RF cubiertos: RF-6.1
  - Depende de: T-15, T-30
  - Archivos involucrados: `02_booleansearch.ipynb` (C13, tras la celda markdown «2. Búsqueda secuencial»)
  - Hecho cuando: C13 se ejecuta con la colección real y muestra, para cada término de prueba, su categoría declarada y los documentos que lo contienen, en el orden de RF-1.11; las listas coinciden con las DF validadas en T-31. Esta celda queda en la sección 2 aunque se escriba después de elegir los términos.

## Fase H — Equivalencia, medición e informe

- [x] T-34 Celda C23 (parte 1): `ejecutar_protegido` y `describir_diferencia`
  - RF cubiertos: RF-11.3, RF-11.4, RF-11.5
  - Depende de: T-3
  - Archivos involucrados: `02_booleansearch.ipynb` (C23; pruebas en C24)
  - Hecho cuando: la prueba T-EQ (parte de equivalencia) pasa en C24: `ejecutar_protegido` devuelve `("ok", ...)` o captura la excepción como `("error", ...)`; `describir_diferencia` incluye la consulta, ambas listas completas, los ids exclusivos de cada una, y avisa cuando solo difiere el orden o hay repeticiones.

- [x] T-35 Celda C23 (parte 2): `construir_filas_informe`
  - RF cubiertos: RF-12.11, RF-13.1, RF-13.2, RF-13.5, RF-13.6, RF-13.7, RF-13.10
  - Depende de: T-18, T-25
  - Archivos involucrados: `02_booleansearch.ipynb` (C23; pruebas en C24)
  - Hecho cuando: la prueba T-EQ (parte de informe) pasa en C24: las filas siguen el orden de las consultas y tienen consulta escrita, tipo, categorías, documentos recuperados y las dos medias con 9 decimales; la advertencia «media 0 por resolución del reloj» aparece cuando `cero` es verdadero; los rechazos aparecen como «rechazada» sin tiempos; no hay columnas de desviación, mínimo, máximo ni cociente entre mecanismos.

- [x] T-36 Celda C25: generar y preparar las 33 consultas
  - RF cubiertos: RF-5.4, RF-5.6, RF-10.3, RF-10.4
  - Depende de: T-24, T-26, T-31
  - Archivos involucrados: `02_booleansearch.ipynb` (C25, tras la celda markdown «5. Comparación»)
  - Hecho cuando: C25 genera las 33 consultas con los términos reales, las prepara y la comprobación T-REAL5 pasa (`rechazos` vacío con los términos de prueba válidos); C25 no imprime nada.

- [x] T-37 Celda C26 (parte 1): equivalencia de los dos mecanismos en las 33 consultas
  - RF cubiertos: RF-5.4, RF-9.5, RF-11.1, RF-11.2, RF-11.3, RF-11.5
  - Depende de: T-16, T-22, T-28, T-34, T-36
  - Archivos involucrados: `02_booleansearch.ipynb` (C26)
  - Hecho cuando: la comprobación T-REAL4 pasa para las 33 consultas: `ejecutar_protegido(resolver_secuencial, ...)` y `ejecutar_protegido(resolver_indice, ...)` devuelven exactamente la misma lista en el mismo orden; si difieren, el `assert` falla con el mensaje de `describir_diferencia`; los resultados quedan guardados en `recuperados` para el informe; C26 está antes de C27.

- [x] T-38 Celda C26 (parte 2): identidades booleanas
  - RF cubiertos: RF-11.6
  - Depende de: T-37
  - Archivos involucrados: `02_booleansearch.ipynb` (C26)
  - Hecho cuando: la comprobación T-REAL4 (identidades) pasa con un término de cada categoría y ambos mecanismos: `a AND a`, `a OR a` y `NOT NOT a` devuelven el resultado de `a`; `a AND NOT a` devuelve vacío; `a OR NOT a` devuelve los 18 documentos; estas comprobaciones no se miden.

- [x] T-39 Celda C27: bucle de medición
  - RF cubiertos: RF-11.2, RF-12.1, RF-12.3, RF-12.4, RF-12.5, RF-12.6, RF-12.7, RF-12.8, RF-12.10, RF-13.4, RF-13.9
  - Depende de: T-17, T-37, T-38
  - Archivos involucrados: `02_booleansearch.ipynb` (C27)
  - Hecho cuando: C27 se ejecuta sin error y produce 33 `ResultadoMedicion` y el tiempo total en milisegundos desde el inicio del primer calentamiento hasta el fin de la última medición; inspección de que C26 precede a C27 (RF-11.2), de que cada consulta se mide primero con la búsqueda secuencial y luego con el índice, de que los ejecutables usan consultas ya preparadas (I-7: no reciben `textos`) y de que no hay carga ni construcción del índice dentro de la ventana medida.

- [x] T-40 Celda C28: informe comparativo
  - RF cubiertos: RF-12.2, RF-12.9, RF-12.11, RF-13.1, RF-13.2, RF-13.3, RF-13.4, RF-13.5, RF-13.6, RF-13.7, RF-13.9, RF-13.10
  - Depende de: T-35, T-39
  - Archivos involucrados: `02_booleansearch.ipynb` (C28)
  - Hecho cuando: C28 muestra la tabla con consulta, tipo, categorías, documentos recuperados y media en milisegundos con 9 decimales fijos de la búsqueda secuencial y del índice, más el tiempo total de las mediciones; la salida no contiene desviación, mínimo, máximo, cociente entre mecanismos ni ninguna afirmación automática de cuál es más rápido; inspección de la salida contra RF-13 (plan §10).

- [x] T-41 Celda C29: tiempo de toda la Parte A frente al presupuesto
  - RF cubiertos: RF-13.4 (complemento); RNF-9
  - Depende de: T-2, T-40
  - Archivos involucrados: `02_booleansearch.ipynb` (C29)
  - Hecho cuando: C29 muestra el tiempo transcurrido desde `T_INICIO` y, si supera `PRESUPUESTO_SEGUNDOS`, un aviso con el tiempo real; no usa `assert` para el presupuesto porque la aceptación es humana (spec §9.1).

- [x] T-42 Celda MD-F: lectura de resultados y lista de verificaciones por inspección
  - RF cubiertos: RF-13.3, RF-13.8
  - Depende de: T-40
  - Archivos involucrados: `02_booleansearch.ipynb` (MD-F)
  - Hecho cuando: MD-F contiene la lectura de resultados del autor apoyada solo en datos del informe de C28, y la lista de requisitos verificados por inspección de RNF-3 con espacio para registrar su resultado; inspección (plan §6.5).

## Fase I — Cierre y verificación global

- [x] T-43 Inspección estática del notebook
  - RF cubiertos: RF-1.2, RF-1.3, RF-1.7, RF-1.8, RF-2.11, RF-6.2, RF-6.3, RF-6.6, RF-7.3, RF-7.4, RF-9.6, RF-9.7
  - Depende de: T-14, T-19, T-41, T-42
  - Archivos involucrados: `02_booleansearch.ipynb` (MD-F para registrar el resultado)
  - Hecho cuando: las búsquedas de texto de I-1 a I-8 (plan §1.4 y §6.3) dan el resultado esperado y el resultado queda registrado en MD-F; se revisa el camino de error de RF-1.2 con la colección ausente según plan §10; no existen archivos `.py` ni de salida nuevos en el repositorio.

- [x] T-44 Ejecución de referencia: *Restart & Run All* dos veces (resuelve P8 y P9)
  - RF cubiertos: ninguno nuevo; verificación global de RF-1 a RF-13 (RNF-1, RNF-9, RNF-10)
  - Depende de: T-43
  - Archivos involucrados: `02_booleansearch.ipynb`
  - Hecho cuando: con el kernel recién iniciado, el notebook termina sin errores ni aserciones fallidas; en la segunda ejecución coinciden resultados, inventario, tamaños, DF e informe salvo los tiempos (RNF-1); queda anotado el tiempo de C29 frente a 5 minutos (cierra P8) y si alguna media salió en 0 por la resolución del reloj (cierra P9).

- [ ] T-45 Registrar la aceptación humana y el cierre de los pendientes (requiere aprobación humana)
  - RF cubiertos: ninguno nuevo; cierra el último criterio de finalización de la spec (§8, §9.1)
  - Depende de: T-44
  - Archivos involucrados: `specs/001-parte-a-corpus-pequeno/spec.md` (§9.1) y `plan.md` (§9)
  - Hecho cuando: una persona aprueba el cambio (la spec no cambia sin aprobación humana `[P 2]`); spec §9.1 muestra quién acepta el tiempo real, la fecha y el tiempo medido; plan §9 marca P1, P2, P3, P8 y P9 como resueltas.
  - Estado 2026-10-27: los resultados técnicos se registraron en spec §9.1 y plan §9, pero no consta aceptación humana del tiempo ni cierre de PEND-3; por eso la tarea sigue abierta.

---

## Tabla RF → tareas

Todo RF vigente está cubierto por al menos una tarea. **RF sin cubrir: ninguno.** RF-4.3 y RF-4.4 están derogados en la spec y no requieren tarea.

| RF | Tareas |
|---|---|
| RF-1.1 | T-4, T-12 |
| RF-1.2 | T-4, T-43 |
| RF-1.3 | T-43 |
| RF-1.4 | T-4 |
| RF-1.5 | T-4 |
| RF-1.6 | T-4 |
| RF-1.7 | T-43 |
| RF-1.8 | T-43 |
| RF-1.9 | T-2 |
| RF-1.10 | T-4, T-12 |
| RF-1.11 | T-10 |
| RF-1.12 | T-4, T-13 |
| RF-2.1 | T-6, T-8 |
| RF-2.2 | T-14 |
| RF-2.3 | T-6 |
| RF-2.4 | T-6 |
| RF-2.5 | T-6 |
| RF-2.6 | T-6 |
| RF-2.7 | T-7 |
| RF-2.8 | T-7 |
| RF-2.9 | T-6 |
| RF-2.10 | T-6 |
| RF-2.11 | T-43 |
| RF-2.12 | T-6, T-8 |
| RF-2.13 | T-5, T-9 |
| RF-2.14 | T-6 |
| RF-2.15 | T-6 |
| RF-2.16 | T-6 |
| RF-2.17 | T-6 |
| RF-2.18 | T-6, T-7, T-9 |
| RF-2.19 | T-7 |
| RF-2.20 | T-6 |
| RF-2.21 | T-6 |
| RF-2.22 | T-6 |
| RF-3.1 | T-11, T-13 |
| RF-3.2 | T-9, T-11 |
| RF-3.3 | T-11, T-13 |
| RF-4.1 | T-13 |
| RF-4.2 | T-10, T-12 |
| RF-4.3 | Derogado en la spec: no requiere tarea |
| RF-4.4 | Derogado en la spec: no requiere tarea |
| RF-4.5 | T-10, T-13 |
| RF-4.6 | T-10 |
| RF-4.7 | T-10 |
| RF-5.1 | T-8 |
| RF-5.2 | T-3, T-8 |
| RF-5.3 | T-15, T-20 |
| RF-5.4 | T-24, T-36, T-37 |
| RF-5.5 | T-8 |
| RF-5.6 | T-24, T-36 |
| RF-5.7 | T-8 |
| RF-6.1 | T-15, T-33 |
| RF-6.2 | T-15, T-43 |
| RF-6.3 | T-15, T-43 |
| RF-6.4 | T-15 |
| RF-6.5 | T-15 |
| RF-6.6 | T-15, T-16, T-43 |
| RF-7.1 | T-20, T-23, T-28 |
| RF-7.2 | T-20 |
| RF-7.3 | T-43 |
| RF-7.4 | T-43 |
| RF-8.1 | T-23, T-32 |
| RF-8.2 | T-23, T-29 |
| RF-8.3 | T-23, T-28 |
| RF-8.4 | T-23, T-29 |
| RF-9.1 | T-16, T-21 |
| RF-9.2 | T-16, T-21 |
| RF-9.3 | T-16, T-21, T-28 |
| RF-9.4 | T-16, T-22, T-24 |
| RF-9.5 | T-16, T-22, T-37 |
| RF-9.6 | T-24, T-43 |
| RF-9.7 | T-16, T-22, T-43 |
| RF-9.8 | T-22 |
| RF-10.1 | T-2, T-27, T-30, T-31 |
| RF-10.2 | T-27, T-30, T-31 |
| RF-10.3 | T-26, T-36 |
| RF-10.4 | T-26, T-36 |
| RF-10.5 | T-27, T-30 |
| RF-10.6 | T-27, T-31 |
| RF-10.7 | T-27, T-31 |
| RF-11.1 | T-37 |
| RF-11.2 | T-37, T-39 |
| RF-11.3 | T-34, T-37 |
| RF-11.4 | T-34 |
| RF-11.5 | T-34, T-37 |
| RF-11.6 | T-38 |
| RF-12.1 | T-17, T-39 |
| RF-12.2 | T-18, T-40 |
| RF-12.3 | T-2, T-17, T-39 |
| RF-12.4 | T-19, T-39 |
| RF-12.5 | T-17, T-19, T-39 |
| RF-12.6 | T-19, T-39 |
| RF-12.7 | T-19, T-39 |
| RF-12.8 | T-17, T-39 |
| RF-12.9 | T-18, T-40 |
| RF-12.10 | T-19, T-39 |
| RF-12.11 | T-17, T-35, T-40 |
| RF-13.1 | T-3, T-35, T-40 |
| RF-13.2 | T-35, T-40 |
| RF-13.3 | T-40, T-42 |
| RF-13.4 | T-39, T-40, T-41 |
| RF-13.5 | T-25, T-26, T-35, T-40 |
| RF-13.6 | T-26, T-35, T-40 |
| RF-13.7 | T-35, T-40 |
| RF-13.8 | T-42 |
| RF-13.9 | T-39, T-40 |
| RF-13.10 | T-24, T-35, T-40 |

## Pendientes del plan y tarea que los cierra

| Pendiente | Tarea | Observación |
|---|---|---|
| P1 Los 9 términos de prueba | T-30 (y T-31 los valida) | Depende de las DF reales; el usuario decide el método de exploración (celda temporal o función auxiliar). |
| P2 Codificación y caracteres mal decodificados | T-14 | Se observa con el inventario de T-13. |
| P3 Comportamiento real de la API de NLTK | T-12 | Con la colección ya instalada en T-1. |
| P8 Tiempo total frente a 5 minutos | T-44 | Medido en la ejecución de referencia; la aceptación la registra T-45. |
| P9 Resolución real del reloj | T-44 | Se anota si alguna media salió en 0. |

## Avisos para la aprobación

1. **Orden de tareas distinto al de celdas.** Por dependencias, `C13` (sección 2) se escribe en T-33, después de elegir los términos. Su posición final en el notebook sigue siendo la del plan §1.3.
2. **Método de exploración de términos (P1).** El plan dejó abierto si se usa una celda temporal o una función auxiliar. T-30 lo decide el usuario al empezar; no cambia el resto de las tareas.
3. **Revisión de RF-1.2 con la colección ausente.** El plan (§10) la fija como inspección sin precisar cómo reproducirla; T-43 la deja a criterio de quien la ejecute, sin inventar un procedimiento.
4. **T-45 modifica documentos aprobados** (spec §9.1 y plan §9). Por eso exige aprobación humana explícita antes de tocarlos.
5. **Ninguna tarea toca código existente.** Por eso no hay tareas marcadas [EXISTENTE] (plan §8).
