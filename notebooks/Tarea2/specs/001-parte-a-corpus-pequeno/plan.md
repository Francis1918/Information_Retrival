# Plan técnico — Spec 001: Parte A, corpus pequeño

| | |
|---|---|
| **Estado** | Aprobado por una persona el 2026-10-22 (con la actualización de `requirements.txt`, ver §8 y §9) |
| **Fecha** | 2026-10-22 |
| **Entradas** | Constitución (con la enmienda del 2026-10-22), Spec 001 revisión 2, notebook `02_booleansearch.ipynb` (solo enunciado y celdas vacías), plan de Tarea1 (solo como referencia de formato) |
| **Alcance de este documento** | El CÓMO: bloques de celdas, datos, algoritmos en pseudocódigo, contratos, decisiones y verificación. No contiene código de producción. |

**Convenciones.** `[RF-x.y]` y `[RNF-n]` remiten a la spec; `[P n]` al principio n de la constitución; `[CL-n]` al caso límite n de la spec §6; `[DTn]` a una decisión técnica de §5; `[Pn]` en §9 a una decisión pendiente. Los identificadores del notebook están en español, sin tildes ni eñes, para evitar problemas de herramientas (el idioma lo exige `[P 6]`, `[RNF-11]`). En los bloques de pseudocódigo, `←` es asignación.

---

## 0. Verificación contra la constitución

| Principio | Cómo lo respeta este plan |
|---|---|
| P 1 Simplicidad | Un solo archivo (`02_booleansearch.ipynb`). Solo biblioteca estándar y NLTK (`unicodedata`, `re`, `time`, `gc`, `sys`, `platform`, `importlib`, `collections`, `typing`). Sin stemming, palabras vacías, ranking, frases, parser, paralelismo ni persistencia. Las consultas son árboles de tuplas, no texto que se interprete [DT2]. |
| P 2 Autoridad de spec | La spec revisión 2 es la fuente. No se detectó ninguna discrepancia. Las interpretaciones del plan que la spec deja abiertas están en §9. |
| P 3 Lógica/E-S | Las funciones de lógica (celdas L) reciben datos en memoria y no leen disco ni imprimen. La lectura vive en C04, la presentación en celdas P, la medición en C27. La ruta de datos es una sola variable (`RUTA_NLTK_DATA`) en C02. |
| P 4 Tests | Solo `assert` dentro del notebook. La equivalencia de mecanismos (C26) precede a cualquier medición (C27). Calentamiento y 30 ejecuciones, sin carga ni tokenización en la ventana medida. |
| P 5 Persistencia | Nada se escribe. NLTK solo lee la colección ya instalada. No se descarga nada. Si falta, la ejecución falla con mensaje claro. El id es el nombre de archivo. |
| P 6 Idioma | Código, comentarios, markdown, mensajes y salidas en español. Los comentarios explican el porqué técnico; el razonamiento largo va en celdas markdown. |

---

## 1. Estructura de módulos

### 1.1 Módulos como archivos: No aplica

La constitución prohíbe cualquier archivo de código fuera del notebook `[P 1]`. La «estructura de módulos» es una **estructura de bloques de celdas** dentro de `02_booleansearch.ipynb`.

### 1.2 Tipos de celda

| Tipo | Significado | Reglas |
|---|---|---|
| **Cfg** | Configuración | Solo constantes. La única variable de ruta vive aquí `[RF-1.9]`. |
| **L** | Lógica | Solo `def`, clases y constantes. No lee disco, no imprime, no importa. `[P 3]` |
| **E/S** | Lectura | Única celda que toca la colección. `[P 3]` |
| **O** | Orquestación | Llama a funciones L y guarda variables. No imprime. |
| **P** | Presentación | Imprime tablas y mensajes. No define lógica. |
| **M** | Medición | Bucle de medición con el reloj real. Imprime solo al final de C28. |
| **V** | Verificación | `assert` sobre datos sintéticos o reales. |
| **MD** | Markdown | Razonamiento largo. |

### 1.3 Orden de celdas

Las celdas nuevas se **insertan inmediatamente después** de la celda markdown de cada sección del enunciado (ver §8). Ninguna celda existente se modifica.

| ID | Sección del enunciado | Tipo | Contenido | Cubre |
|---|---|---|---|---|
| C01 | 1 | Code (importaciones) | Importaciones de biblioteca estándar. NLTK se importa dentro de C04. | P 1 |
| C02 | 1 | Cfg | `RUTA_NLTK_DATA = None`, `DOCUMENTOS_ESPERADOS = 18`, `CALENTAMIENTOS = 3`, `REPETICIONES = 30`, `UMBRAL_FRECUENTE = 12`, `MAX_POCO_FRECUENTE = 2`, `TOP_DF = 10`, `PRESUPUESTO_SEGUNDOS = 300`, `TERMINOS_PRUEBA` (3 por categoría) y marca de inicio `T_INICIO`. | RF-1.9, RF-10.1, RF-10.2, RF-12.3, RNF-9 |
| C03 | 1 | L | Tipos de datos, excepciones y `formatear_tabla`. | §2, §4 |
| C04 | 1 | E/S | `obtener_lector_nltk`, `extraer_textos` y su ejecución: produce `textos` y `version_nltk`. | RF-1.1 a RF-1.6, RF-1.10 |
| C05 | 1 | L | Colección: `ordenar_ids`, `validar_cantidad`, `construir_documentos`, `resumir_tamanos`. | RF-1.11, RF-4.2, RF-4.5 a RF-4.7 |
| C06 | 1 | L | Normalización: `clasificar_caracter`, `normalizar_texto`, `normalizar_termino`. | RF-2, RF-5.1, RF-5.2, RF-5.5 |
| C07 | 1 | V | Pruebas de C03 a C06 con datos sintéticos (lector falso, ejemplos de normalización, prueba diferencial). | RNF-3 |
| C08 | 1 | L | Inventario: `describir_tratamiento`, `inventariar_caracteres`. | RF-3 |
| C09 | 1 | O | `ids`, `validar_cantidad`, `docs`, `tamanos`, `inventario`. | RF-4.2 |
| C10 | 1 | P | Versiones de Python y NLTK, número de documentos, inventario, tabla de tamaños. | RF-1.12, RF-3.1, RF-3.3, RF-4.1, RF-4.5 |
| MD-N | 1 | MD | Regla de normalización por clase, motivo y colisiones aceptadas. | RF-2.2, RNF-7, CL-3, CL-19 a CL-21 |
| C11 | 2 | L | Búsqueda secuencial: `buscar_secuencial`, `terminos_de_consulta`, `evaluar_presencia`, `resolver_secuencial`. | RF-6, RF-9.7 |
| C12 | 2 | V | Pruebas de C11. | RNF-3 |
| C13 | 2 | P | Consultas secuenciales de un término con los 9 términos de prueba y sus resultados. | RF-6.1, enunciado §2 |
| C14 | 3 | L | `medir_media_ms`, `formatear_ms`. | RF-12 |
| C15 | 3 | V | Pruebas de C14 con reloj y ejecutable falsos. | RF-12.1, RF-12.3, RF-12.8, RF-12.11 |
| MD-M | 3 | MD | Protocolo de medición: qué entra y qué no en el tiempo. | RF-12.4, RF-12.6, RF-12.7 |
| C16 | 4 | L | Índice y fusiones: `construir_indice`, `consultar_indice`, `fusionar_and`, `fusionar_or`, `complementar`, `resolver_indice`, `df_de`, `mayores_df`, `distribucion_df`, `revisar_invariantes_indice`. | RF-7, RF-8, RF-9 |
| C17 | 4 | V | Pruebas de C16, incluida la prueba exhaustiva contra un oráculo. | RNF-3 |
| C18 | 4 | O | `indice = construir_indice(docs)` y `assert` de invariantes. | RF-7.1, RF-8.3 |
| C19 | 4 | P | Tamaño del vocabulario, listas y DF de los términos de prueba, top 10, distribución de DF. | RF-8.1, RF-8.2, RF-8.4 |
| C20 | 4 | L | Consultas: constructores, `preparar_consulta`, `preparar_consultas`, `texto_consulta`, `tipo_consulta`, `generar_consultas_prueba`, `clasificar_df`, `revisar_terminos_prueba`. | RF-5, RF-9, RF-10 |
| C21 | 4 | V | Pruebas de C20. | RNF-3 |
| C22 | 4 | O+V | Revisa los términos de prueba contra el índice y la secuencial. | RF-10.2, RF-10.6, RF-10.7 |
| MD-T | 4 | MD | Elección y justificación de los 9 términos con su DF. | RF-10.2, RF-10.7, RNF-7 |
| C23 | 5 | L | `ejecutar_protegido`, `describir_diferencia`, `construir_filas_informe`. | RF-11, RF-13 |
| C24 | 5 | V | Pruebas de C23. | RNF-3 |
| C25 | 5 | O | Genera las 33 consultas, las prepara y separa las rechazadas. | RF-10.3, RF-10.4, RF-5.6 |
| C26 | 5 | V | Equivalencia de los dos mecanismos en las 33 consultas y en las identidades. | RF-11 |
| C27 | 5 | M | Bucle de medición: por consulta, primero secuencial y luego índice. | RF-12 |
| C28 | 5 | P | Informe comparativo y tiempo total de las mediciones. | RF-13 |
| C29 | 5 | P | Tiempo de toda la Parte A frente al presupuesto. | RNF-9, RF-13.4 |
| MD-F | 5 | MD | Lectura de resultados del autor y lista de verificaciones por inspección. | RF-13.3, RF-13.8, RNF-3 |

