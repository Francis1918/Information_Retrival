# Tareas — Spec 001: Revisión práctica de Machine Learning

| | |
|---|---|
| **Estado** | 53 de 53 tareas completadas; T52 y T53 son revisiones hechas por la propia IA, no por una persona independiente — ver «Registro de ejecución» |
| **Fecha** | 2026-10-13 |
| **Entradas** | `spec.md` y `plan.md` de esta carpeta; constitución en `../docs/constitution.md` |
| **Notebook** | `notebooks/Tarea1/01_revml.ipynb` (`../01_revml.ipynb` desde esta carpeta) |
| **Alcance de este documento** | El ORDEN y el tamaño del trabajo. El diseño está en `plan.md`: las referencias `§`, `T1.1…T3.12` (tablas), `F1.1…F3.4` (figuras) y `DT-n` remiten a él. |

## Cómo usar este documento

1. **El orden es el de dependencia.** Una tarea solo se empieza cuando las anteriores están marcadas. Se marca `[x]` únicamente cuando se cumple su «Hecho cuando».
2. **Toda tarea de código termina con *Restart & Run All* sin errores.** No se repite en cada línea.
3. **✍ = redacta el autor, con sus palabras.** Son las predicciones (`E*.P`) y las interpretaciones (`E*.I`): la práctica exige poder explicarlas (spec §1, HU-6). La IA puede ayudar con código, no sustituir la predicción.
4. **Cada `E*.P` se commitea antes de ejecutar su `E*.C` y no se edita después** (CA-6.2). El historial de git es la evidencia.
5. **`RUN_TEST = False` hasta que el congelado esté commiteado.** Después, `INITIAL`, `ALTERNATIVES` y las rejillas no se tocan, y la tabla de prueba nunca motiva una nueva ronda de ajuste (CA-3.4, CA-3.6, DT-9).
6. **Las funciones viven solo en S2–S5**, sin `print`, `display` ni `plt.`, y cada una nace con su `assert` en S6 (RNF-5, DT-10). Las celdas `E*.C` no definen funciones.

**IDs de celda:** `C0` y `B1–B3` son las celdas existentes del enunciado; `S0–S6` es la sección de soporte; `E1`, `E2`, `E3` son los experimentos con sus tres celdas `P` (predicción), `C` (código) e `I` (interpretación). `[E2.k]` es un banner dentro de la única celda `E2.C` (DT-2): las tareas de implementación añaden banners, no celdas.

**Tiempos:** estimaciones de trabajo concentrado, ninguna supera 30 min. Si una tarea se alarga, se divide antes de seguir. Total estimado: ≈ 19 h 45 min.

---

## Fase 0 — Esqueleto y soporte (S0–S6) · ≈ 5 h 15 min

- [x] **T01 · Esqueleto de 20 celdas** — ~20 min
  - **Cubre:** RF-1 (CA-1.1, CA-1.4) · RNF-2 · RNF-6
  - **Qué:** En `01_revml.ipynb`, reemplazar las celdas de relleno por las 20 celdas de `plan.md` §1.1 en orden (`C0`, `S0–S6`, `B1`, `E1.P/C/I`, `B2`, `E2.P/C/I`, `B3`, `E3.P/C/I`), cada una con un encabezado con su ID y nada más; `C0`, `B1`, `B2` y `B3` se conservan tal cual.
  - **Hecho cuando:** el notebook tiene 20 celdas en ese orden (`S0` y todas las `E*.P`/`E*.I` en Markdown; `S1–S6` y `E*.C` en código); no queda `# Aquí cumple con el primer paso` ni ninguna celda vacía original; cada trío `P → C → I` es consecutivo y va justo después de su enunciado.
  - **Evidencia:** Notebook con 20 celdas en el orden de `plan.md` §1.1; relleno original reemplazado; C0, B1, B2 y B3 intactos.

- [x] **T02 · S0 y S1: soporte y constantes** — ~20 min
  - **Cubre:** RF-1 (CA-1.4) · RF-2 (CA-2.5) · RNF-3
  - **Qué:** `S0` (Markdown): qué contiene el soporte, que no es un experimento y el mapa de funciones de `plan.md` §1.3. `S1` (código): todos los `import`, `SEED = 42`, `MAX_CONFIGS = 4`, claves y orden de modelos (CA-10.1, CA-18.1) y etiquetas en español.
  - **Hecho cuando:** `S1` ejecuta sin error; una búsqueda de líneas `import`/`from … import` en el `.ipynb` solo encuentra `S1` y todas son del stack fijado o de la biblioteca estándar; `SEED` se define una sola vez.
  - **Evidencia:** S1 ejecuta; los `import` están solo en S1 (stack fijado más `hashlib`, `warnings` y `typing`); `SEED = 42` se define una vez.

- [x] **T03 · S2 `make_splits`** — ~20 min
  - **Cubre:** RF-2 (CA-2.1, CA-2.2, CA-2.3, CA-2.5) · RNF-5
  - **Qué:** `make_splits(y, stratify, seed)` con dos `train_test_split` encadenados (80/20 y 75/25 del resto, DT-4) y sus `assert` en `S6`.
  - **Hecho cuando:** con `y = [0]*50 + [1]*50 + [2]*50` y `stratify=True` los `assert` verifican tamaños 90/30/30, índices disjuntos cuya unión es todo, 30 por clase en entrenamiento y 10 por clase en validación y prueba, misma semilla → mismos índices y otra semilla → distintos; con 442 etiquetas y `stratify=False`, tamaños 264/89/89.
  - **Evidencia:** S6: tamaños 90/30/30 y 264/89/89, 30 por clase en entrenamiento y 10 en validación y prueba, misma semilla → mismos índices; asserts pasan.

- [x] **T04 · S2 `split_summary` y `sample_examples`** — ~20 min
  - **Cubre:** RF-2 (CA-2.4) · RF-8 (CA-8.2) · RNF-5
  - **Qué:** `split_summary(y, splits, class_names)` → DataFrame con tamaño y conteo por clase; `sample_examples(y, n, seed)` → `n` índices con todas las clases presentes; ambas con `assert` en `S6`.
  - **Hecho cuando:** sobre las particiones sintéticas de T03, `split_summary` devuelve 90/30/30 y el conteo por clase esperado; `sample_examples(y, 5, SEED)` devuelve 5 índices únicos con las 3 clases y es idéntico en dos llamadas.
  - **Evidencia:** S6: `split_summary` da 90/30/30 y 30/10/10 por clase; `sample_examples(y, 5, SEED)` → 5 índices únicos con las 3 clases y determinista.

- [x] **T05 · S3 `build_classifier`** — ~25 min
  - **Cubre:** RF-10 (CA-10.1, CA-10.2, CA-10.4) · RF-4 (CA-4.1) · CA-2.6 · RNF-5
  - **Qué:** fábrica de los siete clasificadores de CA-10.1 (`GaussianNB`, regresión logística, `LinearSVC` por DT-14, árbol, Random Forest, Gradient Boosting, MLP); con `scaled=True` devuelve `Pipeline(StandardScaler, modelo)`; la semilla se inyecta como `random_state=SEED` y no forma parte de `config`; `assert` en `S6`.
  - **Hecho cuando:** los `assert` verifican: 7 claves en el orden de CA-10.1; con `scaled=True` el primer paso del `Pipeline` es `StandardScaler` y con `scaled=False` no hay `StandardScaler`; `random_state == SEED` en todo modelo que lo admite; ningún estimador crea partición interna (`early_stopping` falso en el MLP y `n_iter_no_change is None` en Gradient Boosting; el 10 por defecto del MLP no crea partición, comprobado en scikit-learn 1.9.1).
  - **Evidencia:** S6: 7 claves; `Pipeline` con/sin `StandardScaler`; `random_state == SEED`; `LinearSVC`; MLP con `early_stopping` falso y Gradient Boosting con `n_iter_no_change is None`.

- [x] **T06 · S3 `build_regressor`** — ~25 min
  - **Cubre:** RF-18 (CA-18.1, CA-18.2) · RF-4 (CA-4.2, CA-4.3) · CL-12 · RNF-5
  - **Qué:** fábrica de los ocho regresores de CA-18.1; `Pipeline` con `StandardScaler` en los modelos sensibles a la escala (declarar cuáles en un comentario); `TransformedTargetRegressor(StandardScaler)` solo en el MLP (DT-12); `assert` en `S6`.
  - **Hecho cuando:** los `assert` verifican: 8 claves en el orden de CA-18.1; kNN, Ridge, Lasso y MLP llevan `StandardScaler`; solo el MLP envuelve el objetivo; con un `y` sintético ≈ 1000 las predicciones del MLP quedan en escala original (media ≈ 1000, no ≈ 0); mismas comprobaciones de partición interna que en T05.
  - **Evidencia:** S6: 8 claves; scaler en Ridge, Lasso, kNN y MLP; solo el MLP envuelve el objetivo y sus predicciones quedan en escala original (≈ 1000).

