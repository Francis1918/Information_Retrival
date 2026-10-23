# Contexto del agente
Actualizado: 2026-10-22 22:37

## 1. Estado del proyecto (protegido)
- Objetivo y stack: notebook único `notebooks/Tarea2/02_booleansearch.ipynb` que compara búsqueda secuencial e índice invertido sobre libros de Gutenberg (Partes A, B y C). Python 3.13; biblioteca estándar, NLTK (solo lee la colección ya instalada), numpy, matplotlib, psutil, ipykernel. Verificación solo con `assert`; todo en español.
- Implementado y funcionando (verificado):
  - Dependencias `nltk==3.10.4` y `psutil==7.2.2` añadidas, líneas anteriores intactas | evidencia: `git diff` (3 líneas añadidas) y `pip install --dry-run` resolvió nltk | archivos: `requirements.txt`
  - Documentación aprobada y en git (commits `b5a0d8e`, `eac0817`, `f3dfd4e`, `b0e0642`, `7ce7992`) | evidencia: `git log`; las 107 filas de la tabla RF de `tasks.md` coinciden con los 107 RF de la spec y ninguna tarea depende de otra posterior | archivos: `notebooks/Tarea2/docs/constitution.md`, `notebooks/Tarea2/specs/001-parte-a-corpus-pequeno/{spec,plan,tasks}.md`
  - Enunciado del notebook intacto: 9 celdas markdown y 9 celdas de código, todas vacías | evidencia: lectura del `.ipynb` el 2026-10-22 | archivo: `02_booleansearch.ipynb`
- Implementado sin verificar: ninguno (no hay código escrito).
- Cómo verificar: hoy, `git status` y `git log`. Tras implementar: *Restart & Run All* de `02_booleansearch.ipynb` (los `assert` de las celdas V detienen la ejecución) y las inspecciones I-1 a I-8 del plan §1.4.
- No tocar sin preguntar: constitución, spec, plan, tasks, celdas markdown del enunciado y las líneas previas de `requirements.txt`.

## 2. Progreso actual
- Trabajo en curso: Parte A, corpus pequeño (puntos 1 a 5) | spec, plan y tasks en `notebooks/Tarea2/specs/001-parte-a-corpus-pequeno/` | etapa SDD: etapas 1 a 5 aprobadas (constitución, spec, auditoría QA, plan, tareas); la 6, implementación, **no ha empezado**.
- Hecho (con evidencia):
  - Constitución aprobada y enmendada (se añadió la colección NLTK para la Parte A).
  - Spec revisión 2 aprobada: 85 hallazgos de la auditoría QA resueltos (tabla en spec §9).
  - Plan aprobado: bloques de celdas `C01`–`C29`, 39 funciones, 22 decisiones técnicas.
  - Tasks aprobado: 45 tareas (`T-1` a `T-45`), 107 RF cubiertos; RF-4.3 y RF-4.4 derogados.
  - Evidencia: encabezados «Aprobado» de plan y tasks, y `git log`.
- En curso: nada a medias. Lo único sin guardar en git es esta carpeta `notebooks/Tarea2/Context/` (`git status` el 2026-10-22).
- Pendiente, en orden:
  1. `T-1` (lo hace el usuario): `! .venv\Scripts\pip install -r requirements.txt` y `! .venv\Scripts\python -m nltk.downloader gutenberg`.
  2. `T-2` a `T-45` en el orden de `tasks.md` (empieza por `C01`/`C02`, tras la celda markdown «1. Preparación del corpus»).
  3. Después: spec `002` (Parte B, corpus grande) y `003` (Parte C, escalabilidad), cada una con su ciclo completo.
- Decisiones tomadas y por qué:
  - Exactamente 18 documentos o la ejecución se detiene, sin preguntas al usuario (la constitución exige `assert` y la spec excluye interfaz).
  - Coincidencia por token completo, no por subcadena: un índice no puede devolver lo mismo que una búsqueda por subcadena y se rompería la comparación.
  - Consultas como árboles de tuplas, sin parser (principio 1). Composición implementada: 33 consultas.
  - Medición: 3 calentamientos y 30 ejecuciones, media en milisegundos con 9 decimales; no se mide la construcción del índice.
  - Ninguna tarea toca código existente porque no lo hay; el enunciado se conserva.
- Bloqueos y dudas abiertas:
  - NLTK y la colección **no están instalados** (comprobado en el `.venv` y en `nltk_data`).
  - Pendientes del plan que se cierran ejecutando: P1 términos de prueba (`T-30`, el usuario elige el método de exploración), P2 codificación (`T-14`), P3 API de NLTK (`T-12`), P8 y P9 tiempo y reloj (`T-44`).
  - Spec §9.1: el presupuesto de 5 minutos está sin aceptar; `T-45` modifica spec y plan y exige aprobación.
  - La carpeta externa de la Parte B no se ha inspeccionado.
  - La rama activa se llama `tarea1-revml` aunque el trabajo es la Tarea 2.
- Reglas del usuario: detenerse tras cada etapa y esperar aprobación; no escribir código ni ejecutar tareas hasta que lo pida; no cambiar lo aprobado sin su aprobación; avisar si algo choca con la constitución; no hacer commit sin que lo pida; todo en español.
