# Spec 001: Parte A, corpus pequeño (puntos 1 a 5 del notebook)

**Estado:** revisión 2 (auditoría QA) aprobada por una persona el 2026-10-22; una duda abierta (§9.1), que se valida en la primera ejecución completa · **Fecha:** 2026-10-22 · **Etapa SDD:** 2 (especificación: qué y por qué, sin plan técnico)

## 1. Contexto y objetivo

El proyecto compara dos mecanismos de recuperación de información, la búsqueda secuencial y el índice invertido, sobre libros del Proyecto Gutenberg. Esta funcionalidad es la primera etapa: trabajar con un corpus pequeño, los 18 documentos de la colección Gutenberg de NLTK, donde los resultados pueden revisarse con detalle antes de escalar a miles de libros en las Partes B y C.

**Objetivo:** dejar demostrado, sobre esos 18 documentos, que (a) los textos se preparan con una regla de normalización única y documentada, (b) ambos mecanismos resuelven consultas de un término y booleanas (AND, OR, NOT) y devuelven exactamente los mismos documentos, y (c) se conoce el tiempo medio de cada consulta con cada mecanismo.

**Por qué importa:** la regla de normalización y las funciones de consulta se reutilizarán sin cambios en las Partes B y C. Un error o una ambigüedad aquí contaminaría todas las mediciones posteriores, y una comparación de tiempos solo es válida si ambos mecanismos devuelven lo mismo.

**Decisiones previas que condicionan esta spec:**
- La constitución fue enmendada (aprobado el 2026-10-22): la Parte A lee la colección Gutenberg de NLTK ya instalada, en solo lectura, sin descargar nada, y el id de documento es el nombre de archivo. Las reglas de la carpeta de datos con id numérico aplican solo a la Parte B.
- La máquina de trabajo dispone de 96 GB de RAM, por lo que no hay presión de memoria en esta etapa.
- Prerrequisito del entorno: la biblioteca NLTK y la colección Gutenberg deben estar instaladas antes de ejecutar. Al 2026-10-22 no lo están en el equipo de trabajo. Su instalación es un paso manual previo, fuera del notebook (constitución, principio 5).
- Revisión 2 (2026-10-22): incorpora las aclaraciones de la auditoría QA. Los hallazgos se citan como H-n; ver §9.

## 2. Usuarios

- **Estudiante autor:** desarrolla, ejecuta y entrega el trabajo. Necesita resultados correctos, reproducibles y fáciles de explicar.
- **Evaluador (docente):** revisa que se cumpla el enunciado: normalización documentada, doble lazo en la búsqueda secuencial, índice con vocabulario y listas, igualdad de resultados y 30 ejecuciones por medición.
- **Etapas posteriores del propio proyecto (Partes B y C):** reutilizan la normalización y las consultas definidas aquí.

## 3. Historias de usuario

- **HU-1.** Como estudiante, quiero cargar los 18 documentos y ver cuántos son y cuánto miden, para confirmar que trabajo con la colección completa.
- **HU-2.** Como estudiante, quiero una regla única y documentada de normalización que cubra mayúsculas, tildes, números, apóstrofos y símbolos, para que documentos y consultas se comparen en igualdad de condiciones.
- **HU-3.** Como estudiante, quiero buscar un término de forma secuencial y con el índice, para comparar dos mecanismos sobre las mismas consultas.
- **HU-4.** Como estudiante, quiero combinar términos con AND, OR y NOT, para evaluar consultas booleanas con ambos mecanismos.
- **HU-5.** Como evaluador, quiero ver que ambos mecanismos devuelven exactamente los mismos documentos antes de leer cualquier tiempo, para confiar en la comparación.
- **HU-6.** Como estudiante, quiero el tiempo medio de 30 ejecuciones por consulta y mecanismo en milisegundos, para comparar rendimiento con términos frecuentes, poco frecuentes e inexistentes.
- **HU-7.** Como estudiante, quiero errores claros cuando un término de consulta no es válido, para no confundir una consulta mal formada con un término sin resultados.
- **HU-8.** Como evaluador, quiero ver un inventario de todos los caracteres presentes en los textos y cómo se trata cada uno, para comprobar que ninguno quedó sin criterio.
- **HU-9.** Como estudiante, quiero inspeccionar las listas de documentos y la DF de varios términos, para entender cómo está construido el índice.
- **HU-10.** Como estudiante, quiero saber cuánto tarda en total la parte de mediciones, para validar que la Parte A cabe en el presupuesto de tiempo.

**Trazabilidad entre historias y requisitos:**

| Historia | Requisitos que la cumplen |
|---|---|
| HU-1 | RF-1, RF-4 |
| HU-2 | RF-2, RF-3 |
| HU-3 | RF-5, RF-6, RF-7 |
| HU-4 | RF-9, RF-10 |
| HU-5 | RF-11 |
| HU-6 | RF-10, RF-12, RF-13 |
| HU-7 | RF-1.2, RF-1.4, RF-1.5, RF-4.2, RF-5.2, RF-5.6, RF-11.3 |
| HU-8 | RF-3 |
| HU-9 | RF-8 |
| HU-10 | RF-13.4, RF-13.9, RNF-9 |

## 4. Requisitos funcionales

Notación EARS: ubicuo («El sistema debe…»), evento («Cuando…»), estado («Mientras…»), no deseado («Si… entonces…») y opcional («Donde…»).

Algunos requisitos agrupan varios comportamientos (RF-2.6, RF-2.10, RF-8.3, RF-10.3 y RF-12.3); cada comportamiento se verifica por separado.

