# Contexto del agente
Actualizado: 2026-10-24 16:34

## 1. Estado del proyecto (protegido)
- Objetivo y stack: notebook único `notebooks/Tarea2/02_booleansearch.ipynb` que compara búsqueda secuencial e índice invertido sobre libros de Gutenberg (Partes A, B y C). Python 3.13; biblioteca estándar, NLTK (solo lee la colección ya instalada), numpy, matplotlib, psutil, ipykernel. Verificación solo con `assert`; todo en español.
- Implementado y funcionando (verificado):
  - Dependencias `nltk==3.10.4` y `psutil==7.2.2` añadidas, líneas anteriores intactas | evidencia: `git diff` (3 líneas añadidas) y `import nltk` en el `.venv` | archivos: `requirements.txt`
  - Colección Gutenberg instalada fuera del repositorio (T-1) | evidencia: `gutenberg.fileids()` devuelve 18 archivos (2026-10-24) | ruta: `C:\Users\bravo\AppData\Roaming\nltk_data`
  - Documentación aprobada y en git (commits `b5a0d8e`, `eac0817`, `f3dfd4e`, `b0e0642`, `7ce7992`) | evidencia: `git log`; las 107 filas de la tabla RF de `tasks.md` coinciden con los 107 RF de la spec | archivos: `notebooks/Tarea2/docs/constitution.md`, `notebooks/Tarea2/specs/001-parte-a-corpus-pequeno/{spec,plan,tasks}.md`
  - Enunciado del notebook intacto: 9 celdas markdown y las 9 celdas de código originales, vacías | evidencia: lectura del `.ipynb` el 2026-10-24 | archivo: `02_booleansearch.ipynb`
  - Notebook, celdas nuevas verificadas: C01, C02, C03, C04, C07, C06, C05, C08, C08V, C09, C10, MD-N, C11, C12, C14, C15, MD-M, C16, C17, C20, C21, C18, C19, C22, MD-T, C13, C23, C24, C25, C26, C27, C28, C29, MD-F | evidencia: salida de las celdas V en el kernel de PyCharm (últimas tareas abajo) | archivo: `02_booleansearch.ipynb`
- Implementado sin verificar: ninguno.
- Cómo verificar: ejecutar las celdas V del notebook (los `assert` detienen la ejecución) y, al final, *Restart & Run All* de `02_booleansearch.ipynb` más las inspecciones I-1 a I-8 del plan §1.4.
- No tocar sin preguntar: constitución, spec, plan, tasks (salvo marcar `[x]`), celdas markdown del enunciado y las líneas previas de `requirements.txt`.

## 2. Progreso actual
- Trabajo en curso: Parte A, corpus pequeño (puntos 1 a 5) | spec, plan y tasks en `notebooks/Tarea2/specs/001-parte-a-corpus-pequeno/` | etapa SDD: 6, implementación (T-1 a T-44 pedidas de corrido por el usuario el 2026-10-24)
- Hecho (con evidencia):
  - Constitución aprobada y enmendada; spec revisión 2 aprobada (85 hallazgos resueltos); plan aprobado; tasks aprobado (45 tareas, 107 RF).
  - Implementación (etapa 6): hechas T-1 a T-44 (44 tareas). Últimas: T-44: Restart & Run All: intento 1 falló por referencias adelantadas a CATEGORIAS en C13 y C19 (corregido); intentos 2 y 3 pasan sin errores; las 11 huellas de resultados coinciden entre ambas ejecuciones (RNF-1); C29: 261,0 s y 81,3 s frente a 300 s (P8 cerrado, dentro del presupuesto); ninguna media en 0, reloj de 100 ns (P9 cerrado); T-43: I-1 a I-8 sin hallazgos en las 30 celdas de código (11 import solo en C01, 40 llamadas sin textos, 0 .py nuevos); RF-1.2 y RF-1.4 comprobados con el código real y la ausencia simulada en memoria; resultados registrados en MD-F; la revisión humana sigue pendiente; T-42: MD-F con la lectura de resultados (6 puntos apoyados solo en datos del informe) y la tabla de 12 verificaciones por inspección con columna de resultado pendiente de registrar