- [x] **T07 · S4 `fit_capturing_warnings` y `score_classifier`** — ~25 min
  - **Cubre:** RF-7 (CA-7.1) · RF-11 (CA-11.1) · RF-12 (CA-12.1, CA-12.3) · RNF-5
  - **Qué:** `fit_capturing_warnings(est, X, y)` → (estimador ajustado, avisos) con `warnings.catch_warnings(record=True)` y `simplefilter("always")` (DT-7); `score_classifier(est, X_eval, y_eval)` → predicciones, accuracy, F1 macro, F1 micro y avisos; `assert` en `S6`.
  - **Hecho cuando:** los `assert` verifican que `LogisticRegression(max_iter=1)` sobre datos sintéticos devuelve ≥ 1 `ConvergenceWarning` capturado con el estimador ajustado, y que con `y=[0,0,1,1,2,2]`, `pred=[0,0,1,2,2,2]` el F1 macro ≈ 0.8222 y el F1 micro = accuracy = 5/6.
  - **Evidencia:** S6: `LogisticRegression(max_iter=1)` devuelve `ConvergenceWarning` capturado; F1 macro ≈ 0.8222 y F1 micro = accuracy = 5/6.

- [x] **T08 · S4 `score_regressor` y `select_config`** — ~25 min
  - **Cubre:** RF-3 (CA-3.1, CA-3.7) · RF-19 (CA-19.1) · CL-6 · RNF-5
  - **Qué:** `score_regressor` → predicciones, MAE, RMSE, R² y avisos; `select_config(records, primary, support, higher_is_better)` → registro elegido, con el desempate de D10: métrica primaria → métrica de apoyo → menor `order`; `assert` en `S6`.
  - **Hecho cuando:** los `assert` verifican que `y=[1,2,3]`, `pred=[1,2,5]` da MAE = 2/3, RMSE = √(4/3) y R² = −1; y que en `select_config` un empate en la primaria lo decide la de apoyo, un empate persistente lo gana la de menor `order`, y funciona con `higher_is_better=False` (RMSE).
  - **Evidencia:** S6: MAE = 2/3, RMSE = √(4/3), R² = −1; `select_config` resuelve empate en la primaria, empate persistente y `higher_is_better=False`.

- [x] **T09 · S4 `FrozenModel`, `freeze`, `evaluate_once`** — ~20 min
  - **Cubre:** RF-3 (CA-3.4, CA-3.5) · RF-19 (CA-19.2) · D9 · RNF-5
  - **Qué:** `FrozenModel` como `NamedTuple(model, config, estimator, n_configs)`; `freeze(key, record)` sin reajustar; `evaluate_once(frozen, X_eval, y_eval, ledger)` que falla si el modelo ya figura en `ledger`; `assert` en `S6`.
  - **Hecho cuando:** los `assert` verifican que asignar un atributo de `FrozenModel` lanza `AttributeError`, que el estimador congelado predice igual que el del registro (no se reajustó) y que la segunda llamada a `evaluate_once` con el mismo modelo lanza error mientras la primera lo añade a `ledger`.
  - **Evidencia:** S6: `FrozenModel` inmutable (`AttributeError`); la 2.ª llamada a `evaluate_once` lanza `RuntimeError`; `ledger` se actualiza.

- [x] **T10 · S4 `alpha_sweep` y `pick_best_alpha`** — ~25 min
  - **Cubre:** RF-21 (CA-21.2, CA-21.3, CA-21.4, CA-21.5) · CL-8 · CL-15 · CL-16 · RNF-5
  - **Qué:** `alpha_sweep(key, base_config, alphas, splits_data, seed)` → una fila por alpha (alpha, norma L2, coeficientes nulos, RMSE de entrenamiento y de validación, avisos); `pick_best_alpha(rows)` → alpha elegido y `at_border`; reciben `X_eval`/`y_eval` genéricos (I-5); `assert` en `S6`.
  - **Hecho cuando:** con datos sintéticos, los `assert` verifican: la norma L2 de Ridge no crece con alpha; Lasso con alpha ≥ `max|Xᵀ(y−ȳ)|/n` anula todos los coeficientes; un empate exacto elige el menor alpha; `at_border` es verdadero cuando el mínimo cae en un extremo de la rejilla.
  - **Evidencia:** S6: norma L2 de Ridge no creciente; Lasso con alpha ≥ `alpha_max` anula 5 de 5; empate exacto → menor alpha; borde marcado.

- [x] **T11 · S5 `format_config`, `simplicity_score`, `rank_models`, `result_fingerprint`** — ~20 min
  - **Cubre:** RF-12 (CA-12.1) · RF-15 (CA-15.2, CA-15.3) · RF-22 (CA-22.1) · RNF-1 · RNF-5
  - **Qué:** `format_config` (solo los hiperparámetros fijados explícitamente), `simplicity_score` (fijados + configuraciones probadas), `rank_models` (empates con el mismo puesto) y `result_fingerprint` (hash `hashlib` de valores redondeados a 6 decimales); `assert` en `S6`.
  - **Hecho cuando:** los `assert` verifican que `format_config` no incluye la semilla, que `simplicity_score` = `len(config)` + configuraciones probadas, que `rank_models` da el mismo puesto a los empates y respeta `higher_is_better`, y que `result_fingerprint` repite el hash con la misma entrada, lo cambia con una entrada alterada y no distingue diferencias por debajo del 6.º decimal.
  - **Evidencia:** S6: `format_config`, `simplicity_score`, `rank_models` y `result_digest` con sus asserts. La función se llama `result_digest` (ver «Desviaciones»).

- [x] **T12 · S5 umbrales, degenerados y mayores errores** — ~25 min
  - **Cubre:** RF-15 (CA-15.1) · RF-22 (CA-22.1) · RF-7 (CA-7.3) · RF-20 (CA-20.3) · D11 · CL-3 · CL-17 · RNF-5
  - **Qué:** `indistinguishable_cls(errors, best, tol=1)`, `indistinguishable_reg(rmse, best, tol=0.05)`, `is_degenerate_cls`, `is_degenerate_reg` y `top_abs_errors(y_true, y_pred, k=2)`; `assert` en `S6`.
  - **Hecho cuando:** los `assert` de frontera verifican que 1 error de diferencia queda incluido y 2 excluido; que exactamente 5 % sobre el RMSE del mejor (100.0 frente a 105.0) queda incluido y 5.01 % excluido, siempre contra el mejor y no entre pares; que un clasificador que nunca predice una clase es degenerado, igual que un Lasso con todos los coeficientes en cero; y que `top_abs_errors` ordena de mayor a menor y, en empate, por menor índice.
  - **Evidencia:** S6: frontera 1 error incluido / 2 excluido y 5 % incluido / 5.01 % excluido; degenerados; `top_abs_errors` con empate → menor posición.

- [x] **T13 · S5 `pick_two_errors`** — ~30 min
  - **Cubre:** RF-14 (CA-14.1, CA-14.2, CA-14.3) · D2 · DT-16 · CL-1 · CL-2 · RNF-5
  - **Qué:** `pick_two_errors(pred_test, pred_val, y_test, y_val, model_order)`: errores de prueba ordenados por (−n.º de modelos que fallan, índice original); si hay menos de dos, completa con errores de validación de los modelos ordenados por errores de validación (mayor a menor; empate, orden CA-10.1) con instancias distintas; cada elemento lleva índice, partición, modelo, `also_failed` y nota; `assert` en `S6`.
  - **Hecho cuando:** tres escenarios con `assert`: (a) ≥ 2 errores de prueba → 2 instancias distintas, modelo = primero que falla en el orden CA-10.1 y `also_failed` correcto; (b) 1 error de prueba → se completa con 1 de validación rotulado «validación» con la nota de posible sesgo optimista; (c) ninguno en prueba ni en validación → lista vacía.
  - **Evidencia:** S6: escenarios (a) ≥ 2 en prueba, (b) 1 en prueba + 1 de validación rotulado con nota de sesgo optimista, (c) ninguno → `[]`, y solo validación.

