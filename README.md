

# Clasificación y Métricas de Aprendizaje — Dataset Bancario


**Pontificia Universidad Javeriana — Bogotá, Colombia**  
Procesamiento de Datos a Gran Escala · Samuel Clavijo · 2026

</div>

---

##  Descripción

Este proyecto implementa un pipeline completo de **clasificación supervisada** sobre el *Bank Marketing Dataset* del repositorio UCI Machine Learning. El objetivo es predecir si un cliente de un banco portugués suscribirá un **depósito a plazo fijo** (`y = yes/no`) a partir de sus características demográficas, financieras y de contacto con la campaña.

Todo el procesamiento se realiza sobre **Apache Spark** en un clúster distribuido, aprovechando sus capacidades de procesamiento paralelo a gran escala.

---

##  Objetivo

> Analizar y comparar el rendimiento de diferentes modelos de clasificación sobre datos bancarios reales, aplicando un pipeline completo de ML que incluye exploración, limpieza, balanceo, codificación y evaluación con métricas estándar de la industria.


---

## 🗃️ Dataset

| Atributo | Detalle |
|---|---|
| **Fuente** | [UCI ML Repository — Bank Marketing](https://archive.ics.uci.edu/ml/datasets/bank+marketing) |
| **Registros** | 45.211 |
| **Variables** | 17 (16 features + 1 target) |
| **Variable objetivo** | `y` — ¿Suscribió el depósito? (`yes` / `no`) |
| **Desbalance** | ~88.3% `no` / ~11.7% `yes` |
| **Separador** | `;` (formato europeo) |

### Variables principales

| Variable | Tipo | Descripción |
|---|---|---|
| `age` | Numérica | Edad del cliente |
| `job` | Categórica | Tipo de ocupación (12 categorías) |
| `marital` | Categórica | Estado civil |
| `education` | Categórica | Nivel educativo |
| `balance` | Numérica | Saldo anual promedio (euros) |
| `duration` | Numérica | Duración del último contacto (segundos) |
| `campaign` | Numérica | Contactos durante la campaña actual |
| `poutcome` | Categórica | Resultado de campaña anterior |
| `y` | Binaria | **Variable objetivo** |

---

## 🔄 Pipeline de Procesamiento

```
Carga HDFS → Diagnóstico → Casting → EDA → Limpieza → Balanceo → Codificación → Pipeline ML → Modelos → Evaluación
```

### Etapas detalladas

#### 1. 🔍 Exploración (EDA)
- Estadísticas descriptivas con `describe()`
- Histogramas de variables numéricas
- Boxplots bivariados (variable vs. `y`)
- Matriz de correlación de Pearson (PySpark ML Stat)
- Pairplot y gráficos categóricos cruzados

#### 2. 🧹 Limpieza
- Detección de nulos por columna
- Filtro de outliers: `previous > 30` eliminados
- Eliminación de `pdays` (81.7% valores `-1`, sin información útil)

#### 3. ⚖️ Balanceo
- Técnica: **Random Oversampling** de la clase minoritaria (`yes`)
- Resultado: dataset balanceado ~50/50 en `df04`

#### 4. 🔤 Codificación de Categóricas
Para cada variable categórica:
```python
StringIndexer  →  OneHotEncoder
  (índice)         (vector binario)
```

#### 5. 🔧 Pipeline ML
```python
Pipeline(stages=[
    StringIndexer × 9,
    OneHotEncoder × 9,
    StringIndexer (y → label),
    VectorAssembler (→ features)
])
```

#### 6. 🤖 Modelos Entrenados

| Modelo | Clase PySpark | Split |
|---|---|---|
| Regresión Logística | `LogisticRegression` | 80% train / 20% test |
| Árbol de Decisión | `DecisionTreeClassifier` | 80% train / 20% test |

#### 7. 📈 Métricas de Evaluación

- **Accuracy** — proporción de predicciones correctas
- **Precision** — exactitud en predicciones positivas
- **Recall** — capacidad de detectar positivos reales
- **F1-Score** — balance entre precisión y recall
- **AUC-ROC** — capacidad discriminativa global del modelo

---

## 🛠️ Tecnologías

| Herramienta | Versión | Uso |
|---|---|---|
| Python | 3.8+ | Lenguaje base |
| PySpark | 3.x | Procesamiento distribuido y ML |
| Hadoop HDFS | 3.x | Almacenamiento del dataset |
| pandas | 1.x+ | Visualización local |
| seaborn / matplotlib | Latest | Gráficas estadísticas |
| scikit-learn | Latest | Curva ROC complementaria |

---

## ⚙️ Configuración del Clúster

```python
configura = SparkConf()
configura.set("spark.scheduler.mode", "FAIR")
configura.setMaster("spark://10.43.97.177:7077")
configura.setAppName("Taller_Banca_Metricas_Clavijo")

sparkClavijo = SparkSession.builder.config(conf=configura).getOrCreate()
```

> El notebook está configurado para ejecutarse en un clúster Spark standalone. Para correrlo en modo local, cambiar `.setMaster("spark://...")` por `.setMaster("local[*]")`.

---

## 🚀 Cómo ejecutar

```bash
# 1. Clonar el repositorio
git clone https://github.com/<tu-usuario>/taller-clasificacion-banca.git
cd taller-clasificacion-banca

# 2. Instalar dependencias
pip install pyspark pandas seaborn matplotlib scikit-learn findspark

# 3. Asegurarse de tener el dataset en la ruta configurada
#    Por defecto: file:///Almacen/bank-full.csv
#    Ajustar la ruta en la celda de carga de datos

# 4. Abrir el notebook
jupyter notebook TallerClavijo_Final.ipynb
```

---

## 📊 Resultados

Los modelos se comparan usando F1-Score y AUC-ROC como métricas principales, dado el desbalance original del dataset. El gráfico de barras horizontales al final del notebook muestra el ranking de modelos.

> Los resultados exactos dependen de la ejecución en el clúster (determinista por `seed=4321` en el split).

---

## 📝 Notas Importantes

- `duration` es el predictor más fuerte pero **no debe usarse en producción** (no se conoce antes de la llamada).
- El notebook usa `seed=42` para el oversampling y `seed=4321` para el train/test split, garantizando reproducibilidad.
- La columna `pdays` fue eliminada por tener 81.7% de valores especiales (`-1`).
- El modelo final se guarda en formato Parquet: `output.parquet`.
- El pipeline entrenado se persiste en: `modeloPipeline/`.

---

## 👤 Autor

**Samuel Clavijo**  
Pontificia Universidad Javeriana · Bogotá, Colombia  
Materia: Procesamiento de Datos a Gran Escala · 2026

---

<div align="center">
  <sub>Hecho con ☕ y mucho PySpark</sub>
</div>
