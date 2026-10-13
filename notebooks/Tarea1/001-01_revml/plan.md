# Plan técnico — Spec 001: Revisión práctica de Machine Learning

| | |
|---|---|
| **Estado** | Aprobado |
| **Fecha** | 2026-10-13 |
| **Entradas** | Constitución del proyecto y Spec 001 (únicas fuentes) |
| **Alcance de este documento** | El CÓMO: estructura de celdas, datos, protocolo, contratos, decisiones y verificación. No contiene código de producción. |

**Convenciones.** `[RF-x]`, `[CA-x.y]`, `[RNF-n]`, `[CL-n]`, `[Dn]` remiten a la spec; `[P n]` remite al principio n de la constitución. Cada elemento técnico indica el requisito que cubre. Identificadores y comentarios técnicos del notebook en inglés; celdas Markdown y documentación en español [P 6, RNF-4].

---

## 1. Estructura de celdas del notebook

Toda la implementación vive en `01_revml.ipynb`; no hay módulos, paquetes, scripts ni archivos de datos [P 3, P 5, D7, RNF-2]. La «estructura de celdas» sustituye a la estructura de módulos.

### 1.1 Orden de celdas

| ID | Tipo | Contenido | Cubre |
|---|---|---|---|
| C0 | MD (existente) | Título, propósito, objetivos, uso de IA, organización | — |
| S0 | MD | Sección de soporte: qué contiene, que no es un experimento y mapa de funciones | CA-1.4 |
| S1 | Code | Imports del stack, `SEED`, constantes del protocolo, orden de modelos y etiquetas en español | CA-2.5, RNF-3, P 1 |
| S2 | Code | Lógica de datos y particiones | RF-2, RF-8 |
| S3 | Code | Fábricas de modelos (clasificadores y regresores) | RF-4, RF-10, RF-18, CA-2.6 |
| S4 | Code | Lógica de protocolo: ajuste, selección, congelado, evaluación, barrido, avisos | RF-3, RF-7, RF-21 |
| S5 | Code | Lógica de análisis: métricas, comparación contra el mejor, errores, huella | RF-12…RF-15, RF-19…RF-22 |
| S6 | Code | Comprobaciones (`assert`) de toda la lógica de S2–S5 | RNF-5, P 4 |
| B1 | MD (existente) | Enunciado del bloque 1 | — |
| E1.P | MD | Predicción inicial de E1 | RF-1, RF-6, CA-8.4 |
| E1.C | Code | Implementación y resultados de E1 | RF-8, RF-2 |
| E1.I | MD | Interpretación de E1 | CA-8.5, CA-6.3 |
| B2 | MD (existente) | Enunciado del bloque 2 | — |
| E2.P | MD | Predicciones de E2 y criterio de sencillez | RF-9 |
| E2.C | Code | Implementación y resultados de E2 | RF-10…RF-14 |
| E2.I | MD | Interpretación de E2 | RF-14, RF-15 |
| B3 | MD (existente) | Enunciado del bloque 3 | — |
| E3.P | MD | Predicciones de E3 (flexibilidad y regularización) | RF-17, CA-21.1 |
| E3.C | Code | Implementación y resultados de E3 | RF-16, RF-18…RF-21 |
| E3.I | MD | Interpretación de E3 | RF-22 |

Las celdas de relleno actuales (`# Aquí cumple con el primer paso`, las celdas de código vacías y la celda Markdown vacía) se **reemplazan** por las celdas anteriores. Cada experimento es exactamente `P → C → I`, tres celdas consecutivas y en ese orden, inmediatamente después de su bloque de enunciado [CA-1.1, P 2]. Las celdas S0–S6 no son experimentos y no se rigen por CA-1.1 [CA-1.4].

### 1.2 Invariantes de estructura (verificables por inspección)

| ID | Invariante | Cubre |
|---|---|---|
| I-1 | Las `def` aparecen solo en S2–S5; las celdas `E*.C` no definen funciones. | P 3, RNF-5 |
| I-2 | S2–S5 no contienen `print`, `display` ni `plt.`. Mostrar y graficar ocurre solo en `E*.C`. | P 3, RNF-5 |
| I-3 | No aparece `open`, `read_csv`, `to_csv`, `savefig` ni `joblib.dump` en ninguna celda. | P 5, RNF-2 |
| I-4 | Los `import` están solo en S1 y pertenecen al stack o a la biblioteca estándar. No se importa `IPython` (`display` es el integrado del kernel). | P 1, RNF-3 |
| I-5 | `X_test` / `y_test` aparecen solo en `E1.C` (declaración y tamaños), `E2.C` y `E3.C` (sección de prueba). Las funciones de S2–S5 reciben `X_eval` / `y_eval` genéricos. | CA-3.5, CA-3.6, CA-21.5 |
| I-6 | Toda fuente de aleatoriedad usa `SEED`; no aparecen `np.random.seed` ni `random_state=<número>`. | CA-2.5, RNF-1 |
| I-7 | No se usa `early_stopping`, `validation_fraction`, `n_iter_no_change` ni validación cruzada. | CA-2.6, §7 spec |

### 1.3 Inventario de funciones de lógica (S2–S5)

Ninguna imprime, muestra ni grafica; devuelven valores (listas, diccionarios, DataFrames, arrays). Cada una tiene al menos un `assert` en S6 [P 3, P 4, RNF-5].

