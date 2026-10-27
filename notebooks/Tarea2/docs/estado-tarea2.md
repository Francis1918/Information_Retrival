# Estado de la Tarea 2

Actualizado: 2026-10-27.

El notebook `../02_booleansearch.ipynb` contiene las partes A, B y C. Una ejecución completa desde un kernel nuevo terminó sin errores y dejó las tablas y dos gráficas en sus salidas. **Estado: implementado y ejecutado; cierre formal pendiente.**

| Parte | Qué está implementado | Evidencia y alcance |
|---|---|---|
| A | Normalización, búsqueda secuencial, índice invertido y comparación de 33 consultas sobre 18 libros de NLTK. | Celdas C01–C29 y sus salidas. C29 mostró 74,5 s en la ejecución del 2026-10-27. La auditoría anterior de la spec 001 registró PEND-3, aún sin resolución demostrada. |
| B | Inventario de archivos, tres consultas y subconjuntos de 50 a 1.000 libros. | B1 registró 78.107 documentos válidos; B3 comprobó resultados iguales para cada consulta y tamaño y guardó los tiempos y recursos observados. |
| C | Gráfica tiempo/documentos para `water`, ajustes lineales y R². | C1 y C2 guardaron ambas gráficas; R² de esa ejecución: 0,882056 secuencial y 0,715334 índice. |

La ejecución completa tomó 740,1 s y el mayor subconjunto llegó a una RSS de 4.917,38 MiB. Es la memoria total observada del proceso con el índice, no el tamaño exclusivo del índice. Los tiempos cambian entre ejecuciones.

## Pendientes

- Resolver el hallazgo PEND-3 de `../specs/001-parte-a-corpus-pequeno/validation.md`: la implementación repite normalizaciones y búsquedas que RNF-9 pide no repetir.
- T-45: el usuario confirmó su finalización el 2026-10-27. No se han documentado el nombre, la fecha ni el tiempo de la aceptación original; la confirmación no permite deducirlos.
- Resolver la discrepancia con el principio 3 de `constitution.md`: B3 reúne en una celda la lectura de documentos y las mediciones de recursos, aunque el principio pide celdas separadas.
- Realizar una revisión integral de las partes B y C contra el enunciado y la constitución. Sus documentos en `../specs/002-parte-b-corpus-grande/` y `../specs/003-parte-c-escalabilidad/` son descripciones retrospectivas, no evidencias de aprobación previa.

La ruta local del corpus grande está en C02; su colección se usa en modo de solo lectura y debe existir antes de ejecutar B1. La guía de continuidad está en `../Context/contexto-tarea2.md`.