### 1.4 Invariantes de estructura (verificables por inspección del `.ipynb`)

| ID | Invariante | Cubre |
|---|---|---|
| I-1 | Las celdas L no contienen `print`, `input`, `open`, `nltk` ni `import`. | P 3 |
| I-2 | `import` solo en C01; la única excepción es la importación dinámica de NLTK dentro de `obtener_lector_nltk`. | P 1, RF-1.4 |
| I-3 | No aparecen `open(`, `write`, `download`, `savefig`, `.to_`, `pickle`, `json.dump`. | P 5, RNF-5 |
| I-4 | No aparece `input(` en ninguna celda. | §7 spec |
| I-5 | Ni `resolver_secuencial` ni `buscar_secuencial` mencionan `indice`. | RF-6.3, RF-9.7 |
| I-6 | `fusionar_*` y `complementar` no usan `set` ni los operadores de conjuntos (`&`, `\|`, `-`); solo el oráculo de C17 puede usarlos. | contexto del proyecto, P 3 |
| I-7 | El texto original (`textos`) no se pasa a `buscar_secuencial`, `resolver_secuencial` ni `construir_indice`; solo `docs`. | RF-12.4 |
| I-8 | El repositorio no contiene `.py` ni archivos de salida nuevos. | P 1 |

---

## 2. Modelo de datos

### 2.1 Tipos

| Tipo | Forma | Notas |
|---|---|---|
| `textos` | `dict[str, str]`: id → texto decodificado | Id = nombre de archivo con extensión `[RF-1.10]`. Se conserva para tamaños e inventario. |
| `ids` | `list[str]` | Orden ascendente por puntos de código `[RF-1.11]`. |
| `Documento` | `NamedTuple(id: str, tokens: list[str])` | Desempaqueta como tupla, lo más rápido para los dos lazos. |
| `docs` | `list[Documento]` en el orden de `ids` | Único dato que reciben la búsqueda secuencial y la construcción del índice. |
| `Indice` | `dataclass(postings: dict[str, list[str]], universo: list[str])` | Listas de ids ordenadas y sin repeticiones `[RF-7.1]`. `universo` = ids de todos los documentos. |
| Consulta | Árbol de tuplas: `("TERMINO", texto)`, `("AND", izq, der)`, `("OR", izq, der)`, `("NOT", op)` | «Cruda» lleva el texto escrito; «preparada» lleva el token normalizado. |
| `ConsultaPrueba` | `NamedTuple(texto: str, tipo: str, cruda: tuple, categorias: tuple[str, ...])` | `texto` = consulta con sus términos reales `[RF-13.5]`. |
| `ConsultaPreparada` | `NamedTuple(prueba: ConsultaPrueba, preparada: tuple)` | |
| `Rechazo` | `NamedTuple(prueba: ConsultaPrueba, motivo: str)` | `[RF-5.6, RF-13.10]` |
| `FilaInventario` | `NamedTuple(codigo: int, simbolo: str, clase: str, tratamiento: str, apariciones: int)` | `[RF-3.3]` |
| `ProblemaTermino` | `NamedTuple(termino: str or None, df: int or None, motivo: str)` | `[RF-10.6]` |
| `ResultadoMedicion` | `NamedTuple(prueba, n_documentos: int, media_seq_ms: float, media_idx_ms: float, cero_seq: bool, cero_idx: bool)` | `[RF-13.1, RF-12.11]` |
| Excepciones | `ErrorDeColeccion(Exception)`, `TerminoInvalido(Exception)` con atributos `original` y `cantidad_tokens` o `tipo` | Mensajes en español `[RNF-4]`. |
| Clases de carácter | `apostrofo`, `letra`, `digito`, `marca`, `guion`, `espacio`, `control`, `formato`, `puntuacion_simbolo`, `no_clasificado` | `[RF-2.13]` |

### 2.2 Ejemplo ilustrativo

Colección sintética de 3 documentos (**no es contenido real de NLTK**):

```
textos = {
  "b.txt": "Café, don't stop. Café!",
  "a.txt": "The cat — the Cat's hat",
  "c.txt": ""
}
ids  = ["a.txt", "b.txt", "c.txt"]                      # orden por puntos de código
docs = [ Documento("a.txt", ["the","cat","the","cats","hat"]),
         Documento("b.txt", ["cafe","dont","stop","cafe"]),
         Documento("c.txt", []) ]                       # documento vacío se conserva [CL-1]
indice.postings = { "the":["a.txt"], "cat":["a.txt"], "cats":["a.txt"], "hat":["a.txt"],
                    "cafe":["b.txt"], "dont":["b.txt"], "stop":["b.txt"] }
indice.universo = ["a.txt","b.txt","c.txt"]

consulta cruda  = ("AND", ("TERMINO","The"), ("NOT", ("TERMINO","Café")))
consulta prep.  = ("AND", ("TERMINO","the"), ("NOT", ("TERMINO","cafe")))
texto_consulta  = "The AND NOT Café"
resolver_indice      → NOT cafe = ["a.txt","c.txt"]; AND con ["a.txt"] → ["a.txt"]
resolver_secuencial  → ["a.txt"]                        # mismo resultado, sin usar el índice
```

---

## 3. Algoritmos en pseudocódigo

### 3.1 Lectura de la colección (celda C04, E/S) `[RF-1.1 a RF-1.6, RF-1.9, RF-1.10, RF-1.12]`

```
FUNCIÓN obtener_lector_nltk(ruta_nltk_data, nombre_modulo = "nltk"):
    TRY    modulo ← importar_dinamicamente(nombre_modulo)
    EXCEPT ImportError → LANZAR ErrorDeColeccion("Falta la biblioteca NLTK; instálela a mano (sin descarga automática)")
    SI ruta_nltk_data ≠ NADA: modulo.data.path.insert(0, ruta_nltk_data)     # solo memoria, no escribe disco
    TRY    lector ← modulo.corpus.gutenberg
           lector.fileids()                                                  # fuerza la búsqueda de la colección
    EXCEPT LookupError → LANZAR ErrorDeColeccion("Falta la colección Gutenberg; instálela a mano (sin descarga automática)")
    RETORNAR (lector, modulo.__version__)

FUNCIÓN extraer_textos(lector):
    textos ← {}
    PARA id EN lector.fileids():                                             # el id es el nombre de archivo
        TRY    textos[id] ← lector.raw(id)                                   # NLTK decodifica con la codificación de la colección
        EXCEPT (UnicodeDecodeError, OSError) → LANZAR ErrorDeColeccion("No se puede leer «" + id + "»")
    RETORNAR textos                                                          # texto sin transformar; "" si el archivo está vacío
```

### 3.2 Colección: orden, cantidad y tamaños (C05, L) `[RF-1.11, RF-4.2, RF-4.5 a RF-4.7]`

```
FUNCIÓN ordenar_ids(textos):       RETORNAR ordenar(textos.claves)           # str compara por puntos de código
FUNCIÓN validar_cantidad(ids, esperados):
    SI longitud(ids) ≠ esperados: LANZAR ErrorDeColeccion("Se hallaron N documentos y se esperaban 18")
FUNCIÓN construir_documentos(textos, ids):
    RETORNAR [ Documento(id, normalizar_texto(textos[id])) PARA id EN ids ]
FUNCIÓN resumir_tamanos(textos, docs):
    filas ← [ (d.id, longitud(textos[d.id]), longitud(d.tokens)) PARA d EN docs ]   # caracteres Unicode; tokens = ocurrencias
    totales ← (longitud(docs), suma(caracteres), suma(tokens))
    RETORNAR (filas, totales)
```

### 3.3 Normalización (C06, L) `[RF-2.1 a RF-2.22]`