| Celda | Función | Entrada → salida | Cubre |
|---|---|---|---|
| S2 | `make_splits(y, stratify, seed)` | etiquetas → índices `train/val/test` (60/20/20, dos pasos) | CA-2.1…2.3 |
| S2 | `split_summary(y, splits, class_names)` | → DataFrame de tamaños y conteo por clase | CA-2.4 |
| S2 | `sample_examples(y, n, seed)` | → 5 índices con las 3 clases presentes | CA-8.2 |
| S3 | `build_classifier(key, config, seed, scaled)` | → estimador (Pipeline con `StandardScaler` si `scaled`) | CA-10.2, CA-10.4, CA-4.1 |
| S3 | `build_regressor(key, config, seed)` | → estimador (Pipeline; `TransformedTargetRegressor` solo en MLP) | CA-18.2, CA-4.2 |
| S4 | `fit_capturing_warnings(est, X, y)` | → (estimador ajustado, lista de avisos) | CA-7.1 |
| S4 | `score_classifier(est, X_eval, y_eval)` | → predicciones, métricas, avisos | CA-11.1, CA-12.1 |
| S4 | `score_regressor(est, X_eval, y_eval)` | → predicciones, MAE, RMSE, R², avisos | CA-19.1 |
| S4 | `select_config(records, primary, support, higher_is_better)` | → registro elegido (con desempate) | CA-3.1, CA-3.7 |
| S4 | `alpha_sweep(key, base_config, alphas, splits_data, seed)` | → filas del barrido | CA-21.2, CA-21.3 |
| S4 | `pick_best_alpha(rows)` | → alpha elegido y `at_border` | CA-21.4 |
| S4 | `freeze(key, record)` | → `FrozenModel` inmutable | CA-3.4 |
| S4 | `evaluate_once(frozen, X_eval, y_eval, ledger)` | → resultado; falla si el modelo ya figura en `ledger` | CA-3.5, CA-19.2 |
| S5 | `format_config(config)` | → texto con solo los hiperparámetros fijados explícitamente | CA-3.8, CA-12.1 |
| S5 | `simplicity_score(frozen, n_configs)` | → hiperparámetros fijados + configuraciones probadas | CA-15.2 |
| S5 | `indistinguishable_cls(errors, best, tol=1)` | → modelos a ≤ 1 error del mejor | CA-15.1 |
| S5 | `indistinguishable_reg(rmse, best, tol=0.05)` | → modelos a ≤ 5 % del RMSE del mejor | CA-22.1 |
| S5 | `rank_models(values, higher_is_better)` | → posiciones (empates con el mismo puesto) | CA-15.3, CA-22.1 |
| S5 | `is_degenerate_cls(y_pred, classes)` / `is_degenerate_reg(est)` | → bool (clase nunca predicha / todos los coeficientes en cero) | CA-7.3 |
| S5 | `pick_two_errors(...)` | → hasta dos errores rotulados (prueba, luego validación) | CA-14.1…14.3 |
| S5 | `top_abs_errors(y_true, y_pred, k=2)` | → índices con mayor error absoluto | CA-20.3 |
| S5 | `result_fingerprint(*arrays)` | → hash `hashlib` de valores redondeados | RNF-1 |

---

## 2. Modelo de datos

Los datasets se cargan con `load_iris` y `load_diabetes` de scikit-learn, sin descargas [RNF-2, P 5]. Para modelar se usan arreglos NumPy (`X`, `y`); pandas se usa solo para tablas de presentación (DT-3).

### 2.1 Iris — clasificación [RF-8, RF-2]

| Aspecto | Valor |
|---|---|
| Qué es una instancia | Una flor de iris (una observación) |
| `X` | `ndarray` `float64`, forma `(150, 4)` |
| `y` | `ndarray` `int64`, forma `(150,)`, valores `{0, 1, 2}` |
| Características (nombre de la librería → significado) | `sepal length (cm)` → longitud del sépalo; `sepal width (cm)` → ancho del sépalo; `petal length (cm)` → longitud del pétalo; `petal width (cm)` → ancho del pétalo |
| Clases (nombre de la librería) | `setosa` (0), `versicolor` (1), `virginica` (2); 50 ejemplos cada una |
| Partición (60/20/20, estratificada) | entrenamiento 90, validación 30, prueba 30; 30/10/10 por clase en cada una |

Ejemplo de filas de `X` e `y`:

```
X[0]   = [5.1, 3.5, 1.4, 0.2]   y[0]   = 0   (setosa)
X[50]  = [7.0, 3.2, 4.7, 1.4]   y[50]  = 1   (versicolor)
X[100] = [6.3, 3.3, 6.0, 2.5]   y[100] = 2   (virginica)
```

Los nombres de clase se convierten con `.tolist()` antes de mostrarlos (con NumPy 2 el repr de `np.str_` ensucia las tablas) [RNF-7].

### 2.2 Diabetes — regresión [RF-16, RF-18]

| Aspecto | Valor |
|---|---|
| Qué es una instancia | Un paciente con diabetes |
| `X` | `ndarray` `float64`, forma `(442, 10)` |
| `y` | `ndarray` `float64`, forma `(442,)`; medida cuantitativa de progresión de la enfermedad un año después de la línea base; rango 25–346 |
| Variables | `age`, `sex`, `bmi`, `bp`, `s1`…`s6` (edad, sexo, índice de masa corporal, presión arterial media y seis mediciones de suero) |
| Transformación de origen | Cada columna de `X` viene centrada y escalada (media ≈ 0, suma de cuadrados = 1) con estadísticos del conjunto completo |
| Partición (60/20/20, sin estratificar) | entrenamiento 264, validación 89, prueba 89 |

Ejemplo de fila de `X` e `y` (valores tal como vienen; **no** son unidades originales) [CA-16.3, CL-11]:

```
X[0] = [ 0.038076,  0.050680,  0.061696,  0.021872, -0.044223,
        -0.034821, -0.043401, -0.002592,  0.019907, -0.017646]
y[0] = 151.0
```

### 2.3 Estructuras internas (tipos de datos del protocolo)

| Estructura | Forma | Ejemplo | Cubre |
|---|---|---|---|
| `config` | `dict` solo con los hiperparámetros que el autor escribe; **sin** la semilla | `{"C": 1.0, "max_iter": 1000}` | CA-3.8, D8 |
| `record` | `dict` por configuración probada | `{"model": "logistic_regression", "order": 0, "config": {...}, "f1_macro": 0.93, "accuracy": 0.93, "warnings": [], "degenerate": False, "estimator": <ajustado en train>}` | CA-3.3 |
| `FrozenModel` | `NamedTuple(model, config, estimator, n_configs)`; inmutable | — | CA-3.4, D9 |
| `ledger` | `set` de modelos ya evaluados en prueba | `{"gaussian_nb", ...}` | CA-3.5 |
| `splits` | `dict` con índices `"train"`, `"val"`, `"test"` | tamaños 90/30/30 | CA-2.1, CA-2.3 |

---

## 3. Pseudocódigo del protocolo experimental

### 3.1 Orden obligatorio (idéntico en E2 y E3)

```
 0. SEED fijada en S1; ninguna otra fuente de aleatoriedad                        [CA-2.5]
 1. Cargar el dataset                                                              [RF-8, RF-16]
 2. Declarar la partición train/val/test (60/20/20) y mostrar tamaños              [RF-2]
 3. Declarar (antes de cualquier fit): configuración inicial y alternativas
    por modelo (≤ 4 contando la inicial), métricas de decisión y desempate;
    en E3 además rejillas de alpha                                                 [CA-3.2, CA-3.7, CA-3.8, CA-21.2]
 4. Para cada modelo: ajustar cada configuración SOLO con train y puntuar en val   [CA-3.1]
 5. Elegir la configuración por métrica de val con desempate declarado             [CA-3.7, D10]
 6. Congelar: configuración + modelo ajustado solo con train; sin reentrenar        [CA-3.4, D9]
 7. Comprobar que el congelado reproduce los números de val del registro
 8. SI RUN_TEST: evaluar cada modelo congelado una única vez sobre la misma prueba [CA-3.5, CA-12.2]
 9. Construir tablas, figuras y análisis a partir de esas predicciones (sin re-predecir)
10. Escribir la interpretación (celda I) con base en lo mostrado
```