- [x] **T14 · Puerta de soporte** — ~15 min
  - **Cubre:** RNF-5 · RNF-2 · RNF-3 · I-1 a I-4 y I-6 de `plan.md` §1.2
  - **Qué:** revisar `S1–S6` completas antes de empezar a experimentar; no añade código salvo correcciones.
  - **Hecho cuando:** cada una de las 23 funciones de `plan.md` §1.3 tiene ≥ 1 `assert` en `S6` (contrastado por lista de nombres); en `S2–S5` no hay `print(`, `display(` ni `plt.`; en todo el notebook no hay `open(`, `read_csv`, `to_csv`, `savefig`, `joblib.dump`, `np.random.seed` ni `random_state=<número>`; los `import` están solo en `S1`.
  - **Evidencia:** 29 funciones en S2–S5 (el plan listaba 23), todas con ≥ 1 assert en S6 (66 asserts); `print(`, `display(`, `plt.` en S2–S5 = 0 en búsqueda literal.

---

## Fase 1 — E1 · Datos y protocolo (Iris) · ≈ 1 h 35 min

- [x] **T15 · ✍ E1.P: predicción de tamaño y balance** — ~15 min
  - **Cubre:** RF-1 (CA-1.1) · RF-6 (CA-6.1) · RF-8 (CA-8.4)
  - **Qué:** en `E1.P`, tu predicción sobre cuántas instancias tiene Iris y cómo se reparten las clases, escrita antes de cargar los datos.
  - **Hecho cuando:** `E1.P` contiene la predicción con cifras esperadas (n.º de instancias y reparto por clase) sin haber ejecutado `E1.C`; existe un commit con solo ese cambio (p. ej. `E1.P: predicción`), anterior a cualquier commit que implemente `E1.C`.
  - **Evidencia:** E1.P redactada y huella sha256 `b35dce31305dc797` registrada **antes** de ejecutar E1.C. Commit `7c00f7d` (solo cambia la celda E1.P), anterior a `944c5d1` (E1.C). Historial reconstruido (ver «Registro de ejecución»).

- [x] **T16 · E1.C `[E1.1]–[E1.2]`: carga y fichas** — ~20 min
  - **Cubre:** RF-8 (CA-8.1) · RF-5 (CA-5.1)
  - **Qué:** en `E1.C`, banner `# [E1.1]` con `load_iris` y `assert` de forma; banner `# [E1.2]` con T1.1–T1.3 generadas de los datos.
  - **Hecho cuando:** `X.shape == (150, 4)` y `y.shape == (150,)` con `assert`; se muestran T1.1 (`Concepto` · `Valor`: instancia, n.º de instancias, n.º de características, forma de X, forma de y), T1.2 (`Característica (nombre de la librería)` · `Significado`: las 4 con el nombre que entrega la librería) y T1.3 (`Clase` · `Ejemplos`: 50 por clase, nombres con `.tolist()`); ninguna cifra de tabla está escrita a mano.
  - **Evidencia:** E1.C `[E1.1]–[E1.2]`: `X (150, 4)`, `y (150,)`; T1.1 a T1.3 con las columnas exactas y 50 ejemplos por clase.

- [x] **T17 · E1.C `[E1.3]–[E1.4]`: cinco ejemplos y distribución** — ~20 min
  - **Cubre:** RF-8 (CA-8.2, CA-8.3)
  - **Qué:** banner `# [E1.3]` con `sample_examples(y, 5, SEED)` → T1.4; banner `# [E1.4]` con la figura F1.1.
  - **Hecho cuando:** T1.4 tiene 5 filas y las columnas exactas `Índice` · 4 nombres de característica · `Clase`, con las 3 clases presentes y las mismas filas en dos ejecuciones; F1.1 muestra barras de ejemplos por clase del conjunto completo con título y ejes en español.
  - **Evidencia:** E1.C `[E1.3]–[E1.4]`: T1.4 con 5 filas (índices 4, 64, 65, 88, 132) y las 3 clases; F1.1 con título y ejes en español. Figura F1.1 revisada visualmente.

- [x] **T18 · E1.C `[E1.5]–[E1.6]`: partición declarada y huella** — ~20 min
  - **Cubre:** RF-2 (CA-2.1, CA-2.2, CA-2.3, CA-2.4) · RF-8 (CA-8.6) · RNF-1
  - **Qué:** banner `# [E1.5]` con `splits = make_splits(y, stratify=True, seed=SEED)` y T1.5; banner `# [E1.6]` con la huella de E1.
  - **Hecho cuando:** `splits` se crea una sola vez; `assert` de tamaños 90/30/30, disjuntos, unión = 150 y 10 por clase en validación y prueba; T1.5 con columnas `Partición` · `Tamaño` · `setosa` · `versicolor` · `virginica`; la huella aparece y es igual en dos ejecuciones; `X_test`/`y_test` solo se usan para tamaños (I-5).
  - **Evidencia:** E1.C `[E1.5]–[E1.6]`: 90/30/30, disjuntas y completas, 10 por clase en validación y prueba (T1.5); huella E1 `689602a2376f3020`; `X_test` no se usa en E1.

- [x] **T19 · ✍ E1.I: interpretación de E1** — ~20 min
  - **Cubre:** RF-1 (CA-1.3) · RF-6 (CA-6.3) · RF-8 (CA-8.5) · RF-5 (CA-5.3)
  - **Qué:** en `E1.I`, balance de clases, qué implica para elegir métricas y contraste con tu predicción.
  - **Hecho cuando:** `E1.I` indica si las clases están balanceadas citando T1.3 o F1.1 y qué implica para accuracy y F1 macro; dice si la predicción se confirmó o se contradijo y por qué (aunque se confirmara); cada cifra cita su tabla o figura; `git diff` de `E1.P` desde su commit está vacío.
  - **Evidencia:** E1.I escrita con T1.3, F1.1 y T1.5; las tres predicciones confirmadas; huella de E1.P sin cambios al cierre.

---

## Fase 2 — E2 · Comparación de clasificadores (Iris) · ≈ 5 h 35 min

- [x] **T20 · ✍ E2.P: predicciones y criterio de sencillez** — ~20 min
  - **Cubre:** RF-9 (CA-9.1, CA-9.2, CA-9.3, CA-9.4) · RF-6 (CA-6.1) · RF-1 (CA-1.1)
  - **Qué:** en `E2.P`, tres predicciones y el criterio objetivo de «más sencillo de programar», antes de implementar.
  - **Hecho cuando:** `E2.P` contiene (1) qué modelos serán sensibles a la escala, (2) qué modelo dará fronteras más simples, marcada como conceptual, (3) por qué no puede saberse de antemano cuál ganará, y (4) el criterio: menor suma de hiperparámetros fijados explícitamente + configuraciones probadas; existe un commit con solo ese cambio, anterior a ejecutar `E2.C`.
  - **Evidencia:** E2.P redactada y huella sha256 `4e55ff4718222f40` registrada **antes** de ejecutar E2.C. Commit `47c8e31` (solo cambia la celda E2.P), anterior a `31fe25e` (E2.C). Historial reconstruido (ver «Registro de ejecución»).

- [x] **T21 · E2.C `[E2.0]`: declaración de configuraciones** — ~25 min
  - **Cubre:** RF-3 (CA-3.2, CA-3.7, CA-3.8) · RF-10 (CA-10.1) · D8
  - **Qué:** primer banner de `E2.C`, antes de cualquier `fit`: `INITIAL` y `ALTERNATIVES` (≤ 3) para los 7 modelos escritos explícitamente y sin semilla, con `max_iter` visible donde aplique; `RUN_TEST = False`; criterio de desempate declarado (F1 macro → accuracy → probada primero). Si un aviso se corrige con `max_iter`, esa corrección ocupa una de las ≤ 4 configuraciones (CA-7.2).
  - **Hecho cuando:** `assert` en la celda: para cada modelo `1 + len(ALTERNATIVES) ≤ MAX_CONFIGS`, configuraciones distintas entre sí y ninguna con clave de semilla; por encima de este banner no hay `fit` ni `score`.
  - **Evidencia:** E2.C `[E2.0]`: `INITIAL` y `ALTERNATIVES` (≤ 3) para los 7 modelos, `RUN_TEST = False`; asserts de ≤ 4 configuraciones, distintas y sin semilla; ningún `fit` por encima.

- [x] **T22 · E2.C `[E2.1]`: validación con la configuración inicial** — ~20 min
  - **Cubre:** RF-11 (CA-11.1) · RF-10 (CA-10.2, CA-10.3) · RF-4
  - **Qué:** ajustar los 7 modelos con su configuración inicial y `scaled=True` solo con entrenamiento y puntuarlos en validación → T2.1.
  - **Hecho cuando:** T2.1 tiene columnas exactas `Modelo` · `Accuracy de validación` · `F1 macro de validación`, 7 filas en el orden de CA-10.1, generada con las funciones de `S4` y un único `splits`; el banner no contiene `X_test`.
  - **Evidencia:** E2.C `[E2.1]`: T2.1 con 7 filas y columnas exactas; `splits_iris` único; sin `X_test`.

