# Spec 001: Parte A, corpus pequeño (puntos 1 a 5 del notebook)

**Estado:** aprobada por una persona el 2026-10-22 · **Fecha:** 2026-10-22 · **Etapa SDD:** 2 (especificación: qué y por qué, sin plan técnico)

## 1. Contexto y objetivo

El proyecto compara dos mecanismos de recuperación de información, la búsqueda secuencial y el índice invertido, sobre libros del Proyecto Gutenberg. Esta funcionalidad es la primera etapa: trabajar con un corpus pequeño, los 18 documentos de la colección Gutenberg de NLTK, donde se puede verificar todo a mano antes de escalar a miles de libros en las Partes B y C.

**Objetivo:** dejar demostrado, sobre esos 18 documentos, que (a) los textos se preparan con una regla de normalización única y documentada, (b) ambos mecanismos resuelven consultas de un término y booleanas (AND, OR, NOT) y devuelven exactamente los mismos documentos, y (c) se conoce el tiempo medio de cada consulta con cada mecanismo.

**Por qué importa:** la regla de normalización y las funciones de consulta se reutilizarán sin cambios en las Partes B y C. Un error o una ambigüedad aquí contaminaría todas las mediciones posteriores, y una comparación de tiempos solo es válida si ambos mecanismos devuelven lo mismo.

**Decisiones previas que condicionan esta spec:**
- La constitución fue enmendada (aprobado el 2026-10-22): la Parte A lee la colección Gutenberg de NLTK ya instalada, en solo lectura, sin descargar nada, y el id de documento es el nombre de archivo. Las reglas de la carpeta de datos con id numérico aplican solo a la Parte B.
- La máquina de trabajo dispone de 96 GB de RAM, por lo que no hay presión de memoria en esta etapa.

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

## 4. Requisitos funcionales

Notación EARS: ubicuo («El sistema debe…»), evento («Cuando…»), estado («Mientras…»), no deseado («Si… entonces…») y opcional («Donde…»).

### RF-1. Carga de la colección
- **RF-1.1** (ubicuo) El sistema debe cargar el texto original completo de cada uno de los documentos de la colección Gutenberg de NLTK, identificando cada documento por su nombre de archivo.
- **RF-1.2** (no deseado) Si la colección no está disponible en el equipo, entonces el sistema debe detenerse con un mensaje en español que indique que falta la colección y que debe instalarse manualmente, sin intentar descargarla.
- **RF-1.3** (ubicuo) El sistema no debe modificar, copiar ni escribir nada en la colección ni en el repositorio.

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

### RF-3. Inventario de caracteres
- **RF-3.1** (evento) Cuando se haya cargado la colección, el sistema debe mostrar el inventario de caracteres distintos presentes en los documentos originales, con su número de apariciones, su clase y el tratamiento que recibe según RF-2.
- **RF-3.2** (no deseado) Si existe un carácter que no pertenece a ninguna clase documentada, entonces el sistema debe listarlo como «no clasificado» y tratarlo como separador, para que pueda revisarse.

### RF-4. Conteo y tamaños
- **RF-4.1** (evento) Cuando se haya cargado la colección, el sistema debe mostrar el número de documentos.
- **RF-4.2** (no deseado) Si el número de documentos no es 18, entonces el sistema debe mostrar una advertencia en español con la cantidad hallada y la esperada, y preguntar si se desea continuar o detener la ejecución.
- **RF-4.3** (evento) Cuando la respuesta sea afirmativa y explícita, el sistema debe continuar con los documentos hallados, calcular el universo de RF-9.3 y el umbral de RF-10.1 sobre esa cantidad, y dejar constancia visible de que la colección no tiene 18 documentos en todos los resultados posteriores.
- **RF-4.4** (no deseado) Si la respuesta es negativa, no es explícitamente afirmativa o no es posible preguntar (ejecución sin entrada interactiva), entonces el sistema debe detenerse sin continuar.
- **RF-4.5** (evento) Cuando se haya normalizado la colección, el sistema debe mostrar, por documento, su identificador, su tamaño en caracteres del texto original y su número de tokens, y además los totales de documentos, caracteres y tokens.