### 3.2 Núcleo común

```
FUNCTION run_protocol(task, X, y, model_keys, initial, alternatives, metric_cfg, run_test):
    splits ← make_splits(y, stratify = (task == "classification"), seed = SEED)
    ASSERT disjoint(splits) AND union(splits) == all_indices
    X_tr, y_tr ← X[splits.train], y[splits.train]
    X_va, y_va ← X[splits.val],   y[splits.val]

    FOR key IN model_keys:                                    # orden CA-10.1 / CA-18.1
        configs ← [initial[key]] + alternatives[key]          # declaradas antes de cualquier fit
        ASSERT len(configs) ≤ MAX_CONFIGS AND all_distinct(configs)
        FOR order, config IN enumerate(configs):
            est ← build(key, config, SEED)                    # Pipeline con scaler cuando corresponde
            est, w_fit ← fit_capturing_warnings(est, X_tr, y_tr)
            metrics, y_pred_va, w_pred ← score(est, X_va, y_va)
            records[key].append({order, config, metrics, w_fit + w_pred,
                                 degenerate(y_pred_va or est), est})
        chosen ← select_config(records[key], primary, support)   # solo val
        frozen[key] ← freeze(key, chosen)                        # sin refit
        ASSERT rescore(frozen[key], X_va, y_va) == chosen.metrics

    IF NOT run_test: RETURN records, frozen                    # fase de ajuste
    ledger ← ∅
    FOR key IN model_keys:
        result[key] ← evaluate_once(frozen[key], X[splits.test], y[splits.test], ledger)
    ASSERT len(ledger) == len(model_keys)
    RETURN records, frozen, result
```

### 3.3 E2 — clasificación (específico)

```
scaled ← True para los siete modelos finales                              [CA-10.2]
diagnostic ← FOR key: build(key, initial[key], SEED, scaled = False)       # solo val; no cuenta como configuración;
              score on X_va → accuracy, F1 macro                           #   no cambia la elegida; no toca prueba [CA-10.4]
primary ← F1 macro (val); support ← accuracy (val); ties → primera probada [D10, CA-3.7]
best_obs ← argmax F1 macro de prueba (empate → accuracy → orden CA-10.1)   # se rotula «mejor observado, no seleccionado» [CA-15.1]
errors_by_model[m] ← #{i : pred_test[m][i] ≠ y_test[i]}
not_distinguishable ← {m : errors_by_model[m] − errors_by_model[best_obs] ≤ 1}   # contra el mejor, no entre pares [D11]
simplicity[m] ← len(frozen[m].config) + len(records[m])                   # menor = más sencillo [CA-15.2, CA-9.4]
```

Selección de los dos errores [RF-14, D2, CL-1…CL-2] ⚑:

```
FUNCTION pick_two_errors(pred_test, pred_val, y_test, y_val, model_order):
    chosen ← []
    test_errs ← {i : ∃ m, pred_test[m][i] ≠ y_test[i]}                    # instancias distintas, unión de los 7 modelos
    ORDER test_errs BY (−n_models_failing(i), original_index(i))          # ⚑ regla del plan
    FOR i IN test_errs WHILE len(chosen) < 2:
        chosen.append({i, partition = "prueba",
                       model = first m in model_order that fails on i,    # ⚑
                       also_failed = n_models_failing(i) − 1})
    IF len(chosen) < 2:                                                    # CA-14.2
        FOR m IN models ORDER BY (−val_errors(m), model_order):
            FOR i IN val_errors_of(m) ORDER BY original_index WHILE len(chosen) < 2:
                IF i NOT IN chosen: chosen.append({i, partition = "validación", model = m,
                                                   note = "posible sesgo optimista"})
    IF chosen == []: declarar ausencia de errores y su limitación           # CA-14.3
    RETURN chosen
```

### 3.4 E3 — regresión y barrido (específico)

```
declare ALPHA_GRID["ridge"] ← logspace(-2, 4, 13)                          # ⚑ declaradas antes de ver resultados [CA-21.2]
declare ALPHA_GRID["lasso"] ← logspace(-3, 2, 11)                          # debe llegar a «todos los coeficientes en cero»
ASSERT len(grid) ≥ 8 AND ratios_constantes(grid)                           # log-espaciada

FOR key IN ("ridge", "lasso"):
    FOR base IN non_alpha_configs[key]:                                    # normalmente 1; ≤ MAX_CONFIGS [CA-3.2]
        FOR alpha IN ALPHA_GRID[key]:                                      # barrido: excepción única al límite [D8]
            est ← build_regressor(key, {**base, "alpha": alpha}, SEED)
            est, w ← fit_capturing_warnings(est, X_tr, y_tr)
            rows.append({key, alpha, l2_norm(coef), n_zero(coef) si Lasso,
                         rmse(y_tr, est(X_tr)), rmse(y_va, est(X_va)), w})   # [CA-21.3]
    best ← argmin rmse_val; empate exacto → menor alpha                    # [D3, CA-21.4]
    at_border ← best.alpha IN (grid[0], grid[-1])                          # declarar y explicar; NO extender [CA-21.4, CL-15]
    ASSERT para Lasso: n_zero(coef en grid[-1]) == n_features             # [CL-8]
    chosen[key] ← best                                                     # pasa al paso 6 del protocolo (congelado)

# Prueba solo con RUN_TEST; el barrido nunca toca prueba                    [CA-21.5]
target transform: solo MLP, TransformedTargetRegressor(StandardScaler)     # ajustada con train, revertida antes de métricas [CA-4.2]
                  resto: sin transformar; se declara                       # [CA-4.3]
best_obs ← argmin RMSE de prueba (empate → MAE → orden CA-18.1)
not_distinguishable ← {m : rmse[m] ≤ 1.05 · rmse[best_obs]}                # sobre el RMSE del mejor [D11, CA-22.1]
baseline ← R²(y_test, constante = media(y_train))                          # ASSERT ≤ 0 [CA-22.3]
ASSERT orden por R² == orden por RMSE                                      # misma partición [CA-22.1]
two_largest ← top_abs_errors(y_test, pred_test[m], 2) para LR y MLP        # escala original [CA-20.3]
```