### RF-1. Carga de la colección
- **RF-1.1** (ubicuo) El sistema debe cargar el texto original completo de cada uno de los documentos de la colección Gutenberg de NLTK, identificando cada documento por su nombre de archivo.
- **RF-1.2** (no deseado) Si la colección no está disponible en el equipo, entonces el sistema debe detenerse con un mensaje en español que indique que falta la colección y que debe instalarse manualmente, sin intentar descargarla.
- **RF-1.3** (ubicuo) El sistema no debe modificar, copiar ni escribir nada en la colección ni en el repositorio.
- **RF-1.4** (no deseado) Si la biblioteca NLTK no puede importarse, entonces el sistema debe detenerse con un mensaje en español que indique que falta la biblioteca y que debe instalarse manualmente, sin intentar instalarla.
- **RF-1.5** (no deseado) Si algún archivo de la colección no puede leerse o decodificarse, entonces el sistema debe detenerse con un mensaje en español que nombre el archivo, sin sustituir ni descartar caracteres en silencio.
- **RF-1.6** (evento) Cuando se lea un archivo, el sistema debe decodificarlo con la codificación con la que la colección lo entrega y conservar su texto sin transformaciones previas; un archivo vacío debe conservarse como documento de 0 caracteres y 0 tokens.
- **RF-1.7** (ubicuo) La prohibición de «copiar» de RF-1.3 se refiere a duplicar o exportar archivos de la colección; no impide mantener su texto en memoria para procesarlo.
- **RF-1.8** (ubicuo) La prohibición de escribir «en el repositorio» de RF-1.3 no alcanza al archivo del propio notebook: guardarlo con sus salidas al ejecutarlo está permitido (constitución, principio 5).
- **RF-1.9** (ubicuo) La ubicación de la colección debe definirse en un único lugar de configuración, con la instalación estándar de NLTK como valor por defecto (constitución, principio 3).
- **RF-1.10** (ubicuo) El identificador de cada documento debe ser su nombre de archivo completo, con extensión (por ejemplo «austen-emma.txt»).
- **RF-1.11** (ubicuo) El orden «de menor a mayor» de RF-6.1, RF-7.1, RF-8.2 y RF-9.5 debe ser el orden ascendente por puntos de código Unicode del identificador completo, distinguiendo mayúsculas de minúsculas. El orden de identificadores de la Parte B lo fijará la spec de esa parte (por número entero), sin cambiar el comportamiento de las operaciones.
- **RF-1.12** (evento) Cuando se haya cargado la colección, el sistema debe mostrar las versiones de Python y de NLTK en uso.

