# Parte C: escalabilidad — descripción retrospectiva

Estado al 2026-10-27: **implementada y ejecutada; revisión integral pendiente**. Este documento describe el notebook existente; no acredita una spec, un plan ni unas tareas aprobadas antes de la implementación.

## Alcance

C1 reutiliza los resultados de la consulta `water` medidos en B3 para 50, 100, 250, 500 y 1.000 documentos. Muestra el número de tokens, los tiempos medios y una gráfica con ambos mecanismos en función del número de documentos.

C2 ajusta, mediante mínimos cuadrados, una función `t(n) = an + b` a los tiempos de cada método y muestra parámetros, R² y gráficas. La función de ajuste recibe datos en memoria; una serie constante produce R² no definido. Los `assert` verifican una recta conocida y el caso constante.

## Evidencia observada

El 2026-10-27 C1 y C2 terminaron sin errores y el notebook guardó sus gráficas. El R² observado fue 0,882056 para la búsqueda secuencial y 0,715334 para el índice. Estos valores describen una ejecución, una consulta y cinco tamaños; no demuestran una complejidad asintótica ni justifican extrapolar a otros corpus.

## Pendiente

Revisar integralmente los resultados y la conformidad con el enunciado y la constitución, incluyendo la dependencia de C1 y C2 respecto a los resultados de B3. No se ha registrado aceptación humana de esta parte.