---

## 4. Contrato de cada experimento

Cada experimento es `P → C → I` [CA-1.1]. La celda `C` es **una sola** y se organiza con banners `# [E2.k] …`. Las tablas se generan de resultados, nunca se transcriben [RF-5]. Los encabezados, títulos y ejes van en español [RNF-4].

### 4.1 E1 — Datos y protocolo (Iris)

**Cubre:** RF-8, RF-2 (Iris). Transversales: RF-1, RF-5, RF-6.

| Celda | Contrato |
|---|---|
| **E1.P** | Predicción sobre el tamaño del dataset y el balance entre clases, registrada antes de cargar los datos [CA-8.4, CA-6.1]. No se edita tras ejecutar [CA-6.2]. |
| **E1.C** | `[E1.1]` cargar con `load_iris`; asserts de forma `X (150, 4)`, `y (150,)` [CA-8.1]. `[E1.2]` fichas y tablas T1.1–T1.3. `[E1.3]` cinco ejemplos al azar con `SEED` y las tres clases → T1.4 [CA-8.2]. `[E1.4]` figura F1.1 [CA-8.3]. `[E1.5]` declarar la partición con `make_splits` y mostrar T1.5 [CA-8.6, CA-2.1…2.4]. `[E1.6]` huella. |
| **E1.I** | Indica si las clases están balanceadas y qué implica para elegir métricas [CA-8.5]. Contrasta la predicción: confirmada o contradicha, y por qué [CA-6.3]. Cita cifras con su tabla o figura [CA-5.3]. |

**Salidas:**

| ID | Tipo | Columnas / contenido exacto | Cubre |
|---|---|---|---|
| T1.1 | Tabla | `Concepto` · `Valor` (instancia, n.º de instancias, n.º de características, forma de X, forma de y) | CA-8.1 |
| T1.2 | Tabla | `Característica (nombre de la librería)` · `Significado` | CA-8.1 |
| T1.3 | Tabla | `Clase` · `Ejemplos` | CA-8.1 |
| T1.4 | Tabla | `Índice` · `sepal length (cm)` · `sepal width (cm)` · `petal length (cm)` · `petal width (cm)` · `Clase` (5 filas) | CA-8.2 |
| F1.1 | Figura | Barras de ejemplos por clase (conjunto completo); título y ejes en español | CA-8.3 |
| T1.5 | Tabla | `Partición` · `Tamaño` · `setosa` · `versicolor` · `virginica` (entrenamiento, validación, prueba) | CA-2.4, CA-8.6 |

### 4.2 E2 — Comparación de clasificadores (Iris)

**Cubre:** RF-9…RF-15. Transversales: RF-1…RF-7.

| Celda | Contrato |
|---|---|
| **E2.P** | Tres predicciones: (1) qué modelos son sensibles a la escala [CA-9.1]; (2) qué modelo da fronteras más simples (conceptual) [CA-9.2]; (3) por qué no puede saberse el ganador [CA-9.3]. Además declara el criterio de «más sencillo de programar»: menor suma de hiperparámetros fijados explícitamente + configuraciones probadas [CA-9.4, D6]. |
| **E2.C** | `[E2.0]` declaración: `INITIAL`, `ALTERNATIVES` (≤ 3 por modelo), `RUN_TEST` — antes de cualquier `fit` [CA-3.8]. `[E2.1]` validación con la inicial → T2.1 [RF-11]. `[E2.2]` diagnóstico sin estandarizar → T2.2 [CA-10.4]. `[E2.3]` ajuste limitado → T2.3 [CA-3.3]. `[E2.4]` congelado → T2.4 [CA-3.4]. `[E2.5]` prueba única → T2.5, T2.6, F2.1 [RF-12, RF-13]. `[E2.6]` análisis → T2.7–T2.9 [RF-15]. `[E2.7]` avisos y degenerados → T2.10–T2.11 [RF-7]. `[E2.8]` errores → T2.12 [RF-14]. `[E2.9]` huella. |
| **E2.I** | Responde: (1) mejor desempeño con «mejor observado, no seleccionado» y lista de no distinguibles, siempre «en esta partición» [CA-15.1, CA-15.7]; (2) más sencillo según el criterio declarado [CA-15.2]; (3) accuracy vs F1 macro [CA-15.3]; (4) macro vs micro, incluido micro = accuracy [CA-15.4]; (5) MLP vs lineales sin afirmar ventaja general [CA-15.5]. Contrasta las tres predicciones usando T2.2 para CA-9.1 [CA-15.6, CA-6.3]. Separa lo concluible de la hipótesis sobre la causa de los errores [CA-14.6]. Comenta cada aviso y resultado degenerado [CA-7.1, CA-7.3, CA-7.4]. Justifica la configuración elegida, el desempate y la elección de errores [RNF-9]. |

**Salidas:**

| ID | Tipo | Columnas / contenido exacto | Cubre |
|---|---|---|---|
| T2.1 | Tabla | `Modelo` · `Accuracy de validación` · `F1 macro de validación` (7 filas, configuración inicial) | CA-11.1 |
| T2.2 | Tabla | `Modelo` · `Accuracy sin estandarizar (validación)` · `F1 macro sin estandarizar (validación)` · `F1 macro estandarizado (validación)` · `Diferencia de F1 macro` | CA-10.4 |
| T2.3 | Tabla | `Modelo` · `N.º` · `Hiperparámetros` · `Accuracy de validación` · `F1 macro de validación` · `Avisos` · `Elegida` | CA-3.3, CA-11.2 |
| T2.4 | Tabla | `Modelo` · `Transformación` · `Hiperparámetros fijados` · `Configuraciones probadas` | CA-3.4 |
| **T2.5** | Tabla | **`Modelo` · `Transformación` · `Hiperparámetros principales` · `Accuracy de prueba` · `F1 macro de prueba`** (7 filas) | CA-12.1, CA-12.2 |
| T2.6 | Tabla | `Modelo` · `F1 micro de prueba` · `F1 macro de prueba` | CA-12.3 |
| F2.1 | Figura | Tres matrices de confusión de prueba (Naive Bayes, SVM lineal, MLP); filas y columnas con los nombres de clase de la librería; conteos | CA-13.1, CA-13.2 |
| T2.7 | Tabla | `Modelo` · `Errores de prueba` · `Diferencia con el mejor observado` · `No distinguible del mejor (≤ 1 error)` | CA-15.1, D11 |
| T2.8 | Tabla | `Modelo` · `Posición por F1 macro` · `Posición por accuracy` · `¿Coincide?` | CA-15.3 |
| T2.9 | Tabla | `Modelo` · `Hiperparámetros fijados` · `Configuraciones probadas` · `Suma (menor = más sencillo)` | CA-15.2 |
| T2.10 | Tabla | `Modelo` · `Etapa (ajuste / validación / prueba)` · `Categoría` · `Mensaje` | CA-7.1, CA-7.4 |
| T2.11 | Tabla | `Modelo` · `Clases nunca predichas (prueba)` · `Degenerado` | CA-7.3 |
| T2.12 | Tabla | `Instancia (índice original)` · `Partición` · `Modelo` · `Otros modelos que también fallaron` · 4 características originales · `Clase real` · `Clase predicha` · `Nota` | CA-14.4, CA-14.2 |