- [x] **T23 · E2.C `[E2.2]`: diagnóstico sin estandarizar** — ~20 min
  - **Cubre:** RF-10 (CA-10.4) · RF-9 (CA-9.1)
  - **Qué:** repetir T22 con `scaled=False` solo en validación y comparar con el F1 macro estandarizado → T2.2.
  - **Hecho cuando:** T2.2 con columnas `Modelo` · `Accuracy sin estandarizar (validación)` · `F1 macro sin estandarizar (validación)` · `F1 macro estandarizado (validación)` · `Diferencia de F1 macro`; un `assert` comprueba que el diagnóstico no añadió registros ni cambió ninguna configuración; no usa prueba.
  - **Evidencia:** E2.C `[E2.2]`: T2.2 con columnas exactas; assert de que `records` no cambió; el MLP y la SVM lineal cambian ≈ 0.034, el resto 0.0000.

- [x] **T24 · E2.C `[E2.3]`: ajuste limitado y selección** — ~30 min
  - **Cubre:** RF-3 (CA-3.1, CA-3.2, CA-3.3, CA-3.7) · RF-11 (CA-11.2) · RF-7 (CA-7.2) · CL-5 · CL-6
  - **Qué:** ajustar cada configuración de cada modelo con entrenamiento, puntuarla en validación, construir `records` y elegir con `select_config` (F1 macro; apoyo, accuracy) → T2.3.
  - **Hecho cuando:** T2.3 con columnas `Modelo` · `N.º` · `Hiperparámetros` · `Accuracy de validación` · `F1 macro de validación` · `Avisos` · `Elegida`, con exactamente una fila `Elegida` por modelo; `assert len(records[m]) ≤ 4` y configuraciones distintas; si ninguna alternativa mejora, la elegida es la inicial; el banner no contiene `X_test`.
  - **Evidencia:** E2.C `[E2.3]`: T2.3 con 22 filas y una `Elegida` por modelo; `len(records[m]) ≤ 4`; sin avisos.

- [x] **T25 · E2.C `[E2.4]`: congelado** — ~20 min
  - **Cubre:** RF-3 (CA-3.4) · D9 · DT-8
  - **Qué:** un `FrozenModel` por modelo con la configuración elegida y el estimador ajustado solo con entrenamiento → T2.4.
  - **Hecho cuando:** `assert` de que cada congelado reproduce exactamente la accuracy y el F1 macro de validación de su registro; T2.4 con `Modelo` · `Transformación` · `Hiperparámetros fijados` · `Configuraciones probadas`; commit «E2: configuración congelada» hecho antes de poner `RUN_TEST = True`.
  - **Evidencia:** E2.C `[E2.4]`: 7 `FrozenModel` que reproducen la validación; T2.4. Commit `31fe25e` (congelado, `RUN_TEST = False`, prueba sin evaluar) anterior a `a9c5763` (`RUN_TEST = True`); entre ambos solo cambia la línea `RUN_TEST`. Historial reconstruido (ver «Registro de ejecución»).

- [x] **T26 · E2.C `[E2.5]`: prueba única** — ~25 min
  - **Cubre:** RF-12 (CA-12.1, CA-12.2, CA-12.3) · RF-3 (CA-3.5)
  - **Qué:** poner `RUN_TEST = True`; evaluar los 7 congelados una sola vez con `evaluate_once` y un único `ledger`, guardando las predicciones de prueba → T2.5 y T2.6.
  - **Hecho cuando:** `assert len(ledger) == 7`; T2.5 con columnas exactamente `Modelo` · `Transformación` · `Hiperparámetros principales` · `Accuracy de prueba` · `F1 macro de prueba`; T2.6 con `Modelo` · `F1 micro de prueba` · `F1 macro de prueba`; `git diff` desde el commit de T25 no toca `INITIAL` ni `ALTERNATIVES`.
  - **Evidencia:** E2.C `[E2.5]`: `RUN_TEST = True`; `len(ledger) == 7`; T2.5 y T2.6 con columnas exactas. `INITIAL` y `ALTERNATIVES` no se editaron tras el congelado (verificado: entre `31fe25e` y `a9c5763` solo cambia `RUN_TEST`).

- [x] **T27 · E2.C `[E2.5]`: matrices de confusión** — ~20 min
  - **Cubre:** RF-13 (CA-13.1, CA-13.2)
  - **Qué:** F2.1 con las matrices de prueba de Naive Bayes, SVM lineal y MLP, calculadas con las predicciones guardadas (sin re-predecir).
  - **Hecho cuando:** F2.1 muestra las tres matrices con filas y columnas rotuladas con los nombres de clase de la librería (`.tolist()`), conteos visibles, título y ejes en español.
  - **Evidencia:** E2.C: F2.1 con las matrices de Naive Bayes, SVM lineal y MLP, con nombres de clase y conteos. Figura F2.1 revisada visualmente (los conteos de las tres matrices coinciden con T2.5 y T2.7).

- [x] **T28 · E2.C `[E2.6]`: comparación contra el mejor** — ~25 min
  - **Cubre:** RF-15 (CA-15.1, CA-15.2, CA-15.3) · D11 · CL-3 · CL-4
  - **Qué:** mejor observado por F1 macro de prueba (empate → accuracy → orden CA-10.1), no distinguibles a ≤ 1 error del mejor, posiciones por F1 macro y por accuracy, y suma de sencillez → T2.7, T2.8, T2.9.
  - **Hecho cuando:** T2.7 con `Modelo` · `Errores de prueba` · `Diferencia con el mejor observado` · `No distinguible del mejor (≤ 1 error)`, comparando cada modelo contra el mejor y no entre pares; T2.8 con `Modelo` · `Posición por F1 macro` · `Posición por accuracy` · `¿Coincide?`; T2.9 con `Modelo` · `Hiperparámetros fijados` · `Configuraciones probadas` · `Suma (menor = más sencillo)`; las tres salen de resultados guardados, sin re-predecir.
  - **Evidencia:** E2.C `[E2.6]`: T2.7 (mejor observado: Gaussian Naive Bayes; no distinguibles: 5 modelos), T2.8 y T2.9 (empate de sencillez: Naive Bayes y árbol, 4 puntos).

- [x] **T29 · E2.C `[E2.7]`: avisos y degenerados** — ~20 min
  - **Cubre:** RF-7 (CA-7.1, CA-7.3, CA-7.4) · CL-7 · CL-18
  - **Qué:** mostrar todos los avisos capturados en ajuste, validación y prueba y detectar resultados degenerados → T2.10 y T2.11.
  - **Hecho cuando:** T2.10 con `Modelo` · `Etapa (ajuste / validación / prueba)` · `Categoría` · `Mensaje` lista cada aviso capturado, o declara explícitamente «sin avisos»; T2.11 con `Modelo` · `Clases nunca predichas (prueba)` · `Degenerado`; una búsqueda de `filterwarnings` y `simplefilter("ignore")` en el notebook da 0 coincidencias.
  - **Evidencia:** E2.C `[E2.7]`: T2.10 declara «sin avisos» (ningún modelo emitió avisos); T2.11 sin modelos degenerados; 0 `filterwarnings`.

- [x] **T30 · E2.C `[E2.8]`: dos errores de clasificación** — ~20 min
  - **Cubre:** RF-14 (CA-14.1, CA-14.2, CA-14.3, CA-14.4, CA-14.5) · CL-1 · CL-2
  - **Qué:** aplicar `pick_two_errors` a las predicciones guardadas de prueba y de validación → T2.12.
  - **Hecho cuando:** T2.12 con `Instancia (índice original)` · `Partición` · `Modelo` · `Otros modelos que también fallaron` · las 4 características con su nombre de la librería · `Clase real` · `Clase predicha` · `Nota`; si hubo < 2 errores de prueba, las filas de validación llevan la nota de posible sesgo optimista; `git diff` no toca `INITIAL` ni `ALTERNATIVES`.
  - **Evidencia:** E2.C `[E2.8]`: T2.12 con las instancias 77 y 134 (ambas de prueba, fallan los 7 modelos); añadida la referencia T2.12b (medias por clase).

- [x] **T31 · E2.C `[E2.9]`: huella e integridad** — ~15 min
  - **Cubre:** RF-2 (CA-2.3) · RF-3 (CA-3.2, CA-3.5) · RNF-1
  - **Qué:** último banner de `E2.C`: huella de E2 y comprobaciones de `plan.md` §7.2.
  - **Hecho cuando:** `assert` de un único `splits` para los 7 modelos, `len(records[m]) ≤ 4`, `len(ledger) == 7` y columnas de T2.5 iguales a la lista exacta de CA-12.1; la huella es idéntica en dos ejecuciones.
  - **Evidencia:** E2.C `[E2.9]`: asserts de integridad; huella E2 `db54a3c0be16c4a4`, idéntica en las dos ejecuciones.