### RF-5. Términos de consulta
- **RF-5.1** (evento) Cuando se reciba un término de consulta, el sistema debe normalizarlo con la configuración de RF-2 y obtener exactamente un token («Café» se consulta como «cafe», «1815» es válido).
- **RF-5.2** (no deseado) Si el término normalizado produce cero tokens o más de uno (por ejemplo vacío, solo símbolos como «!!!» o con guion como «well-known»), entonces el sistema debe detenerse con un mensaje en español que indique el término original y cuántos tokens produjo, sin devolver resultado alguno.
- **RF-5.3** (evento) Cuando el término sea válido pero no aparezca en ningún documento, el sistema debe devolver un resultado vacío y no un error.
- **RF-5.4** (ubicuo) Las reglas de RF-5.1 a RF-5.3 deben aplicarse igual en ambos mecanismos y a cada operando de una consulta booleana.

### RF-6. Búsqueda secuencial
- **RF-6.1** (evento) Cuando se consulte un término válido, el sistema debe devolver los identificadores de todos los documentos que lo contienen, sin repeticiones y en orden ascendente por nombre de archivo (de menor a mayor), aunque el término aparezca varias veces en un documento.
- **RF-6.2** (ubicuo) El sistema debe resolver la búsqueda recorriendo cada documento y, dentro de cada documento, cada uno de sus tokens, como exige el enunciado.
- **RF-6.3** (estado) Mientras se ejecuta una búsqueda secuencial, el sistema no debe usar el índice invertido ni ninguna otra estructura que asocie términos con documentos de forma anticipada.

### RF-7. Índice invertido
- **RF-7.1** (ubicuo) El sistema debe construir un índice invertido que contenga el vocabulario de la colección, es decir, todos los tokens distintos tras la normalización, y para cada término la lista de documentos donde aparece, sin repeticiones y en el orden de RF-6.1.
- **RF-7.2** (evento) Cuando se consulte un término válido mediante el índice, el sistema debe devolver la lista asociada al término, o una lista vacía si el término no está en el vocabulario.
- **RF-7.3** (ubicuo) El índice debe existir solo mientras dure la ejecución y no debe guardarse en disco.

### RF-8. Inspección de listas y frecuencia documental
- **RF-8.1** (evento) Cuando el índice esté construido, el sistema debe mostrar el tamaño del vocabulario y, para todos los términos de prueba de RF-10, su lista de documentos y su frecuencia documental (DF, número de documentos que contienen el término).
- **RF-8.2** (evento) Cuando el índice esté construido, el sistema debe mostrar los 10 términos de mayor DF (desempate por orden alfabético ascendente) y la distribución de DF, es decir, cuántos términos del vocabulario tienen DF igual a 1, 2, y así hasta el número de documentos de la colección.
- **RF-8.3** (ubicuo) La DF de cada término del vocabulario debe ser igual al número de documentos de su lista, y la suma de todas las DF debe ser igual a la suma, sobre los documentos, de sus tokens distintos.

### RF-9. Consultas booleanas
- **RF-9.1** (evento) Cuando se evalúe AND entre dos operandos, el sistema debe devolver los documentos presentes en ambos.
- **RF-9.2** (evento) Cuando se evalúe OR entre dos operandos, el sistema debe devolver los documentos presentes en al menos uno.
- **RF-9.3** (evento) Cuando se evalúe NOT sobre un operando, el sistema debe devolver todos los documentos de la colección que no están en ese operando; el universo son todos los documentos de la colección cargada (18 en el caso esperado).
- **RF-9.4** (opcional) Donde un operando sea el resultado de otra operación (por ejemplo «a AND NOT b» o «(a OR b) AND NOT c»), el sistema debe evaluar primero la operación interna y usar su resultado como operando.
- **RF-9.5** (ubicuo) Toda operación booleana debe devolver identificadores sin repeticiones y en el orden de RF-6.1, y debe estar disponible tanto en la búsqueda secuencial como en el índice.
- **RF-9.6** (ubicuo) El sistema no debe interpretar consultas escritas como texto libre ni como lenguaje de consulta; las consultas compuestas se forman combinando operaciones sobre términos.