### 4.3 E3 — Regresión y regularización (Diabetes)

**Cubre:** RF-16…RF-22, RF-2 (Diabetes). Transversales: RF-1…RF-7.

| Celda | Contrato |
|---|---|
| **E3.P** | **Una sola celda** con dos predicciones: (a) si un modelo más flexible produce siempre menor error en datos nuevos, declarando qué modelos considera más flexibles [CA-17.1]; (b) cómo cambian coeficientes y errores al aumentar la penalización [CA-21.1]. |
| **E3.C** | `[E3.0]` declaración: `INITIAL`, `ALTERNATIVES`, `NON_ALPHA_CONFIGS`, rejillas de alpha, `RUN_TEST` [CA-3.8, CA-21.2]. `[E3.1]` carga y descripción → T3.1, comprobación de transformación [RF-16]. `[E3.2]` partición 60/20/20 sin estratificar → T3.2 [CA-18.3]. `[E3.3]` declaración de limitación y de transformación del objetivo [CA-16.4, CA-4.3]. `[E3.4]` barrido → T3.3, F3.1, F3.2 [RF-21]. `[E3.5]` ajuste limitado de los demás → T3.4 [CA-3.3]. `[E3.6]` congelado → T3.5 [CA-3.4, CA-21.4]. `[E3.7]` prueba única → T3.6 [RF-19]. `[E3.8]` análisis → T3.7, T3.8, T3.10 [RF-22]. `[E3.9]` diagnóstico → F3.3, F3.4, T3.9 [RF-20]. `[E3.10]` avisos y degenerados → T3.11, T3.12 [RF-7]. `[E3.11]` huella. |
| **E3.I** | Responde las seis preguntas [RF-22]: (1) mejor desempeño por RMSE, MAE de apoyo, R² y no distinguibles ≤ 5 % [CA-22.1]; (2) MAE vs RMSE [CA-22.2]; (3) R² negativo [CA-22.3]; (4) efecto de la regularización con base en el barrido [CA-22.4]; (5) explicaciones de errores grandes, marcando hipótesis [CA-22.5]; (6) causalidad con al menos un motivo concreto [CA-22.6]. Contrasta RF-17 reformulada en observable y RF-21 [CA-17.2, CA-22.7, CA-6.3]. Declara si el óptimo cayó en el borde [CA-21.4]. |

**Salidas:**

| ID | Tipo | Columnas / contenido exacto | Cubre |
|---|---|---|---|
| T3.1 | Tabla | `Variable` · `Descripción (según el dataset)`, más líneas con el significado del objetivo y la comprobación de transformaciones | CA-16.1, CA-16.2 |
| T3.2 | Tabla | `Partición` · `Tamaño` (264 / 89 / 89) | CA-2.4, CA-18.3 |
| T3.3 | Tabla | `Modelo` · `Intensidad (alpha)` · `Norma L2 de los coeficientes` · `Coeficientes nulos` (solo Lasso) · `RMSE de entrenamiento` · `RMSE de validación` | CA-21.3 |
| F3.1 | Figura | Evolución de los coeficientes frente a alpha (escala log), Ridge y Lasso | CA-21.6 |
| F3.2 | Figura | RMSE de entrenamiento y validación frente a alpha (escala log), Ridge y Lasso | CA-21.6 |
| T3.4 | Tabla | `Modelo` · `N.º` · `Hiperparámetros` · `RMSE de validación` · `MAE de validación` · `Avisos` · `Elegida` | CA-3.3 |
| T3.5 | Tabla | `Modelo` · `Transformación` · `Hiperparámetros fijados` · `Configuraciones probadas` | CA-3.4 |
| **T3.6** | Tabla | **`Modelo` · `MAE` · `RMSE` · `R²`** (8 filas) | CA-19.1, CA-19.2 |
| T3.7 | Tabla | `Modelo` · `RMSE de prueba` · `Diferencia relativa con el mejor` · `No distinguible del mejor (≤ 5 %)` | CA-22.1, D11 |
| T3.8 | Tabla | `Modelo` · `Posición por RMSE` · `Posición por MAE` · `Posición por R²` | CA-22.1, CA-22.2 |
| F3.3 | Figura | Valores reales frente a predicciones de prueba: regresión lineal y MLP | CA-20.1 |
| F3.4 | Figura | Residuos de prueba: regresión lineal y MLP | CA-20.2 |
| T3.9 | Tabla | `Modelo` · `Instancia (índice original)` · `Valor real` · `Predicción` · `Error` · 10 variables tal como vienen (2 filas por modelo) | CA-20.3, CL-11 |
| T3.10 | Tabla | `Predictor` · `R² de prueba` (media de entrenamiento) | CA-22.3 |
| T3.11 | Tabla | `Modelo` · `Etapa` · `Categoría` · `Mensaje` | CA-7.1, CA-7.4 |
| T3.12 | Tabla | `Modelo` · `Degenerado` · `Motivo` (p. ej. todos los coeficientes en cero) | CA-7.3, CL-8, CL-16 |

### 4.4 Declaración de configuraciones (forma, no valores)

La spec reserva al autor elegir y declarar la configuración inicial [CA-3.8]; este plan solo fija la forma. Ejemplo ilustrativo **no vinculante**:

```
INITIAL["logistic_regression"]      = {"C": 1.0, "max_iter": 1000}
ALTERNATIVES["logistic_regression"] = [{"C": 0.1, "max_iter": 1000}, {"C": 10.0, "max_iter": 1000}]   # ≤ 3
```

La semilla no forma parte de `config` y se inyecta como `random_state = SEED` [CA-3.8, CA-2.5].

---

## 5. Contrato de «interfaz» del notebook

El notebook no tiene CLI ni API. Su interfaz es la ejecución completa:

