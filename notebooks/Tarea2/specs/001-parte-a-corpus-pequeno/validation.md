# Validación de la entrega: Spec 001, Parte A (corpus pequeño)

> **Nota de estado, 2026-10-27:** Este informe conserva la auditoría histórica del 2026-10-24 sobre el commit identificado abajo. El notebook se ejecutó después con las partes B y C sin errores; esa ejecución no es una nueva auditoría integral y no demuestra resuelto PEND-3. Se añadieron registros técnicos a spec §9.1 y plan §9; sigue sin constar la aceptación humana requerida por T-45. Para el estado actualizado de toda la tarea, ver `../../docs/estado-tarea2.md`.

| | |
|---|---|
| **Fecha** | 2026-10-24, 20:28 |
| **Veredicto** | **RECHAZADO** |
| **Qué se audita** | `notebooks/Tarea2/02_booleansearch.ipynb`, commit `4686821` («Tarea 2 implementacion pt1.»), contra `spec.md`, `plan.md`, `tasks.md` y `docs/constitution.md` |
| **Entorno** | Windows 11, Python 3.13.14 del `.venv` del proyecto, NLTK 3.10.4, colección Gutenberg de NLTK (18 documentos), kernel `python3` del `.venv` |
| **Alcance de este informe** | Solo constata y reporta: no corrige nada ni propone soluciones. El único archivo escrito en el repositorio es este informe. |

## 1. Veredicto y pendientes

**RECHAZADO.** Los 105 requisitos funcionales vigentes pasan, no hay construcción fuera de alcance y no hay regresiones, pero quedan tres pendientes que impiden aprobar:

| ID | Pendiente | Dónde está | Etapa en que se originó |
|---|---|---|---|
| PEND-1 | La duda abierta 1 de la spec no tiene su registro. §9.1 sigue diciendo «Aceptación: pendiente de registrar», el criterio de finalización 11 (dudas resueltas o aceptadas por una persona, con registro en esa sección) no se cumple y la tarea T-45 no se ejecutó: figura como `[x/]` en `tasks.md` y su «Hecho cuando» no se cumple. La casilla de cada uno de los 12 criterios de la spec §8 sigue sin marcar. | `spec.md` §9 (línea 304) y §8; `tasks.md` T-45 (línea 318) | Implementación (T-45 sin ejecutar; exige aprobación humana) |
| PEND-2 | `plan.md` §9 sigue listando P1, P2, P3, P8 y P9 como pendientes («Siguen pendientes P1, P2, P3, P8 y P9», línea 600, y tabla de las líneas 604 a 612), aunque la ejecución ya los resolvió. | `plan.md` §9; tarea T-45 | Implementación (T-45 sin ejecutar) |
| PEND-3 | Incumplimiento de RNF-9 («cada término de consulta [se normaliza] una sola vez antes de medir, aunque se use en varias consultas; ninguna etapa debe repetir trabajo ya hecho»). Los 9 términos distintos de prueba se normalizan 84 veces en el flujo de producción: 57 en C25, 9 en C13, 9 en C22 y 9 en C26 (M-8). La búsqueda secuencial de los 9 términos también se repite en C13, C22 y C26 (lectura del código). Por esta discrepancia entre código y spec, el principio 2 de la constitución queda NO CUMPLE. | Celdas C25 (`preparar_consultas`), C13, C22 y C26; `plan.md` §3.7 (`preparar_consulta` normaliza cada hoja de cada consulta) y §1.3 (C25) | Plan (el diseño no prevé normalizar una sola vez por término); la implementación siguió el plan |

## 2. Qué se leyó y cómo se verificó

- **Leídos íntegros:** `spec.md` (107 RF, 12 RNF, 12 criterios de finalización), `plan.md`, `tasks.md` (45 tareas) y `constitution.md` (6 principios). La spec define 107 RF y la tabla RF → tareas de `tasks.md` tiene 107 filas, de modo que ninguno quedó sin tarea. RF-4.3 y RF-4.4 están derogados en la spec.
- **Ejecución de la suite (sección 3):** *Restart & Run All* con un kernel recién iniciado, con `nbclient` sobre el notebook del repositorio, de principio a fin. La copia ejecutada se guardó fuera del repositorio (no se usó el botón del IDE, que reescribiría las salidas del `.ipynb` entregado, y la auditoría no puede modificar otros archivos).
- **Pruebas del propio notebook:** son los `assert` de las celdas V, O y L (T-LEC, T-CLASES, T-NORM, T-TERM, T-DIF, T-COL, T-INV, T-SEQ, T-MED, T-FMT, T-IDX, T-FUS, T-EXH, T-CON, T-DF, T-EQ y las comprobaciones T-REAL1 a T-REAL5). Una celda silenciosa significa que todos sus `assert` pasaron.
- **Verificaciones manuales de la auditoría** (comandos concretos; sus scripts están en el scratchpad de la sesión, fuera del repositorio):

| Código | Qué es | Apéndice |
|---|---|---|
| **EJ** | Resultado de la ejecución de auditoría: 30 de 30 celdas de código ejecutadas, 0 errores. `EJ(C07)` significa que esa celda terminó sin error. | A |
| **M-1** a **M-9** | Rutas de fallo y recuentos sobre los datos reales, con una copia en memoria del notebook (M-1: RF-11.3 y RF-11.4 con un índice corrupto; M-2: RF-11.5 con un mecanismo que falla; M-3: RF-10.6; M-4: RF-4.2 con 17 y 19 documentos; M-5: flujo de RF-5.6; M-6: 33 llamadas por consulta y mecanismo en C27; M-7: RF-1.2 y RF-1.4 con el código real; M-8: contadores de llamadas por celda; M-9: huellas de reproducibilidad) | B |
| **IND** | Verificación diferencial independiente: recalcula con expresiones regulares y conjuntos, sin usar código del notebook, los tamaños, el inventario, el vocabulario, la DF, las listas de los 9 términos y el número de documentos de las 33 consultas, y lo compara con lo que imprime el notebook | C |
| **INS** | Inspecciones estáticas I-1 a I-8 del plan §1.4, orden de dependencias entre celdas, idioma, alcance, `ast` de las funciones críticas | D |
| **SNAP** | Foto del repositorio y de la colección antes y después de la auditoría | D |

## 3. Ejecución de la suite

| | |
|---|---|
| Resultado | **TERMINÓ SIN ERRORES**: 30 de 30 celdas de código con contenido ejecutadas, 0 salidas de error |
| Duración de toda la Parte A (salida de C29) | **96,8 s** frente al presupuesto de 300 s, sin aviso |
| Tiempo total de las mediciones (salida de C28) | 86,1 s |
| Celdas con salida | C10, C13, C19, C28 y C29. Las otras 25 celdas son silenciosas: sus `assert` pasaron. |
| Reproducibilidad | C10, C13 y C19 son idénticas byte a byte a las salidas guardadas en el entregable; C28 coincide en todas las columnas salvo los tiempos; las huellas de las 9 estructuras reproducibles coinciden con las de las ejecuciones de referencia del 2026-10-24 (M-9). |

**La salida completa de esta ejecución está en el Apéndice A.**

## 4. Requisito por requisito (RF-1 a RF-13)

Resultados posibles: PASA, FALLA o SIN COBERTURA. «Verificación manual» indica que el requisito no tiene test automático y se comprobó con un comando concreto.

**Resumen: 107 requisitos en la spec; 105 PASA, 0 FALLA, 0 SIN COBERTURA, 2 derogados (NO APLICA).** De los 105 que pasan, 80 tienen test automático y 25 incluyen verificación manual (23 solo manual y 2 test y verificación manual); los demás casos con observación (O-1 y O-2, sección 10) pasan con test automático.

