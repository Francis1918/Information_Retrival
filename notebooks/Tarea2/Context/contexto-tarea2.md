# Contexto de la Tarea 2

Actualizado: 2026-10-27.

## Objetivo y estado

El notebook único `notebooks/Tarea2/02_booleansearch.ipynb` compara búsqueda secuencial e índice invertido con libros del Proyecto Gutenberg. Las partes A, B y C están implementadas y hay una ejecución completa guardada sin errores de celda. **La ejecución correcta no equivale al cierre formal**: siguen abiertos los hallazgos y la revisión humana que se detallan abajo.

- Parte A: corpus Gutenberg de NLTK con 18 documentos, normalización, búsquedas, índice y 33 consultas.
- Parte B: inventario de la colección externa, tres consultas y cinco subconjuntos crecientes.
- Parte C: gráfica de tiempos, ajuste lineal y R² para ambos métodos.
- Python 3.13; biblioteca estándar, NLTK, numpy, matplotlib, psutil e ipykernel. Los resultados, el índice y las gráficas viven en memoria y en las salidas del notebook; no se descargan ni modifican los datos desde el notebook.

## Datos y configuración

- La Parte A usa la colección Gutenberg ya instalada para NLTK; `RUTA_NLTK_DATA` controla su búsqueda.
- La Parte B usa `RUTA_CORPUS_GRANDE`, definida en C02: `C:\Users\bravo\PycharmProjects\DataRI\Tarea2\txt-files.tar\cache\epub`. Es una colección previamente extraída, fuera del repositorio y de solo lectura. Una carpeta numérica corresponde a un libro; se exige exactamente un `pg<N>.txt` que coincida con el número de carpeta y se registran las carpetas excluidas.
- Los identificadores de B son enteros y los subconjuntos se ordenan numéricamente. `TAMANOS_CORPUS = (50, 100, 250, 500, 1000)`; se usan los primeros documentos válidos, de forma anidada.
- Todas las mediciones usan 3 calentamientos y 30 repeticiones por consulta y mecanismo; no incluyen carga, tokenización ni construcción del índice. El reloj y la carga del equipo afectan los tiempos.

## Evidencia de ejecución

El 2026-10-27 se ejecutó el notebook completo desde un kernel nuevo con `nbclient` y se guardaron sus salidas: 35 celdas de código ejecutadas, sin errores guardados, incluidas las comprobaciones con `assert`. Duración total observada: 740,1 s; C29 informó 74,5 s para la Parte A. Los tiempos dependen del equipo y no son garantías.

- B1 inspeccionó 78.107 carpetas numéricas y registró 78.107 documentos válidos y 0 carpetas excluidas.
- B3 verificó igualdad de resultados antes de medir, para las tres consultas en cada tamaño. El subconjunto de 1.000 documentos tuvo 81.718.278 tokens y 460,10 MiB en archivos TXT; la RSS del proceso con índice fue 4.917,38 MiB. Es memoria del proceso, no tamaño exacto del índice.
- C1 y C2 guardaron sus dos gráficas. En esa ejecución, para `water`, los R² fueron 0,882056 (secuencial) y 0,715334 (índice). Son observaciones de una ejecución, no propiedades universales ni evidencia para extrapolar.
- El detalle y el alcance de las verificaciones de la Parte A se conservan en `specs/001-parte-a-corpus-pequeno/validation.md`. Ese informe auditó el commit `4686821` el 2026-10-24; no es una auditoría integral posterior de las partes B y C.

## Pendientes de cierre

1. La auditoría de la Parte A rechazó la entrega en su momento: PEND-3 indica que RNF-9 exige normalizar cada término una sola vez y evitar trabajo repetido, pero las celdas C13, C22, C25 y C26 repiten normalizaciones y búsquedas. La ejecución posterior sin errores no demuestra que se haya corregido. Revisar y resolver antes de declarar conformidad con la spec 001.
2. T-45 está terminada según confirmó el usuario el 2026-10-27; P1, P2, P3, P8 y P9 tienen evidencia técnica registrada en el plan. La confirmación no especificó el nombre de quien aceptó, ni la fecha o el tiempo de la aceptación original; no se deben inventar esos datos. PEND-3 es un hallazgo distinto y sigue sin resolución demostrada.
3. El principio 3 de `docs/constitution.md` pide que lectura de documentos, medición de memoria/disco y gráficas estén en celdas separadas. En B3 la lectura y las mediciones de recursos están en la misma celda; debe corregirse o resolverse con el usuario antes de declarar conformidad. Las gráficas están en C1 y C2.
4. Las notas retrospectivas de B y C en `specs/002-parte-b-corpus-grande/` y `specs/003-parte-c-escalabilidad/` describen lo realizado; no representan un ciclo SDD previo ni una aprobación humana inventada. Falta una revisión integral de B y C contra los requisitos y la constitución.

## Ejecución y reglas de trabajo

Ejecutar las celdas en orden desde un kernel nuevo. C27 hace las repeticiones de A; B3 hace las de B para los cinco tamaños; C1 y C2 reutilizan los resultados de B3. Cada una de esas celdas se ejecuta una sola vez por recorrido. Las salidas guardadas pueden inspeccionarse sin volver a ejecutar el experimento largo.

El usuario pidió trabajar en español, no hacer commit sin que lo solicite y no hacer push. Los documentos aprobados solo se cambian con su aprobación; el 2026-10-27 autorizó actualizarlos y confirmó que T-45 ya estaba terminada. La rama activa observada era `tarea1-revml`.
