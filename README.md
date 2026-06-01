# Proyecto 1er Bimestre – Sistema de Recuperación de Información

**Integrantes:**  Danny Constante - Alexander Paillacho - Andrés Pérez 
**Asignatura:** Recuperación de Información  
**Profesor:** Iván Carrera  
**Fecha:** Junio 2026  

## Descripción del proyecto

Este proyecto implementa un sistema de recuperación de información que compara modelos clásicos (Jaccard, TF-IDF + coseno, BM25) con un modelo semántico moderno basado en embeddings (sentence-transformers) y una base de datos vectorial (FAISS).

El sistema indexa un corpus de más de 51,000 documentos del conjunto de noticias financieras de Reuters (8 archivos CSV). Permite realizar consultas en texto libre, obtener rankings de documentos y evaluar la calidad mediante métricas estándar: Precisión, Recall y MAP.

## Estructura del repositorio

- `proyecto_1b.ipynb` – Notebook principal con todo el código.
- `limpieza.py` – Módulo de preprocesamiento (generado automáticamente al ejecutar el notebook).
- `README.md` – Este archivo.

## Requisitos técnicos

- Python 3.8 o superior
- Las siguientes librerías (se instalan automáticamente en la Celda 0):
  - `kagglehub` – descarga del corpus Reuters.
  - `sentence-transformers` – embeddings semánticos.
  - `faiss-cpu` – base de datos vectorial.
  - `scikit-learn` – TF-IDF y PCA.
  - `pandas`, `numpy`, `tqdm`, `matplotlib`, `seaborn`.
  - `nltk` – tokenización, stopwords, lematización.

Se recomienda ejecutar el notebook en Google Colab con acelerador GPU activado para un cálculo rápido de los embeddings.

## Instrucciones de ejecución

1. Sube el notebook `proyecto_1b.ipynb` a Google Colab o ábrelo en un entorno Jupyter.
2. Ejecuta todas las celdas en orden (de la 0 a la 10). Cada celda está documentada.
   - La primera ejecución descargará el corpus de Kaggle (requiere conexión a Internet) y los modelos de Hugging Face.
   - El preprocesamiento e indexado pueden tomar unos minutos.
   - La generación de embeddings semánticos puede tardar entre 10 y 20 minutos en una GPU T4 de Colab.
3. Al final de la Celda 7 se inicia la interfaz de línea de comandos (CLI). Allí podrás probar el sistema.

## Explicación de cada celda

| Celda | Nombre | Función |
|-------|--------|---------|
| 0 | Instalación de dependencias | Instala kagglehub, sentence-transformers, faiss-cpu, etc. |
| 1 | Importaciones | Carga todas las librerías necesarias (pandas, numpy, sklearn, nltk, etc.). |
| 2 | Carga del corpus | Descarga el dataset Reuters desde Kaggle, lee los 8 archivos CSV, unifica los documentos y añade trazabilidad (columna csv_origen). |
| 3 | Módulo limpieza.py | Define dos pipelines de preprocesamiento: uno léxico (lematización, stopwords, solo letras) y otro semántico (limpieza superficial conservando sintaxis). |
| 3.1 | Ejecución del pipeline | Aplica el preprocesamiento a todo el corpus y genera columnas: texto_semantico, texto_lexico_str, tokens_lexicos. |
| 4 | Índice invertido y metadatos | Construye un índice invertido {término: {doc_id: frecuencia}} y un diccionario de metadatos (longitud, tokens, título, etc.). |
| 5.1 | Modelo Jaccard | Implementa la similitud de Jaccard sobre conjuntos de tokens binarios. |
| 5.2 | Modelo TF-IDF + coseno | Usa TfidfVectorizer de scikit-learn y calcula similitud coseno entre la consulta y los documentos. |
| 5.3 | Modelo BM25 | Implementación manual de Okapi BM25 con k1=1.5, b=0.75 y normalización por longitud. |
| 6 | Modelo semántico + FAISS | Genera embeddings con all-mpnet-base-v2, los normaliza y los indexa en FAISS (IndexFlatIP). La búsqueda se hace por producto interno (equivalente a coseno). |
| 7 | Interfaz CLI | Bucle interactivo que permite escribir consultas, elegir modelo (1-4) y navegar por los resultados paginados. |
| 8 | Batería de pruebas automatizada | Ejecuta 5 consultas predefinidas y muestra los top-5 de cada modelo en formato tabla. |
| 9 | Análisis espacial (PCA) | Reduce los embeddings a 2D con PCA, grafica la nube del corpus y los resultados de cada modelo, mostrando la dispersión (varianza) de cada modelo. |
| 10 | Evaluación cuantitativa (MAP) | Construye qrels por pooling (consenso de los top-10 de cada modelo). Calcula Precision@25, Recall@25, AP@25 y MAP. Presenta tablas y una matriz de calor. |