```
CONSTANTE APOSTROFOS ← { U+0027, U+2018, U+2019, U+02BC }

FUNCIÓN clasificar_caracter(c):                                  # RF-2.13; una sola clase por carácter
    SI c ∈ APOSTROFOS:                      RETORNAR "apostrofo"           # prioridad sobre las demás clases
    cat ← categoria_unicode(c)
    SI cat = "Nd":                          RETORNAR "digito"
    SI cat empieza con "L":                 RETORNAR "letra"
    SI cat empieza con "M":                 RETORNAR "marca"
    SI cat = "Pd" O c = U+2212:             RETORNAR "guion"
    SI cat empieza con "Z":                 RETORNAR "espacio"             # incluye espacio duro, U+2028 y U+2029
    SI cat = "Cc":                          RETORNAR "control"
    SI cat = "Cf":                          RETORNAR "formato"
    SI cat ∈ {"Cn","Co","Cs"}:              RETORNAR "no_clasificado"
    RETORNAR "puntuacion_simbolo"                                          # P, S, No, Nl

FUNCIÓN normalizar_texto(texto):
    t ← NFKD(texto)                          # pasos 1 y 2 de RF-2.18: compatibilidad (fi, ², ½, ancho completo) y marcas separadas
    tabla ← {}                               # una entrada por carácter DISTINTO de t, no por carácter del texto
    PARA c EN conjunto(t):
        SEGÚN clasificar_caracter(c):
            "marca", "formato":  tabla[c] ← ""                  # se eliminan sin separar la palabra (RF-2.15, RF-2.21)
            "apostrofo":         tabla[c] ← "'"                 # unifica variantes
            "letra":             tabla[c] ← minuscula_simple(c) # carácter a carácter: "ß" se conserva, sin sigma final (RF-2.17)
            "digito":            (sin entrada: se conserva)
            OTRA CLASE:          tabla[c] ← " "                 # separador (RF-2.9, 2.10, 2.20, 2.22)
    s ← t.traducir(tabla)                    # tras esto solo quedan letras, dígitos, "'" y espacios
    s ← reemplazar(s, "'{2,}", " ")          # RF-2.19: apóstrofos consecutivos separan
    s ← reemplazar(s, "(?<=alfanumérico)'(?=alfanumérico)", "")   # RF-2.7: un apóstrofo aislado entre alfanuméricos une
    s ← reemplazar(s, "'", " ")              # RF-2.8: cualquier otro apóstrofo separa
    RETORNAR dividir_por_espacios(s)         # tokens = secuencias contiguas de letras y dígitos (RF-2.6)

FUNCIÓN normalizar_termino(termino):                              # RF-5.1, RF-5.2, RF-5.5
    SI termino NO es texto:  LANZAR TerminoInvalido(original = termino, tipo = nombre_del_tipo)
    tokens ← normalizar_texto(termino)                            # los espacios alrededor desaparecen solos
    SI longitud(tokens) ≠ 1: LANZAR TerminoInvalido(original = termino, cantidad_tokens = longitud(tokens))
    RETORNAR tokens[0]
```

### 3.4 Inventario (C08, L) `[RF-3.1 a RF-3.3]`

```
FUNCIÓN describir_tratamiento(c):                  # una sola fuente de verdad: la propia normalización
    r ← normalizar_texto("a" + c + "b")
    SI r = ["a","b"]:    RETORNAR "separador"
    SI r = ["ab"]:       RETORNAR "se elimina sin separar" (o «se une» si es apóstrofo)
    SI r = ["a" + x + "b"]: SI x = c: "se conserva" SINO "se convierte en «x»"
FUNCIÓN inventariar_caracteres(textos):
    conteo ← contar_caracteres(todos los textos juntos)            # suma de los 18 documentos
    PARA c, n EN conteo ordenado por punto de código:
        simbolo ← c SI es visible, SI NO el nombre Unicode (o «sin nombre»)
        filas.agregar(FilaInventario(codigo(c), simbolo, clasificar_caracter(c), describir_tratamiento(c), n))
    RETORNAR filas                                                  # los de clase «no_clasificado» quedan visibles (RF-3.2)
```

### 3.5 Búsqueda secuencial (C11, L) `[RF-6.1 a RF-6.6, RF-9.7]`

```
FUNCIÓN buscar_secuencial(docs, token):                           # un término: los dos lazos del enunciado
    resultado ← []
    PARA CADA documento EN docs:                                  # lazo 1: documentos, ya en orden ascendente
        PARA CADA t EN documento.tokens:                          # lazo 2: tokens
            SI t = token:                                         # token completo, nunca subcadena (RF-6.4)
                resultado.agregar(documento.id)
                ROMPER                                            # pasa al documento siguiente (RF-6.5)
    RETORNAR resultado

FUNCIÓN resolver_secuencial(docs, consulta):
    SI consulta[0] = "TERMINO": RETORNAR buscar_secuencial(docs, consulta[1])
    buscados ← terminos_de_consulta(consulta)                     # conjunto de trabajo del tamaño de la consulta, creado en cada llamada
    resultado ← []
    PARA CADA documento EN docs:                                  # cada documento se evalúa una sola vez (RF-9.7)
        pendientes ← copia(buscados);  presentes ← {}
        PARA CADA t EN documento.tokens:
            SI t ∈ pendientes: mover t de pendientes a presentes
                               SI pendientes vacío: ROMPER        # ya se conoce la presencia de todos
        SI evaluar_presencia(consulta, presentes): resultado.agregar(documento.id)
    RETORNAR resultado

FUNCIÓN evaluar_presencia(consulta, presentes):                   # recursión sobre el árbol
    SEGÚN consulta[0]: "TERMINO" → consulta[1] ∈ presentes
                       "AND"     → evaluar(izq) Y evaluar(der)
                       "OR"      → evaluar(izq) O evaluar(der)
                       "NOT"     → NO evaluar(op)
FUNCIÓN terminos_de_consulta(consulta): conjunto de los tokens de las hojas «TERMINO»
```

### 3.6 Índice y fusiones (C16, L) `[RF-7, RF-8, RF-9]`

```
FUNCIÓN construir_indice(docs):
    postings ← {}
    PARA CADA documento EN docs:                                  # orden ascendente ⇒ cada lista queda ordenada por construcción
        PARA CADA t EN documento.tokens:
            lista ← postings.obtener(t)
            SI lista = NADA:                    postings[t] ← [documento.id]
            SINO SI lista[último] ≠ documento.id: lista.agregar(documento.id)    # sin repeticiones sin usar conjuntos
    RETORNAR Indice(postings, universo = [d.id PARA d EN docs])

FUNCIÓN consultar_indice(indice, token): RETORNAR indice.postings.obtener(token, [])    # lista nueva si no existe; las listas del índice no se mutan
FUNCIÓN fusionar_and(a, b):                                       # dos punteros sobre listas ordenadas
    i ← 0; j ← 0; salida ← []
    MIENTRAS i < |a| Y j < |b|:
        SI a[i] = b[j]: salida.agregar(a[i]); i++; j++
        SINO SI a[i] < b[j]: i++  SINO j++
    RETORNAR salida
FUNCIÓN fusionar_or(a, b):  igual, pero se agrega el menor de los dos y, si son iguales, se agrega una vez y avanzan ambos; luego se vuelcan los restos
FUNCIÓN complementar(universo, a):
    i ← 0; salida ← []
    PARA u EN universo: SI i < |a| Y a[i] = u: i++  SINO salida.agregar(u)
    RETORNAR salida
FUNCIÓN resolver_indice(indice, consulta):
    SEGÚN consulta[0]: "TERMINO" → consultar_indice(indice, consulta[1])
                       "AND" → fusionar_and(resolver(izq), resolver(der))
                       "OR"  → fusionar_or(resolver(izq), resolver(der))
                       "NOT" → complementar(indice.universo, resolver(op))
```

Auxiliares de inspección `[RF-8]`:

```
df_de(indice, token)            → longitud(consultar_indice(indice, token))
mayores_df(indice, cantidad)    → ordenar todos los términos por (−DF, término) y devolver los primeros `cantidad`
                                   más cuántos términos comparten la DF máxima [RF-8.2, RF-8.4]
distribucion_df(indice, n)      → para k = 1..n: cuántos términos tienen DF = k (incluye ceros)
revisar_invariantes_indice(indice, docs) → lista de violaciones de:
    (a) cada lista está ordenada y sin repeticiones          [RF-7.1]
    (b) la suma de todas las DF = suma de tokens distintos por documento   [RF-8.3]
    (c) todo término de algún documento está en el vocabulario, y viceversa
```

### 3.7 Consultas y términos de prueba (C20, L) `[RF-5.4, RF-5.6, RF-9, RF-10]`