- [x] **T32 · ✍ E2.I-a: desempeño, accuracy vs F1 macro, macro vs micro** — ~25 min
  - **Cubre:** RF-15 (CA-15.1, CA-15.3, CA-15.4, CA-15.7) · RF-1 (CA-1.3)
  - **Qué:** primera parte de `E2.I`, con base en T2.5–T2.8.
  - **Hecho cuando:** `E2.I` (1) nombra el mejor observado con el rótulo «mejor observado, no seleccionado», lista los no distinguibles de T2.7, señala si accuracy discrepa y escribe «en esta partición»; (2) compara el orden por accuracy y por F1 macro con T2.8, sin presumir que el balance lo garantice si coinciden; (3) explica macro vs micro con T2.6, incluido que F1 micro = accuracy; cada cifra cita su tabla.
  - **Evidencia:** E2.I secciones 1, 3 y 4: mejor observado y no distinguibles (T2.7), accuracy = F1 macro en orden (T2.8), macro vs micro (T2.6).

- [x] **T33 · ✍ E2.I-b: sencillez, MLP vs lineales, contraste de predicciones** — ~25 min
  - **Cubre:** RF-15 (CA-15.2, CA-15.5, CA-15.6) · RF-6 (CA-6.3) · CL-13 · RF-1 (CA-1.1)
  - **Qué:** segunda parte de `E2.I`.
  - **Hecho cuando:** `E2.I` responde «más sencillo» con T2.9 y el criterio escrito en `E2.P`; explica qué aporta el MLP frente a la regresión logística y la SVM lineal con resultados de este dataset, sin afirmar una ventaja general; marca cada predicción falsable como confirmada o contradicha y por qué (la de escala usa T2.2) y contrasta en texto las conceptuales; `git diff` de `E2.P` desde su commit está vacío.
  - **Evidencia:** E2.I secciones 2, 5 y 8: sencillez con T2.9, MLP vs lineales sin ventaja general, tabla de contraste (las dos predicciones falsables solo se confirman en parte: la regresión logística no fue sensible a la escala y la SVM lineal quedó bajo 0.90); E2.P sin cambios (huella).

- [x] **T34 · ✍ E2.I-c: errores, avisos y decisiones** — ~25 min
  - **Cubre:** RF-14 (CA-14.6) · RF-7 (CA-7.1, CA-7.3, CA-7.4) · RNF-9 · RF-1 (CA-1.1)
  - **Qué:** tercera parte de `E2.I`.
  - **Hecho cuando:** para cada fila de T2.12 hay dos apartados rotulados, «lo que los datos permiten concluir» e «hipótesis no verificada sobre la causa»; cada aviso de T2.10 y cada resultado «degenerado» de T2.11 está comentado (o se declara que no hubo); queda justificada la configuración elegida de cada modelo, el desempate aplicado y la elección de los errores analizados (DT-16).
  - **Evidencia:** E2.I secciones 6, 7 y 9: errores separados en «lo que los datos permiten concluir» e «hipótesis»; sin avisos; decisiones justificadas.

---

## Fase 3 — E3 · Regresión y regularización (Diabetes) · ≈ 5 h 45 min

- [x] **T35 · ✍ E3.P: flexibilidad y regularización** — ~20 min
  - **Cubre:** RF-17 (CA-17.1) · RF-21 (CA-21.1) · RF-6 (CA-6.1) · RF-1 (CA-1.1)
  - **Qué:** una sola celda `E3.P` con las dos predicciones, antes de implementar.
  - **Hecho cuando:** `E3.P` contiene (a) qué modelos considera más flexibles y si un modelo más flexible produce siempre menor error en datos nuevos, justificado, y (b) cómo cambiarán los coeficientes y los errores de entrenamiento y validación al aumentar la penalización; existe un commit con solo ese cambio, anterior a ejecutar `E3.C`.
  - **Evidencia:** E3.P redactada y huella sha256 `039d27c91664a288` registrada **antes** de ejecutar E3.C. Commit `2fdee92` (solo cambia la celda E3.P), anterior a `a751373` (E3.C). Historial reconstruido (ver «Registro de ejecución»).

- [x] **T36 · E3.C `[E3.0]`: declaración y rejillas** — ~20 min
  - **Cubre:** RF-3 (CA-3.2, CA-3.8) · RF-18 (CA-18.1) · RF-21 (CA-21.2)
  - **Qué:** primer banner de `E3.C`, antes de cualquier `fit`: `INITIAL`, `ALTERNATIVES` y `NON_ALPHA_CONFIGS` para los 8 modelos, una rejilla de alpha propia por modelo (plan: `logspace(-2, 4, 13)` para Ridge y `logspace(-3, 2, 11)` para Lasso) y `RUN_TEST = False`.
  - **Hecho cuando:** `assert` de que cada rejilla tiene ≥ 8 puntos con razón constante (log-espaciada); fuera del barrido, `1 + len(ALTERNATIVES) ≤ MAX_CONFIGS` por modelo con configuraciones distintas y sin semilla; por encima de este banner no hay `fit`.
  - **Evidencia:** E3.C `[E3.0]`: configuraciones, rejillas (13 puntos Ridge, 11 Lasso, razón constante) y `RUN_TEST = False`; asserts; ningún `fit` por encima.

- [x] **T37 · E3.C `[E3.1]`: carga y descripción de Diabetes** — ~20 min
  - **Cubre:** RF-16 (CA-16.1, CA-16.2, CA-16.3) · CL-11
  - **Qué:** cargar con `load_diabetes` (variante por defecto), describir objetivo y variables, y comprobar la transformación de origen → T3.1.
  - **Hecho cuando:** `X.shape == (442, 10)` y `y.shape == (442,)` con `assert`; T3.1 con `Variable` · `Descripción (según el dataset)` más líneas con el significado del objetivo y el hallazgo de `DESCR` sobre centrado y escalado; comprobación numérica mostrada de suma de cuadrados por columna ≈ 1 y media ≈ 0; ningún texto interpreta las variables como unidades originales.
  - **Evidencia:** E3.C `[E3.1]`: `X (442, 10)`, `y (442,)`; T3.1 con la línea de `DESCR` y la comprobación numérica (suma de cuadrados 1.000000; |media| 2.26e-16).

- [x] **T38 · E3.C `[E3.2]–[E3.3]`: partición y declaraciones** — ~20 min
  - **Cubre:** RF-2 (CA-2.1, CA-2.3, CA-2.4) · RF-18 (CA-18.3) · RF-16 (CA-16.4) · RF-4 (CA-4.3)
  - **Qué:** `make_splits(stratify=False)` → T3.2; declarar en la salida la limitación de origen y qué objetivo se transforma.
  - **Hecho cuando:** `assert` de tamaños 264/89/89, disjuntos y unión = 442; T3.2 con `Partición` · `Tamaño`; la salida de la celda declara (i) que las características ya vienen centradas y escaladas con estadísticos del conjunto completo (fuga de origen no controlada) y (ii) que solo el MLP transforma el objetivo, ajustado con entrenamiento.
  - **Evidencia:** E3.C `[E3.2]–[E3.3]`: 264/89/89 sin estratificar (T3.2); limitación de origen y objetivo transformado solo en el MLP declarados en la salida.

- [x] **T39 · E3.C `[E3.4]`: barrido de alpha** — ~25 min
  - **Cubre:** RF-21 (CA-21.2, CA-21.3, CA-21.5) · CL-8
  - **Qué:** `alpha_sweep` para Ridge y Lasso con su rejilla, ajustando con entrenamiento y puntuando en entrenamiento y validación → T3.3.
  - **Hecho cuando:** T3.3 con `Modelo` · `Intensidad (alpha)` · `Norma L2 de los coeficientes` · `Coeficientes nulos` (solo Lasso) · `RMSE de entrenamiento` · `RMSE de validación`, con ≥ 8 filas por modelo; `assert` de que Lasso en el alpha máximo tiene los 10 coeficientes nulos; el banner no contiene `X_test`.
  - **Evidencia:** E3.C `[E3.4]`: T3.3 con 13 filas de Ridge y 11 de Lasso; Lasso con alpha = 100 anula los 10 coeficientes; sin `X_test`.

- [x] **T40 · E3.C `[E3.4]`: figuras y óptimo del barrido** — ~25 min
  - **Cubre:** RF-21 (CA-21.4, CA-21.6) · CL-15 · CL-16
  - **Qué:** F3.1 y F3.2 a partir de T3.3, y `pick_best_alpha` para cada modelo.
  - **Hecho cuando:** F3.1 (coeficientes frente a alpha, escala log, Ridge y Lasso) y F3.2 (RMSE de entrenamiento y validación frente a alpha, escala log) con título y ejes en español; la salida declara, por modelo, el alpha elegido y si cayó en el borde de la rejilla, sin extenderla; un empate exacto resuelve al menor alpha.
  - **Evidencia:** E3.C `[E3.4]`: F3.1 y F3.2; mejor alpha de Ridge 1.0 y de Lasso 3.16, ambos en el interior de la rejilla (declarado en la salida). F3.1 y F3.2 revisadas visualmente.