| RF | Test que lo cubre | Resultado | Evidencia |
|---|---|---|---|
| RF-1.1 | T-LEC (C07), T-REAL1 (C09) | PASA | EJ(C07, C09). IND: los 18 ids coinciden con `fileids()` de NLTK y `len(raw)` coincide con la tabla de C10 en los 18 documentos. |
| RF-1.2 | T-LEC (C07, módulo sin colección) y revisión del camino de error | PASA | EJ(C07). M-7: con la colección ausente (simulada en memoria con el código real de C03 y C04) se lanza `ErrorDeColeccion` «Falta la colección Gutenberg de NLTK; instálela a mano (el notebook no la descarga).» |
| RF-1.3 | Inspección I-3 (sin test automático) | PASA (verificación manual) | INS I-3: sin `open(`, `write`, `download`, `savefig`, `.to_`, `pickle` ni `json.dump` en las 30 celdas. SNAP: la huella de los 19 archivos de la colección (11.802.669 bytes) y la de los archivos del repositorio son idénticas antes y después de la auditoría. |
| RF-1.4 | T-LEC (módulo inexistente) | PASA | EJ(C07). M-7: «Falta la biblioteca NLTK; instálela a mano (el notebook no la instala ni la descarga).» |
| RF-1.5 | T-LEC (`UnicodeDecodeError` y `OSError` con lector falso) | PASA | EJ(C07): el mensaje nombra el archivo. Con la colección real no hay fallo: IND, el lector declara `latin1` y decodifica los 18. |
| RF-1.6 | T-LEC (archivo vacío da `""`), T-INV | PASA | EJ(C07, C08V). IND: codificación `latin1`, `raw()` devuelve `str` en los 18 documentos. |
| RF-1.7 | Inspección I-3 | PASA (verificación manual) | INS I-3 y SNAP: nada se copia ni se escribe; el texto solo se mantiene en memoria. |
| RF-1.8 | Inspección del archivo guardado | PASA (verificación manual) | El `.ipynb` entregado trae guardadas las salidas de C10, C13, C19, C28 y C29 (Apéndice D); I-3 no halla ninguna otra escritura. |
| RF-1.9 | Inspección: variable única | PASA (verificación manual) | INS `ruta.py`: la única asignación de ruta es `RUTA_NLTK_DATA = None` en C02; solo C02 y C04 la mencionan. T-LEC prueba su efecto. |
| RF-1.10 | T-LEC, T-COL | PASA | EJ(C07). IND: los ids de C10 son `austen-emma.txt`, etc., con extensión. |
| RF-1.11 | T-COL (`ordenar_ids`) | PASA | EJ(C07): `["B.txt","a-1.txt","a.txt","a10.txt","a2.txt","b.txt"]`. IND: el orden de C10 es `sorted(fileids())`. |
| RF-1.12 | Inspección de la salida de C10 | PASA (verificación manual) | C10 imprime «Python 3.13.14 \| NLTK 3.10.4» (Apéndice A). |
| RF-2.1 | T-TERM, T-NORM | PASA | EJ(C07). M-8: C09 llama 115 veces a `normalizar_texto` (18 documentos y 97 caracteres del inventario) y T-TERM comprueba que `normalizar_termino` usa la misma función. |
| RF-2.2 | Inspección de MD-N | PASA (verificación manual) | MD-N contiene las 10 clases con regla y motivo, el orden de aplicación, las colisiones aceptadas y la codificación (INS `md.py`: 13 de 13 elementos). |
| RF-2.3 | T-NORM, T-DIF | PASA | EJ(C07). T-DIF compara 67.692 puntos de código (BMP completo y muestra de los planos 1 y 2) con una referencia independiente. |
| RF-2.4 | T-NORM, T-DIF | PASA | EJ(C07): «Café» y «año» dan `cafe` y `ano`. |
| RF-2.5 | T-NORM, T-DIF | PASA | EJ(C07): «ß æ ø đ ł» se conservan. |
| RF-2.6 | T-NORM | PASA | EJ(C07): «1815», «b52» y «3.14» (da `3` y `14`). |
| RF-2.7 | T-NORM | PASA | EJ(C07): «don't» da `dont`, «king’s» da `kings`, «5'10» da `510`. |
| RF-2.8 | T-NORM | PASA | EJ(C07): «dogs'», «'tis» y «'hola'». |
| RF-2.9 | T-NORM | PASA | EJ(C07): «well-known» da `well` y `known`; «a--b». |
| RF-2.10 | T-NORM, T-DIF | PASA | EJ(C07). |
| RF-2.11 | Inspección I-1 y alcance | PASA (verificación manual) | INS `alcance.py`: sin reducción a raíz, palabras vacías, corrección ortográfica ni sinónimos (sus únicas coincidencias son falsos positivos, ver Apéndice D). |
| RF-2.12 | Inspección: una sola función | PASA (verificación manual) | M-8: `normalizar_texto` es la única función de normalización y la usan los documentos (C09) y los términos (C13, C22, C25, C26). |
| RF-2.13 | T-CLASES, T-DIF | PASA | EJ(C07): las 10 clases con 48 caracteres representativos y 67.692 puntos de código contra una tabla por categoría Unicode. |
| RF-2.14 | T-NORM, T-DIF | PASA | EJ(C07): «ﬁ» da `fi`, «x²» da `x2`, «ＡＢ» da `ab`. |
| RF-2.15 | T-NORM, T-DIF | PASA | EJ(C07): marca combinable eliminada («Cafe» más U+0301). |
| RF-2.16 | T-NORM, T-DIF | PASA | EJ(C07): «Привет», «Ωμέγα». |
| RF-2.17 | T-NORM | PASA | EJ(C07): «ß» se conserva; «ΑΣ» da `ασ`. |
| RF-2.18 | T-NORM, T-DIF | PASA | EJ(C07). |
| RF-2.19 | T-NORM | PASA | EJ(C07): «don''t», «a'''b»; «a`b» y «a´b» separan. |
| RF-2.20 | T-NORM | PASA | EJ(C07): «_x_», «a--b»; el inventario de C10 trata `_` y `*` como separador. |
| RF-2.21 | T-NORM, T-CLASES | PASA | EJ(C07): guion suave y ancho cero se eliminan sin partir la palabra. |
| RF-2.22 | T-NORM, T-DIF | PASA | EJ(C07): espacio duro y U+2028 separan. |
| RF-3.1 | T-INV (C08V), salida de C10 | PASA | EJ(C08V). C10 muestra 97 caracteres distintos con su clase, tratamiento y apariciones; IND: las apariciones por carácter coinciden con el recuento independiente. |
| RF-3.2 | T-DIF, T-INV | PASA | EJ(C07, C08V): U+0378 y U+E000 dan `no_clasificado` y separan. En la colección real hay 0 no clasificados (C10 lo informa). |
| RF-3.3 | T-INV, salida de C10 | PASA (ver observación O-1) | C10 da U+XXXX, carácter o nombre, clase, tratamiento y apariciones; la suma coincide con IND. |
| RF-4.1 | T-REAL1 (C09), salida de C10 | PASA | C10: «Documentos en la colección: 18». |
| RF-4.2 | T-COL, T-REAL1 | PASA | EJ(C07, C09). M-4: con 17 y con 19 documentos se lanza `ErrorDeColeccion` «Se hallaron 17 (o 19) documentos y se esperaban 18; la ejecución se detiene.» |
| RF-4.3 | Derogado en la spec | NO APLICA | Reemplazado por RF-4.2. |
| RF-4.4 | Derogado en la spec | NO APLICA | Reemplazado por RF-4.2. |
| RF-4.5 | T-COL, salida de C10 | PASA | Tabla de 18 filas y fila TOTAL. IND: caracteres y tokens por documento y totales (11.793.318 y 2.182.038) coinciden con el cálculo independiente. |
| RF-4.6 | T-COL | PASA | EJ(C07): «cafe» dos veces cuenta 4 tokens. IND: tokens como ocurrencias. |
| RF-4.7 | T-COL | PASA | EJ(C07). IND: caracteres igual a `len(raw)` en los 18 documentos. |
| RF-5.1 | T-TERM | PASA | EJ(C07): «Café» da `cafe`. |
| RF-5.2 | T-TERM | PASA | EJ(C07): «», «!!!», «well-known» y «new york» lanzan `TerminoInvalido` con el original y los tokens. M-3. |
| RF-5.3 | T-SEQ, T-IDX, T-EXH | PASA | EJ(C12, C17). C13 muestra los 3 términos inexistentes con «(ninguno)». |
| RF-5.4 | T-CON, T-EXH | PASA | EJ(C17, C21). Una sola función `preparar_consulta` alimenta a ambos mecanismos. |
| RF-5.5 | T-TERM | PASA | EJ(C07): espacios de los extremos ignorados; `None`, `5` y `b"casa"` lanzan `TerminoInvalido` con el tipo. |
| RF-5.6 | T-CON, T-EQ, M-5 | PASA (ver observación O-2) | EJ(C21, C24). M-5: `preparar_consultas` separa 6 rechazadas y prepara las otras 27. |
| RF-5.7 | T-TERM, T-CON | PASA | EJ(C07, C21): «and» es un término ordinario. |
| RF-6.1 | T-SEQ | PASA | EJ(C12). C13: listas en orden y sin repetidos; IND coincide. |
| RF-6.2 | Inspección I-5 (`ast`) | PASA (verificación manual) | INS: `buscar_secuencial` tiene 2 `for` anidados y 1 `break`. |
| RF-6.3 | Inspección I-5 | PASA (verificación manual) | INS I-5: `buscar_secuencial` y `resolver_secuencial` no mencionan el índice; T-SEQ prohíbe nombres de índice en su código. |
| RF-6.4 | T-SEQ | PASA | EJ(C12): «cat» no encuentra «cats» ni «category». |
| RF-6.5 | T-SEQ | PASA | EJ(C12): con el término en primera posición se recorre 1 token; ausente, recorre todos. |
| RF-6.6 | Inspección I-5 | PASA (verificación manual) | INS: `buscar_secuencial` solo menciona `append`, `docs`, `id_documento`, `resultado`, `t`, `token` y `tokens` (sin conjuntos ni contadores). `resolver_secuencial` crea un conjunto de trabajo por llamada: interpretación P4 del plan, aprobada. |
| RF-7.1 | T-IDX, T-REAL2 (C18) | PASA | EJ(C17, C18). IND: vocabulario de 43.137 términos, igual al independiente. |
| RF-7.2 | T-IDX | PASA | EJ(C17): un ausente da `[]` sin crear entradas. |
| RF-7.3 | Inspección I-3 | PASA (verificación manual) | INS I-3 y SNAP: el índice no se escribe en disco. |
| RF-7.4 | Inspección | PASA (verificación manual) | M-8: el índice real se construye una vez, en C18, y no se guarda. |
| RF-8.1 | T-IDX, salida de C19 | PASA | C19: tamaño del vocabulario y lista y DF de los 9 términos de prueba; IND coincide. |
| RF-8.2 | T-IDX, salida de C19 | PASA | C19 muestra el top 10 con desempate alfabético y la distribución de k=1 a 18; IND coincide con ambos. |
| RF-8.3 | T-IDX, T-REAL2 | PASA | EJ(C17, C18): `revisar_invariantes_indice` detecta 5 defectos construidos a propósito y no da violaciones en el índice real. |
| RF-8.4 | T-IDX | PASA | EJ(C17). C19: «Términos que comparten la DF máxima (18): 225»; IND coincide. |
| RF-9.1 | T-FUS, T-EXH | PASA | EJ(C17): AND coincide con el álgebra de conjuntos en todos los pares de subconjuntos de 2 universos de 4 ids. |
| RF-9.2 | T-FUS, T-EXH | PASA | EJ(C17). |
| RF-9.3 | T-FUS, T-EXH | PASA | EJ(C17). IND: «NOT algoritmo» recupera 18 documentos. |
| RF-9.4 | T-EXH | PASA | EJ(C17): 3.280 consultas de profundidad hasta 2 con secuencial, índice y oráculo coinciden. |
| RF-9.5 | T-EXH, T-REAL4 | PASA | EJ(C17, C26). |
| RF-9.6 | Inspección I-1 y alcance | PASA (verificación manual) | INS: las consultas son árboles de tuplas; sin analizador, `eval` ni `exec`. |
| RF-9.7 | T-EXH, T-SEQ y C12, I-5 | PASA | EJ(C12, C17): se deja de recorrer el documento al conocer todos los términos (2 tokens vistos); `resolver_secuencial` no menciona el índice. |
| RF-9.8 | T-EXH | PASA | EJ(C17): la composición está implementada y probada. |
| RF-10.1 | T-DF | PASA | EJ(C21): DF 0, 1, 2, 3, 11, 12 y 18 dan inexistente, poco frecuente, poco frecuente, intermedia, intermedia, frecuente y frecuente. Umbrales 12 y 2 en C02. |
| RF-10.2 | T-DF, T-REAL3 (C22), MD-T | PASA | EJ(C21, C22). MD-T justifica los 9 términos (INS `md.py`). |
| RF-10.3 | T-CON, C25 | PASA | EJ(C21, C25): 33 consultas con 9, 6, 6, 3, 6 y 3 por familia. C28 muestra las 33 filas. |
| RF-10.4 | T-CON, MD-T | PASA | EJ(C21): sin términos repetidos en una consulta, cada término en algún AND u OR, c distinto de a y b. MD-T documenta la asignación. |
| RF-10.5 | T-DF, C22 | PASA | EJ(C21, C22). M-3: «mountain» (DF 11) detiene la ejecución. |
| RF-10.6 | T-DF, M-3 | PASA | EJ(C21). M-3: con «mountain» (DF 11) como frecuente la ejecución se detiene con un mensaje en español que nombra el término y su DF (11); también con «well-known» (2 tokens) y con solo 2 términos poco frecuentes. |
| RF-10.7 | T-DF, T-REAL3, MD-T | PASA | EJ(C21, C22). IND: la DF de C19 (índice) y la de C13 (secuencial) coinciden en los 9 términos. |
| RF-11.1 | T-EXH, T-REAL4 (C26) | PASA | EJ(C17, C26): las 33 consultas dan la misma lista en el mismo orden. |
| RF-11.2 | Inspección del orden de celdas | PASA (verificación manual) | INS: orden real `C26` antes de `C27`; M-1: tras una divergencia no existe `resultados_medicion`. |
| RF-11.3 | T-EQ, M-1 | PASA | EJ(C24). M-1: con el índice sin el documento de «macbeth», C26 se detiene nombrando la consulta y ambas listas, sin medir. |
| RF-11.4 | T-EQ, M-1 | PASA | EJ(C24). M-1: el mensaje muestra ambas listas completas y los ids exclusivos. |
| RF-11.5 | T-EQ, M-2 | PASA | EJ(C24). M-2: con un índice que falla (`ZeroDivisionError`) C26 se detiene. |
| RF-11.6 | T-REAL4 (identidades en C26) | PASA | EJ(C26): 5 identidades con cada uno de los 9 términos y ambos mecanismos. |
| RF-12.1 | T-MED, M-6 | PASA | EJ(C15). M-6: C27 llama 33 veces (3 + 30) por consulta y mecanismo. |
| RF-12.2 | T-FMT, salida de C28 | PASA | EJ(C15). IND: 9 decimales en las 66 medias. |
| RF-12.3 | T-MED, C02, MD-M | PASA | EJ(C15). `CALENTAMIENTOS = 3` en C02, documentado en MD-M; M-6 cuenta 3 calentamientos. |
| RF-12.4 | T-MED, I-7, M-6 | PASA (test y verificación manual) | INS I-7: 40 llamadas, ninguna con `textos`. M-6: 0 normalizaciones y 0 construcciones durante C27. |
| RF-12.5 | T-MED, M-6 | PASA | M-6: cada consulta se mide por separado con cada mecanismo. |
| RF-12.6 | Inspección, M-6 | PASA (verificación manual) | M-6: la ventana medida contiene solo `resolver_secuencial` o `resolver_indice` con la consulta preparada. |
| RF-12.7 | Inspección | PASA (verificación manual) | INS `alcance.py`: el reloj solo se usa en C02, C27 y C29; C29 informa el total de la Parte A (RNF-9), no el de cada fase. |
| RF-12.8 | T-MED | PASA | EJ(C15): 2 lecturas del reloj por ejecución medida; C27 usa `time.perf_counter_ns`. |
| RF-12.9 | T-FMT, salida de C28 | PASA | EJ(C15). IND: `\d+\.\d{9}` en las 66 medias. |
| RF-12.10 | Inspección, M-6 | PASA (verificación manual) | M-6: orden `secuencial ×33, índice ×33` por consulta, sin intercalar. |
| RF-12.11 | T-MED, T-EQ | PASA | EJ(C15, C24). En esta ejecución ninguna media salió en 0 (IND). |
| RF-13.1 | T-EQ, salida de C28 | PASA | EJ(C24). C28 tiene una fila por consulta con tipo, categorías, documentos y las dos medias. |
| RF-13.2 | T-EQ, salida de C28 | PASA (test y verificación manual) | EJ(C24). IND: 7 columnas y ninguna palabra de desviación, mínimo ni máximo en la salida. |
| RF-13.3 | Inspección de C28 | PASA (verificación manual) | IND: ninguna palabra de comparación en la salida automática. |
| RF-13.4 | Inspección de C28 | PASA (verificación manual) | C28: «Tiempo total de las mediciones, calentamientos incluidos: 86089.202700000 ms (86.1 s)». |
| RF-13.5 | T-CON, T-EQ | PASA | EJ(C21, C24). C28 trae la consulta escrita y los 6 tipos. |
| RF-13.6 | T-CON, T-EQ | PASA | EJ(C21, C24). C28: «frecuente, poco frecuente», etc., en el orden de la consulta. |
| RF-13.7 | Inspección | PASA (verificación manual) | C28 añade columnas de consulta, documentos y observación y el total, sin otros estadísticos. |
| RF-13.8 | Inspección de MD-F | PASA (verificación manual) | MD-F cita solo datos del informe: las afirmaciones sobre órdenes de magnitud, «water» y «king», y las 33 consultas se cumplen con los datos de esta ejecución. |
| RF-13.9 | Inspección de C27 | PASA (verificación manual) | M-6: el inicio se marca antes del bucle y después de C26, e incluye los calentamientos. |
| RF-13.10 | T-CON, T-EQ, M-5 | PASA (ver observación O-2) | EJ(C21, C24): las rechazadas salen como «rechazada» sin tiempos. |

## 5. Criterios de finalización de la spec (§8)

| # | Criterio | Resultado | Evidencia |
|---|---|---|---|
| 1 | RF-1 a RF-13 implementados y cada criterio comprobado por verificación automática o inspección documentada | CUMPLE | Sección 4: 105 vigentes PASA, 2 derogados. Las inspecciones están documentadas en MD-F. |
| 2 | Los 18 documentos cargados, con número y tamaños; si el conteo no es 18, la ejecución se detiene | CUMPLE | C10 (18 documentos y tabla de tamaños); M-4. |
| 3 | Configuración de normalización con su motivo, inventario documentado y aplicada igual a documentos y consultas | CUMPLE | MD-N; inventario de C10; M-8 (misma función). |
| 4 | Exactamente 3 términos por categoría, DF comprobada, 24 consultas obligatorias y 9 compuestas | CUMPLE | C22 sin error; C25: 33 consultas; MD-T. |
| 5 | Ambos mecanismos con listas idénticas en todas las consultas antes de cualquier tiempo | CUMPLE | C26 antes de C27; M-1. |
| 6 | Cada consulta con su media de 30 ejecuciones en ms por mecanismo, sin carga ni tokenización | CUMPLE | C28; M-6. |
| 7 | Todo el texto mostrado en español y nada escrito en disco | CUMPLE | INS `idioma.py` (8 coincidencias, todas falsos positivos), I-3 y SNAP. |
| 8 | La Parte A completa en no más de 5 minutos y el tiempo total de las mediciones a la vista | CUMPLE | C29: 96,8 s (la ejecución de referencia más lenta del proyecto, 261,0 s, tampoco lo superó); C28 muestra 86,1 s. |
| 9 | El notebook muestra las versiones de Python y NLTK | CUMPLE | C10. |
| 10 | Las verificaciones por inspección de RNF-3 quedan documentadas | CUMPLE | MD-F: tabla de 12 verificaciones con su resultado (la propia celda indica que la revisión de una persona sigue pendiente). |
| 11 | Las dudas abiertas de la sección 9 están resueltas o aceptadas por una persona, con registro en esa sección | **NO CUMPLE** | `spec.md` línea 304: «Aceptación: pendiente de registrar». Ver PEND-1. |
| 12 | La spec fue aprobada por una persona antes de pasar al plan | CUMPLE | `spec.md` línea 3: «revisión 2 (auditoría QA) aprobada por una persona el 2026-10-22»; `plan.md` aprobado el mismo día. |

## 6. Principios de la constitución

| # | Principio | Resultado | Evidencia |
|---|---|---|---|
| 1 | Simplicidad | CUMPLE | INS `alcance.py`: solo la biblioteca estándar (11 módulos importados en C01) y NLTK de forma dinámica; sin stemming, palabras vacías, ranking, frases, parser, persistencia ni paralelismo; `git ls-files '*.py'` vacío; no hay archivos de salida. |
| 2 | Autoridad de spec | **NO CUMPLE** | La spec no se modificó (`git diff --stat 4cfcda6 HEAD` solo toca el notebook, `tasks.md` y el contexto), pero queda una discrepancia entre el código y la spec sin corregir: RNF-9 (PEND-3). Además T-45 y el registro de §9.1 están pendientes (PEND-1). |
| 3 | Desacoplamiento lógica y E/S | CUMPLE | INS I-1: las 9 celdas de solo lógica no contienen `print`, `input`, `open`, `nltk` ni `import`; la lectura vive en C04; `RUTA_NLTK_DATA` es la única variable de ruta. |
| 4 | Política de tests | CUMPLE | Sin framework de pruebas (imports de C01); solo `assert`; equivalencia de resultados antes de medir (C26 antes de C27; M-1); 3 calentamientos y 30 ejecuciones sin carga ni tokenización (M-6). La parte de la Parte B no aplica. |
| 5 | Persistencia | CUMPLE | INS I-3 y SNAP: no se escribe ni se descarga nada desde el notebook; falla con mensaje claro si falta la colección (M-7); id igual al nombre de archivo. |
| 6 | Idioma | CUMPLE | INS `idioma.py`: código, comentarios, docstrings y mensajes en español; las 8 coincidencias son falsos positivos (la palabra reservada «for», la consulta «the AND NOT cafe», la palabra «error» y el literal `'error'`). |

## 7. Requisitos no funcionales (informativo)

| RNF | Resultado | Evidencia |
|---|---|---|
| RNF-1 Reproducibilidad | CUMPLE | Salidas idénticas entre la ejecución de auditoría y el entregable salvo los tiempos; huellas iguales (M-9). |
| RNF-2 Validez de la medición | CUMPLE | M-6. |
| RNF-3 Verificabilidad | CUMPLE | Sección 4; los valores mostrados en RF-4.1, RF-4.5, RF-8.1, RF-8.2 y RF-13.1 se calculan con funciones cubiertas por T-COL, T-IDX y T-EQ, y IND los confirma sobre los datos reales. |
| RNF-4 Idioma | CUMPLE | INS `idioma.py`. |
| RNF-5 Solo memoria | CUMPLE | INS I-3 y SNAP. |
| RNF-6 Recursos | CUMPLE | No se optimizó memoria; la ejecución completa terminó en 96,8 s. |
| RNF-7 Trazabilidad de decisiones | CUMPLE | MD-N y MD-T. |
| RNF-8 Reutilización | CUMPLE | Las funciones reciben datos en memoria (INS I-1). |
| RNF-9 Eficiencia de ejecución | **NO CUMPLE** | Cumple la carga y normalización de la colección una vez (1 llamada de `construir_documentos` en C09), el índice una vez (1 llamada en C18) y el presupuesto de 5 minutos; no cumple la normalización única por término ni «ninguna etapa debe repetir trabajo ya hecho» (PEND-3, M-8). |
| RNF-10 Validez de la ejecución | CUMPLE | Ejecución completa en orden con kernel nuevo; INS `huerfanos.py`: una sola referencia adelantada, dentro de una función que no se llama antes de definirse su dependencia. |
| RNF-11 Idioma ampliado | CUMPLE | INS `idioma.py`; se usan los términos técnicos del enunciado «token» y «postings». |
| RNF-12 Equipo de referencia | CUMPLE | C10 muestra las versiones de Python y NLTK. |

## 8. Construcción fuera de alcance (spec §7)

**No se encontró ninguna.** INS `alcance.py` busca en las 30 celdas de código cada tema de la spec §7:

| Fuera de alcance | Hallazgo |
|---|---|
| Partes B y C: descarga de Gutenberg, subconjuntos crecientes, gráfica, ajuste lineal y R² | No construido (sin `matplotlib`, `numpy` ni ajustes) |
| Medición de memoria y almacenamiento | No construido |
| Desviación, mínimo y máximo de los tiempos; tiempo de construcción del índice | No construido (el reloj solo se usa en C02, C27 y C29; C18 no mide) |
| Reducción a raíz, palabras vacías, ranking, frases, lenguaje o analizador de consultas, corrección ortográfica, sinónimos | No construido |
| Persistencia, paralelismo, descarga, copia o extracción de datos | No construido |
| Interfaz o servicio; preguntas o decisiones humanas durante la ejecución | No construido (sin `input`) |
| Búsqueda por subcadena | No construido (comparación `==` de tokens completos) |
| Instalar o descargar NLTK o la colección desde el notebook | No construido |

## 9. Regresiones

**Ninguna.** La suite es un único notebook y sus 30 celdas de código terminaron sin error. El commit auditado solo cambia `02_booleansearch.ipynb`, `tasks.md` y `contexto-tarea2.md` (`git diff --stat 4cfcda6 HEAD`), así que no toca ningún archivo de otra funcionalidad ni de `Tarea1`. Los notebooks de `Tarea1` no forman parte de la suite pedida y no se ejecutaron.

## 10. Observaciones no bloqueantes

- **O-1 (RF-3.3, plan).** Tres caracteres de control (U+000A, U+000D, U+001A) aparecen con el nombre «sin nombre» en C10; el plan §3.4 lo prevé. El resto de los caracteres invisibles muestra su nombre Unicode.
- **O-2 (RF-5.6 y RF-13.10, spec y plan).** El rechazo de una consulta con un término inválido funciona a nivel de función (M-5: 27 preparadas, 6 rechazadas), pero en el flujo del notebook la ejecución se detiene antes, en C22 (RF-10.6) y en C25 (T-REAL5), así que una consulta rechazada nunca llega al informe. Es una tensión entre RF-10.6 y RF-5.6, resuelta a favor de RF-10.6.
- **O-3 (desviaciones del plan, documentadas en `contexto-tarea2.md`).** Hay una celda no prevista (C08V, que aloja T-INV en lugar de C07), una función auxiliar (`es_termino`), una constante (`ENCABEZADOS_INFORME`) y un parámetro opcional (`pruebas` en `construir_filas_informe`). C13 y C19 se corrigieron tras el primer *Restart & Run All*, que reveló que usaban `CATEGORIAS` antes de definirse.
- **O-4 (tiempos).** Las ejecuciones completas del proyecto dieron 261,0 s, 81,3 s, 86,6 s y, en esta auditoría, 96,8 s para toda la Parte A. La variación entre ejecuciones es grande y la más lenta quedó a 39 s del presupuesto.
- **O-5 (metadatos).** El notebook declara `language_info.version` «2.7.6» (ya estaba así en el enunciado original, sin cambios en esta entrega); el kernel que lo ejecutó es Python 3.13.14.
- **O-6 (revisión humana).** MD-F indica que la revisión de una persona de las verificaciones por inspección sigue pendiente.

## 11. Integridad de la auditoría

- No se modificó ningún archivo del repositorio salvo este informe. SNAP (Apéndice D): `git status` limpio antes y después de las ejecuciones, huellas del notebook, la spec, el plan, `tasks.md`, la constitución, el contexto y `requirements.txt` iguales, y colección Gutenberg igual (19 archivos, huella `ea8869a181fa3e06`).
- Las copias ejecutadas, las copias instrumentadas y los scripts de verificación están en el scratchpad de la sesión, fuera del repositorio.
- No se hizo commit ni push.

## Apéndice A. Salida completa de la ejecución de auditoría (Restart & Run All)

Kernel recién iniciado, notebook completo de principio a fin (`nbclient`), duración total de la ejecución 101,1 s incluyendo el arranque del kernel; la Parte A tardó 96,8 s según C29. Las celdas que no aparecen son silenciosas y sus `assert` pasaron.

```text
=========== C10 (ejecución 11) ===========
Python 3.13.14 | NLTK 3.10.4
Documentos en la colección: 18

Inventario de caracteres de los textos originales (suma de los documentos)
Código | Carácter   | Clase              | Tratamiento         | Apariciones
-------+------------+--------------------+---------------------+------------
U+000A | sin nombre | control            | separador           |   256.893
U+000D | sin nombre | control            | separador           |    48.430
U+001A | sin nombre | control            | separador           |         2
U+0020 | SPACE      | espacio            | separador           | 2.000.723
U+0021 | !          | puntuacion_simbolo | separador           |     8.660
U+0022 | "          | puntuacion_simbolo | separador           |    31.160
U+0024 | $          | puntuacion_simbolo | separador           |        12
U+0025 | %          | puntuacion_simbolo | separador           |         3
U+0026 | &          | puntuacion_simbolo | separador           |        85
U+0027 | '          | apostrofo          | se une              |    21.682
U+0028 | (          | puntuacion_simbolo | separador           |     1.821
U+0029 | )          | puntuacion_simbolo | separador           |     1.827
U+002A | *          | puntuacion_simbolo | separador           |       283
U+002B | +          | puntuacion_simbolo | separador           |         1
U+002C | ,          | puntuacion_simbolo | separador           |   192.368
U+002D | -          | guion              | separador           |    28.030
U+002E | .          | puntuacion_simbolo | separador           |    81.851
U+002F | /          | puntuacion_simbolo | separador           |        23
U+0030 | 0          | digito             | se conserva         |     5.580
U+0031 | 1          | digito             | se conserva         |    27.496
U+0032 | 2          | digito             | se conserva         |    19.221
U+0033 | 3          | digito             | se conserva         |    12.421
U+0034 | 4          | digito             | se conserva         |     9.287
U+0035 | 5          | digito             | se conserva         |     7.171
U+0036 | 6          | digito             | se conserva         |     6.561
U+0037 | 7          | digito             | se conserva         |     6.030
U+0038 | 8          | digito             | se conserva         |     5.910
U+0039 | 9          | digito             | se conserva         |     5.753
U+003A | :          | puntuacion_simbolo | separador           |    47.625
U+003B | ;          | puntuacion_simbolo | separador           |    27.942
U+003C | <          | puntuacion_simbolo | separador           |         2
U+003D | =          | puntuacion_simbolo | separador           |         1
U+003E | >          | puntuacion_simbolo | separador           |         3
U+003F | ?          | puntuacion_simbolo | separador           |    10.342
U+0040 | @          | puntuacion_simbolo | separador           |         3
U+0041 | A          | letra              | se convierte en «a» |    32.314
U+0042 | B          | letra              | se convierte en «b» |    13.864
U+0043 | C          | letra              | se convierte en «c» |     9.378
U+0044 | D          | letra              | se convierte en «d» |    13.331
U+0045 | E          | letra              | se convierte en «e» |    10.725
U+0046 | F          | letra              | se convierte en «f» |     8.170
U+0047 | G          | letra              | se convierte en «g» |     9.887
U+0048 | H          | letra              | se convierte en «h» |    14.889
U+0049 | I          | letra              | se convierte en «i» |    42.742
U+004A | J          | letra              | se convierte en «j» |     8.459
U+004B | K          | letra              | se convierte en «k» |     2.022
U+004C | L          | letra              | se convierte en «l» |    15.100
U+004D | M          | letra              | se convierte en «m» |    15.582
U+004E | N          | letra              | se convierte en «n» |     7.146
U+004F | O          | letra              | se convierte en «o» |    15.745
U+0050 | P          | letra              | se convierte en «p» |     6.664
U+0051 | Q          | letra              | se convierte en «q» |       685
U+0052 | R          | letra              | se convierte en «r» |    11.295
U+0053 | S          | letra              | se convierte en «s» |    16.830
U+0054 | T          | letra              | se convierte en «t» |    26.711
U+0055 | U          | letra              | se convierte en «u» |     1.501
U+0056 | V          | letra              | se convierte en «v» |     1.189
U+0057 | W          | letra              | se convierte en «w» |    12.741
U+0058 | X          | letra              | se convierte en «x» |       179
U+0059 | Y          | letra              | se convierte en «y» |     3.905
U+005A | Z          | letra              | se convierte en «z» |       988
U+005B | [          | puntuacion_simbolo | separador           |       131
U+005D | ]          | puntuacion_simbolo | separador           |       131
U+005F | _          | puntuacion_simbolo | separador           |     1.116
U+0060 | `          | puntuacion_simbolo | separador           |       209
U+0061 | a          | letra              | se conserva         |   698.889
U+0062 | b          | letra              | se conserva         |   125.982
U+0063 | c          | letra              | se conserva         |   176.471
U+0064 | d          | letra              | se conserva         |   387.163
U+0065 | e          | letra              | se conserva         | 1.108.892
U+0066 | f          | letra              | se conserva         |   201.069
U+0067 | g          | letra              | se conserva         |   162.161
U+0068 | h          | letra              | se conserva         |   635.854
U+0069 | i          | letra              | se conserva         |   534.949
U+006A | j          | letra              | se conserva         |     7.487
U+006B | k          | letra              | se conserva         |    64.654
U+006C | l          | letra              | se conserva         |   360.213
U+006D | m          | letra              | se conserva         |   214.450
U+006E | n          | letra              | se conserva         |   607.945
U+006F | o          | letra              | se conserva         |   662.391
U+0070 | p          | letra              | se conserva         |   129.509
U+0071 | q          | letra              | se conserva         |     6.867
U+0072 | r          | letra              | se conserva         |   491.107
U+0073 | s          | letra              | se conserva         |   540.033
U+0074 | t          | letra              | se conserva         |   800.450
U+0075 | u          | letra              | se conserva         |   250.710
U+0076 | v          | letra              | se conserva         |    82.640
U+0077 | w          | letra              | se conserva         |   188.551
U+0078 | x          | letra              | se conserva         |     8.981
U+0079 | y          | letra              | se conserva         |   172.135
U+007A | z          | letra              | se conserva         |     4.537
U+007D | }          | puntuacion_simbolo | separador           |       389
U+007E | ~          | puntuacion_simbolo | separador           |         1
U+00E6 | æ          | letra              | se conserva         |         2
U+00E8 | è          | letra              | se convierte en «e» |         2
U+00E9 | é          | letra              | se convierte en «e» |         2
U+00EE | î          | letra              | se convierte en «i» |         1
Caracteres distintos: 97 | no clasificados (se tratan como separador): 0

Tamaño de cada documento (caracteres del texto original y tokens tras la normalización)
Documento               | Caracteres | Tokens
------------------------+------------+----------
austen-emma.txt         |    887.071 |   160.999
austen-persuasion.txt   |    466.292 |    83.659
austen-sense.txt        |    673.022 |   119.965
bible-kjv.txt           |  4.332.554 |   852.256
blake-poems.txt         |     38.153 |     6.839
bryant-stories.txt      |    249.439 |    46.321
burgess-busterbrown.txt |     84.663 |    15.969
carroll-alice.txt       |    144.395 |    26.688
chesterton-ball.txt     |    457.450 |    82.245
chesterton-brown.txt    |    406.629 |    72.391
chesterton-thursday.txt |    320.525 |    58.300
edgeworth-parents.txt   |    935.158 |   167.720
melville-moby_dick.txt  |  1.242.990 |   216.201
milton-paradise.txt     |    468.220 |    80.089
shakespeare-caesar.txt  |    112.310 |    20.530
shakespeare-hamlet.txt  |    162.881 |    29.698
shakespeare-macbeth.txt |    100.351 |    17.833
whitman-leaves.txt      |    711.215 |   124.335
TOTAL (18 documentos)   | 11.793.318 | 2.182.038

=========== C13 (ejecución 14) ===========
Búsqueda secuencial de cada término de prueba, con la categoría que se le declaró
Categoría declarada | Término    | Documentos | Documentos que lo contienen
--------------------+------------+------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
frecuente           | water      | 18         | austen-emma.txt, austen-persuasion.txt, austen-sense.txt, bible-kjv.txt, blake-poems.txt, bryant-stories.txt, burgess-busterbrown.txt, carroll-alice.txt, chesterton-ball.txt, chesterton-brown.txt, chesterton-thursday.txt, edgeworth-parents.txt, melville-moby_dick.txt, milton-paradise.txt, shakespeare-caesar.txt, shakespeare-hamlet.txt, shakespeare-macbeth.txt, whitman-leaves.txt
frecuente           | king       | 15         | austen-emma.txt, bible-kjv.txt, blake-poems.txt, bryant-stories.txt, carroll-alice.txt, chesterton-ball.txt, chesterton-brown.txt, chesterton-thursday.txt, edgeworth-parents.txt, melville-moby_dick.txt, milton-paradise.txt, shakespeare-caesar.txt, shakespeare-hamlet.txt, shakespeare-macbeth.txt, whitman-leaves.txt
frecuente           | fortune    | 12         | austen-emma.txt, austen-persuasion.txt, austen-sense.txt, bryant-stories.txt, chesterton-brown.txt, chesterton-thursday.txt, edgeworth-parents.txt, melville-moby_dick.txt, shakespeare-caesar.txt, shakespeare-hamlet.txt, shakespeare-macbeth.txt, whitman-leaves.txt
poco frecuente      | macbeth    | 1          | shakespeare-macbeth.txt
poco frecuente      | ahab       | 2          | bible-kjv.txt, melville-moby_dick.txt
poco frecuente      | emma       | 2          | austen-emma.txt, austen-persuasion.txt
inexistente         | internet   | 0          | (ninguno)
inexistente         | smartphone | 0          | (ninguno)
inexistente         | algoritmo  | 0          | (ninguno)

=========== C19 (ejecución 20) ===========
Tamaño del vocabulario: 43.137 términos

Los 10 términos de mayor DF (a igual DF, en orden alfabético)
Posición | Término | DF
---------+---------+---
1        | a       | 18
2        | about   | 18
3        | after   | 18
4        | against | 18
5        | age     | 18
6        | all     | 18
7        | alone   | 18
8        | am      | 18
9        | among   | 18
10       | an      | 18
Términos que comparten la DF máxima (18): 225

Distribución de DF: cuántos términos aparecen en exactamente k de los 18 documentos
k (documentos) | Términos
---------------+---------
1              | 23.146
2              | 6.421
3              | 3.623
4              | 2.203
5              | 1.571
6              | 1.202
7              | 896
8              | 828
9              | 638
10             | 532
11             | 392
12             | 335
13             | 321
14             | 267
15             | 236
16             | 154
17             | 147
18             | 225

Términos de prueba: lista de documentos (postings) y frecuencia documental (DF)
Categoría      | Término    | DF | Documentos
---------------+------------+----+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
frecuente      | water      | 18 | austen-emma.txt, austen-persuasion.txt, austen-sense.txt, bible-kjv.txt, blake-poems.txt, bryant-stories.txt, burgess-busterbrown.txt, carroll-alice.txt, chesterton-ball.txt, chesterton-brown.txt, chesterton-thursday.txt, edgeworth-parents.txt, melville-moby_dick.txt, milton-paradise.txt, shakespeare-caesar.txt, shakespeare-hamlet.txt, shakespeare-macbeth.txt, whitman-leaves.txt
frecuente      | king       | 15 | austen-emma.txt, bible-kjv.txt, blake-poems.txt, bryant-stories.txt, carroll-alice.txt, chesterton-ball.txt, chesterton-brown.txt, chesterton-thursday.txt, edgeworth-parents.txt, melville-moby_dick.txt, milton-paradise.txt, shakespeare-caesar.txt, shakespeare-hamlet.txt, shakespeare-macbeth.txt, whitman-leaves.txt
frecuente      | fortune    | 12 | austen-emma.txt, austen-persuasion.txt, austen-sense.txt, bryant-stories.txt, chesterton-brown.txt, chesterton-thursday.txt, edgeworth-parents.txt, melville-moby_dick.txt, shakespeare-caesar.txt, shakespeare-hamlet.txt, shakespeare-macbeth.txt, whitman-leaves.txt
poco frecuente | macbeth    | 1  | shakespeare-macbeth.txt
poco frecuente | ahab       | 2  | bible-kjv.txt, melville-moby_dick.txt
poco frecuente | emma       | 2  | austen-emma.txt, austen-persuasion.txt
inexistente    | internet   | 0  | (ninguno)
inexistente    | smartphone | 0  | (ninguno)
inexistente    | algoritmo  | 0  | (ninguno)

=========== C28 (ejecución 29) ===========
Informe comparativo: media de 30 ejecuciones por consulta y mecanismo, en milisegundos
Consulta                            | Tipo               | Categorías                             | Documentos | Búsqueda secuencial (ms) | Índice (ms) | Observación
------------------------------------+--------------------+----------------------------------------+------------+--------------------------+-------------+------------
water                               | término único      | frecuente                              | 18         | 5.700723333              | 0.001630000 |
king                                | término único      | frecuente                              | 15         | 8.232383333              | 0.000376667 |
fortune                             | término único      | frecuente                              | 12         | 27.428683333             | 0.000583333 |
macbeth                             | término único      | poco frecuente                         | 1          | 60.040403333             | 0.000763333 |
ahab                                | término único      | poco frecuente                         | 2          | 53.217513333             | 0.000370000 |
emma                                | término único      | poco frecuente                         | 2          | 51.751836667             | 0.000213333 |
internet                            | término único      | inexistente                            | 0          | 62.911363333             | 0.000196667 |
smartphone                          | término único      | inexistente                            | 0          | 36.408186667             | 0.000710000 |
algoritmo                           | término único      | inexistente                            | 0          | 38.430590000             | 0.000223333 |
water AND king                      | AND                | frecuente, frecuente                   | 15         | 16.309966667             | 0.001893333 |
fortune AND macbeth                 | AND                | frecuente, poco frecuente              | 1          | 90.086110000             | 0.001533333 |
water AND internet                  | AND                | frecuente, inexistente                 | 0          | 123.331520000            | 0.000513333 |
ahab AND emma                       | AND                | poco frecuente, poco frecuente         | 0          | 74.918470000             | 0.000986667 |
macbeth AND smartphone              | AND                | poco frecuente, inexistente            | 0          | 60.378746667             | 0.000730000 |
algoritmo AND internet              | AND                | inexistente, inexistente               | 0          | 127.676460000            | 0.000470000 |
king OR fortune                     | OR                 | frecuente, frecuente                   | 17         | 57.919606667             | 0.002066667 |
water OR ahab                       | OR                 | frecuente, poco frecuente              | 18         | 69.555680000             | 0.002493333 |
king OR smartphone                  | OR                 | frecuente, inexistente                 | 15         | 90.543953333             | 0.001116667 |
emma OR macbeth                     | OR                 | poco frecuente, poco frecuente         | 3          | 93.997226667             | 0.001633333 |
ahab OR algoritmo                   | OR                 | poco frecuente, inexistente            | 2          | 110.865240000            | 0.005973333 |
internet OR smartphone              | OR                 | inexistente, inexistente               | 0          | 88.099850000             | 0.003103333 |
NOT fortune                         | NOT                | frecuente                              | 6          | 26.603010000             | 0.001266667 |
NOT emma                            | NOT                | poco frecuente                         | 16         | 112.230196667            | 0.008166667 |
NOT algoritmo                       | NOT                | inexistente                            | 18         | 75.874196667             | 0.001150000 |
water AND NOT macbeth               | a AND NOT b        | frecuente, poco frecuente              | 17         | 95.601883333             | 0.014723333 |
king AND NOT internet               | a AND NOT b        | frecuente, inexistente                 | 15         | 99.652506667             | 0.012196667 |
ahab AND NOT fortune                | a AND NOT b        | poco frecuente, frecuente              | 1          | 125.814253333            | 0.001646667 |
emma AND NOT smartphone             | a AND NOT b        | poco frecuente, inexistente            | 2          | 113.850766667            | 0.005096667 |
algoritmo AND NOT water             | a AND NOT b        | inexistente, frecuente                 | 0          | 97.610943333             | 0.001333333 |
internet AND NOT macbeth            | a AND NOT b        | inexistente, poco frecuente            | 0          | 114.832690000            | 0.001640000 |
(ahab OR smartphone) AND NOT king   | (a OR b) AND NOT c | poco frecuente, inexistente, frecuente | 0          | 114.044916667            | 0.003846667 |
(fortune OR algoritmo) AND NOT emma | (a OR b) AND NOT c | frecuente, inexistente, poco frecuente | 10         | 133.035280000            | 0.005320000 |
(water OR macbeth) AND NOT internet | (a OR b) AND NOT c | frecuente, poco frecuente, inexistente | 18         | 151.241520000            | 0.039500000 |

Tiempo total de las mediciones, calentamientos incluidos: 86089.202700000 ms (86.1 s)

=========== C29 (ejecución 30) ===========
Tiempo transcurrido desde el inicio de la Parte A: 96.8 s (presupuesto: 300 s)

--- resumen: celdas de código con contenido: 30 | ejecutadas: 30 | sin salida (silenciosas, es decir, sus assert pasaron): ['C01', 'C02', 'C03', 'C04', 'C05', 'C06', 'C07', 'C08', 'C08V', 'C09', 'C11', 'C12', 'C14', 'C15', 'C16', 'C17', 'C18', 'C20', 'C21', 'C22', 'C23', 'C24', 'C25', 'C26', 'C27']
```

## Apéndice B. Comprobaciones manuales sobre una copia en memoria (M-1 a M-9)

Se ejecutaron C01 a C26 en un kernel nuevo con contadores de llamadas (`sys.monitoring`, sin alterar las funciones) y después sondas que fuerzan las rutas de fallo con datos reales. 0 errores en la copia instrumentada.

```text
errores en la copia instrumentada: ninguno
== HUELLAS (comparar con las de las ejecuciones de referencia del 2026-10-24) ==
textos                   a0161533d8284957
ids                      f5854f8360490b35
inventario               2b82cc984951f344
tamanos                  495270c5699d9f1b
indice                   688bb5eab91335dd
top DF y distribucion    6bd1590dacff0dc5
terminos de prueba       cb3f8bd0686ab4b0
consultas                950b0cc489a8e836
recuperados              21677498482cbc6f
== RNF-9: normalizaciones y construcciones por celda (solo celdas donde cambia algún contador) ==
  C07   {'normalizar_termino': 11, 'normalizar_texto': 67750, 'construir_documentos': 1}
  C08V  {'normalizar_texto': 29}
  C09   {'normalizar_texto': 115, 'construir_documentos': 1}
  C13   {'normalizar_termino': 9, 'normalizar_texto': 9}
  C17   {'construir_indice': 7}
  C18   {'construir_indice': 1}
  C21   {'normalizar_termino': 92, 'normalizar_texto': 91, 'construir_indice': 1}
  C22   {'normalizar_termino': 9, 'normalizar_texto': 9}
  C25   {'normalizar_termino': 57, 'normalizar_texto': 57}
  C26   {'normalizar_termino': 9, 'normalizar_texto': 9}
términos distintos de las 33 consultas: 9 | hojas (apariciones de un término) en las 33 consultas: 57
total normalizar_termino en todo el notebook hasta C26: 187 | construir_indice: 9 | construir_documentos: 2
== RF-11.3 / RF-11.4 con datos reales: el índice pierde el documento de «macbeth» ==
se detuvo con AssertionError. Mensaje:
La consulta «macbeth» no da el mismo resultado con la búsqueda secuencial y con el índice.
  Con la búsqueda secuencial: ['shakespeare-macbeth.txt']
  Con el índice: []
  Solo con la búsqueda secuencial: ['shakespeare-macbeth.txt']
  Solo con el índice: []
¿se llegó a medir algún tiempo? (existe resultados_medicion): False
== RF-11.5 con datos reales: el índice falla (ZeroDivisionError) y la secuencial no ==
se detuvo con AssertionError. Mensaje:
La consulta «water» no da el mismo resultado con la búsqueda secuencial y con el índice.
  Con la búsqueda secuencial: ['austen-emma.txt', 'austen-persuasion.txt', 'austen-sense.txt', 'bible-kjv.txt', 'blake-poems.txt', 'bryant-stories.txt', 'burgess-busterbrown.txt', 'carroll-alice.txt', 'chesterton-ball.txt', 'chesterton-brown.txt', 'chesterton-thursday.txt', 'edgeworth-parents.txt', 'melville-m
== RF-10.6 con datos reales: un término deja de cumplir su categoría / deja de ser válido ==
«mountain» (DF 11) como frecuente -> se detuvo: Los términos de prueba no son válidos: «mountain» (DF 11): Su DF (11) no cumple el umbral de la categoría «frecuente».
«well-known» como inexistente -> se detuvo: Los términos de prueba no son válidos: «well-known» (DF None): El término 'well-known' produjo 2 tokens y debe producir exactamente uno.
solo 2 poco frecuentes -> se detuvo: Los términos de prueba no son válidos: «None» (DF None): Se esperan 3 términos en la categoría «poco frecuente» y hay 2.
== RF-4.2 con datos reales: 17 y 19 documentos ==
17 -> ErrorDeColeccion: Se hallaron 17 documentos y se esperaban 18; la ejecución se detiene.
19 -> ErrorDeColeccion: Se hallaron 19 documentos y se esperaban 18; la ejecución se detiene.
== RF-5.6 / RF-13.10 en el flujo real: un término de prueba inválido ==
el flujo se detiene en C25 (no continúa con las demás consultas): Hay consultas rechazadas con los términos de prueba: «well-known»: El término 'well-known' produjo 2 tokens y debe producir exactamente uno. | «well-known AND internet»: El término 'well-known' produj
preparar_consultas por sí sola: preparadas 27 | rechazadas 6 | ejemplo: well-known -> El término 'well-known' produjo 2 tokens y debe producir exactamente uno.
== RF-12.1 / RF-12.3 / RF-12.4 / RF-12.7 / RF-12.10: C27 real con 2 consultas y llamadas contadas ==
llamadas en orden (mecanismo, nº de llamadas seguidas): [('secuencial', 33), ('indice', 33), ('secuencial', 33), ('indice', 33)]
calentamientos + repeticiones esperados por mecanismo y consulta: 3 + 30 = 33
normalizaciones/construcciones durante C27 (cambios en los contadores): {'normalizar_termino': 0, 'normalizar_texto': 0, 'construir_documentos': 0, 'construir_indice': 0}
recolector de basura activo al terminar: True
```

## Apéndice C. Verificación diferencial independiente (IND)

Recalcula con expresiones regulares y conjuntos, sin usar funciones del notebook, lo que el notebook imprime, y lo compara con las salidas de la ejecución de auditoría.

```text
documentos en la tabla de C10: 18 | ids iguales a los de NLTK en orden por puntos de código: True
caracteres por documento iguales (len de raw): True
tokens por documento iguales (método independiente): True
totales independientes: 11793318 2182038 | fila TOTAL del notebook: TOTAL (18 documentos)   | 11.793.318 | 2.182.038
inventario: filas 97 | caracteres distintos independientes: 97 | apariciones por carácter iguales: True
vocabulario independiente: 43137 | notebook: 43.137
distribución de DF 1..18 igual: True | suma = 43137
top 10 independiente: ['a', 'about', 'after', 'against', 'age', 'all', 'alone', 'am', 'among', 'an'] | comparten la DF máxima: 225
DF y listas de los 9 términos de prueba iguales a C19: True
filas del informe: 33 | nº de documentos igual al oráculo independiente en todas: True
mismatches: []
9 decimales fijos en las 66 medias: True
encabezados del informe: ['Consulta', 'Tipo', 'Categorías', 'Documentos', 'Búsqueda secuencial (ms)', 'Índice (ms)', 'Observación']
palabras de dispersión o comparación en toda la salida de C28: ninguna
columna Observación vacía en las 33 filas (ninguna media en 0): True
total mostrado: ['Tiempo total de las mediciones, calentamientos incluidos: 86089.202700000 ms (86.1 s)']
codificación que declara el lector de NLTK: {'latin1'} | raw devuelve str: True | ningún documento vacío: True
```

## Apéndice D. Inspecciones estáticas, reproducibilidad e integridad (INS y SNAP)

Inspecciones I-1 a I-8 del plan, orden de dependencias entre celdas, idioma, alcance, `ast` de las funciones críticas, comparación con las salidas guardadas en el entregable y foto del repositorio y de la colección antes y después.

```text
### inspeccion.py: invariantes I-1 a I-8 del plan §1.4
celdas de código con contenido: 30 ['C01', 'C02', 'C03', 'C04', 'C05', 'C06', 'C07', 'C08', 'C08V', 'C09', 'C10', 'C11', 'C12', 'C13', 'C14', 'C15', 'C16', 'C17', 'C18', 'C19', 'C20', 'C21', 'C22', 'C23', 'C24', 'C25', 'C26', 'C27', 'C28', 'C29']

I-1  celdas L sin print, input, open, nltk ni import:
   C03   sin apariciones
   C05   sin apariciones
   C06   sin apariciones
   C08   sin apariciones
   C11   sin apariciones
   C14   sin apariciones
   C16   sin apariciones
   C20   sin apariciones
   C23   sin apariciones

I-2  sentencias import/from y apariciones de «import» fuera de C01:
   C01   sentencias en líneas [2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12] | otras: ['# C01 · Importaciones (solo biblioteca estándar; NLTK se importa de forma dinámica dentro de C04).']
   C04   sentencias en líneas - | otras: ['modulo = importlib.import_module(nombre_modulo)']

I-3  open(, write, download, savefig, .to_, pickle, json.dump en todas las celdas de código:
   open(      sin apariciones
   write      sin apariciones
   download   sin apariciones
   savefig    sin apariciones
   .to_       sin apariciones
   pickle     sin apariciones
   json.dump  sin apariciones

I-4  input( en alguna celda: sin apariciones

I-5  buscar_secuencial y resolver_secuencial no mencionan el índice:
   buscar_secuencial: menciona indice/índice/postings: False
   resolver_secuencial: menciona indice/índice/postings: False

I-6  fusionar_* y complementar sin conjuntos ni &, |, -:
   fusionar_and: set/frozenset = False, operadores = ninguno
   fusionar_or: set/frozenset = False, operadores = ninguno
   complementar: set/frozenset = False, operadores = ninguno

I-7  el texto original (textos) no se pasa a buscar_secuencial, resolver_secuencial ni construir_indice:
   llamadas revisadas: 40 | pasan `textos`: ninguna

I-8  archivos del repositorio (git):
   git status --short: []
   archivos .py versionados: ninguno
   archivos sin versionar: ninguno

### huerfanos.py: nombres usados antes de definirse, en el orden real del notebook
referencias a nombres que aún no existen: 1
   ('C05', 'dentro de «construir_documentos»', 'normalizar_texto')

### idioma.py: palabras inglesas en comentarios, docstrings y mensajes
comentarios, docstrings y mensajes con palabras inglesas comunes: 8
   ('C11', ['for'], '# C11 · Búsqueda secuencial: dos lazos «for» anidados sobre la lista de tokens de cada doc')
   ('C12', ['the'], '# El ejemplo del plan (sección 2.2): the AND NOT cafe')
   ('C12', ['error'], 'un término ausente da lista vacía, no un error')
   ('C21', ['error'], '# una forma inválida es un error de uso')
   ('C21', ['error'], '# un error de uso no se oculta')
   ('C21', ['and'], '«and» escrito como término es un término ordinario')
   ('C23', ['error'], "('ok', resultado) o ('error', descripción). Captura el fallo para poder compararlo con el ")
   ('C24', ['error'], "captura la excepción como ('error', ...)")

### alcance.py: señales de lo que la spec §7 deja fuera
módulos importados (C01) y los de la constitución permitidos (stdlib, nltk, numpy, matplotlib, psutil, ipykernel): ['collections', 'dataclasses', 'gc', 'importlib', 'itertools', 'platform', 're', 'sys', 'time', 'typing', 'unicodedata']
importados fuera de la lista estándar usada: ninguno
¿se usan numpy, matplotlib, psutil, pandas, statistics, threading, multiprocessing, concurrent, pickle, shelve, sqlite3, subprocess, os, pathlib, shutil, urllib, socket? {'numpy': False, 'matplotlib': False, 'psutil': False, 'pandas': False, 'statistics': False, 'threading': False, 'multiprocessing': False, 'concurrent': False, 'pickle': False, 'shelve': False, 'sqlite3': False, 'subprocess': False, 'os': False, 'pathlib': False, 'shutil': False, 'urllib': False, 'socket': False}
reducción a raíz / stemming / lematización       -> [('C04', 8, 'except ImportError as error:')]
palabras vacías                                  -> sin apariciones
ranking / tf-idf / bm25                          -> sin apariciones
consultas de frase                               -> sin apariciones
analizador de consultas (parser) / eval          -> sin apariciones
corrección ortográfica / sinónimos               -> sin apariciones
estadísticos de dispersión                       -> [('C24', 84, 'assert not any(palabra in texto_encabezados for palabra in ("desviació'), ('C24', 85, 'assert not any(palabra in " ".join(" ".join(f) for f in filas).lower()')]
memoria / almacenamiento                         -> sin apariciones
gráficas / ajuste lineal / R²                    -> sin apariciones
descarga / red                                   -> sin apariciones
persistencia                                     -> sin apariciones
interacción con el usuario                       -> sin apariciones
búsqueda por subcadena / parcial                 -> [('C15', 76, 'assert lineas_tabla[0].index("|") == lineas_tabla[1].index("+") == lin'), ('C15', 79, 'assert lineas_tabla[2].startswith("a AND b  | 0.5") and lineas_tabla[4'), ('C21', 22, 'assert not any(nombre.startswith("resolver") or nombre in ("buscar_sec')]
paralelismo                                      -> sin apariciones

usos del reloj (time.* o reloj) por celda:
   C02 ['T_INICIO = time.perf_counter()']
   C03 ['cero_seq: bool                # la media salió en 0 por la resolución del reloj']
   C15 ['"""Reloj de mentira; cada llamada a su ejecutable avanza el reloj lo que dura se', 'def __call__(self):  # lectura del reloj']
   C23 ['observacion = "media 0 por resolución del reloj (ambos mecanismos)"', 'observacion = "media 0 por resolución del reloj (búsqueda secuencial)"', 'observacion = "media 0 por resolución del reloj (índice)"']
   C24 ['assert "media 0 por resolución del reloj" in filas[1][6] and "índice" in filas[1', 'assert "media 0 por resolución del reloj" in filas[3][6] and "búsqueda secuencia']
   C27 ['reloj = time.perf_counter_ns  # monótono, de la mayor resolución disponible y de', 'inicio_mediciones_ns = reloj()', 'lambda arbol=consulta.preparada: resolver_secuencial(docs, arbol), CALENTAMIENTO']
   C29 ['transcurrido_s = time.perf_counter() - T_INICIO']

### ruta.py: variable única de ruta (RF-1.9)
asignaciones de nivel de celda cuyo nombre sugiere una ruta o carpeta:
   C02 RUTA_NLTK_DATA = None
celdas que mencionan RUTA_NLTK_DATA: ['C02', 'C04']

### ausente.py: RF-1.2 y RF-1.4 con el código real del notebook
--- colección ausente
ErrorDeColeccion: Falta la colección Gutenberg de NLTK; instálela a mano (el notebook no la descarga).
causa encadenada: LookupError
--- biblioteca ausente
ErrorDeColeccion: Falta la biblioteca NLTK; instálela a mano (el notebook no la instala ni la descarga).
causa encadenada: ModuleNotFoundError

### lazos.py: estructura de la búsqueda secuencial (ast)
buscar_secuencial: lazos for = 2, profundidad máxima de for anidados = 2, break = 1
nombres que menciona: ['append', 'docs', 'id_documento', 'resultado', 't', 'token', 'tokens']
sentencias while/comprensiones/lambda: 0
resolver_secuencial: lazos for = 2, profundidad máxima de for anidados = 2, break = 1
nombres que menciona: ['add', 'append', 'buscados', 'buscar_secuencial', 'consulta', 'docs', 'es_termino', 'evaluar_presencia', 'id_documento', 'pendientes', 'presentes', 'remove', 'resultado', 'set', 't', 'terminos_de_consulta', 'tokens']
sentencias while/comprensiones/lambda: 0

### i6.py: fusiones sin conjuntos (ast)
fusionar_and: usa set/frozenset = False, operadores &,|,-,^ = ninguno, literales de conjunto = 0, punteros while = 1
fusionar_or: usa set/frozenset = False, operadores &,|,-,^ = ninguno, literales de conjunto = 0, punteros while = 1
complementar: usa set/frozenset = False, operadores &,|,-,^ = ninguno, literales de conjunto = 0, punteros while = 0
resolver_indice: usa set/frozenset = False, operadores &,|,-,^ = ninguno, literales de conjunto = 0, punteros while = 0

### comparar_salidas.py y comparar_c28.py: salidas guardadas en el entregable frente a la ejecución de auditoría
celdas con salida (entregado): ['C10', 'C13', 'C19', 'C28', 'C29'] | (auditoría): ['C10', 'C13', 'C19', 'C28', 'C29']
C10: salida idéntica byte a byte -> True (8753 caracteres)
C13: salida idéntica byte a byte -> True (2113 caracteres)
C19: salida idéntica byte a byte -> True (2917 caracteres)
C29 entregado: Tiempo transcurrido desde el inicio de la Parte A: 86.6 s (presupuesto: 300 s) | auditoría: Tiempo transcurrido desde el inicio de la Parte A: 96.8 s (presupuesto: 300 s)
C28 total entregado: Tiempo total de las mediciones, calentamientos incluidos: 73047.989500000 ms (73.0 s)
C28 total auditoría: Tiempo total de las mediciones, calentamientos incluidos: 86089.202700000 ms (86.1 s)
filas de la tabla: 34 34
columnas consulta, tipo, categorías, documentos y observación idénticas en las 34 filas (encabezado + 33): True

### foto.py: antes y después de la auditoría
--- antes
HEAD: 4686821 Tarea 2 implementacion pt1.
git status --short: limpio
archivos sin versionar: ninguno
archivos .py versionados: ninguno
sha256 notebooks/Tarea2/02_booleansearch.ipynb: cb04767c5760757c
sha256 notebooks/Tarea2/specs/001-parte-a-corpus-pequeno/spec.md: 2cb628910da1ee74
sha256 notebooks/Tarea2/specs/001-parte-a-corpus-pequeno/plan.md: c4307d6fc800a11f
sha256 notebooks/Tarea2/specs/001-parte-a-corpus-pequeno/tasks.md: 91611c45a1bef5b4
sha256 notebooks/Tarea2/docs/constitution.md: 5e5bb0cca8341f6e
sha256 notebooks/Tarea2/Context/contexto-tarea2.md: 31ccf3cf37ad4b75
sha256 requirements.txt: 9612cd42d48f9173
colección Gutenberg: 19 archivos en C:\Users\bravo\AppData\Roaming\nltk_data\corpora\gutenberg | huella conjunta ea8869a181fa3e06 | tamaño 11802669 bytes
validation.md existe: False
--- después
HEAD: 4686821 Tarea 2 implementacion pt1.
git status --short: limpio
archivos sin versionar: ninguno
archivos .py versionados: ninguno
sha256 notebooks/Tarea2/02_booleansearch.ipynb: cb04767c5760757c
sha256 notebooks/Tarea2/specs/001-parte-a-corpus-pequeno/spec.md: 2cb628910da1ee74
sha256 notebooks/Tarea2/specs/001-parte-a-corpus-pequeno/plan.md: c4307d6fc800a11f
sha256 notebooks/Tarea2/specs/001-parte-a-corpus-pequeno/tasks.md: 91611c45a1bef5b4
sha256 notebooks/Tarea2/docs/constitution.md: 5e5bb0cca8341f6e
sha256 notebooks/Tarea2/Context/contexto-tarea2.md: 31ccf3cf37ad4b75
sha256 requirements.txt: 9612cd42d48f9173
colección Gutenberg: 19 archivos en C:\Users\bravo\AppData\Roaming\nltk_data\corpora\gutenberg | huella conjunta ea8869a181fa3e06 | tamaño 11802669 bytes
validation.md existe: False
--- diff
(sin diferencias)

### mdf.py: afirmaciones de MD-F (lectura de resultados) contra los datos de esta ejecución
(2) índice menor que secuencial en 33 de 33 | razón mínima secuencial/índice: 3497 (>= 100: True )
(2) índice máximo (ms): 0.0395 (< 0,2: True ) | secuencial mín y máx (ms): 5.700723333 151.24152 (máx > 100: True ; mín < 50: True )
(3) water y king más rápidos que los 3 inexistentes: 8.232383333 < 36.408186667 -> True
(4) mayor / menor de los 3 inexistentes: 1.73 (> 1,5: True )
(5) observaciones vacías: True
```