```
CATEGORIAS ← ("frecuente", "poco frecuente", "inexistente")

FUNCIÓN preparar_consulta(cruda):                                 # valida TODO el árbol antes de evaluar nada (RF-5.6)
    SEGÚN cruda[0]: "TERMINO" → ("TERMINO", normalizar_termino(cruda[1]))     # puede lanzar TerminoInvalido
                    "AND","OR" → (op, preparar(izq), preparar(der));  "NOT" → ("NOT", preparar(op))
FUNCIÓN preparar_consultas(pruebas):
    preparadas ← [];  rechazos ← []
    PARA CADA p EN pruebas:
        TRY preparadas.agregar(ConsultaPreparada(p, preparar_consulta(p.cruda)))
        EXCEPT TerminoInvalido como e: rechazos.agregar(Rechazo(p, mensaje_en_espanol(e)))     # solo se rechaza esa consulta
    RETORNAR (preparadas, rechazos)

FUNCIÓN clasificar_df(df, umbral_frecuente, max_poco_frecuente):
    SI df = 0: "inexistente";  SI df ≥ umbral_frecuente: "frecuente"
    SI 1 ≤ df ≤ max_poco_frecuente: "poco frecuente";  EN OTRO CASO "intermedia"        # RF-10.5

FUNCIÓN revisar_terminos_prueba(indice, docs, terminos, umbral, max_poco):
    problemas ← []
    PARA categoria, lista EN terminos:
        SI |lista| ≠ 3: problema «se esperan 3 términos»                                 # RF-10.2
        PARA texto EN lista:
            TRY token ← normalizar_termino(texto)  EXCEPT TerminoInvalido: problema «inválido»; CONTINUAR
            df_i ← df_de(indice, token);  df_s ← |buscar_secuencial(docs, token)|
            SI df_i ≠ df_s: problema «DF distinta entre mecanismos»                      # RF-10.7
            SI clasificar_df(df_i, ...) ≠ categoria: problema «no cumple el umbral»      # RF-10.6
    problema por cada token repetido entre los 9 términos
    RETORNAR problemas

FUNCIÓN generar_consultas_prueba(terminos):                       # determinista; reparte con contadores rotativos [RF-10.4]
    uso ← {categoria: 0};   siguiente(c) = terminos[c][uso[c] mod 3] y luego uso[c]++
    COMBOS ← [(F,F), (F,P), (F,I), (P,P), (P,I), (I,I)]                  # F=frecuente, P=poco frecuente, I=inexistente
    1.  9 de un término:    PARA c, k: TERMINO(terminos[c][k])                       # no consume contadores
    2.  6 AND:              PARA (c1,c2) EN COMBOS: AND(siguiente(c1), siguiente(c2))
    3.  6 OR:               igual, continuando los contadores
    4.  3 NOT:              PARA c: NOT(siguiente(c))
    5.  6 «a AND NOT b»:    PARA (c1,c2) con c1 ≠ c2, en orden: AND(siguiente(c1), NOT(siguiente(c2)))
    6.  3 «(a OR b) AND NOT c»:  PARA c: (ca, cb) = las otras dos categorías en orden;
                                 AND(OR(siguiente(ca), siguiente(cb)), NOT(siguiente(c)))
    cada ConsultaPrueba guarda: texto con los términos reales, tipo, árbol crudo y categorías en el orden de la consulta
    RETORNAR la lista en ese orden                                  # el orden fijo de RF-12.10
```

`texto_consulta` escribe la consulta con paréntesis alrededor de los operandos AND/OR; `tipo_consulta` devuelve `término único`, `AND`, `OR`, `NOT`, `a AND NOT b` o `(a OR b) AND NOT c` `[RF-13.5]`.

### 3.8 Equivalencia (C23 y C26) `[RF-11]`

```
FUNCIÓN ejecutar_protegido(mecanismo, *argumentos):
    TRY RETORNAR ("ok", mecanismo(*argumentos))
    EXCEPT Exception como e: RETORNAR ("error", nombre_del_tipo(e) + ": " + mensaje(e))

FUNCIÓN describir_diferencia(texto, a, b):                        # RF-11.4
    mensaje con: la consulta; ambas listas completas; ids solo en una y solo en la otra;
    si tienen los mismos ids pero distinto orden o repeticiones, lo dice

CELDA C26 (V):
    PARA CADA cp EN preparadas:                                   # las 33 consultas
        ra ← ejecutar_protegido(resolver_secuencial, docs, cp.preparada)
        rb ← ejecutar_protegido(resolver_indice, indice, cp.preparada)
        ASSERT ra = rb, describir_diferencia(cp.prueba.texto, ra, rb)       # incluye RF-11.5: un error y un "ok" difieren
        recuperados[cp.prueba.texto] ← ra[1]
    PARA CADA a EN un término por categoría:                      # RF-11.6, sin medir
        esperados: a AND a = a;  a OR a = a;  NOT NOT a = a;  a AND NOT a = [];  a OR NOT a = universo
        ASSERT ambos mecanismos devuelven lo esperado
```

### 3.9 Medición (C14, L; C27, M) `[RF-12]`

```
FUNCIÓN medir_media_ms(ejecutable, calentamientos, repeticiones, reloj_ns):
    SI repeticiones < 1: LANZAR ValueError
    gc_estaba_activo ← gc.activo();  gc.desactivar()              # evita pausas del recolector dentro de la ventana medida
    TRY:
        REPETIR calentamientos VECES: ejecutable()                # no cuentan (RF-12.3)
        tiempos ← []
        REPETIR repeticiones VECES:
            t0 ← reloj_ns();  ejecutable();  t1 ← reloj_ns();  tiempos.agregar(t1 − t0)    # cada ejecución por separado (RF-12.8)
    FINALLY: SI gc_estaba_activo: gc.activar()
    media_ns ← suma(tiempos) / repeticiones
    RETORNAR (media_ns / 1_000_000, media_ns = 0)                 # el segundo valor activa la advertencia de RF-12.11

FUNCIÓN formatear_ms(valor): RETORNAR valor con 9 decimales fijos, redondeo al más cercano    # RF-12.9

CELDA C27 (M):
    inicio ← reloj()
    PARA CADA cp EN preparadas (orden fijo):                      # RF-12.10
        (seq, cero_s) ← medir_media_ms(CIERRE{ resolver_secuencial(docs, cp.preparada) }, CALENTAMIENTOS, REPETICIONES, reloj)
        (idx, cero_i) ← medir_media_ms(CIERRE{ resolver_indice(indice, cp.preparada) },   CALENTAMIENTOS, REPETICIONES, reloj)
        resultados.agregar(ResultadoMedicion(cp.prueba, |recuperados[...]|, seq, idx, cero_s, cero_i))
    total_ms ← (reloj() − inicio) / 1_000_000                     # primer calentamiento hasta fin de la última medición (RF-13.9)
```

`reloj` es `time.perf_counter_ns` (monótono, mayor resolución disponible, tiempo de pared). Los términos ya están normalizados antes de entrar al bucle `[RF-12.4]`.

### 3.10 Informe (C23, L; C28, P) `[RF-13]`

```
FUNCIÓN construir_filas_informe(resultados, rechazos):
    PARA CADA r EN resultados:  [texto, tipo, categorias unidas por «, », n_documentos, formatear_ms(seq), formatear_ms(idx),
                                 «media 0 por resolución del reloj» si cero_seq O cero_idx]
    PARA CADA x EN rechazos:    [texto, tipo, categorias, «rechazada», «—», «—», motivo]
    RETORNAR filas en el orden fijo de RF-10.3
CELDA C28: imprimir formatear_tabla(encabezados, filas) y el tiempo total de las mediciones. No imprime desviación, mínimo, máximo ni cociente entre mecanismos.
CELDA C29: imprimir el tiempo transcurrido desde T_INICIO; SI supera PRESUPUESTO_SEGUNDOS, mostrar un aviso con el tiempo real (la aceptación es humana, spec §9.1).
```

---

## 4. Contrato de las funciones

Todas las funciones L reciben datos en memoria, devuelven valores, **no imprimen y no leen disco**. Los errores son excepciones con mensaje en español. Los `ValueError` indican uso incorrecto desde el código, no entrada del usuario.

