# Parte B: corpus grande — descripción retrospectiva

Estado al 2026-10-27: **implementada y ejecutada; revisión integral pendiente**. Este documento describe el notebook existente; no acredita una spec, un plan ni unas tareas aprobadas antes de la implementación.

## Alcance

El notebook lee una colección previamente extraída fuera del repositorio, configurada en `RUTA_CORPUS_GRANDE` de C02. B1 inspecciona las carpetas numéricas y acepta un libro cuando existe exactamente un archivo `pg<N>.txt` cuyo número coincide con la carpeta. Registra las carpetas que no cumplen y evita ids numéricos duplicados. No descarga, extrae ni modifica datos.

B2 prepara `water`, `water AND king` y `macbeth OR ahab`, y comprueba las operaciones booleanas con identificadores numéricos. B3 selecciona subconjuntos anidados de 50, 100, 250, 500 y 1.000 documentos en orden numérico, normaliza con la función de A, construye un índice para cada tamaño y verifica igualdad de resultados antes de medir.

Para cada consulta y mecanismo se hacen 3 calentamientos y 30 ejecuciones medidas. La tabla registra documentos, tokens, coincidencias, medias, tamaño total de los TXT y RSS del proceso antes y después de construir el índice. La RSS es aproximada y no equivale al tamaño exacto del índice. La lectura usa UTF-8 con o sin BOM y, cuando falla, Latin-1; registra los ids que requieren esa alternativa.

## Evidencia observada

En la ejecución completa del 2026-10-27, B1 registró 78.107 documentos válidos y ninguna carpeta excluida; B3 terminó sin errores en los cinco tamaños. Con 1.000 libros se contaron 81.718.278 tokens y 460,10 MiB de TXT; la RSS con índice fue 4.917,38 MiB. Ningún archivo procesado requirió Latin-1. Las cifras y tiempos dependen de los datos y del equipo.

## Pendiente

El principio 3 de la constitución pide separar en celdas la lectura y la medición de memoria y disco; B3 las reúne. Se necesita resolver esta discrepancia y revisar la Parte B de forma integral. Una ejecución sin errores no sustituye esa revisión ni una aceptación humana.