### RF-2. Configuración de normalización y tokenización
- **RF-2.1** (ubicuo) El sistema debe aplicar una única configuración de normalización y tokenización a todos los documentos y a todos los términos de consulta, en todos los experimentos.
- **RF-2.2** (ubicuo) El sistema debe documentar en texto explicativo la regla aplicada a cada clase de carácter y el motivo de cada decisión.
- **RF-2.3** (evento) Cuando el texto contenga mayúsculas, incluidas letras con tilde y la ñ, el sistema debe convertirlas a minúsculas.
- **RF-2.4** (evento) Cuando una letra lleve tilde, diéresis, virgulilla o cedilla, el sistema debe reemplazarla por su letra base («café» pasa a «cafe», «año» pasa a «ano»).
- **RF-2.5** (evento) Cuando aparezca una letra sin equivalente base (por ejemplo «æ», «ø» o «ß»), el sistema debe conservarla como letra.
- **RF-2.6** (evento) Cuando aparezcan dígitos, el sistema debe conservarlos, y una secuencia contigua de letras y dígitos debe formar un único token («1815», «b52»).
- **RF-2.7** (evento) Cuando aparezca un apóstrofo, recto o tipográfico, entre dos caracteres alfanuméricos, el sistema debe eliminarlo y unir ambos lados en un solo token («don't» pasa a «dont», «king's» pasa a «kings»).
- **RF-2.8** (evento) Cuando aparezca un apóstrofo que no esté entre dos caracteres alfanuméricos, el sistema debe tratarlo como separador.
- **RF-2.9** (evento) Cuando aparezca un guion, un guion medio o una raya, el sistema debe tratarlo como separador («well-known» produce los tokens «well» y «known»).
- **RF-2.10** (evento) Cuando aparezca puntuación, un símbolo, un espacio, un salto de línea, un carácter de control o cualquier otro carácter no clasificado, el sistema debe tratarlo como separador y no generar token a partir de él.
- **RF-2.11** (ubicuo) El sistema no debe aplicar reducción a raíz, eliminación de palabras vacías, corrección ortográfica ni sinónimos.
- **RF-2.12** (ubicuo) «Todos los experimentos» de RF-2.1 son todos los de esta spec y los de cualquier parte posterior que reutilice la normalización.
- **RF-2.13** (ubicuo) El sistema debe clasificar cada carácter en una sola de estas clases, según su categoría general Unicode: apóstrofo (U+0027, U+2018, U+2019 y U+02BC, que tiene prioridad sobre las demás clases), letra (L), dígito decimal (Nd), marca combinable (M), guion o raya (Pd y el signo menos U+2212), espacio o separador (Z, incluidos el espacio duro y los separadores de línea y de párrafo), control (Cc), formato invisible (Cf), puntuación, símbolo u otro número (P, S, No y Nl) y no clasificado (sin asignar, uso privado o sustituto).
- **RF-2.14** (evento) Cuando aparezca un carácter con equivalente de compatibilidad (ligaduras como «ﬁ», superíndices como «²», fracciones como «½», formas de ancho completo), el sistema debe sustituirlo por su equivalente antes de aplicar el resto de las reglas («ﬁ» pasa a «fi», «²» pasa a «2», «ａ» pasa a «a»).
- **RF-2.15** (evento) Cuando una letra lleve cualquier marca diacrítica combinable (acento agudo o grave, circunflejo, diéresis, virgulilla, cedilla, anillo, caron, macron, ogonek, breve u otra), el sistema debe eliminar la marca y conservar la letra base.
- **RF-2.16** (evento) Cuando aparezca una letra de cualquier alfabeto, el sistema debe conservarla como letra; esto incluye las que no tienen equivalente base («ß», «ø», «đ», «ł», «ð», «þ», «æ», «œ», «ı») y las de alfabetos no latinos.
- **RF-2.17** (ubicuo) La conversión a minúsculas de RF-2.3 debe ser la conversión simple de cada carácter, no el plegado completo: «ß» se conserva y no se convierte en «ss».
- **RF-2.18** (ubicuo) El sistema debe aplicar las reglas en este orden: (1) sustitución por equivalentes de compatibilidad (RF-2.14); (2) eliminación de marcas diacríticas (RF-2.4 y RF-2.15); (3) conversión a minúsculas (RF-2.3 y RF-2.17); (4) eliminación de caracteres invisibles (RF-2.21); (5) tratamiento de apóstrofos (RF-2.7, RF-2.8 y RF-2.19); (6) separación en tokens por los demás caracteres (RF-2.6, RF-2.9, RF-2.10, RF-2.20 y RF-2.22).
- **RF-2.19** (evento) Cuando aparezcan dos o más apóstrofos consecutivos, el sistema debe tratarlos como separador. El acento grave («`») y el acento agudo suelto («´») no son apóstrofos: son símbolos y separan.
- **RF-2.20** (evento) Cuando aparezca cualquier carácter de la clase guion o raya (guion-menos, guion no separable, guion medio, raya, barra horizontal, signo menos), el subrayado «_» o el asterisco «*», el sistema debe tratarlo como separador; una secuencia como «--» separa por la misma razón.
- **RF-2.21** (evento) Cuando aparezca un carácter invisible de formato (guion suave, ancho cero, marca de orden de bytes, marcas de dirección), el sistema debe eliminarlo sin separar la palabra en la que aparece.
- **RF-2.22** (evento) Cuando aparezca un espacio Unicode, incluido el espacio duro, o un separador de línea o de párrafo, el sistema debe tratarlo como separador.

### RF-3. Inventario de caracteres
- **RF-3.1** (evento) Cuando se haya cargado la colección, el sistema debe mostrar el inventario de caracteres distintos presentes en los documentos originales, con su número de apariciones, su clase y el tratamiento que recibe según RF-2.
- **RF-3.2** (no deseado) Si existe un carácter que no pertenece a ninguna clase documentada, entonces el sistema debe listarlo como «no clasificado» y tratarlo como separador, para que pueda revisarse.
- **RF-3.3** (ubicuo) El inventario debe sumar las apariciones de los 18 documentos y mostrar, por cada carácter distinto, su código Unicode (U+XXXX), el carácter visible o su nombre cuando sea invisible, su clase según RF-2.13, el tratamiento aplicado y su número de apariciones.

### RF-4. Conteo y tamaños
- **RF-4.1** (evento) Cuando se haya cargado la colección, el sistema debe mostrar el número de documentos.
- **RF-4.2** (no deseado) Si el número de documentos no es exactamente 18, entonces el sistema debe detenerse con un mensaje en español que indique la cantidad hallada y la esperada, sin ofrecer continuar.
- **RF-4.3** (derogado en la revisión 2) ~~Cuando la respuesta sea afirmativa y explícita, el sistema debe continuar con los documentos hallados, calcular el universo de RF-9.3 y el umbral de RF-10.1 sobre esa cantidad, y dejar constancia visible de que la colección no tiene 18 documentos en todos los resultados posteriores.~~ Reemplazado por RF-4.2: no se permite ejecutar con una cantidad distinta de 18. El número se conserva por trazabilidad.
- **RF-4.4** (derogado en la revisión 2) ~~Si la respuesta es negativa, no es explícitamente afirmativa o no es posible preguntar (ejecución sin entrada interactiva), entonces el sistema debe detenerse sin continuar.~~ Reemplazado por RF-4.2: no hay pregunta al usuario. El número se conserva por trazabilidad.
- **RF-4.5** (evento) Cuando se haya normalizado la colección, el sistema debe mostrar, por documento, su identificador, su tamaño en caracteres del texto original y su número de tokens, y además los totales de documentos, caracteres y tokens.
- **RF-4.6** (ubicuo) El número de tokens de RF-4.5 debe contar ocurrencias (cada aparición cuenta), no tokens distintos.
- **RF-4.7** (ubicuo) El tamaño en caracteres de RF-4.5 debe ser el número de caracteres Unicode del texto decodificado tal como lo entrega la colección, sin traducir ni eliminar saltos de línea.

### RF-5. Términos de consulta
- **RF-5.1** (evento) Cuando se reciba un término de consulta, el sistema debe normalizarlo con la configuración de RF-2; el término es válido solo si el resultado es exactamente un token («Café» se consulta como «cafe», «1815» es válido).
- **RF-5.2** (no deseado) Si el término normalizado produce cero tokens o más de uno (por ejemplo vacío, solo símbolos como «!!!» o con guion como «well-known»), entonces el sistema debe detenerse con un mensaje en español que indique el término original y cuántos tokens produjo, sin devolver resultado alguno.
- **RF-5.3** (evento) Cuando el término sea válido pero no aparezca en ningún documento, el sistema debe devolver un resultado vacío y no un error.
- **RF-5.4** (ubicuo) Las reglas de RF-5.1 a RF-5.3 deben aplicarse igual en ambos mecanismos y a cada operando de una consulta booleana.
- **RF-5.5** (ubicuo) Un término de consulta debe ser texto y los espacios al inicio y al final deben ignorarse. Un valor que no sea texto debe tratarse como término inválido, con un mensaje en español que indique su tipo.
- **RF-5.6** (no deseado) Si un término de consulta es inválido (RF-5.2 o RF-5.5), entonces el sistema debe rechazar solo esa consulta: no debe evaluarla, medirla ni compararla, debe informar el rechazo con el mensaje de RF-5.2 y debe continuar con las demás consultas. En una consulta compuesta, todos sus términos deben validarse antes de evaluar cualquier parte. Este requisito precisa el sentido de «detenerse» en RF-5.2.
- **RF-5.7** (ubicuo) Las palabras «and», «or» y «not» escritas como términos son términos ordinarios; los operadores no se escriben como texto (RF-9.6).

### RF-6. Búsqueda secuencial
- **RF-6.1** (evento) Cuando se consulte un término válido, el sistema debe devolver los identificadores de todos los documentos que lo contienen, sin repeticiones y en orden ascendente por nombre de archivo (de menor a mayor), aunque el término aparezca varias veces en un documento.
- **RF-6.2** (ubicuo) El sistema debe resolver la búsqueda recorriendo cada documento y, dentro de cada documento, cada uno de sus tokens, como exige el enunciado.
- **RF-6.3** (estado) Mientras se ejecuta una búsqueda secuencial, el sistema no debe usar el índice invertido ni ninguna otra estructura que asocie términos con documentos de forma anticipada.
- **RF-6.4** (ubicuo) Un documento contiene un término solo si alguno de sus tokens es idéntico al token del término; no se admite la coincidencia por subcadena («cat» no encuentra «category»).
- **RF-6.5** (evento) Cuando un token de un documento coincide con el término consultado, el sistema debe pasar al documento siguiente sin recorrer los tokens restantes.
- **RF-6.6** (ubicuo) La búsqueda secuencial no debe usar conjuntos, contadores ni otras estructuras calculadas de antemano por documento o por término; solo la lista de tokens de cada documento.

### RF-7. Índice invertido
- **RF-7.1** (ubicuo) El sistema debe construir un índice invertido que contenga el vocabulario de la colección, es decir, todos los tokens distintos tras la normalización, y para cada término la lista de documentos donde aparece, sin repeticiones y en el orden de RF-6.1.
- **RF-7.2** (evento) Cuando se consulte un término válido mediante el índice, el sistema debe devolver la lista asociada al término, o una lista vacía si el término no está en el vocabulario.
- **RF-7.3** (ubicuo) El índice debe existir solo mientras dure la ejecución y no debe guardarse en disco.
- **RF-7.4** (ubicuo) Por «ejecución» en RF-7.3 se entiende la sesión de ejecución del notebook, de principio a fin.

### RF-8. Inspección de listas y frecuencia documental
- **RF-8.1** (evento) Cuando el índice esté construido, el sistema debe mostrar el tamaño del vocabulario y, para todos los términos de prueba de RF-10, su lista de documentos y su frecuencia documental (DF, número de documentos que contienen el término).
- **RF-8.2** (evento) Cuando el índice esté construido, el sistema debe mostrar los 10 términos de mayor DF (desempate por orden alfabético ascendente) y la distribución de DF, es decir, cuántos términos del vocabulario tienen DF igual a 1, 2, y así hasta el número de documentos de la colección.
- **RF-8.3** (ubicuo) La DF de cada término del vocabulario debe ser igual al número de documentos de su lista, y la suma de todas las DF debe ser igual a la suma, sobre los documentos, de sus tokens distintos.
- **RF-8.4** (evento) Cuando se muestren los 10 términos de mayor DF (RF-8.2), el sistema debe indicar además cuántos términos del vocabulario comparten la DF máxima; el desempate alfabético debe seguir el orden de RF-1.11.

### RF-9. Consultas booleanas
- **RF-9.1** (evento) Cuando se evalúe AND entre dos operandos, el sistema debe devolver los documentos presentes en ambos.
- **RF-9.2** (evento) Cuando se evalúe OR entre dos operandos, el sistema debe devolver los documentos presentes en al menos uno.
- **RF-9.3** (evento) Cuando se evalúe NOT sobre un operando, el sistema debe devolver todos los documentos de la colección que no están en ese operando; el universo son los 18 documentos de la colección.
- **RF-9.4** (opcional) Donde un operando sea el resultado de otra operación (por ejemplo «a AND NOT b» o «(a OR b) AND NOT c»), el sistema debe evaluar primero la operación interna y usar su resultado como operando.
- **RF-9.5** (ubicuo) Toda operación booleana debe devolver identificadores sin repeticiones y en el orden de RF-6.1, y debe estar disponible tanto en la búsqueda secuencial como en el índice.
- **RF-9.6** (ubicuo) El sistema no debe interpretar consultas escritas como texto libre ni como lenguaje de consulta; las consultas compuestas se forman combinando operaciones sobre términos.
- **RF-9.7** (ubicuo) La búsqueda secuencial debe resolver cada consulta booleana evaluando cada documento una sola vez, deteniendo el recorrido de un documento en cuanto conoce la presencia de todos los términos de la consulta, sin usar el índice ni la combinación de listas del índice. Su tiempo medido incluye todo el trabajo de resolverla (RF-12.6).
- **RF-9.8** (ubicuo) La composición de operaciones (RF-9.4) es opcional; AND y OR entre términos y NOT sobre un término son obligatorios.

### RF-10. Consultas de prueba
- **RF-10.1** (ubicuo) El sistema debe clasificar los términos de prueba por frecuencia documental: **frecuente** si aparece en al menos 12 de los 18 documentos, **poco frecuente** si aparece en 1 o 2, e **inexistente** si no aparece en ninguno.
- **RF-10.2** (ubicuo) El sistema debe usar exactamente 3 términos de prueba por categoría (9 en total), elegidos y justificados en texto, y comprobar automáticamente que cada uno es un término válido (RF-5.1) y cumple el umbral de su categoría.
- **RF-10.3** (ubicuo) El sistema debe incluir, como mínimo, estas 24 consultas obligatorias:
  - una consulta de un término por cada término de prueba (9);
  - una consulta AND y una OR por cada combinación de categorías, incluida una misma categoría con dos términos distintos (6 de cada una, 12 en total);
  - una consulta NOT por cada categoría (3).

  Donde se implemente la composición de operaciones (RF-9.4), debe incluir además estas 9 consultas compuestas, hasta un total de 33:
  - una consulta «a AND NOT b» por cada par ordenado de categorías distintas (6);
  - una consulta «(a OR b) AND NOT c», con a y b de categorías distintas, por cada categoría de c (3).
- **RF-10.4** (ubicuo) Los términos de prueba deben repartirse entre las consultas de modo que: (a) cada término de prueba aparezca al menos en una consulta AND u OR; (b) ninguna consulta repita un término; (c) en «(a OR b) AND NOT c», c sea de la categoría indicada y distinto de a y de b; (d) la asignación concreta quede documentada en texto.
- **RF-10.5** (ubicuo) Los términos con DF entre 3 y 11 no pertenecen a ninguna categoría y no deben usarse como términos de prueba.
- **RF-10.6** (no deseado) Si un término de prueba deja de ser válido o de cumplir el umbral de su categoría (por ejemplo, al cambiar la configuración de normalización), entonces el sistema debe detener la ejecución con un mensaje en español que nombre el término y su DF.
- **RF-10.7** (ubicuo) El sistema debe calcular la DF de cada término de prueba con el índice y confirmarla con la búsqueda secuencial; ambas deben coincidir. La justificación de cada término debe indicar su DF y por qué representa su categoría.

### RF-11. Equivalencia de resultados
- **RF-11.1** (evento) Cuando ambos mecanismos resuelvan la misma consulta, el sistema debe comprobar que devuelven exactamente la misma lista de identificadores, en el mismo orden.
- **RF-11.2** (ubicuo) El sistema debe completar esa comprobación para todas las consultas de prueba antes de medir o comparar cualquier tiempo.
- **RF-11.3** (no deseado) Si los dos mecanismos difieren en alguna consulta, entonces el sistema debe detenerse indicando la consulta y los identificadores que difieren, sin reportar tiempos.
- **RF-11.4** (ubicuo) Dos resultados difieren si tienen distintos identificadores, repeticiones o distinto orden. Cuando difieran, el mensaje de RF-11.3 debe mostrar ambas listas completas y los identificadores presentes en una y ausentes en la otra.
- **RF-11.5** (no deseado) Si un mecanismo rechaza la consulta o falla y el otro no, entonces el sistema debe tratarlo como una diferencia y detenerse como en RF-11.3.
- **RF-11.6** (ubicuo) Además de las consultas de RF-10.3, el sistema debe comprobar la equivalencia, sin medirlas, de «a AND a» y «a OR a» sobre al menos un término de cada categoría y, donde se implemente la composición, de «a AND NOT a», «a OR NOT a» y «NOT NOT a». Debe verificar además que los resultados esperados se cumplen: «a AND a», «a OR a» y «NOT NOT a» devuelven el resultado de «a»; «a AND NOT a» devuelve vacío; «a OR NOT a» devuelve los 18 documentos.

### RF-12. Medición de tiempos
- **RF-12.1** (evento) Cuando se mida una consulta con un mecanismo, el sistema debe ejecutar la misma consulta 30 veces y calcular la media aritmética de los tiempos.
- **RF-12.2** (ubicuo) El sistema debe expresar los tiempos en milisegundos, mostrando hasta 9 decimales.
- **RF-12.3** (evento) Cuando vaya a iniciarse una medición, el sistema debe ejecutar antes 3 veces la consulta con el mismo mecanismo, en calentamiento, y esas ejecuciones no deben contarse entre las 30. La cantidad de calentamientos debe ser un valor único y documentado, reutilizable en las Partes B y C.
- **RF-12.4** (estado) Mientras se mide el tiempo de una consulta, el sistema no debe incluir la carga de archivos, la normalización o tokenización de los documentos, la normalización de los términos de consulta ni la construcción del índice.
- **RF-12.5** (ubicuo) El sistema debe medir por separado cada consulta con cada mecanismo.
- **RF-12.6** (ubicuo) El tiempo de una consulta debe incluir todo el trabajo de resolverla con el mecanismo (recorrido o consulta, combinación de resultados y obtención del universo de NOT) y no debe incluir las exclusiones de RF-12.4 ni la comprobación de equivalencia de RF-11.
- **RF-12.7** (ubicuo) El sistema no debe medir ni reportar en la Parte A el tiempo de carga, de normalización ni de construcción del índice.
- **RF-12.8** (ubicuo) El tiempo debe medirse con un reloj monótono de la mayor resolución disponible, como tiempo de pared; cada ejecución se mide por separado y la media se calcula sobre las 30 mediciones individuales.
- **RF-12.9** (ubicuo) Los tiempos deben mostrarse en milisegundos con 9 decimales fijos, con redondeo al valor más cercano.
- **RF-12.10** (ubicuo) Las mediciones deben hacerse consulta por consulta, en el orden fijo de RF-10.3: para cada una se mide primero la búsqueda secuencial (calentamiento y 30 ejecuciones) y luego el índice (calentamiento y 30 ejecuciones), sin intercalar ejecuciones de consultas distintas.
- **RF-12.11** (no deseado) Si la media de una consulta resulta 0 por la resolución del reloj, entonces el informe debe advertirlo junto a esa consulta.

### RF-13. Informe comparativo
- **RF-13.1** (evento) Cuando concluyan las mediciones, el sistema debe mostrar una tabla con una fila por consulta que indique su tipo (término único o booleana), la categoría de sus términos, el número de documentos recuperados y la media en milisegundos de la búsqueda secuencial y del índice.
- **RF-13.2** (ubicuo) El informe solo debe reportar la media de las 30 ejecuciones; no debe incluir desviación, mínimo ni máximo.
- **RF-13.3** (ubicuo) El sistema debe reportar lo medido sin exigir ni afirmar que un mecanismo sea más rápido que el otro.
- **RF-13.4** (evento) Cuando concluyan las mediciones, el sistema debe mostrar el tiempo total empleado en todas ellas.
- **RF-13.5** (ubicuo) La tabla de RF-13.1 debe incluir además una columna con la consulta escrita con sus términos reales (por ejemplo «a AND NOT b» con los términos elegidos), y la columna de tipo debe distinguir término único, AND, OR, NOT, «a AND NOT b» y «(a OR b) AND NOT c».
- **RF-13.6** (ubicuo) Para consultas con términos de categorías distintas, la columna de categoría debe listar la categoría de cada término en el orden de la consulta (por ejemplo «frecuente, poco frecuente»).
- **RF-13.7** (ubicuo) RF-13.2 limita los estadísticos de tiempo a la media; no limita las demás columnas de la tabla ni el tiempo total de RF-13.4.
- **RF-13.8** (ubicuo) RF-13.3 se aplica a la salida automática; las conclusiones que el autor redacte en celdas explicativas se permiten si se apoyan en los datos medidos.
- **RF-13.9** (ubicuo) El tiempo total de RF-13.4 debe abarcar desde el inicio del primer calentamiento hasta el fin de la última medición, incluidos los calentamientos y excluida la comprobación de RF-11.
- **RF-13.10** (evento) Cuando una consulta haya sido rechazada (RF-5.6), el informe debe incluirla marcada como «rechazada», sin tiempos.

## 5. Requisitos no funcionales

- **RNF-1. Reproducibilidad.** Con la misma colección, dos ejecuciones deben producir los mismos resultados de consulta, el mismo inventario y las mismas tablas de tamaños; solo los tiempos pueden variar.
- **RNF-2. Validez de la medición.** El tiempo de consulta debe corresponder solo a resolver la consulta; las fases de preparación se hacen aparte y no se miden en esta etapa (RF-12.7).
- **RNF-3. Verificabilidad.** Todo requisito debe poder comprobarse. Cuando su resultado es observable en un valor calculado o en el estado del notebook, la verificación debe ser automática y detener la ejecución al fallar; esto incluye los valores mostrados en RF-4.1, RF-4.5, RF-8.1, RF-8.2 y RF-13.1, cuyo formato de presentación se revisa por inspección. Se verifican por inspección documentada: RF-1.3, RF-2.2, RF-2.11, RF-6.2, RF-6.3, RF-6.6, RF-7.3, RF-9.6, la justificación de RF-10.2, RF-13.3, RNF-4, RNF-5, RNF-7 y RNF-11.
- **RNF-4. Idioma.** Todo texto mostrado, los mensajes de error y las explicaciones deben estar en español (constitución, principio 6).
- **RNF-5. Solo memoria.** Colección normalizada, índice y resultados deben existir solo en memoria y en las salidas visibles; no se persiste nada y no se descarga nada (constitución, principio 5).
- **RNF-6. Recursos.** La colección y su índice caben holgadamente en memoria; no se requiere optimizar el uso de memoria ni procesar por lotes en esta etapa.
- **RNF-7. Trazabilidad de decisiones.** Las decisiones de normalización y la elección de términos de prueba deben quedar razonadas en texto explicativo, no solo implícitas.
- **RNF-8. Reutilización.** La normalización y las operaciones de consulta deben quedar definidas de forma que las Partes B y C las usen sin modificar su comportamiento.
- **RNF-9. Eficiencia de ejecución.** La colección debe cargarse y normalizarse una sola vez, el índice construirse una sola vez y cada término de consulta normalizarse una sola vez antes de medir, aunque se use en varias consultas; ninguna etapa debe repetir trabajo ya hecho, salvo las repeticiones propias de la medición (RF-12.1 y RF-12.3). La Parte A completa, con todas las consultas de RF-10.3 y ambos mecanismos, debe terminar en no más de 5 minutos en el equipo de referencia (RNF-12), contados desde la primera celda posterior a las importaciones hasta el fin del informe, con el kernel recién iniciado y sin esperas humanas. Lo valida el autor en la primera ejecución completa; si se supera, se informa el tiempo real y una persona decide si lo acepta (ver duda 1). Nota: el tiempo lo domina la búsqueda secuencial, que es justamente lo que se compara.
- **RNF-10. Validez de la ejecución.** La ejecución de referencia es la completa y en orden, con el kernel recién iniciado. Los resultados de celdas ejecutadas fuera de orden, o tras cambiar la configuración sin repetir lo que depende de ella, no son válidos. Los resultados dependen de las versiones de Python y NLTK mostradas en RF-1.12.
- **RNF-11. Idioma ampliado.** El código y los comentarios también deben estar en español (constitución, principio 6). Los mensajes que generen las bibliotecas o el intérprete fuera del control del proyecto no entran en RNF-4; las siglas AND, OR, NOT y DF son términos técnicos admitidos.
- **RNF-12. Equipo de referencia.** El equipo de referencia es el de trabajo del autor (96 GB de RAM y Python 3.13 con las versiones del entorno del proyecto); las versiones de Python y NLTK usadas se muestran al ejecutar (RF-1.12).

## 6. Casos límite

1. **Documento sin tokens tras normalizar.** Se conserva en la colección, cuenta como documento del universo de NOT y se muestra con 0 tokens.
2. **Término con mayúsculas o tilde en la consulta** («Café»): se normaliza igual que el texto y coincide con «cafe».
3. **Colisión por quitar tildes** («año» y «ano» son el mismo término): es una consecuencia aceptada de RF-2.4 y debe quedar señalada en la documentación de la regla.
4. **Término solo de dígitos** («1815»): es válido y se busca como cualquier otro.
5. **Letras y dígitos pegados** («b52»): un único token.
6. **Apóstrofos tipográficos** (’ y ‘): equivalentes al recto; entre alfanuméricos se eliminan, en los extremos de una palabra («'hola'») separan.
7. **Guion al final de línea** (palabra partida entre dos líneas): produce dos tokens; es una limitación conocida de RF-2.9.
8. **Término con guion** («well-known») o vacío o solo símbolos: error explícito según RF-5.2, no resultado vacío.
9. **Término válido que no existe:** resultado vacío sin error; en una consulta NOT devuelve todos los documentos.
10. **Término presente en los 18 documentos:** NOT devuelve vacío.
11. **Identidades booleanas:** «a AND NOT a» devuelve vacío y «a OR NOT a» devuelve los 18 documentos, con ambos mecanismos.
12. **Operando vacío en AND/OR:** AND con vacío devuelve vacío; OR con vacío devuelve el otro operando.
13. **Mismo término como ambos operandos** («a AND a», «a OR a»): devuelve el resultado de «a».
14. **Término que aparece muchas veces en un mismo documento:** el documento aparece una sola vez.
15. **Colección ausente o con cantidad distinta de 18:** si falta, error explícito (RF-1.2); si la cantidad difiere, la ejecución se detiene (RF-4.2).
16. **Tiempos muy pequeños** (microsegundos o menos): la media en milisegundos debe conservar precisión suficiente para distinguir mecanismos (RF-12.2).
17. **Divergencia entre mecanismos:** se detiene antes de reportar ningún tiempo (RF-11.3).
18. **Letras sin equivalente base** («æ», «ø», «ß»): se conservan como letras (RF-2.5), no se descartan como símbolos.
19. **Colisiones por unir apóstrofos o por pasar a minúsculas:** «it's» y «its», «we'll» y «well», «he'll» y «hell», «I'll» y «ill», «can't» y «cant», «Will» y «will» son el mismo término. Es una consecuencia aceptada de RF-2.3 y RF-2.7 y debe quedar señalada en la documentación de la regla.
20. **Otras colisiones por quitar marcas:** «résumé» y «resume», «façade» y «facade». Consecuencia aceptada de RF-2.4 y RF-2.15.
21. **Números con puntuación interna:** «3.14» produce «3» y «14»; «1,000» produce «1» y «000»; «1:1» produce «1» y «1»; «5'10» produce «510». Son consecuencias de RF-2.6, RF-2.7 y RF-2.10; se documentan y no se corrigen.
22. **Apóstrofos en los extremos o dobles:** «dogs'» produce «dogs», «'tis» produce «tis» y «''» separa (RF-2.8 y RF-2.19).
23. **Convenciones de Gutenberg:** la raya escrita «--», el énfasis «_palabra_» y el asterisco separan (RF-2.20).
24. **Caracteres invisibles y espacios especiales:** el guion suave, el ancho cero y la marca de orden de bytes se eliminan sin partir la palabra; el espacio duro y los separadores de línea y de párrafo separan (RF-2.21 y RF-2.22).
25. **Alfabetos no latinos, URLs y correos:** las letras de cualquier alfabeto se conservan (RF-2.16); las URLs y los correos se fragmentan por sus símbolos en varios tokens.
26. **Términos con espacios internos o que no son texto** («new york», un número, `None`): inválidos según RF-5.1 y RF-5.5; los espacios al inicio y al final se ignoran.
27. **«and», «or» y «not» como términos:** términos ordinarios (RF-5.7).
28. **Operaciones booleanas extremas:** «NOT NOT a» devuelve el resultado de «a» (donde se implemente la composición); NOT de una consulta que ya devuelve los 18 documentos devuelve vacío; AND u OR con ambos operandos vacíos devuelve vacío.
29. **Un mecanismo rechaza o falla y el otro no:** es una diferencia y se detiene (RF-11.5).
30. **Reloj de baja resolución:** si la resolución del reloj es mayor que la duración de la consulta, la media puede resultar 0 y se advierte (RF-12.11); una primera ejecución más lenta queda cubierta por el calentamiento (RF-12.3).
31. **Estado del notebook y versiones:** celdas ejecutadas fuera de orden, o tras cambiar la configuración, dan resultados no válidos; una interrupción obliga a repetir desde cero; los resultados dependen de las versiones mostradas (RNF-10).
32. **Término de prueba que cambia de categoría** al cambiar la normalización: la ejecución se detiene (RF-10.6).
33. **Archivo vacío, archivo ilegible o biblioteca ausente:** archivo vacío es un documento de 0 tokens (RF-1.6); archivo ilegible detiene la ejecución nombrando el archivo (RF-1.5); biblioteca ausente detiene la ejecución (RF-1.4).

**Requisito que verifica cada caso límite:**

| Caso | Requisito |
|---|---|
| 1 | RF-1.6, RF-4.5 |
| 2 | RF-5.1 |
| 3, 20 | RF-2.4, RF-2.15 |
| 4, 5 | RF-2.6 |
| 6, 22 | RF-2.7, RF-2.8, RF-2.19 |
| 7, 23 | RF-2.9, RF-2.20 |
| 8, 26, 27 | RF-5.1, RF-5.2, RF-5.5, RF-5.6, RF-5.7 |
| 9, 10 | RF-5.3, RF-9.3 |
| 11, 13, 28 | RF-11.6 |
| 12 | RF-9.1, RF-9.2, RF-10.3 |
| 14 | RF-6.1 |
| 15, 33 | RF-1.2, RF-1.4, RF-1.5, RF-1.6, RF-4.2 |
| 16, 30 | RF-12.2, RF-12.9, RF-12.11 |
| 17, 29 | RF-11.3, RF-11.5 |
| 18, 25 | RF-2.5, RF-2.16 |
| 19, 21 | RF-2.3, RF-2.6, RF-2.7, RF-2.10 |
| 24 | RF-2.21, RF-2.22 |
| 31 | RNF-10 |
| 32 | RF-10.6 |

## 7. Fuera de alcance

- Partes B y C: corpus grande, descarga de Gutenberg, subconjuntos crecientes, gráfica de tiempo contra número de documentos y ajuste lineal con R². Los requisitos RF-2.1, RF-12.3 y RNF-8 solo piden que lo definido aquí sea reutilizable por ellas; no obligan a implementar nada de esas partes.
- Medición de memoria y almacenamiento.
- Estadísticos de dispersión de los tiempos (desviación, mínimo, máximo) y tiempo de construcción del índice.
- Reducción a raíz, palabras vacías, ranking, consultas de frase, lenguaje o analizador de consultas, corrección ortográfica y sinónimos.
- Persistencia del índice o de resultados, ejecución en paralelo, descarga, copia o extracción de datos.
- Interfaz de usuario o servicio; es un trabajo de experimentación.
- Búsqueda por subcadena o coincidencia parcial de palabras: la coincidencia es siempre de token completo (RF-6.4).
- Preguntas o decisiones humanas durante la ejecución.
- Instalar o descargar NLTK o la colección: es un paso manual previo, fuera del notebook.
- Cualquier decisión de diseño técnico, estructura de código o herramientas: corresponden al plan.

## 8. Criterios de finalización

- [ ] RF-1 a RF-13 implementados y cada criterio de aceptación comprobado por verificación automática o inspección documentada.
- [ ] Los 18 documentos cargados, con número de documentos y tamaños mostrados; si el conteo no es 18, la ejecución se detiene con error.
- [ ] La configuración de normalización, con el motivo de cada decisión y el inventario de caracteres, está documentada y se aplica por igual a documentos y consultas.
- [ ] Exactamente 3 términos de prueba por categoría (frecuente, poco frecuente, inexistente) con su DF comprobada contra el umbral, las 24 consultas obligatorias de RF-10.3 definidas y, donde se implemente la composición, las 9 compuestas.
- [ ] Ambos mecanismos devuelven listas idénticas en todas las consultas de prueba antes de cualquier tiempo.
- [ ] Cada consulta tiene su media de 30 ejecuciones en milisegundos por mecanismo, sin incluir carga ni tokenización.
- [ ] Todo el texto mostrado está en español y nada se escribió en disco.
- [ ] La Parte A completa termina en no más de 5 minutos (RNF-9) y el tiempo total de las mediciones queda a la vista.
- [ ] El notebook muestra las versiones de Python y NLTK (RF-1.12).
- [ ] Las verificaciones por inspección de RNF-3 quedan documentadas.
- [ ] Las dudas abiertas de la sección 9 están resueltas o aceptadas por una persona, con registro en esa sección.
- [ ] La spec fue aprobada por una persona antes de pasar al plan.

## 9. Dudas abiertas

1. **[NECESITA ACLARACIÓN]** Presupuesto de 5 minutos de RNF-9: es una estimación, no una medición. Debe validarse en la primera ejecución completa (responsable: el autor). Si se supera, se informa el tiempo real y una persona decide si lo acepta. **Aceptación:** pendiente de registrar (persona, fecha y tiempo real).

### Resueltas en la revisión 1 (2026-10-22)

- **Calentamiento:** 3 ejecuciones por consulta y mecanismo, como valor único y reutilizable en las Partes B y C (RF-12.3).
- **Precisión de tiempos:** milisegundos con hasta 9 decimales (RF-12.2).
- **Cobertura de consultas:** 33 consultas sistemáticas por categorías en lugar de un mínimo de cuatro (RF-10.3).
- **Orden de identificadores:** ascendente por nombre de archivo, de menor a mayor (RF-6.1, RF-7.1, RF-9.5).
- **Inspección de DF:** todos los términos de prueba, los 10 de mayor DF, la distribución de DF y la verificación de DF para todo el vocabulario (RF-8).
- **Cantidad distinta de 18 documentos:** advertencia y pregunta de continuar o detener. *Reemplazado en la revisión 2: la ejecución se detiene (RF-4.2).*

### Resueltas en la revisión 2 (2026-10-22, auditoría QA)

| Tema | Requisitos | Hallazgos |
|---|---|---|
| Cantidad de documentos | RF-4.2 (modificado), RF-4.3 y RF-4.4 (derogados), RF-9.3 y RF-10.1 (modificados) | H-16, 17, 18, 52, 59, 61, 75, 76, 77 |
| Lectura y entorno | RF-1.4 a RF-1.9, RF-1.12, RF-4.7, RNF-12, §1 | H-1, 4, 5, 20, 43, 58, 60, 72, 83 |
| Identificadores y orden | RF-1.10, RF-1.11, RF-8.4 | H-2, 3, 28, 79 |
| Alcance y clases de caracteres | RF-2.12, RF-2.13 | H-6, 7, 53 |
| Reglas de normalización | RF-2.14 a RF-2.22, casos 19 a 25 | H-8, 9, 10, 11, 12, 13, 14, 62, 63, 64, 65, 66 |
| Inventario | RF-3.3 | H-15 |
| Tokens y tamaños | RF-4.6, RF-4.7 | H-19, 20 |
| Términos de consulta | RF-5.1 (modificado), RF-5.5, RF-5.6, RF-5.7 | H-21, 22, 49, 67 |
| Búsqueda secuencial | RF-6.4 a RF-6.6, RF-9.7 | H-23, 24, 25, 26, 78 |
| Vida del índice | RF-7.4 | H-27 |
| Composición | RF-9.8, RF-10.3 (modificado) | H-50 |
| Términos de prueba | RF-10.2 (modificado), RF-10.4 a RF-10.7 | H-29, 30, 31, 32, 51, 74 |
| Equivalencia | RF-11.4 a RF-11.6 | H-33, 68, 69, 73 |
| Tiempos | RF-12.6 a RF-12.11, RNF-2 (modificado) | H-34, 35, 36, 37, 54, 70, 80 |
| Informe | RF-13.5 a RF-13.10 | H-38, 39, 40, 41, 42 |
| Verificación | RNF-3 (modificado) | H-55, 56 |
| Presupuesto y repetición | RNF-9 (modificado), §9.1 | H-44, 57, 84, 85 |
| Idioma y validez de la ejecución | RNF-10, RNF-11 | H-45, 71, 82 |
| Partes B y C | §7, RF-1.11 | H-81 |
| Trazabilidad y atomicidad | HU-8 a HU-10, tablas de trazabilidad, nota de §4 | H-47, 48, 73 |
| Contexto | §1 | H-46 |