| Función | Celda | Parámetros | Retorna | Errores | Cubre |
|---|---|---|---|---|---|
| `formatear_tabla` | C03 | `encabezados: list[str]`, `filas: list[list[str]]` | `str` alineado en columnas | `ValueError` si una fila tiene otra longitud | RF-13.1 |
| `obtener_lector_nltk` | C04 (E/S) | `ruta_nltk_data: str or None`, `nombre_modulo: str = "nltk"` | `(lector, version_nltk)` | `ErrorDeColeccion` (falta NLTK o la colección) | RF-1.2, RF-1.4, RF-1.9, RF-1.12 |
| `extraer_textos` | C04 (E/S) | `lector` con `fileids()` y `raw(id)` | `dict[str, str]` | `ErrorDeColeccion` nombrando el archivo | RF-1.1, RF-1.5, RF-1.6, RF-1.10 |
| `ordenar_ids` | C05 | `textos: dict` | `list[str]` | — | RF-1.11 |
| `validar_cantidad` | C05 | `ids: list`, `esperados: int` | `None` | `ErrorDeColeccion` (hallados y esperados) | RF-4.2 |
| `construir_documentos` | C05 | `textos`, `ids` | `list[Documento]` | — | RF-1.1, RF-2.1 |
| `resumir_tamanos` | C05 | `textos`, `docs` | `(filas, totales)` | — | RF-4.5 a RF-4.7 |
| `clasificar_caracter` | C06 | `caracter: str` de longitud 1 | `str` (clase) | `ValueError` si no es 1 carácter | RF-2.13 |
| `normalizar_texto` | C06 | `texto: str` | `list[str]` (tokens) | — | RF-2.1 a RF-2.22 |
| `normalizar_termino` | C06 | `termino` | `str` (un token) | `TerminoInvalido` (tipo, 0 o más de 1 token) | RF-5.1, RF-5.2, RF-5.5 |
| `describir_tratamiento` | C08 | `caracter: str` | `str` | `ValueError` | RF-3.1 |
| `inventariar_caracteres` | C08 | `textos: dict` | `list[FilaInventario]` | — | RF-3 |
| `buscar_secuencial` | C11 | `docs`, `token: str` | `list[str]` | — | RF-6.1 a RF-6.6 |
| `terminos_de_consulta` | C11 | `consulta` preparada | `set[str]` | `ValueError` si el árbol es inválido | RF-9.7 |
| `evaluar_presencia` | C11 | `consulta`, `presentes: set` | `bool` | `ValueError` | RF-9.1 a RF-9.3 |
| `resolver_secuencial` | C11 | `docs`, `consulta` preparada | `list[str]` | `ValueError` | RF-6, RF-9.5, RF-9.7 |
| `medir_media_ms` | C14 | `ejecutable` sin argumentos, `calentamientos: int`, `repeticiones: int`, `reloj_ns` | `(media_ms: float, cero: bool)` | `ValueError` si `repeticiones < 1`; restaura el recolector aunque `ejecutable` falle | RF-12.1, RF-12.3, RF-12.8, RF-12.11 |
| `formatear_ms` | C14 | `valor_ms: float` | `str` con 9 decimales | — | RF-12.9 |
| `construir_indice` | C16 | `docs` | `Indice` | — | RF-7.1 |
| `consultar_indice` | C16 | `indice`, `token` | `list[str]` | — | RF-7.2 |
| `fusionar_and`, `fusionar_or` | C16 | `a: list`, `b: list` ordenadas | `list[str]` nueva | — | RF-9.1, RF-9.2 |
| `complementar` | C16 | `universo: list`, `a: list` | `list[str]` nueva | — | RF-9.3 |
| `resolver_indice` | C16 | `indice`, `consulta` preparada | `list[str]` | `ValueError` | RF-9, RF-7.2 |
| `df_de` | C16 | `indice`, `token` | `int` | — | RF-8.1 |
| `mayores_df` | C16 | `indice`, `cantidad: int` | `(lista de (término, DF), n_empatados_en_el_maximo)` | — | RF-8.2, RF-8.4 |
| `distribucion_df` | C16 | `indice`, `n_documentos: int` | `list[(k, cuantos_terminos)]` | — | RF-8.2 |
| `revisar_invariantes_indice` | C16 | `indice`, `docs` | `list[str]` violaciones | — | RF-7.1, RF-8.3 |
| `consulta_termino`, `consulta_and`, `consulta_or`, `consulta_not` | C20 | texto u operandos | tupla del árbol | — | RF-9.4, RF-9.6 |
| `preparar_consulta` | C20 | `cruda` | `preparada` | `TerminoInvalido` | RF-5.4, RF-5.6 |
| `preparar_consultas` | C20 | `pruebas: list[ConsultaPrueba]` | `(preparadas, rechazos)` | — | RF-5.6, RF-13.10 |
| `texto_consulta`, `tipo_consulta` | C20 | `cruda` | `str` | — | RF-13.5 |
| `generar_consultas_prueba` | C20 | `terminos: dict[categoria, list[str]]` | `list[ConsultaPrueba]` (33) | `ValueError` si no hay 3 por categoría | RF-10.3, RF-10.4 |
| `clasificar_df` | C20 | `df`, `umbral_frecuente`, `max_poco_frecuente` | `str` | — | RF-10.1, RF-10.5 |
| `revisar_terminos_prueba` | C20 | `indice`, `docs`, `terminos`, `umbral_frecuente`, `max_poco_frecuente` | `list[ProblemaTermino]` | — | RF-10.2, RF-10.6, RF-10.7 |
| `ejecutar_protegido` | C23 | `mecanismo`, `*argumentos` | `("ok", resultado)` o `("error", descripción)` | no propaga | RF-11.5 |
| `describir_diferencia` | C23 | `texto: str`, `resultado_a`, `resultado_b` | `str` | — | RF-11.3, RF-11.4 |
| `construir_filas_informe` | C23 | `resultados`, `rechazos` | `list[list[str]]` | — | RF-13.1, RF-13.5, RF-13.6, RF-13.10 |

---

## 5. Decisiones técnicas

| ID | Decisión | Justificación | Alternativa descartada | Cubre |
|---|---|---|---|---|
| DT1 | Un solo notebook; los «módulos» son bloques de celdas. | Es lo único que permite `[P 1]`. | Módulos `.py`: prohibidos. | P 1, P 3 |
| DT2 | Las consultas son árboles de tuplas creados por funciones. | Sin parser `[P 1]`, `[RF-9.6]`. Las tuplas son inmutables y fáciles de recorrer. | Parser de texto; sobrecarga de operadores en clases (más código sin beneficio). | RF-9.4, RF-9.6 |
| DT3 | Normalización con una tabla de traducción por **carácter distinto** y `str.translate`, más 3 sustituciones con expresión regular para los apóstrofos. | Se clasifica unas decenas de caracteres por documento, no millones; el trabajo masivo ocurre en C. Se hace una sola vez por ejecución `[RNF-9]`. | Bucle Python carácter a carácter (millones de iteraciones); bibliotecas externas de transliteración (fuera del stack `[P 1]`). | RF-2, RNF-9 |
| DT4 | `NFKD` en un solo paso para la compatibilidad (ﬁ, ², ancho completo) y para separar las marcas. | Cubre RF-2.14 y RF-2.15 sin tablas manuales, incluidas marcas no listadas. | `NFKC` más tabla manual de acentos (incompleta). | RF-2.14, RF-2.15 |
| DT5 | Minúsculas aplicadas **carácter a carácter** a las letras. | RF-2.17 pide conversión simple: así «ß» no pasa a «ss» y no actúa la regla contextual de la sigma final. | `str.casefold` (rompe RF-2.5); `lower()` sobre todo el texto (aplica la regla de sigma final). | RF-2.3, RF-2.17 |
| DT6 | `buscar_secuencial` con dos `for` explícitos y comparación `==`; `resolver_secuencial` para consultas compuestas con un conjunto de trabajo del tamaño de la consulta. | El enunciado exige los dos lazos. El camino simple evita el costo del conjunto en consultas de un término. | `token in tokens` o `any(...)` (esconden el segundo lazo); regex o `in` sobre el texto (no recorre tokens); un único camino general (más lento en un término). | RF-6.2, RF-6.5, RF-9.7 |
| DT7 | Índice como diccionario término → lista de ids, construido en un recorrido y ordenado por construcción comprobando el último elemento. | Cumple RF-7.1 sin conjuntos ni ordenación posterior; el orden de `docs` ya es el de RF-1.11. | `set` por término y ordenar al final; `defaultdict(list)` con `sorted`. | RF-7.1 |
| DT8 | AND, OR y NOT del índice por fusión de dos punteros sobre listas ordenadas. | Es lo que describe el contexto del proyecto `[P 3]` y conserva el orden sin ordenar. | Operadores de conjuntos `&`, `\|`, `-` (más breves, pero ocultan la fusión y pierden el orden). | RF-9.1 a RF-9.3 |
| DT9 | Los ids se comparan como cadenas Python. | `str` compara por puntos de código, igual que RF-1.11. | `locale`, `casefold` u orden natural: contradicen RF-1.11. | RF-1.11 |
| DT10 | `time.perf_counter_ns` inyectado como parámetro; cada ejecución se mide por separado y se promedia. | RF-12.8 lo pide; la inyección permite probar con un reloj falso. | Medir el bloque de 30 y dividir; `timeit` (no permite fijar 3+30 ni reloj falso con facilidad); `time.process_time` (CPU, no pared). | RF-12.1, RF-12.8 |
| DT11 | Recolector de basura desactivado dentro de la ventana medida y restaurado en `finally`. | Como solo se reporta la media, una pausa del recolector contamina un valor sin que haya dispersión que la delate. | Dejarlo activo; `gc.freeze`. | RF-12.4 |
| DT12 | Excepciones propias para entorno y entrada; `assert` para verificar. | `[P 4]` pide `assert` para verificar. `assert` desaparecería con optimizaciones y mezclaría errores de entorno con verificación. | Solo `assert`; `print` más `return` (viola P 3 y no detiene la ejecución). | RF-1.2, RF-4.2, RF-5.2 |
| DT13 | `preparar_consulta` valida todo el árbol antes de evaluar; `preparar_consultas` separa preparadas y rechazadas. | RF-5.6 pide rechazar solo la consulta afectada sin evaluar partes. | Validar durante la evaluación (podría evaluar parcialmente). | RF-5.6 |
| DT14 | Generación determinista de las 33 consultas con contadores rotativos y bloque AND completo antes del bloque OR. | Reproducible, cumple RF-10.4 (cada término aparece en un AND u OR; sin repetir término en una consulta). | Azar con semilla; producto cartesiano completo (excede las 33). | RF-10.3, RF-10.4 |
| DT15 | El inventario describe el tratamiento con una **sonda**: normaliza `"a" + c + "b"`. | Una sola fuente de verdad: el inventario no puede divergir de la normalización. | Tabla escrita a mano por clase. | RF-3.1, RF-3.3 |
| DT16 | La normalización se prueba contra una implementación de referencia carácter a carácter sobre todo el BMP. | Cubre las clases completas sin depender de la colección. | Solo ejemplos puntuales. | RF-2, RNF-3 |
| DT17 | La DF de los términos de prueba se calcula con el índice y se confirma con la búsqueda secuencial. | RF-10.7. Detecta un error del índice antes de medir. | Calcularla solo con el índice. | RF-10.7 |
| DT18 | Tablas y mensajes con texto formateado (`formatear_tabla`); sin pandas. | Pandas no está en el stack permitido `[P 1]`; no hay gráficas en la Parte A. | `pandas`, `matplotlib`. | P 1, RF-13 |
| DT19 | Se implementa la composición de operaciones (RF-9.4) sin interruptor. | La spec la declara opcional, pero NOT solo es útil compuesto y el costo de código es mínimo (recursión). Cubre las 33 consultas. | Omitir la composición (24 consultas); añadir un interruptor. | RF-9.4, RF-10.3 |
| DT20 | La lectura usa `nltk.corpus.gutenberg` (`fileids`, `raw`) y no abre archivos directamente. | `[P 5]`: NLTK solo para leer la colección. La decodificación la decide la colección `[RF-1.6]`. | `open` sobre la ruta de NLTK (habría que elegir una codificación); `nltk.download` (prohibido). | RF-1.1, RF-1.3 |
| DT21 | `RUTA_NLTK_DATA = None` usa la búsqueda estándar de NLTK; si tiene valor, se antepone a `nltk.data.path` en memoria. | `[RF-1.9]` pide una única variable con la instalación estándar por defecto. | Variable de entorno externa (no reproducible desde el notebook). | RF-1.9 |
| DT22 | Identificadores sin tildes ni eñes, en español. | `[P 6]` exige español; evita problemas de codificación de herramientas. | Identificadores en inglés (violan `[P 6]`); con tildes. | RNF-11 |