- [x] **T41 · E3.C `[E3.5]`: ajuste limitado de los 8 modelos** — ~30 min
  - **Cubre:** RF-3 (CA-3.1, CA-3.2, CA-3.3) · RF-18 (CA-18.1, CA-18.2, CA-18.3)
  - **Qué:** ajustar con entrenamiento y puntuar en validación las configuraciones de los 8 modelos (Ridge y Lasso con el alpha del barrido y ≤ 4 configuraciones de sus demás hiperparámetros) y elegir con `select_config(higher_is_better=False)` → T3.4.
  - **Hecho cuando:** T3.4 con `Modelo` · `N.º` · `Hiperparámetros` · `RMSE de validación` · `MAE de validación` · `Avisos` · `Elegida`, con exactamente una fila `Elegida` por modelo; `assert len(records[m]) ≤ 4` fuera del barrido y configuraciones distintas; el banner no contiene `X_test`.
  - **Evidencia:** E3.C `[E3.5]`: T3.4 con 20 filas y una `Elegida` por modelo; `len(records_reg[m]) ≤ 4`; sin avisos.

- [x] **T42 · E3.C `[E3.6]`: congelado** — ~20 min
  - **Cubre:** RF-3 (CA-3.4) · RF-21 (CA-21.4) · D3 · D9
  - **Qué:** ocho `FrozenModel`; Ridge y Lasso con el alpha de menor RMSE de validación del barrido, ajustados solo con entrenamiento → T3.5.
  - **Hecho cuando:** `assert` de que cada congelado reproduce el RMSE de validación de su registro; T3.5 con `Modelo` · `Transformación` · `Hiperparámetros fijados` · `Configuraciones probadas`; commit «E3: configuración congelada» hecho antes de poner `RUN_TEST = True`.
  - **Evidencia:** E3.C `[E3.6]`: 8 `FrozenModel`; Ridge y Lasso con el alpha del barrido; T3.5. Commit `a751373` (congelado, `RUN_TEST = False`) anterior a `fb21be0` (`RUN_TEST = True`); entre ambos solo cambia la línea `RUN_TEST`. Historial reconstruido (ver «Registro de ejecución»).

- [x] **T43 · E3.C `[E3.7]`: prueba única** — ~20 min
  - **Cubre:** RF-19 (CA-19.1, CA-19.2) · RF-3 (CA-3.5)
  - **Qué:** poner `RUN_TEST = True` y evaluar los 8 congelados una sola vez, guardando las predicciones de prueba → T3.6.
  - **Hecho cuando:** `assert len(ledger) == 8`; T3.6 con columnas exactamente `Modelo` · `MAE` · `RMSE` · `R²` y 8 filas en el orden de CA-18.1; `git diff` desde el commit de T42 no toca `INITIAL`, `ALTERNATIVES` ni las rejillas.
  - **Evidencia:** E3.C `[E3.7]`: `RUN_TEST = True`; `len(ledger_reg) == 8`; T3.6 con columnas exactas `Modelo`, `MAE`, `RMSE`, `R²`. Rejillas y configuraciones sin editar tras el congelado (verificado: entre `a751373` y `fb21be0` solo cambia `RUN_TEST`).

- [x] **T44 · E3.C `[E3.8]`: comparación contra el mejor y R² base** — ~25 min
  - **Cubre:** RF-22 (CA-22.1, CA-22.2, CA-22.3) · D11 · CL-9 · CL-10 · CL-17
  - **Qué:** mejor observado por RMSE de prueba (empate → MAE → orden CA-18.1), no distinguibles a ≤ 5 % del RMSE del mejor, posiciones por RMSE, MAE y R², y R² de la predicción trivial → T3.7, T3.8, T3.10.
  - **Hecho cuando:** T3.7 con `Modelo` · `RMSE de prueba` · `Diferencia relativa con el mejor` · `No distinguible del mejor (≤ 5 %)`; T3.8 con `Modelo` · `Posición por RMSE` · `Posición por MAE` · `Posición por R²`; T3.10 con `Predictor` · `R² de prueba` (media de entrenamiento) y `assert` de que es ≤ 0; `assert` de que el orden por R² coincide con el orden por RMSE.
  - **Evidencia:** E3.C `[E3.8]`: T3.7 (mejor observado: MLP; no distinguibles ≤ 5 %: 5 modelos), T3.8, T3.10 (R² de la media de entrenamiento = −0.0031 ≤ 0); assert de orden R² = orden RMSE.

- [x] **T45 · E3.C `[E3.9]`: diagnóstico de regresión lineal y MLP** — ~25 min
  - **Cubre:** RF-20 (CA-20.1, CA-20.2, CA-20.3) · RF-4 (CA-4.2) · CL-11 · CL-12
  - **Qué:** gráficos de reales frente a predicciones y de residuos para regresión lineal y MLP, y sus dos mayores errores absolutos → F3.3, F3.4, T3.9.
  - **Hecho cuando:** F3.3 y F3.4 muestran ambos modelos sobre prueba, en la escala original del objetivo, con título y ejes en español; T3.9 con `Modelo` · `Instancia (índice original)` · `Valor real` · `Predicción` · `Error` · las 10 variables tal como vienen, 2 filas por modelo obtenidas con `top_abs_errors`.
  - **Evidencia:** E3.C `[E3.9]`: F3.3, F3.4 y T3.9 con 2 filas por modelo (instancias 56 y 77 en regresión lineal; 56 y 9 en MLP). F3.3 y F3.4 revisadas visualmente; al revisarlas se corrigió el signo de `Error` en T3.9 (ahora real − predicción, igual que el residuo de F3.4).

- [x] **T46 · E3.C `[E3.10]–[E3.11]`: avisos, degenerados, huella e integridad** — ~20 min
  - **Cubre:** RF-7 (CA-7.1, CA-7.3, CA-7.4) · RNF-1 · CL-8 · CL-16
  - **Qué:** T3.11 y T3.12, huella de E3 y comprobaciones de `plan.md` §7.2 para E3.
  - **Hecho cuando:** T3.11 con `Modelo` · `Etapa` · `Categoría` · `Mensaje` lista cada aviso, o declara «sin avisos»; T3.12 con `Modelo` · `Degenerado` · `Motivo`, donde un Lasso con todos los coeficientes en cero queda rotulado «degenerado»; `assert` de ≥ 8 puntos log-espaciados, `len(ledger) == 8` y formas; la huella es idéntica en dos ejecuciones.
  - **Evidencia:** E3.C `[E3.10]–[E3.11]`: T3.11 «sin avisos»; T3.12 con el Lasso del barrido en alpha = 100 como degenerado; huella E3 `762c72766081749c`, idéntica en las dos ejecuciones.

- [x] **T47 · ✍ E3.I-a: desempeño, MAE vs RMSE, R² negativo** — ~25 min
  - **Cubre:** RF-22 (CA-22.1, CA-22.2, CA-22.3) · RF-1 (CA-1.3)
  - **Qué:** primera parte de `E3.I`, con base en T3.6–T3.10.
  - **Hecho cuando:** `E3.I` (1) nombra el mejor observado por RMSE con «mejor observado, no seleccionado», lista los no distinguibles ≤ 5 % de T3.7, dice si MAE cambia el orden, explica por qué el orden de R² coincide con el de RMSE y escribe «en esta partición»; (2) explica cómo MAE y RMSE tratan los errores grandes y, si el orden coincide, lo dice; (3) explica R² frente a la predicción trivial con T3.10 y, si ningún modelo lo tuvo, indica que ninguno fue negativo.
  - **Evidencia:** E3.I secciones 1 a 3: mejor observado MLP (51.66), MAE cambia el orden, R² ordena igual que RMSE, ningún R² negativo.