## Cómo usar la interfaz CLI (Celda 7)

Al ejecutar la Celda 7 aparecerá un menú interactivo:

1. Escribe tu consulta en texto libre (ej. `animals`, `oil prices`).
2. Selecciona el modelo (1: BM25, 2: TF-IDF, 3: Jaccard, 4: Semántico).
3. El sistema mostrará una tabla con los primeros 10 resultados (página 1).
4. Usa las opciones:
   - `S` (Siguiente) para ver más resultados.
   - `A` (Anterior) para retroceder.
   - `N` (Nueva búsqueda) para volver al menú principal.
   - `salir` para terminar.

## Cumplimiento de los requisitos del proyecto

A continuación se detalla cómo el código satisface cada punto del enunciado:

### a. Construcción del índice
- Se leen los 8 archivos CSV del corpus Reuters (51,077 documentos).
- Tokenización, normalización (minúsculas, eliminar no alfabético) y eliminación de stopwords (NLTK + palabras financieras).
- Se construye un índice invertido `{término: {doc_id: frecuencia}}`.

### b. Modelos de recuperación
- **Jaccard**: `jaccard_search` – vectores binarios sobre conjuntos de tokens.
- **TF-IDF + coseno**: `TfidfVectorizer` + `cosine_similarity`.
- **BM25**: implementación manual con parámetros k1=1.5, b=0.75.
- Se permite ejecutar consultas de texto libre y se muestra un ranking ordenado.

### c. Interfaz básica (CLI)
- Interfaz de línea de comandos dentro del notebook usando `input()` y `print()`.
- Permite realizar consultas, elegir modelo y visualizar resultados paginados.

### d. Recuperación semántica con embeddings
- Modelo `all-mpnet-base-v2` (sentence-transformers).
- Embeddings para todos los documentos y para las consultas.
- Almacenamiento en FAISS (base de datos vectorial) con índice de producto interno.
- Búsqueda vectorial y ranking por similitud coseno.

### e. Evaluación de resultados
- Conjunto de consultas de prueba predefinidas.
- Construcción de qrels mediante pooling (consenso de los top-10 de cada modelo).
- Cálculo de Precision@25, Recall@25 y AP@25 por consulta.
- Cálculo del MAP (Mean Average Precision) global para cada modelo.

### f. Comparación de modelos
- Tablas comparativas de MAP y AP.
- Análisis cualitativo en celdas Markdown (tipos de consultas, casos donde el semántico mejora o empeora).
- Gráficos PCA que muestran la dispersión espacial de los resultados de cada modelo.

### g. Requisitos técnicos
- Lenguaje Python.
- Librerías permitidas: numpy, pandas, nltk, scikit-learn, sentence-transformers, faiss-cpu.
- No se utilizan motores de búsqueda completos (Elasticsearch, Solr, Whoosh).
- La implementación de los modelos clásicos es propia o está claramente explicada.

## Nota sobre los qrels (juicios de relevancia)

El proyecto no incluye un archivo de qrels externo. En su lugar, se genera un conjunto de documentos relevantes mediante la técnica de **pooling**: se toman los top-10 resultados de cada modelo para cada consulta, se unen y se consideran relevantes. Esto es una práctica válida en entornos no supervisados y permite demostrar la correcta implementación de las métricas de evaluación. Si se dispusiera de un conjunto de qrels real, el código podría adaptarse fácilmente.