### RF-10. Consultas de prueba
- **RF-10.1** (ubicuo) El sistema debe clasificar los términos de prueba por frecuencia documental: **frecuente** si aparece en al menos dos tercios de los documentos (12 de 18 en el caso esperado, redondeando hacia arriba), **poco frecuente** si aparece en 1 o 2, e **inexistente** si no aparece en ninguno.
- **RF-10.2** (ubicuo) El sistema debe usar al menos 3 términos de prueba por categoría, elegidos y justificados en texto, y comprobar automáticamente que cada uno cumple el umbral de su categoría.
- **RF-10.3** (ubicuo) El sistema debe incluir, como mínimo, estas 33 consultas (con tres términos por categoría):
  - una consulta de un término por cada término de prueba (9);
  - una consulta AND y una OR por cada combinación de categorías, incluida una misma categoría con dos términos distintos (6 de cada una, 12 en total);
  - una consulta NOT por cada categoría (3);
  - una consulta «a AND NOT b» por cada par ordenado de categorías distintas (6);
  - una consulta «(a OR b) AND NOT c», con a y b de categorías distintas, por cada categoría de c (3).

### RF-11. Equivalencia de resultados
- **RF-11.1** (evento) Cuando ambos mecanismos resuelvan la misma consulta, el sistema debe comprobar que devuelven exactamente la misma lista de identificadores, en el mismo orden.
- **RF-11.2** (ubicuo) El sistema debe completar esa comprobación para todas las consultas de prueba antes de medir o comparar cualquier tiempo.
- **RF-11.3** (no deseado) Si los dos mecanismos difieren en alguna consulta, entonces el sistema debe detenerse indicando la consulta y los identificadores que difieren, sin reportar tiempos.

### RF-12. Medición de tiempos
- **RF-12.1** (evento) Cuando se mida una consulta con un mecanismo, el sistema debe ejecutar la misma consulta 30 veces y calcular la media aritmética de los tiempos.
- **RF-12.2** (ubicuo) El sistema debe expresar los tiempos en milisegundos, mostrando hasta 9 decimales.
- **RF-12.3** (evento) Cuando vaya a iniciarse una medición, el sistema debe ejecutar antes 3 veces la consulta con el mismo mecanismo, en calentamiento, y esas ejecuciones no deben contarse entre las 30. La cantidad de calentamientos debe ser un valor único y documentado, reutilizable en las Partes B y C.
- **RF-12.4** (estado) Mientras se mide el tiempo de una consulta, el sistema no debe incluir la carga de archivos, la normalización o tokenización de los documentos, la normalización de los términos de consulta ni la construcción del índice.
- **RF-12.5** (ubicuo) El sistema debe medir por separado cada consulta con cada mecanismo.

### RF-13. Informe comparativo
- **RF-13.1** (evento) Cuando concluyan las mediciones, el sistema debe mostrar una tabla con una fila por consulta que indique su tipo (término único o booleana), la categoría de sus términos, el número de documentos recuperados y la media en milisegundos de la búsqueda secuencial y del índice.
- **RF-13.2** (ubicuo) El informe solo debe reportar la media de las 30 ejecuciones; no debe incluir desviación, mínimo ni máximo.
- **RF-13.3** (ubicuo) El sistema debe reportar lo medido sin exigir ni afirmar que un mecanismo sea más rápido que el otro.
- **RF-13.4** (evento) Cuando concluyan las mediciones, el sistema debe mostrar el tiempo total empleado en todas ellas.

## 5. Requisitos no funcionales

- **RNF-1. Reproducibilidad.** Con la misma colección, dos ejecuciones deben producir los mismos resultados de consulta, el mismo inventario y las mismas tablas de tamaños; solo los tiempos pueden variar.
- **RNF-2. Validez de la medición.** El tiempo de consulta debe corresponder solo a resolver la consulta; las fases de preparación se hacen y se miden aparte.
- **RNF-3. Verificabilidad.** Cada requisito de esta spec debe poder comprobarse con una verificación automática que detenga la ejecución al fallar, sin depender de inspección visual, salvo la decisión humana prevista en RF-4.2.
- **RNF-4. Idioma.** Todo texto mostrado, los mensajes de error y las explicaciones deben estar en español (constitución, principio 6).
- **RNF-5. Solo memoria.** Colección normalizada, índice y resultados deben existir solo en memoria y en las salidas visibles; no se persiste nada y no se descarga nada (constitución, principio 5).
- **RNF-6. Recursos.** La colección y su índice caben holgadamente en memoria; no se requiere optimizar el uso de memoria ni procesar por lotes en esta etapa.
- **RNF-7. Trazabilidad de decisiones.** Las decisiones de normalización y la elección de términos de prueba deben quedar razonadas en texto explicativo, no solo implícitas.
- **RNF-8. Reutilización.** La normalización y las operaciones de consulta deben quedar definidas de forma que las Partes B y C las usen sin modificar su comportamiento.
- **RNF-9. Eficiencia de ejecución.** La colección debe cargarse y normalizarse una sola vez, el índice construirse una sola vez y cada término de consulta normalizarse una sola vez antes de medir; ninguna etapa debe repetir trabajo ya hecho. La Parte A completa, con las 33 consultas y ambos mecanismos, debe terminar en no más de 5 minutos en el equipo de trabajo (ver duda 1). El tiempo lo domina la búsqueda secuencial, que es justamente lo que se compara.

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
15. **Colección ausente o con cantidad distinta de 18:** si falta, error explícito (RF-1.2); si la cantidad difiere, advertencia y decisión humana (RF-4.2 a RF-4.4).
16. **Tiempos muy pequeños** (microsegundos o menos): la media en milisegundos debe conservar precisión suficiente para distinguir mecanismos (RF-12.2).
17. **Divergencia entre mecanismos:** se detiene antes de reportar ningún tiempo (RF-11.3).
18. **Letras sin equivalente base** («æ», «ø», «ß»): se conservan como letras (RF-2.5), no se descartan como símbolos.