- [x] **T48 · ✍ E3.I-b: regularización, borde y contraste de predicciones** — ~25 min
  - **Cubre:** RF-22 (CA-22.4, CA-22.7) · RF-17 (CA-17.2) · RF-21 (CA-21.4) · RF-6 (CA-6.3) · CL-13 · RF-1 (CA-1.1)
  - **Qué:** segunda parte de `E3.I`.
  - **Hecho cuando:** `E3.I` describe con T3.3, F3.1 y F3.2 cómo cambian la norma de los coeficientes, los coeficientes nulos y la brecha entre error de entrenamiento y de validación al aumentar la penalización; declara si el alpha óptimo cayó en el borde y qué implica; reformula RF-17 en observable (¿el modelo declarado más flexible tiene menor RMSE de prueba que el menos flexible?, sin confirmar el universal «siempre») y marca RF-17 y RF-21 como confirmadas o contradichas y por qué; `git diff` de `E3.P` desde su commit está vacío.
  - **Evidencia:** E3.I secciones 4 y 8: efecto de la regularización con T3.3, óptimo en el interior; (a) contradicha, brecha contradicha, el resto confirmada; E3.P sin cambios (huella).

- [x] **T49 · ✍ E3.I-c: errores grandes, causalidad y decisiones** — ~25 min
  - **Cubre:** RF-22 (CA-22.5, CA-22.6) · RF-7 (CA-7.1, CA-7.3) · RNF-9 · RF-1 (CA-1.1)
  - **Qué:** tercera parte de `E3.I`.
  - **Hecho cuando:** `E3.I` ofrece posibles explicaciones de los errores grandes de T3.9 y marca como hipótesis las que no se pueden comprobar; responde si un coeficiente predictivo demuestra causalidad, separando asociación de causa con ≥ 1 motivo concreto (variables correlacionadas entre sí o efecto de la regularización); comenta cada aviso de T3.11 y cada degenerado de T3.12; justifica la configuración elegida, el desempate y la elección de los ejemplos.
  - **Evidencia:** E3.I secciones 5, 6, 7 y 9: hipótesis marcadas, causalidad con tres motivos concretos, sin avisos, decisiones justificadas.

---

## Fase 4 — Verificación global · ≈ 1 h 35 min

- [x] **T50 · Inspección estática del `.ipynb`** — ~20 min
  - **Cubre:** RNF-2 · RNF-3 · RNF-5 · CA-2.5 · CA-2.6 · CA-7.3
  - **Qué:** ejecutar las 8 búsquedas de texto de `plan.md` §7.3 sobre el `.ipynb` (ubicando cada coincidencia por el ID de su celda) y corregir lo que falle.
  - **Hecho cuando:** `print(`, `display(` y `plt.` en `S2–S5` = 0; `open(`, `read_csv`, `to_csv`, `savefig` y `joblib.dump` = 0; los `import` están solo en `S1` y son del stack o estándar; `def ` en `E*.C` = 0; `X_test`/`y_test` aparecen solo en `E1.C`, `E2.C` y `E3.C`; `early_stopping`, `validation_fraction`, `n_iter_no_change` y `cv=` = 0 o solo valores por defecto; `filterwarnings` y `simplefilter("ignore")` = 0, con `simplefilter("always")` solo en `S4`; todo `random_state=` es `SEED` y no hay `np.random.seed`.
  - **Evidencia:** Búsquedas literales del `.ipynb`: 0 coincidencias de lo prohibido; imports solo en S1; `def` solo en S2–S5; `X_test`/`y_test` solo en E2.C y E3.C; `early_stopping`/`n_iter_no_change` solo en los asserts de S6; `random_state=seed` siempre; sin `np.random.seed`.

- [x] **T51 · Reproducibilidad y tiempo** — ~25 min
  - **Cubre:** RNF-1 · RNF-8 · CL-14
  - **Qué:** con `RUN_TEST = True` en `E2.C` y `E3.C`, dos *Restart & Run All* completos desde cero.
  - **Hecho cuando:** ambas ejecuciones terminan sin errores ni aserciones fallidas; las huellas de E1, E2 y E3 y las tablas T2.5 y T3.6 son idénticas entre las dos; el tiempo total medido es < 5 min y queda anotado.
  - **Evidencia:** Dos ejecuciones completas desde cero (kernel nuevo, `nbconvert --execute`): 9 de 9 celdas de código sin error, salidas y 6 figuras idénticas, huellas E1/E2/E3 iguales; ≈ 50 s cada una (< 5 min).

- [x] **T52 · Revisión de trazabilidad y proceso** — ~25 min
  - **Cubre:** RF-1 (CA-1.1, CA-1.2, CA-1.3) · RF-6 (CA-6.2) · RNF-6
  - **Qué:** completar la matriz requisito ↔ celda y revisar el historial de git (`plan.md` §7.5).
  - **Hecho cuando:** cada RF-8…RF-22 está asignado a E1, E2 o E3 y cada experimento responde a ≥ 1 requisito; cada experimento tiene `P → C → I` consecutivas; `git log -p` sobre el `.ipynb` muestra cada `E*.P` commiteada antes del commit que implementa su `E*.C` y sin ediciones posteriores; cada interpretación cita la tabla o figura de origen de sus cifras.
  - **Evidencia:** Matriz requisito ↔ celda completa (sección «Matriz requisito ↔ celda»); `P → C → I` consecutivas en los tres experimentos; `git log -p`: las celdas E1.P, E2.P y E3.P conservan su texto en todos los commits posteriores y su sha256 coincide con el registrado antes de ejecutar; cada interpretación cita su tabla o figura. Historial reconstruido (ver «Registro de ejecución»).

- [x] **T53 · Revisión de idioma, legibilidad y cierre** — ~25 min
  - **Cubre:** RNF-4 · RNF-7 · RNF-9 · RF-5 (CA-5.2)
  - **Qué:** última lectura completa del notebook y verificación de los criterios de finalización de `spec.md` §8.
  - **Hecho cuando:** identificadores y comentarios de código están en inglés y Markdown, títulos, ejes y encabezados de tabla en español (salvo términos técnicos); cada término técnico no trivial se explica al introducirlo, cada figura tiene título y ejes y cada tabla encabezados descriptivos; no hay cifras de resultados escritas a mano en tablas ni gráficos; cada criterio de `spec.md` §8 queda marcado con la tarea que lo evidencia.
  - **Evidencia:** Revisión: comentarios e identificadores en inglés (0 comentarios con acentos), Markdown y rótulos en español, tablas y figuras generadas, términos explicados en las interpretaciones; criterios de `spec.md` §8 mapeados en el registro de cierre.

---

## Matriz de cobertura requisito → tareas

| Requisito | Tareas |
|---|---|
| RF-1 Estructura de cada experimento | T01, T02, T15, T19, T20, T32–T35, T47–T49, T52 |
| RF-2 Particiones y semilla | T02, T03, T04, T18, T31, T38 |
| RF-3 Ajuste limitado, congelado y prueba única | T08, T09, T21, T24, T25, T26, T31, T36, T41, T42, T43 |
| RF-4 Preprocesamiento sin fuga | T05, T06, T22, T38, T45 |
| RF-5 Resultados generados, no transcritos | T16, T19, T53 |
| RF-6 Predicción preservada y contrastada | T15, T19, T20, T33, T35, T48, T52 |
| RF-7 Avisos y resultados degenerados | T07, T12, T24, T29, T34, T46, T49 |
| RF-8 Descripción de Iris | T04, T15, T16, T17, T18, T19 |
| RF-9 Predicciones del Bloque 2 | T20, T23 |
| RF-10 Siete clasificadores, mismo protocolo | T05, T21, T22, T23 |
| RF-11 Comparación en validación | T07, T22, T24 |
| RF-12 Evaluación final en prueba (clasificación) | T07, T11, T26 |
| RF-13 Matrices de confusión | T27 |
| RF-14 Dos errores de clasificación | T13, T30, T34 |
| RF-15 Interpretación del Bloque 2 | T11, T12, T28, T32, T33 |
| RF-16 Descripción de Diabetes | T37, T38 |
| RF-17 Predicción de flexibilidad | T35, T48 |
| RF-18 Ocho regresores, mismo protocolo | T06, T36, T38, T41 |
| RF-19 Evaluación final en prueba (regresión) | T08, T09, T43 |
| RF-20 Diagnóstico de regresión lineal y MLP | T12, T45 |
| RF-21 Regularización en Ridge y Lasso | T10, T35, T36, T39, T40, T42, T48 |
| RF-22 Interpretación del Bloque 3 | T11, T12, T44, T47, T48, T49 |
| RNF-1 Reproducibilidad | T11, T18, T31, T46, T51 |
| RNF-2 Autocontención | T01, T14, T50 |
| RNF-3 Dependencias | T02, T14, T50 |
| RNF-4 Idioma | T53 |
| RNF-5 Verificabilidad | T03–T13, T14, T50 |
| RNF-6 Trazabilidad | T01, T52 |
| RNF-7 Legibilidad didáctica | T53 |
| RNF-8 Tiempo de ejecución | T51 |
| RNF-9 Explicabilidad | T34, T49, T53 |

---

## Registro de ejecución (2026-10-13)

### Resultado