| Aspecto | Contrato | Cubre |
|---|---|---|
| Entrada | Ninguna; datasets desde `load_iris` / `load_diabetes`; sin lectura ni escritura de archivos | RNF-2, P 5 |
| Ejecución válida | Solo *Restart & Run All*, de arriba abajo, con `RUN_TEST = True` | RNF-1, CL-14 |
| Salida correcta | Termina sin errores; aserciones superadas; tablas T*, figuras F* y huellas por experimento | RNF-1, §8 spec |
| Salida con fallo | Una aserción falla con mensaje en inglés que nombra la invariante violada; el notebook se detiene | RNF-5 |
| Tiempo | < 5 minutos en un equipo personal | RNF-8 |
| Avisos | Visibles en T2.10 y T3.11, nunca suprimidos | RF-7 |

---

## 6. Decisiones técnicas

| ID | Decisión | Justificación | Alternativa descartada | Cubre |
|---|---|---|---|---|
| DT-1 | Un solo notebook con sección de soporte (S0–S6) al inicio y experimentos `P → C → I`. | Es lo único que permiten D7, RNF-2 y CA-1.4; separa lógica de presentación. | Módulo `.py` o `utils`: prohibido por la constitución. | P 3, RNF-2 |
| DT-2 | Una **única** celda de código por experimento; funciones solo en soporte. | CA-1.1 exige tres celdas consecutivas; las funciones en soporte mantienen corta la celda. | Varias celdas de código por experimento: rompe la regla de tres celdas. | CA-1.1, P 2 |
| DT-3 | Modelado con arreglos NumPy; pandas solo para tablas. | Evita avisos de nombres de columnas entre arreglos y DataFrames y mantiene idéntico el formato de entrada. | `as_frame=True` de extremo a extremo. | RNF-1, RNF-7 |
| DT-4 ⚑ | Dos `train_test_split` encadenados (80/20 y luego 75/25 del resto) con `random_state = SEED`, `stratify = y` en Iris; `SEED = 42`. | Da 60/20/20 exactos y reproducibles (90/30/30 en Iris, 264/89/89 en Diabetes); 42 es solo una convención sin significado. | `StratifiedShuffleSplit` o k-fold: CV está fuera de alcance. | RF-2, D1 |
| DT-5 | `Pipeline(StandardScaler, modelo)` ajustado solo con entrenamiento; validación y prueba solo se transforman. | La transformación nunca ve datos de validación o prueba; es la forma estándar y verificable. | Estandarizar todo `X` antes de partir (fuga); escalado manual `(x−μ)/σ` (propenso a error y a fuga). | RF-4, CA-10.2, CA-18.2 |
| DT-6 | Barrido de alpha con bucle explícito sobre pipelines. | Cada fila del barrido queda auditable y con escalado de solo entrenamiento. | `RidgeCV`/`LassoCV`/`GridSearchCV` (validación interna, prohibida por CA-2.6 y fuera de alcance); `lasso_path` (oculta el escalado por pipeline). | CA-21.2, CA-2.6 |
| DT-7 | Avisos capturados con `warnings.catch_warnings(record=True)` + `simplefilter("always")` y mostrados en T2.10/T3.11. | El filtro por defecto solo muestra la primera aparición por ubicación y ocultaría repeticiones; capturar y mostrar mantiene todos visibles. | `filterwarnings("ignore")`: prohibido por CA-7.3; filtro por defecto: oculta repeticiones. | RF-7 |
| DT-8 | Registro de configuraciones y `FrozenModel` como `NamedTuple`; verificación de que el congelado reproduce la validación del registro. | Hace explícito y comprobable que no hubo reentrenamiento. | `dict` mutable sin comprobación. | CA-3.4, D9 |
| DT-9 ⚑ | `ledger` para la prueba única e interruptor `RUN_TEST` (falso mientras se ajusta, verdadero en el notebook final). | Con una sola celda por experimento, sin interruptor ver la tabla de prueba sería inevitable mientras se ajusta. | Confiar en la disciplina; separar en dos celdas de código (rompe CA-1.1). | CA-3.5, CA-3.6 |
| DT-10 | No hay funciones de graficado; las figuras se dibujan en línea en `E*.C`. | El principio 3 prohíbe que una función grafique. | Funciones `plot_*(ax, …)`: siguen graficando dentro de una función. | P 3, RNF-5 |
| DT-11 | Tablas con `DataFrame` y `display` integrado; sin `Styler` ni `to_markdown`. | `jinja2` y `tabulate` no están en el stack. | `df.style`, `to_markdown`, `from IPython.display import display`. | P 1, RNF-3 |
| DT-12 ⚑ | Objetivo transformado solo para el MLP, con `TransformedTargetRegressor(StandardScaler)`. | El MLP es sensible a la escala del objetivo (25–346); el envoltorio ajusta con entrenamiento y revierte antes de las métricas. Se declara que los demás no lo transforman. | Transformar el objetivo en todos los modelos; escalado manual de `y` (riesgo de fuga). | CA-4.2, CA-4.3 |
| DT-13 ⚑ | Rejillas: Ridge `logspace(-2, 4, 13)`, Lasso `logspace(-3, 2, 11)`, declaradas antes del barrido, con aserción de que Lasso llega a cero. | Ridge no anula coeficientes y necesita alpha grandes por no dividir entre n; en Lasso el alpha que anula todo es `max|Xᵀ(y−ȳ)|/n`, alrededor de 47 con el escalado de entrenamiento. | Rejilla única compartida (CA-21.2 pide una por modelo); extender la rejilla tras ver resultados (prohibido por CA-21.4). | CA-21.2, CL-8 |
| DT-14 ⚑ | `LinearSVC` como SVM lineal. | SVM lineal en forma primal; su `max_iter` es hiperparámetro explícito para tratar la no convergencia contando la corrección. | `SVC(kernel="linear")`: válida, pero kernelizada y con `max_iter=-1`. | CA-10.1, CL-7 |
| DT-15 | Sin trato especial al MLP: mismo pipeline, partición y sin `early_stopping`; los modelos de boosting con parada temprana desactivada. | La comparación MLP vs lineales solo es válida con el mismo protocolo; `early_stopping` crea una partición interna. | MLP con `early_stopping=True`; `HistGradientBoosting*`. | CA-2.6, CA-15.5, CA-10.3 |
| DT-16 ⚑ | Los dos errores de prueba son los de más modelos fallando (desempate: menor índice original); el «modelo» mostrado es el primero que falla en el orden CA-10.1. | La spec no define cuál elegir; priorizar instancias difíciles para varios modelos es explicable y determinista [RNF-9]. | Elegir al azar con `SEED`; elegir un error por modelo. | CA-14.1, CA-14.4 |
| DT-17 ⚑ | Huella `hashlib` (biblioteca estándar) de valores redondeados a 6 decimales al final de cada `E*.C`. | Convierte «idénticos entre dos ejecuciones» en una comparación objetiva. | Comparar números a ojo. | RNF-1 |
| DT-18 | `joblib`, `scipy` y `seaborn` no se usan; no se usa `n_jobs`. | Están permitidos, no son necesarios; los datos son diminutos y el determinismo es prioritario. | Paralelizar con `joblib`. | P 1, RNF-8 |