---

## 6. Estrategia de tests

No hay framework de pruebas: son `assert` dentro del notebook `[P 4]`. Las celdas V de cada sección usan **datos sintéticos** y se ejecutan antes de tocar la colección real, para fallar pronto.

### 6.1 Pruebas por función (celdas V)

| ID | Celda | Qué se comprueba |
|---|---|---|
| T-LEC | C07 | Con un lector falso: devuelve los ids tal cual; un archivo vacío da `""`; un `UnicodeDecodeError` produce `ErrorDeColeccion` con el nombre; `obtener_lector_nltk` con `nombre_modulo="modulo_inexistente"` lanza `ErrorDeColeccion`. |
| T-COL | C07 | `ordenar_ids(["b.txt","B.txt","a.txt","a-1.txt","a2.txt","a10.txt"])` = `["B.txt","a-1.txt","a.txt","a10.txt","a2.txt","b.txt"]`. `validar_cantidad` falla con 0, 17 y 19 y pasa con 18. `resumir_tamanos` sobre la colección sintética del §2.2: tokens 5, 4 y 0, caracteres = longitud de cada texto, totales correctos. |
| T-NORM | C07 | Ejemplos de la spec: «Café»→`cafe`; «año»→`ano`; «ß æ ø đ ł»→conservadas; «ﬁ»→`fi`; «x²»→`x2`; «ＡＢ»→`ab`; «İSTANBUL»→`istanbul`; «don't»→`dont`; «king’s»→`kings`; «dogs'»→`dogs`; «'tis»→`tis`; «don''t»→`don`,`t`; «well-known»→`well`,`known`; «a--b»→`a`,`b`; «_x_»→`x`; «co­operate»→`cooperate`; «a​b»→`ab`; «a b»→`a`,`b`; «3.14»→`3`,`14`; «5'10»→`510`; «b52»→`b52`; «»→vacío; «!!!»→vacío; «ΑΣ»→`ασ` (sin sigma final); «Ωμέγα»→`ωμεγα`. |
| T-DIF | C07 | Prueba diferencial: `normalizar_texto("a"+c+"b")` coincide con una función de referencia carácter a carácter (solo en el notebook, sin uso en producción) para todos los puntos de código del BMP y una muestra de los planos 1 y 2. Incluye `U+0378` y `U+E000`, que deben dar clase `no_clasificado`. |
| T-TERM | C07 | `normalizar_termino("Café")`=`cafe`; `" casa "`=`casa`; `"and"`=`and`; `""`, `"!!!"`, `"well-known"`, `"new york"`, `None` y `5` lanzan `TerminoInvalido` con el original y la cantidad o el tipo. |
| T-INV | C08 en C09/C10 | Sobre la colección sintética: las cuentas suman la longitud total; `é` aparece con tratamiento «se convierte en «e»»; un apóstrofo, «se une»; `U+0378` aparece como `no_clasificado`. |
| T-SEQ | C12 | `buscar_secuencial` devuelve ids ordenados y sin repetidos; un término repetido en un documento cuenta una vez; el término ausente da `[]`; no es subcadena («cat» no encuentra «cats»). Documento vacío: no aparece. |
| T-IDX | C17 | Listas ordenadas y sin repetidos; vocabulario = unión de tokens; `consultar_indice` de un ausente da `[]` y no crea entradas; `revisar_invariantes_indice` detecta una lista desordenada, una repetición y una DF alterada (se construyen casos defectuosos a propósito). |
| T-FUS | C17 | `fusionar_and`, `fusionar_or` y `complementar` coinciden con el álgebra de conjuntos para **todos** los pares de subconjuntos ordenados de un universo de 4 ids. |
| T-EXH | C17 | Para un vocabulario pequeño y todas las consultas de profundidad hasta 2 (términos, NOT, AND y OR), `resolver_secuencial` = `resolver_indice` = un oráculo con conjuntos (el oráculo solo existe en esta celda). Incluye documentos vacíos y términos ausentes. |
| T-CON | C21 | `generar_consultas_prueba` produce 33 consultas: 9, 6, 6, 3, 6 y 3 por familia en ese orden; cada consulta usa términos distintos; cada término de prueba aparece en al menos un AND u OR; en «(a OR b) AND NOT c» c difiere de a y b. `preparar_consultas` separa rechazos y deja pasar las demás. `texto_consulta` y `tipo_consulta` dan los valores esperados. |
| T-DF | C21 | `clasificar_df` en 0, 1, 2, 3, 11, 12 y 18 da `inexistente`, `poco frecuente`, `poco frecuente`, `intermedia`, `intermedia`, `frecuente`, `frecuente`. `revisar_terminos_prueba` detecta un término en la categoría equivocada, uno inválido, uno repetido y menos de 3 por categoría. |
| T-EQ | C24 | `ejecutar_protegido` captura una excepción como `("error", ...)`. `describir_diferencia` incluye ambas listas completas, los ids exclusivos de cada una y avisa cuando solo difiere el orden o hay repeticiones. `construir_filas_informe` incluye la advertencia de media 0 cuando `cero` es verdadero, marca «rechazada» en los rechazos sin tiempos y respeta el orden de las consultas. |
| T-MED | C15 | Con un ejecutable contador y un reloj falso: el ejecutable se llama exactamente `3 + 30` veces; las 3 primeras no cuentan; con incrementos conocidos la media sale exacta; `cero=True` cuando todos los tiempos son 0; el recolector queda restaurado también si el ejecutable lanza una excepción; `repeticiones=0` lanza `ValueError`. No se usa el reloj real, así que no hay tiempos reportados antes de RF-11.2. |
| T-FMT | C15 | `formatear_ms(0)`=`0.000000000`; `formatear_ms(1.2345678905)` redondea al más cercano; `formatear_tabla` alinea columnas y rechaza filas de otra longitud. |

### 6.2 Comprobaciones sobre datos reales

| ID | Celda | Comprobación | Cubre |
|---|---|---|---|
| T-REAL1 | C09 | `validar_cantidad` con 18 documentos. | RF-4.2 |
| T-REAL2 | C18 | `revisar_invariantes_indice` devuelve lista vacía. | RF-7.1, RF-8.3 |
| T-REAL3 | C22 | `revisar_terminos_prueba` devuelve lista vacía; si no, `assert` con el término y su DF. | RF-10.2, RF-10.6, RF-10.7 |
| T-REAL4 | C26 | Las 33 consultas y las identidades dan el mismo resultado en ambos mecanismos; los resultados esperados se cumplen. | RF-11 |
| T-REAL5 | C25 | `rechazos` está vacío con los términos de prueba válidos. | RF-5.6 |

### 6.3 Inspección estática del `.ipynb` (búsquedas de texto)

Las búsquedas de I-1 a I-8 (§1.4) con el resultado esperado indicado en cada una. La inspección es manual.

### 6.4 Ejecución de referencia

1. *Restart & Run All* con el kernel recién iniciado `[RNF-10]`.
2. Repetirlo: coinciden resultados, inventario, tamaños y DF; solo varían los tiempos `[RNF-1]`.
3. Registrar el tiempo de C29 frente a 5 minutos. La aceptación, si lo supera, es humana `[RNF-9, spec §9.1]`.

### 6.5 Inspección documentada (RNF-3)