## 7. Fuera de alcance

- Partes B y C: corpus grande, descarga de Gutenberg, subconjuntos crecientes, gráfica de tiempo contra número de documentos y ajuste lineal con R².
- Medición de memoria y almacenamiento.
- Estadísticos de dispersión de los tiempos (desviación, mínimo, máximo) y tiempo de construcción del índice.
- Reducción a raíz, palabras vacías, ranking, consultas de frase, lenguaje o analizador de consultas, corrección ortográfica y sinónimos.
- Persistencia del índice o de resultados, ejecución en paralelo, descarga, copia o extracción de datos.
- Interfaz de usuario o servicio; es un trabajo de experimentación.
- Cualquier decisión de diseño técnico, estructura de código o herramientas: corresponden al plan.

## 8. Criterios de finalización

- [ ] RF-1 a RF-13 implementados y cada criterio de aceptación comprobado por verificación automática o inspección documentada.
- [ ] Los 18 documentos cargados, con número de documentos y tamaños mostrados; si el conteo no es 18, se advierte y se pregunta si continuar o detener.
- [ ] La configuración de normalización, con el motivo de cada decisión y el inventario de caracteres, está documentada y se aplica por igual a documentos y consultas.
- [ ] Al menos 3 términos de prueba por categoría (frecuente, poco frecuente, inexistente) con su DF comprobada contra el umbral, y las 33 consultas de RF-10.3 definidas.
- [ ] Ambos mecanismos devuelven listas idénticas en todas las consultas de prueba antes de cualquier tiempo.
- [ ] Cada consulta tiene su media de 30 ejecuciones en milisegundos por mecanismo, sin incluir carga ni tokenización.
- [ ] Todo el texto mostrado está en español y nada se escribió en disco.
- [ ] La Parte A completa termina en no más de 5 minutos y el tiempo total de las mediciones queda a la vista.
- [ ] Las dudas abiertas de la sección 9 están resueltas o aceptadas por una persona.
- [ ] La spec fue aprobada por una persona antes de pasar al plan.

## 9. Dudas abiertas

1. **[NECESITA ACLARACIÓN]** El presupuesto de 5 minutos de RNF-9 es una estimación mía, no una medición: la búsqueda secuencial recorre del orden de millones de tokens por consulta, y las 33 consultas con 3 calentamientos y 30 ejecuciones suman unos miles de recorridos. Debe validarse al ejecutar y ajustarse si el equipo resulta más lento o más rápido.

### Resueltas en la revisión 1 (2026-10-22)

- **Calentamiento:** 3 ejecuciones por consulta y mecanismo, como valor único y reutilizable en las Partes B y C (RF-12.3).
- **Precisión de tiempos:** milisegundos con hasta 9 decimales (RF-12.2).
- **Cobertura de consultas:** 33 consultas sistemáticas por categorías en lugar de un mínimo de cuatro (RF-10.3).
- **Orden de identificadores:** ascendente por nombre de archivo, de menor a mayor (RF-6.1, RF-7.1, RF-9.5).
- **Inspección de DF:** todos los términos de prueba, los 10 de mayor DF, la distribución de DF y la verificación de DF para todo el vocabulario (RF-8).
- **Cantidad distinta de 18 documentos:** advertencia y pregunta de continuar o detener; se detiene si no hay respuesta afirmativa explícita (RF-4.2 a RF-4.4).