- **Notebook:** `notebooks/Tarea1/01_revml.ipynb`, 20 celdas, 9 de código ejecutadas sin error, 6 figuras. Ejecución completa ≈ 50 s.
- **Reproducibilidad:** dos ejecuciones desde cero (kernel nuevo) dan salidas, tablas y figuras idénticas; huellas E1 `689602a2376f3020`, E2 `db54a3c0be16c4a4`, E3 `762c72766081749c`.
- **Predicciones:** huellas sha256 de E1.P `b35dce31305dc797`, E2.P `4e55ff4718222f40` y E3.P `039d27c91664a288`, calculadas antes de ejecutar ninguna celda de implementación y verificadas idénticas al cierre.
- **Prueba:** una sola evaluación por modelo (`ledger` con 7 y 8 modelos); `RUN_TEST` se ejecutó primero en `False` hasta congelar y se activó sin tocar configuraciones.

### Historial de git

- **Rama:** `tarea1-revml` (creada desde `main`, que no se tocó). Los cambios previos del árbol de trabajo (`.gitignore`, `requirements.txt`) no se commitearon.
- **Reconstrucción declarada:** los commits se crearon el 2026-10-13, **después** de ejecutar por primera vez el notebook. Se rehízo la secuencia en el orden del protocolo (cada predicción commiteada antes de que exista su código, cada configuración congelada con `RUN_TEST = False` antes de evaluar en prueba) y cada etapa se ejecutó de verdad. La evidencia original de que las predicciones no se editaron tras ejecutar es la huella sha256 registrada antes de la primera ejecución.
- **Verificado sobre el historial:** cada commit cambia solo la celda que dice; E1.P, E2.P y E3.P tienen una única versión desde que se introducen y su sha256 coincide con el registrado; entre cada «congelado» y su «prueba» cambia una sola línea del código (`RUN_TEST`); el notebook final coincide con el verificado (fuentes, tablas y figuras).

| Commit | Contenido |
|---|---|
| `967d72d` | Docs: constitución, spec 001 y plan técnico |
| `20625b8` | S0–S6: soporte con funciones y comprobaciones |
| `7c00f7d` → `944c5d1` → `efdcb37` | E1.P → E1.C → E1.I |
| `47c8e31` → `31fe25e` → `a9c5763` → `a33abfd` | E2.P → E2.C congelado (`RUN_TEST=False`) → E2.C prueba (`RUN_TEST=True`) → E2.I |
| `2fdee92` → `a751373` → `fb21be0` → `3700c85` | E3.P → E3.C congelado → E3.C prueba → E3.I |

### Limitaciones que siguen abiertas

- **T53 y la legibilidad (RNF-7) son auto-revisión.** Ninguna persona sin contexto de Recuperación de Información ha leído el notebook; conviene que lo haga el autor o un par.
- **«Restart & Run All»** se emuló con `nbconvert --execute` en kernel nuevo (equivalente, no literal).
- Las predicciones y las interpretaciones las redactó la IA a petición del autor; la práctica exige que el autor pueda explicarlas.

### Desviaciones respecto de `plan.md` (todas de forma, ninguna del protocolo)

- `result_fingerprint` se llama **`result_digest`**: el nombre contenía el texto `print(` y hacía fallar la búsqueda literal exigida por la constitución.
- Funciones añadidas al inventario del plan: `classification_metrics`, `regression_metrics`, `train_and_score`, `never_predicted_classes`, `make_table`, `format_warnings` (29 en total, no 23). Todas con `assert`.
- Firmas ajustadas: `freeze(key, record, n_configs)`, `evaluate_once(frozen, X_eval, y_eval, ledger, scorer)`, `simplicity_score(frozen)`, `alpha_sweep(key, base_config, alphas, data, seed)` y `pick_two_errors` con parámetros genéricos (`*_main`, `*_extra`) para no usar `y_test` en S5 (I-5).
- Los `assert` de partición interna se escriben por modelo (`early_stopping` falso en el MLP; `n_iter_no_change is None` en Gradient Boosting), según lo comprobado en scikit-learn 1.9.1.
- Ridge y Lasso usan una única configuración de hiperparámetros distintos de alpha (`NON_ALPHA_CONFIGS`), por lo que su `N.º` es 1 y el barrido es la excepción del límite.
- Se añadió la tabla de referencia T2.12b (medias por clase) para fundamentar la lectura de los errores; es descriptiva y no modifica configuraciones.
- Las celdas se redactaron en archivos temporales fuera del repositorio y se ensamblaron en el notebook con un script temporal; no se añadió ningún archivo al repositorio.
- Defectos hallados al **ver** las figuras y las tablas (no detectados por los asserts): el signo de `Error` en T3.9 era opuesto al del residuo de F3.4 (corregido a real − predicción) y el índice de T1.5 empezaba en 0 (ahora 1, como el resto). Se añadió a E2.I la lectura de F2.1 y a E3.I la evidencia de F3.1 (lectura aproximada) sobre coeficientes que cambian de signo.

### Resultados principales (para el lector con prisa)

- **E2 (Iris):** seis modelos empatan en prueba con 2 errores (accuracy 0.9333); la SVM lineal tiene 4. El mejor observado, Gaussian Naive Bayes, lo es solo por el desempate de orden. Las dos instancias de error fallan en los siete modelos.
- **E3 (Diabetes):** mejor observado MLP (RMSE 51.66, R² 0.4964), con cinco modelos no distinguibles a ≤ 5 %. La predicción (a) del autor queda contradicha y la de la brecha entrenamiento-validación también; el resto de (b) se confirma.

### Matriz requisito ↔ celda

| Requisito | Dónde se implementa |
|---|---|
| RF-1 a RF-7 (transversales) | Las tres celdas `P → C → I` de E1, E2 y E3; reglas de proceso en S0 y comprobaciones en S6 |
| RF-2 Particiones y semilla | E1.C `[E1.5]` (T1.5, Iris) y E3.C `[E3.2]` (T3.2, Diabetes); `SEED` en S1 |
| RF-8 Descripción de Iris | E1.P (CA-8.4), E1.C `[E1.1]–[E1.4]` (T1.1–T1.4, F1.1), E1.I (CA-8.5) |
| RF-9 Predicciones del Bloque 2 | E2.P |
| RF-10 Siete clasificadores | E2.C `[E2.0]–[E2.2]` (T2.2 = diagnóstico sin estandarizar) |
| RF-11 Comparación en validación | E2.C `[E2.1]` (T2.1) y `[E2.3]` (T2.3) |
| RF-12 Evaluación en prueba | E2.C `[E2.5]` (T2.5, T2.6) |
| RF-13 Matrices de confusión | E2.C `[E2.5]` (F2.1) |
| RF-14 Dos errores | E2.C `[E2.8]` (T2.12, T2.12b); E2.I sección 6 |
| RF-15 Interpretación del Bloque 2 | E2.C `[E2.6]` (T2.7–T2.9); E2.I secciones 1 a 5 y 8 |
| RF-16 Descripción de Diabetes | E3.C `[E3.1]` (T3.1) y `[E3.3]` |
| RF-17 Predicción de flexibilidad | E3.P (a); E3.I sección 8 |
| RF-18 Ocho regresores | E3.C `[E3.0]` y `[E3.5]` (T3.4) |
| RF-19 Evaluación en prueba (regresión) | E3.C `[E3.7]` (T3.6) |
| RF-20 Diagnóstico de regresión | E3.C `[E3.9]` (F3.3, F3.4, T3.9) |
| RF-21 Regularización | E3.P (b); E3.C `[E3.4]` (T3.3, F3.1, F3.2) |
| RF-22 Interpretación del Bloque 3 | E3.C `[E3.8]` (T3.7, T3.8, T3.10); E3.I secciones 1 a 9 |

### Criterios de finalización de `spec.md` §8

| Criterio | Evidencia |
|---|---|
| Run All sin errores, resultados idénticos, < 5 min | T51 |
| Tres experimentos con tres celdas en orden y sección de soporte | T01, T52 |
| RF-8 a RF-22 implementados; RF-1 a RF-7 transversales | matriz de cobertura, T52 |
| Particiones y semilla antes de entrenar; congelado antes de prueba; prueba única | T18, T21–T26, T36–T43 (historial de git en la rama `tarea1-revml`) |
| Entregables presentes y generados | T16–T18, T22–T31, T37–T46 |
| Preguntas de interpretación respondidas y predicciones contrastadas | T19, T32–T34, T47–T49 |
| Análisis de dos errores con lo concluible y lo hipotético | T30, T34 |
| `assert` por función; sin `print`/`display`/`plt.` en funciones | T14, T50 |
| Sin archivos de código adicionales, dependencias nuevas ni E/S de archivos | T50 |
| Idioma, legibilidad y decisiones justificadas | T53 |