| Requisito | Cómo se revisa |
|---|---|
| RF-1.3, RF-1.7, RF-1.8, RNF-5 | I-3; ninguna escritura salvo el propio notebook. |
| RF-2.2, RNF-7 | MD-N y MD-T presentes y completos; cada clase con su tratamiento y su motivo. |
| RF-2.11, RF-9.6 | I-1; no hay reducción a raíz, palabras vacías ni análisis de texto de consultas. |
| RF-6.2, RF-6.3, RF-6.6 | I-5 y lectura de `buscar_secuencial`: dos `for` anidados, sin `indice`, sin estructuras calculadas de antemano (ver `[P4]`). |
| RF-7.3, RF-7.4 | El índice solo vive en la sesión; I-3. |
| RF-10.2 (justificación) | MD-T explica cada término con su DF. |
| RF-13.3, RF-13.8 | La salida automática no compara mecanismos; MD-F solo cita datos del informe. |
| RNF-4, RNF-11 | Todo texto en español; los mensajes de bibliotecas quedan fuera. |

---

## 7. Secciones que no aplican

| Sección típica | Motivo |
|---|---|
| Módulos como archivos y paquetes | No aplica: `[P 1]` prohíbe cualquier archivo de código fuera del notebook. |
| API, CLI, endpoints | No aplica: la interfaz es ejecutar el notebook completo; la spec excluye interfaz y servicio (§7). |
| Base de datos y persistencia | No aplica: `[P 5]`, `[RNF-5]`; todo vive en memoria. |
| Red y descargas | No aplica: no se descarga nada `[P 5]`; la instalación es un paso manual previo. |
| Concurrencia y paralelismo | No aplica: prohibido por `[P 1]`. |
| Seguridad y autenticación | No aplica: no hay usuarios, credenciales ni servicio. |
| Despliegue y migraciones | No aplica: trabajo académico de un solo equipo. |
| Gráficas | No aplica en la Parte A: pertenecen a la Parte C (spec §7). |
| Medición de memoria y almacenamiento | No aplica: fuera de alcance (spec §7). |

---

## 8. Cambios sobre lo existente

El notebook `02_booleansearch.ipynb` existe y solo contiene el enunciado (celdas markdown) y celdas vacías. No hay código implementado que reutilizar o extender. Tampoco hay planes anteriores en `specs/` de Tarea2.

| Elemento existente | Qué ocurre | Motivo |
|---|---|---|
| Celdas markdown del enunciado (título, objetivo, secciones 1 a 9) | **Sin cambios.** | Son la fuente de verdad `[P 2]`. |
| Celdas de código vacías de las secciones 1 a 5 | **Se dejan donde están.** Las celdas nuevas se insertan inmediatamente después de la celda markdown de cada sección, antes de la celda vacía. | Integración sin borrar nada. Ver `[P5]` en §9. |
| Secciones de las Partes B y C (6 a 9) y sus celdas vacías | **Sin cambios.** | Fuera de alcance. |
| `notebooks/Tarea1/*` | **No se lee ni se reutiliza código.** | Otra tarea, otra constitución, otro stack. Solo se tomó el formato del plan como referencia. |
| `requirements.txt` | **Se amplía, sin quitar ni cambiar ninguna línea existente.** Se añaden `nltk==3.10.4` y `psutil==7.2.2` (esta última ya instalada en el `.venv`; la constitución la permite y la usarán las Partes B y C) y un comentario que recuerda que la colección Gutenberg se instala aparte. | Aprobado por el usuario el 2026-10-22. Antes el archivo no declaraba NLTK, que la Parte A necesita `[P6]`. Las dependencias de NLTK (`click`, `joblib`, `regex`, `tqdm`) las resuelve pip. |
| Consecuencia | El archivo `02_booleansearch.ipynb` cambia al agregar celdas y guardar salidas. Está permitido por RF-1.8. | |

Módulos reutilizados: ninguno. Módulos extendidos: ninguno.

---

## 9. Decisiones pendientes de confirmar

No se pueden deducir de los documentos y no se inventan aquí.

**Resueltas al aprobar el plan (2026-10-22):** P4 (el conjunto de trabajo de `resolver_secuencial` es aceptable), P5 (las celdas vacías existentes se dejan donde están), P6 (`requirements.txt` declara `nltk` y `psutil`, ver §8) y P7 (orden de las 33 consultas tal como se propuso). La tabla siguiente conserva las preguntas y las razones originales del plan.

**Registro técnico posterior (2026-10-27):** P1: se eligieron y comprobaron los nueve términos de C02; P2: el lector de NLTK declara `latin1` y el inventario quedó en C10; P3: la ejecución y las pruebas de C07 verificaron las operaciones del lector y el tratamiento de ausencia; P8: C29 mostró 74,5 s en la ejecución completa más reciente, por debajo de 300 s; P9: las ejecuciones de referencia anteriores registraron una resolución de 100 ns y ninguna media en cero. El usuario confirmó que T-45 ya estaba terminada; no precisó los datos de la aceptación original. La auditoría de `validation.md` mantiene el hallazgo independiente PEND-3 sobre trabajo repetido de RNF-9.

| ID | Pendiente | Por qué falta | Cuándo se resuelve |
|---|---|---|---|
| P1 | Los 9 términos de prueba (3 frecuentes con DF ≥ 12, 3 poco frecuentes con DF 1 o 2 y 3 inexistentes). Hasta entonces `TERMINOS_PRUEBA` queda con marcadores. Elegir los poco frecuentes exige ver las DF reales; para eso hace falta una celda de exploración temporal (que se elimina al terminar) o aprobar una función auxiliar `terminos_con_df`. | La colección no está instalada: no se conocen las DF. | Después de instalar NLTK y la colección, ejecutando las secciones 1 y 4. |
| P2 | Codificación con la que el lector de NLTK declara decodificar esta colección y si algún archivo muestra caracteres mal decodificados. | NLTK no está instalado: no se puede comprobar. | El inventario de C10 lo mostrará en la primera ejecución. |
| P3 | Que `nltk.corpus.gutenberg`, `fileids()`, `raw()` y `LookupError` se comporten como asume §3.1 en la versión instalada. | NLTK no está instalado en el equipo. | Al instalar NLTK. |
| P4 | Interpretación de RF-6.6: el conjunto de trabajo que crea `resolver_secuencial` en cada llamada (tamaño ≤ número de términos de la consulta) no se considera una estructura «calculada de antemano». | La spec no distingue estructuras creadas durante la consulta de las precalculadas. | Al aprobar este plan. |
| P5 | Qué hacer con las celdas de código vacías existentes: dejarlas (propuesta) o reutilizarlas. | El plan no puede decidir sobre archivos del usuario. | Al aprobar este plan. |
| P6 | Si `requirements.txt` debe declarar `nltk` (hoy no lo lista; `psutil` no es necesario en la Parte A). | Es un archivo existente y la instalación es manual por la spec. | Al aprobar este plan. |
| P7 | Orden dentro de la lista de las 33 consultas: bloque AND completo y luego bloque OR; y para «(a OR b) AND NOT c», a y b son de las dos categorías distintas de c. | RF-10.3 fija las familias pero no el orden entre AND y OR ni qué categorías toman a y b. | Al aprobar este plan. |
| P8 | Si el tiempo total de la Parte A cabe en 5 minutos. | Depende de las DF, del tamaño real de la colección y del equipo; no se estima aquí sin datos. | Primera ejecución completa (spec §9.1). |
| P9 | La resolución real del reloj en este equipo (si las medias del índice salen en 0). | Depende del sistema; lo cubre RF-12.11. | Primera ejecución completa. |

---

## 10. Tabla RF → sección del plan

Columnas: dónde se implementa, dónde se verifica. «Insp.» = inspección documentada (§6.5). Ningún RF vigente queda sin cobertura. Los dos derogados no requieren cobertura.

