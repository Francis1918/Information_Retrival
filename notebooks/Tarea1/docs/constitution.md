# Constitución — 01_revml.ipynb

Principios innegociables: un cambio que viole cualquiera de ellos se rechaza.

1. **Stack fijo.** Solo numpy==2.5.3, pandas==3.0.6, scikit-learn==1.9.1, scipy==1.18.1, matplotlib==3.11.2, seaborn==0.13.2, joblib==1.6.0, notebook e ipykernel==7.4.0; otra dependencia requiere justificación escrita y aprobación. Verificable: todo `import` pertenece a este conjunto o a la biblioteca estándar.
2. **La spec es la ley.** Cada experimento responde a un requisito de la spec y consta de tres celdas seguidas: Markdown con la predicción inicial, código con la implementación y Markdown con la interpretación y las respuestas. Verificable: no hay experimento sin requisito ni requisito sin experimento, y cada uno tiene las tres celdas en ese orden.
3. **Lógica separada de la interfaz.** Toda la implementación vive en `01_revml.ipynb` (sin `.py`, paquetes ni scripts); la lógica son funciones que no imprimen ni grafican, y mostrar resultados ocurre solo fuera de ellas. Verificable: ninguna función contiene `print`, `display` ni `plt.`.
4. **Tests y reproducibilidad.** Cada función de lógica tiene al menos un `assert` en el notebook (sin pytest); semillas fijadas, particiones train/validación/prueba declaradas antes de entrenar y configuración congelada antes de evaluar en prueba, que nunca se usa para ajustar. Verificable: «Restart & Run All» termina sin errores y repite los mismos resultados.
5. **Sin persistencia externa.** El notebook es la única fuente de verdad: los datasets se cargan desde las librerías del stack y las tablas y figuras se generan de los resultados obtenidos. Verificable: ninguna llamada a `open`, `read_csv`, `to_csv`, `savefig` ni `joblib.dump`.
6. **Idioma dividido.** Código, nombres de variables y comentarios técnicos en inglés; celdas Markdown, explicaciones y documentación en español. Verificable: ningún identificador ni comentario está en español y ninguna celda Markdown está en inglés (salvo términos técnicos).
