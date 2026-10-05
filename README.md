# Recuperación de Información

Repositorio oficial de la asignatura **ICCD753 Recuperación de Información**.

Este repositorio contiene el material utilizado durante el curso: clases magistrales, notebooks prácticos, microtutoriales, datasets, actividades y proyectos.

El objetivo de la asignatura es comprender y construir sistemas modernos de recuperación de información, desde los modelos clásicos basados en términos hasta las arquitecturas actuales basadas en representaciones densas, transformers y Retrieval-Augmented Generation (RAG).

Aunque este repositorio está diseñado para los estudiantes de la asignatura, el material está disponible para cualquier persona interesada en aprender sobre Recuperación de Información.

---

## Contenidos del curso

Durante el curso se estudian los principales paradigmas de Recuperación de Información:

- Problema de Recuperación de Información e Information Seeking.
- Modelos clásicos:
  - Recuperación booleana.
  - Modelo de espacio vectorial.
  - TF-IDF.
  - Similaridad coseno.
  - Modelos probabilísticos.
  - BM25.
- Evaluación de sistemas de recuperación:
  - Precision.
  - Recall.
  - F1-score.
  - MAP.
  - DCG/NDCG.
- Construcción de índices invertidos.
- Procesamiento y representación de documentos.
- Recuperación basada en embeddings.
- Dense Retrieval.
- Búsqueda vectorial.
- Re-ranking.
- Transformers para recuperación de texto.
- Sistemas RAG (Retrieval-Augmented Generation).

---

## Organización del repositorio

```
.
├── notebooks/
│   ├── 01_...
│   ├── 02_...
│   ├── ...
│
├── data/
│   ├── datasets utilizados en las actividades
│   └── archivos auxiliares
│
├── slides/
│   └── diapositivas de las clases
│
└── README.md
```

Las actividades prácticas están organizadas progresivamente dentro de la carpeta `notebooks/`.

Cada notebook representa una etapa del aprendizaje y busca que los estudiantes construyan componentes reutilizables para actividades posteriores.

---

# Entorno de trabajo

Los notebooks pueden ejecutarse utilizando cualquiera de los siguientes entornos:

- Google Colab.
- Kaggle Notebooks.
- Jupyter Notebook/JupyterLab local.

Cada estudiante puede elegir el entorno que mejor se adapte a sus necesidades.

---

# Entrega de actividades

Las actividades se desarrollan utilizando repositorios personales de cada estudiante.

Para entregar una actividad:

1. Crear una solución en el repositorio personal del estudiante.
2. Verificar que el notebook se haya ejecutado correctamente.
3. Compartir únicamente el enlace al notebook o solución implementada.

La entrega no consiste en enviar archivos directamente, sino en mantener un historial reproducible del trabajo realizado.

---

# Uso de herramientas de Inteligencia Artificial

Durante el curso se permite y se recomienda utilizar herramientas de asistencia basadas en Inteligencia Artificial, incluyendo:

- ChatGPT.
- Gemini.
- Claude Code.
- Otras herramientas equivalentes.

Sin embargo, el objetivo no es reemplazar el aprendizaje, sino aprender a trabajar profesionalmente con estas herramientas.

Antes de usar una herramienta, el estudiante debe ser hacerse estas preguntas:

1. ¿Qué tarea necesito resolver?
2. ¿Tiene un resultado exacto comprobable o admite varias propuestas válidas?
3. ¿Qué consecuencias tendría un error?
4. ¿Existe una herramienta especializada que la resuelva directamente?
5. ¿Cómo comprobaré el resultado?

Recuerda que la IA debe utilizarse como un asistente de aprendizaje y programación, no como un sustituto del razonamiento técnico.

---

# Colaboración

La colaboración entre estudiantes no solo que está permitida, sino que se valora mucho el trabajo colaborativo.

Se espera que los estudiantes compartan conocimiento, discutan soluciones y aprendan unos de otros.

Sin embargo, cada estudiante debe comprender y poder explicar las soluciones entregadas.

---

# Reutilización de código

Uno de los objetivos del curso es aprender a construir software reutilizable.

Por esta razón, se recomienda:

- Crear funciones y módulos reutilizables.
- Construir pequeñas librerías personales.
- Evitar copiar y pegar código repetido.
- Mejorar implementaciones realizadas en actividades anteriores.

Los componentes desarrollados durante el curso pueden y deben ser utilizados como base para actividades posteriores.

---

# Bibliografía principal

El curso utiliza como referencia libros y artículos clásicos y actuales de Recuperación de Información, incluyendo:

- Baeza-Yates & Ribeiro-Neto. *Modern Information Retrieval*.
- Manning, Raghavan & Schütze. *Introduction to Information Retrieval*.
- Lin, Nogueira & Yates. *Pretrained Transformers for Text Ranking: BERT and Beyond*.

---

# Docente

**Prof. Iván Carrera**  
Escuela Politécnica Nacional  
Asignatura: Recuperación de Información

---

# Licencia

El material de este repositorio tiene fines educativos.

Se permite su uso para aprendizaje y enseñanza, manteniendo la atribución correspondiente.