| RF | Se implementa en | Se verifica en |
|---|---|---|
| RF-1.1 | §3.1 `extraer_textos`; C04; DT20 | T-LEC |
| RF-1.2 | §3.1 `obtener_lector_nltk`; C04 | T-LEC (parte del módulo ausente); Insp. con la colección ausente |
| RF-1.3 | DT20; I-3 | Insp. |
| RF-1.4 | §3.1 `obtener_lector_nltk`; C04 | T-LEC (`modulo_inexistente`) |
| RF-1.5 | §3.1 `extraer_textos` | T-LEC |
| RF-1.6 | §3.1 `extraer_textos`; DT20 | T-LEC (archivo vacío), T-INV |
| RF-1.7 | DT20; I-3 | Insp. |
| RF-1.8 | §8 (guardado del notebook) | Insp. |
| RF-1.9 | C02; DT21 | Insp. (una sola variable de ruta) |
| RF-1.10 | §3.1; §2.1 | T-LEC, T-COL |
| RF-1.11 | §3.2 `ordenar_ids`; DT9 | T-COL |
| RF-1.12 | C04 (`version_nltk`); C10 | Insp. (la salida muestra Python y NLTK) |
| RF-2.1 | §3.3 `normalizar_texto`; una sola función para documentos y términos | T-TERM, T-NORM |
| RF-2.2 | MD-N | Insp. |
| RF-2.3 | §3.3 (clase `letra`); DT5 | T-NORM, T-DIF |
| RF-2.4 | §3.3 (NFKD y clase `marca`); DT4 | T-NORM, T-DIF |
| RF-2.5 | §3.3 (las letras sin descomposición se conservan) | T-NORM, T-DIF |
| RF-2.6 | §3.3 (`digito`, tokens por separadores) | T-NORM |
| RF-2.7 | §3.3 (regex del apóstrofo aislado) | T-NORM |
| RF-2.8 | §3.3 (apóstrofo restante separa) | T-NORM |
| RF-2.9 | §3.3 (clase `guion`) | T-NORM |
| RF-2.10 | §3.3 (otras clases separan) | T-NORM, T-DIF |
| RF-2.11 | I-1; ausencia de reducción a raíz y palabras vacías | Insp. |
| RF-2.12 | C02, C06: alcance de la única función | Insp. |
| RF-2.13 | §3.3 `clasificar_caracter` | T-DIF, T-INV |
| RF-2.14 | §3.3 NFKD; DT4 | T-NORM, T-DIF |
| RF-2.15 | §3.3 NFKD y clase `marca` | T-NORM, T-DIF |
| RF-2.16 | §3.3 clase `letra` | T-NORM, T-DIF |
| RF-2.17 | §3.3; DT5 | T-NORM («ß», «ΑΣ») |
| RF-2.18 | §3.3 (orden de pasos) | T-NORM, T-DIF |
| RF-2.19 | §3.3 (regex `'{2,}`; acentos como símbolos) | T-NORM |
| RF-2.20 | §3.3 (guiones, `_`, `*` separan) | T-NORM |
| RF-2.21 | §3.3 (clases `marca` y `formato` se eliminan) | T-NORM, T-DIF |
| RF-2.22 | §3.3 (clase `espacio`) | T-NORM, T-DIF |
| RF-3.1 | §3.4 `inventariar_caracteres`; C08, C10 | T-INV |
| RF-3.2 | §3.3 (clase `no_clasificado`); §3.4 | T-DIF, T-INV |
| RF-3.3 | §3.4; DT15 | T-INV |
| RF-4.1 | C10 | Insp. (la salida muestra el número) y T-REAL1 |
| RF-4.2 | §3.2 `validar_cantidad`; C09 | T-COL, T-REAL1 |
| RF-4.3 | Derogado en la spec | No aplica |
| RF-4.4 | Derogado en la spec | No aplica |
| RF-4.5 | §3.2 `resumir_tamanos`; C10 | T-COL |
| RF-4.6 | §3.2 (tokens = ocurrencias) | T-COL |
| RF-4.7 | §3.2 (longitud del texto decodificado) | T-COL |
| RF-5.1 | §3.3 `normalizar_termino` | T-TERM |
| RF-5.2 | §3.3 `normalizar_termino`; DT12 | T-TERM |
| RF-5.3 | §3.5, §3.6 (resultado `[]`) | T-SEQ, T-IDX, T-EXH |
| RF-5.4 | §3.7 `preparar_consulta` (una sola vía para ambos mecanismos) | T-CON, T-EXH |
| RF-5.5 | §3.3 `normalizar_termino` | T-TERM |
| RF-5.6 | §3.7 `preparar_consultas`; DT13; C25 | T-CON, T-REAL5 |
| RF-5.7 | §3.3 (no hay palabras reservadas) | T-TERM («and») |
| RF-6.1 | §3.5 `buscar_secuencial` | T-SEQ |
| RF-6.2 | §3.5; DT6 | Insp. (I-5) |
| RF-6.3 | §3.5; I-5 | Insp. |
| RF-6.4 | §3.5 (`==` sobre tokens) | T-SEQ |
| RF-6.5 | §3.5 (`ROMPER`) | T-SEQ |
| RF-6.6 | §3.5; DT6 | Insp.; pendiente `[P4]` |
| RF-7.1 | §3.6 `construir_indice`; DT7 | T-IDX, T-REAL2 |
| RF-7.2 | §3.6 `consultar_indice` | T-IDX |
| RF-7.3 | §2.1; I-3 | Insp. |
| RF-7.4 | §2.1 (vida de la sesión) | Insp. |
| RF-8.1 | §3.6 auxiliares; C19 | T-IDX; Insp. de la salida |
| RF-8.2 | §3.6 `mayores_df`, `distribucion_df`; C19 | T-IDX |
| RF-8.3 | §3.6 `revisar_invariantes_indice`; C18 | T-IDX, T-REAL2 |
| RF-8.4 | §3.6 `mayores_df` | T-IDX |
| RF-9.1 | §3.6 `fusionar_and`; §3.5 `evaluar_presencia` | T-FUS, T-EXH |
| RF-9.2 | §3.6 `fusionar_or`; §3.5 | T-FUS, T-EXH |
| RF-9.3 | §3.6 `complementar`; `indice.universo` | T-FUS, T-EXH |
| RF-9.4 | §3.6 `resolver_indice`; §3.5; DT19 | T-EXH |
| RF-9.5 | §3.5, §3.6 | T-EXH, T-REAL4 |
| RF-9.6 | DT2; I-1 | Insp. |
| RF-9.7 | §3.5 `resolver_secuencial`; I-5 | T-EXH; Insp.; pendiente `[P4]` |
| RF-9.8 | DT19 | T-EXH |
| RF-10.1 | §3.7 `clasificar_df`; C02 | T-DF |
| RF-10.2 | §3.7 `revisar_terminos_prueba`; C22 | T-DF, T-REAL3; MD-T |
| RF-10.3 | §3.7 `generar_consultas_prueba`; C25 | T-CON |
| RF-10.4 | §3.7 `generar_consultas_prueba`; DT14 | T-CON |
| RF-10.5 | §3.7 `clasificar_df` («intermedia») | T-DF |
| RF-10.6 | §3.7 `revisar_terminos_prueba`; C22 | T-DF, T-REAL3 |
| RF-10.7 | §3.7; DT17; C22 | T-DF, T-REAL3; MD-T |
| RF-11.1 | §3.8; C26 | T-EXH, T-REAL4 |
| RF-11.2 | Orden de celdas: C26 antes de C27 | T-MED (no mide con reloj real); Insp. del orden |
| RF-11.3 | §3.8 `describir_diferencia`; C26 | T-EQ |
| RF-11.4 | §3.8 `describir_diferencia` | T-EQ |
| RF-11.5 | §3.8 `ejecutar_protegido` | T-EQ |
| RF-11.6 | §3.8 (C26) | T-REAL4 |
| RF-12.1 | §3.9 `medir_media_ms`; C27 | T-MED |
| RF-12.2 | §3.9 `formatear_ms` | T-FMT |
| RF-12.3 | §3.9; C02 (`CALENTAMIENTOS`) | T-MED |
| RF-12.4 | §3.9 (cierres con términos ya preparados); I-7 | T-MED; Insp. |
| RF-12.5 | §3.9 (una llamada por consulta y mecanismo) | T-MED |
| RF-12.6 | §3.5, §3.6 (todo el trabajo dentro del ejecutable); MD-M | Insp. |
| RF-12.7 | §3.9 (no se mide carga, normalización ni índice) | Insp. |
| RF-12.8 | §3.9; DT10 | T-MED |
| RF-12.9 | §3.9 `formatear_ms` | T-FMT |
| RF-12.10 | §3.9 C27 | Insp. del bucle |
| RF-12.11 | §3.9 (`cero`); §3.10 | T-MED; T-EQ |
| RF-13.1 | §3.10 `construir_filas_informe`; C28 | T-EQ; Insp. de la salida |
| RF-13.2 | §3.10 (sin dispersión) | Insp. |
| RF-13.3 | §3.10; MD-F | Insp. |
| RF-13.4 | §3.9 C27; §3.10 C28 | Insp. |
| RF-13.5 | §3.7 `texto_consulta`, `tipo_consulta`; §3.10 | T-CON |
| RF-13.6 | §3.10 (categorías en el orden de la consulta) | T-CON |
| RF-13.7 | §3.10 | Insp. |
| RF-13.8 | MD-F | Insp. |
| RF-13.9 | §3.9 C27 (`total_ms`) | Insp. |
| RF-13.10 | §3.7 `preparar_consultas`; §3.10 | T-CON, T-EQ |

**Requisitos no funcionales:** RNF-1 → §6.4 · RNF-2 → §3.9, I-7 · RNF-3 → §6 completo · RNF-4 y RNF-11 → §0 (P 6), DT22 · RNF-5 → I-3, §0 (P 5) · RNF-6 → DT3 (sin optimización de memoria) · RNF-7 → MD-N, MD-T · RNF-8 → §4 (funciones reciben datos en memoria; sin supuestos sobre el nombre de archivo salvo el orden) · RNF-9 → DT3, DT7, C29 · RNF-10 → §6.4 · RNF-12 → C10, C29.

**RF con cobertura solo por inspección** (no verificable por `assert`, según RNF-3): RF-1.3, RF-1.7, RF-1.8, RF-2.2, RF-2.11, RF-2.12, RF-6.2, RF-6.3, RF-6.6, RF-7.3, RF-7.4, RF-9.6, RF-11.2 (el orden de celdas), RF-12.4, RF-12.6, RF-12.7, RF-12.10, RF-13.2, RF-13.3, RF-13.4, RF-13.7, RF-13.8, RF-13.9.