- En curso: T-45 espera la aprobación del usuario: modifica spec §9.1 y plan §9; el notebook y T-1 a T-44 están terminados
- IDs de celdas nuevas (para ubicarlas en el notebook): C01=de4ccfa2cd7af425, C02=1d583c16bcef55ae, C03=fb3387ea04c65cd9, C04=df7ac70523cbb5d1, C07=235aa9dd14e72b8b, C06=839e03d50e9033fc, C05=e62c42e3be0b6658, C08=69875a63e49ade5e, C08V=df41dc472a7919ec, C09=20edc1c7ebbe6d11, C10=83924972af68ad57, MD-N=18d893d8ba27829a, C11=799c115849eca02e, C12=dfb05a694a16440d, C14=449b08ab4e1a2e3, C15=a879f647c46d315b, MD-M=1c9cdf4da52a372a, C16=7a7c7150f4c338b2, C17=c86e740b5607126c, C20=79d3ecdc4fd2ff5b, C21=efb4bcacda2cbf1, C18=9b42a45170b88b83, C19=d3e43beec5b4819, C22=11a8f9ea66de90d3, MD-T=8720a35ab95f64f9, C13=87b4872468b28e04, C23=3f16f76848e4949c, C24=e42a2a3f8bd53328, C25=a62ef37984ef64d5, C26=f032b2d7bc59f61a, C27=2a3d3530f5dc1b99, C28=8acc9e158ee0a23, C29=22d5b59fd3e33ffe, MD-F=8fc264a5e12f07f8
- Pendiente, en orden:
  1. Implementación: solo falta `T-45` (necesita la aprobación del usuario para tocar spec §9.1 y plan §9); `T-1` a `T-44` están hechas. Ejecución final limpia el 2026-10-24: 86,6 s.
  2. Después: spec `002` (Parte B, corpus grande) y `003` (Parte C, escalabilidad), cada una con su ciclo completo.
- Decisiones tomadas y por qué:
  - Exactamente 18 documentos o la ejecución se detiene, sin preguntas al usuario (la constitución exige `assert` y la spec excluye interfaz).
  - Coincidencia por token completo, no por subcadena: un índice no puede devolver lo mismo que una búsqueda por subcadena y se rompería la comparación.
  - Consultas como árboles de tuplas, sin parser (principio 1). Composición implementada: 33 consultas.
  - Medición: 3 calentamientos y 30 ejecuciones, media en milisegundos con 9 decimales; no se mide la construcción del índice.
  - Trampa de `edit_notebook`: convierte `\n`, `\t`, `\\` y `\uXXXX` (si hay acentos en el texto) en caracteres reales. En las celdas se usa `\N{NOMBRE}`, `chr(0x…)` o `\xNN`, nunca esas secuencias. En scripts de Bash usar `python -I -X utf8` (`-I` ignora `PYTHONIOENCODING`).
  - T-44: la primera ejecución limpia destapó que C13 y C19 usaban `CATEGORIAS` (definida en C20, más abajo); la sesión interactiva lo ocultaba. Se corrigieron recorriendo `TERMINOS_PRUEBA.items()`. Esas dos celdas se tocaron aunque ya estaban verificadas: el defecto era de esta misma sesión.
  - T-44, tiempos: las dos ejecuciones de referencia dieron 261,0 s y 81,3 s para toda la Parte A (la medición sola: 221 s y 70 s). Misma máquina, distinta carga. Ambas caben en 300 s. Ninguna media en 0; el reloj resuelve 100 ns.
  - T-30: la exploración de términos se hizo con sondas sobre el kernel (no quedó celda ni función auxiliar), sin preguntar al usuario porque pidió ejecutar de corrido; es el camino que el plan propone como principal. Revisar si prefiere otro.
  - T-11: la prueba T-INV vive en la celda `C08V`, justo tras C08, y no en C07 como decía `tasks.md`, porque C07 va antes que C08 y fallaría al ejecutar en orden.
  - Las celdas nuevas se editan y ejecutan con las herramientas de notebook de PyCharm (kernel persistente); `T-1` lo ejecutó el agente porque la colección seguía ausente aunque `tasks.md` ya la marcaba hecha.
- Bloqueos y dudas abiertas:
  - Resueltos al ejecutar, falta registrarlo en plan §9 (`T-45`): P1 términos de prueba (`T-30`), P2 codificación (`T-14`; el lector declara `latin1`), P3 API de NLTK (`T-12`), P8 tiempo (261,0 s, 81,3 s y 86,6 s, todos bajo 300 s) y P9 reloj (ninguna media en 0), ambos en `T-44`.
  - `T-45` modifica spec §9.1 y plan §9: exige aprobación humana. Alguien lo marcó `[x]` en `tasks.md` antes de empezar la implementación, pero NO está hecho: spec y plan siguen sin cambios. Conviene desmarcarlo hasta que se apruebe.
  - La carpeta externa de la Parte B no se ha inspeccionado.
  - La rama activa se llama `tarea1-revml` aunque el trabajo es la Tarea 2.
- Reglas del usuario: no cambiar lo aprobado sin su aprobación; avisar si algo choca con la constitución; no hacer commit sin que lo pida; no hacer push; todo en español.