---

## 7. Estrategia de pruebas y verificación

No existe suite de tests: la verificación son comprobaciones dentro del notebook más inspección estática [P 4].

### 7.1 Comprobaciones de la lógica (celda S6; `assert` sin pytest)

| Función | Comprobación |
|---|---|
| `make_splits` | Con `y = [0]*50 + [1]*50 + [2]*50`: tamaños 90/30/30; disjuntos y cubren todo; 30/10/10 por clase; misma semilla → mismos índices; otra semilla → distintos |
| `sample_examples` | Devuelve 5 índices únicos, las 3 clases presentes, determinista |
| `build_*` | `random_state == SEED`; hay `StandardScaler` solo si corresponde; `n_iter_no_change is None` y `early_stopping` falso |
| `fit_capturing_warnings` | `LogisticRegression(max_iter=1)` produce al menos un `ConvergenceWarning` capturado y el estimador queda ajustado |
| `score_classifier` | `y=[0,0,1,1,2,2]`, `pred=[0,0,1,2,2,2]` → F1 macro = (1 + 2/3 + 4/5)/3 ≈ 0.8222; F1 micro = accuracy = 5/6 |
| `score_regressor` | `y=[1,2,3]`, `pred=[1,2,5]` → MAE = 2/3; RMSE = √(4/3); R² = −1 |
| `select_config` | Empate en primaria → decide la de apoyo; empate persistente → gana la probada primero |
| `evaluate_once` | La segunda llamada con el mismo modelo lanza error |
| `alpha_sweep` / `pick_best_alpha` | Norma L2 de Ridge no creciente con alpha; Lasso con alpha ≥ `alpha_max` → todos nulos; empate → menor alpha; borde detectado |
| `indistinguishable_*` | Frontera exacta: 1 error incluido, 2 excluido; 5 % exacto incluido, 5.01 % excluido |
| `pick_two_errors` | Tres escenarios: ≥ 2 errores de prueba; 1 de prueba + 1 de validación rotulado; ninguno → vacío [CL-1, CL-2] |
| `top_abs_errors` | Orden descendente; empate → menor índice |
| `is_degenerate_*` | Clasificador que nunca predice una clase → verdadero; Lasso con coeficientes nulos → verdadero |
| `build_regressor` (MLP) | Con `y ≈ 1000`, las predicciones quedan en escala original (media ≈ 1000, no ≈ 0) [CL-12] |
| `result_fingerprint` | Misma entrada → mismo hash; entrada alterada → hash distinto |

### 7.2 Comprobaciones de integridad en cada `E*.C`

| Comprobación | Cubre |
|---|---|
| Formas: `X (150, 4)`, `y (150,)`; `X (442, 10)`, `y (442,)` | CA-8.1, CA-16.1 |
| Particiones disjuntas, completas y 90/30/30 y 264/89/89; Iris con 10/10/10 por clase en validación y prueba | CA-2.1…2.4 |
| Misma partición para todos los modelos (un único `splits`) | CA-2.3 |
| `len(records[m]) ≤ 4` (salvo barrido de alpha) y configuraciones distintas | CA-3.2 |
| Congelado reproduce las métricas de validación del registro | CA-3.4 |
| `ledger` con una evaluación por modelo (7 en E2, 8 en E3) | CA-3.5, CA-19.2 |
| Columnas de T2.5 y T3.6 iguales a la lista exacta de la spec | CA-12.1, CA-19.1 |
| Rejilla con ≥ 8 puntos log-espaciados; Lasso en el tope con todos los coeficientes en cero | CA-21.2, CL-8 |
| Orden por R² igual al orden por RMSE; R² de la media de entrenamiento ≤ 0 | CA-22.1, CA-22.3 |
| Suma de cuadrados por columna = 1 y media ≈ 0 en `X` de Diabetes | CA-16.2, CA-16.4 |

### 7.3 Inspección estática del `.ipynb` (búsquedas de texto, sin scripts)

| Búsqueda | Resultado esperado | Cubre |
|---|---|---|
| `print(`, `display(`, `plt.` en S2–S5 | 0 coincidencias | P 3, RNF-5 |
| `open(`, `read_csv`, `to_csv`, `savefig`, `joblib.dump` | 0 | P 5, RNF-2 |
| Líneas `import` / `from … import` | Solo en S1; todas del stack o estándar | P 1, RNF-3 |
| `def ` en `E*.C` | 0 | I-1 |
| `X_test`, `y_test` | Solo en `E1.C`, `E2.C`, `E3.C` | I-5 |
| `early_stopping`, `validation_fraction`, `n_iter_no_change`, `cv=` | 0 o solo en valores por defecto | CA-2.6 |
| `filterwarnings`, `simplefilter("ignore")` | 0; `simplefilter("always")` solo en S4 | CA-7.3 |
| `random_state=` | Siempre `SEED`; sin `np.random.seed` | CA-2.5 |

### 7.4 Ejecución

1. *Restart & Run All* con `RUN_TEST = True`: termina sin errores y sin aserciones fallidas [RNF-1].
2. Repetirlo: las tres huellas (E1, E2, E3) y las tablas coinciden [RNF-1].
3. Medir el tiempo total: < 5 minutos [RNF-8].

### 7.5 Revisión humana (lo que no es mecanizable)

| Criterio | Cubre |
|---|---|
| Cada experimento tiene `P → C → I` consecutivas y en ese orden; cada RF-8…RF-22 mapea a un experimento | CA-1.1, CA-1.2, RNF-6 |
| La predicción no se editó tras ejecutar (se revisa el historial de git: la predicción se confirma antes de ejecutar la implementación) | CA-6.2 |
| Cada predicción falsable se marca confirmada/contradicha y por qué; las conceptuales se contrastan en texto | CA-6.3 |
| La interpretación cita resultados de su propia implementación y la tabla o figura de origen | CA-1.3, CA-5.3 |
| Aviso comentado, degenerado rotulado; «mejor observado, no seleccionado»; «en esta partición» | RF-7, CA-15.1, CA-15.7 |
| Idioma: código en inglés; Markdown y cadenas visibles en español | RNF-4 |
| Términos explicados, figuras con título y ejes, tablas con encabezados descriptivos | RNF-7 |
| Decisiones justificadas: configuración elegida, desempate, errores analizados | RNF-9 |

### 7.6 Criterios de finalización de la spec → verificación

| Criterio (§8 de la spec) | Dónde se verifica |
|---|---|
| Run All sin errores, resultados idénticos, < 5 min | 7.4 |
| Tres experimentos con tres celdas en orden y sección de soporte | 7.5, §1.1 |
| RF-8…RF-22 implementados; RF-1…RF-7 transversales | §9, 7.5 |
| Particiones y semilla antes de entrenar; congelado antes de prueba; prueba única | 7.2, §3.1 |
| Entregables presentes y generados | §4, 7.2 |
| Interpretaciones basadas en resultados; predicciones contrastadas | 7.5 |
| `assert` por función; sin `print`/`display`/`plt.` en funciones | 7.1, 7.3 |
| Sin archivos adicionales, dependencias nuevas ni E/S de archivos | 7.3 |
| Idioma, legibilidad, decisiones justificadas | 7.5 |

---

## 8. Requisitos con cobertura parcial o fuera del alcance de la verificación automática

No se inventa cobertura donde el plan no puede garantizarla:

| Requisito | Situación | Cómo se cubre |
|---|---|---|
| CA-1.1, CA-1.2, RNF-6 | La presencia y el orden de celdas Markdown **no son comprobables desde el notebook** sin leer el `.ipynb` (lo prohíben P 5 y RNF-2) | Inspección estática y revisión humana (7.3, 7.5) |
| CA-6.2 | «No modificar la predicción tras ejecutar» no es comprobable por código | Control de proceso: confirmar la predicción en git antes de ejecutar; revisión del historial (7.5) |
| CA-3.6, CA-14.5 | «No iniciar una nueva ronda de ajuste a partir de la tabla de prueba» es una regla de proceso | `RUN_TEST` reduce el riesgo (DT-9); no lo elimina |
| CA-5.2 | Que no haya cifras escritas a mano en tablas y gráficos no es comprobable por código | Revisión: las tablas salen de variables (7.5) |
| CA-6.3, CA-7.1, CA-14.6, CA-15.x, CA-22.x (contenido) | El plan define **qué debe contener** cada respuesta, no su texto | Contratos de §4 y revisión humana (7.5) |
| RNF-4, RNF-7, RNF-9 | Calidad lingüística y didáctica | Revisión humana (7.5) |

**Ningún RF queda sin una parte del plan que lo implemente.** Las limitaciones son de *verificación*, no de implementación.

---

## 9. Matriz de cobertura RF → plan

| RF | Se implementa / cubre en |
|---|---|
| RF-1 | §1.1, §1.2, §4 (estructura `P → C → I`), 7.5 |
| RF-2 | §2.1, §2.2, §3.1 pasos 2, §3.2, DT-4; E1.C `[E1.5]`, E3.C `[E3.2]`; T1.5, T3.2 |
| RF-3 | §3.1 pasos 3–8, §3.2, DT-8, DT-9; T2.3–T2.4, T3.4–T3.5 |
| RF-4 | DT-5, DT-12; `build_*`; E3.C `[E3.3]` |
| RF-5 | §4 (tablas/figuras generadas), DT-11; 7.5 |
| RF-6 | E*.P y E*.I; 7.5 (CA-6.2 por proceso, §8) |
| RF-7 | DT-7; `fit_capturing_warnings`; T2.10–T2.11, T3.11–T3.12 |
| RF-8 | E1.C `[E1.1–E1.4]`; T1.1–T1.4, F1.1; E1.P (CA-8.4), E1.I (CA-8.5) |
| RF-9 | E2.P (tres predicciones y criterio de sencillez) |
| RF-10 | §3.3, DT-5, DT-14, DT-15; T2.2 (CA-10.4) |
| RF-11 | E2.C `[E2.1]`, `[E2.3]`; T2.1, T2.3 |
| RF-12 | E2.C `[E2.5]`; T2.5, T2.6 |
| RF-13 | F2.1 |
| RF-14 | §3.3 (`pick_two_errors`), DT-16; T2.12; E2.I (CA-14.6) |
| RF-15 | §3.3; T2.7–T2.9; E2.I |
| RF-16 | E3.C `[E3.1]`, `[E3.3]`; T3.1; §2.2 |
| RF-17 | E3.P (CA-17.1); E3.I (CA-17.2) |
| RF-18 | §3.4, DT-5, DT-12; `build_regressor`; T3.4 |
| RF-19 | E3.C `[E3.7]`; T3.6 |
| RF-20 | E3.C `[E3.9]`; F3.3, F3.4, T3.9 |
| RF-21 | §3.4, DT-6, DT-13; E3.C `[E3.4]`; T3.3, F3.1, F3.2; E3.P (CA-21.1) |
| RF-22 | E3.I; T3.7, T3.8, T3.10 |

**Requisitos no funcionales:** RNF-1 → §5, DT-17, 7.4 · RNF-2 → DT-1, I-3 · RNF-3 → I-4, DT-11, DT-18 · RNF-4 → convenciones, 7.5 · RNF-5 → §1.3, 7.1 · RNF-6 → §1.1, §9 · RNF-7 → §2.1, 7.5 · RNF-8 → DT-18, 7.4 · RNF-9 → 7.5.

---

## 10. Riesgos y supuestos

| Riesgo / supuesto | Mitigación |
|---|---|
| El MLP (clasificación y regresión) puede emitir `ConvergenceWarning` con su configuración inicial [CL-7] | Se muestra en T2.10/T3.11; si se corrige con `max_iter`, cuenta como configuración [CA-7.2] |
| El óptimo de alpha puede caer en el borde de la rejilla [CL-15] | Se declara y se explica; no se extiende la rejilla [CA-21.4] |
| Lasso puede quedar constante en el óptimo [CL-16] | Se rotula «degenerado» y se explica [CA-7.3] |
| Con 30 instancias de prueba, diferencias de un error no son concluyentes [CA-15.7] | Criterio ≤ 1 error y redacción «en esta partición» |
| Ejecutar celdas fuera de orden invalida los resultados [CL-14] | Solo *Restart & Run All* cuenta como válido |
| `notebook` no está fijado en versión | Sin efecto en resultados; observación de la spec, no se modifica |
