# Plan de Implementación: Predictor de Sueldos (Clasificación)

Este plan actualiza la arquitectura y el flujo de trabajo basándose en los datos reales encontrados en `data.csv`. Se reemplazan las variables hipotéticas (GPA, Habilidades numéricas) por las variables reales del dataset.

## User Review Required

> [!IMPORTANT]
> **Cambio de Variables (Features):** El dataset `data.csv` no contiene `GPA` ni habilidades del 1 al 10. Para que el modelo funcione correctamente y el formulario web tenga sentido, he seleccionado las siguientes columnas como entradas para la red neuronal:
> 
> **Variables Numéricas (Requerirán un slider o input de número en la web):**
> * `Employer_Reputation_Score (1–100)`
> * `Skill_Demand_Score (1–100)`
> * `Remote_Work_Availability (%)`
> 
> **Variables Categóricas (Requerirán un menú desplegable/Select en la web):**
> * `Degree_Level` (Ej: Bachelor, Master, PhD)
> * `Field_of_Study` (Ej: Engineering, Computer Science, Data Science & AI, etc.)
> * `Region` (Ej: North America, Europe, Asia-Pacific, etc.)

## Open Questions

> [!WARNING]
> ¿Estás de acuerdo con esta selección de variables para el formulario predictivo? Si prefieres incluir otras (como `Top_Industry` o `Country` en lugar de `Region`), házmelo saber antes de empezar a programar.

## Proposed Changes

### Entorno de Machine Learning (`ml/`)

#### [NEW] `ml/train.py`
Script principal de entrenamiento que realizará lo siguiente:
1. **Carga y Preprocesamiento:** 
   * Leerá `data.csv`.
   * Convertirá `Average_Starting_Salary_USD` en 3 clases (Bajo, Medio, Alto) usando cuantiles para evitar desbalanceo de clases.
   * Aplicará `StandardScaler` a las variables numéricas y `OneHotEncoder` a las categóricas.
2. **Entrenamiento:** Entrenará varios modelos (Perceptrón Simple y Perceptrón Multicapa) usando `softmax` y `sparse_categorical_crossentropy`.
3. **Exportación (El cambio clave):**
   * Exportará el modelo ganador a `public/web_model/`.
   * **NUEVO:** Exportará un archivo `scaler_params.json` (medias, varianzas y nombres de columnas codificadas) para que Next.js sepa cómo transformar los datos del usuario.
   * Exportará las métricas (Exactitud, Precisión, Recuerdo, Matriz de Confusión) a `src/data/metrics.json` para facilitar el hot-reload en React.

#### [NEW] `ml/config.json`
Archivo para modificar hiperparámetros (capas, neuronas, activaciones) sin tocar el código Python.

---

### Frontend Web (Next.js)

#### [MODIFY] `src/app/page.js`
La página principal contendrá el **Formulario Predictivo**:
* **Selectores Categóricos:** Dropdowns para "Nivel de Estudios", "Campo de Estudio" y "Región".
* **Inputs Numéricos:** Controles estrictos (0-100) para "Reputación del Empleador", "Demanda de la Habilidad" y "Disponibilidad de Trabajo Remoto".
* **Inferencia en vivo:** Al hacer "Submit", leerá el `scaler_params.json` para normalizar los datos del formulario, cargará el modelo vía TensorFlow.js y predecirá la categoría salarial (Bajo, Medio, Alto).

#### [MODIFY] `src/app/resultados/page.js`
La página de comparación académica:
* Importará directamente `src/data/metrics.json`.
* Renderizará una tabla comparativa con las métricas de rendimiento de cada arquitectura.
* Incluirá la Matriz de Confusión visual para analizar falsos positivos y negativos en cada clase salarial.

## Verification Plan

### Automated Tests
* Ejecutar `python ml/train.py` para asegurar que procesa el CSV, genera el modelo, y guarda los JSON correctamente.

### Manual Verification
* Lanzar el servidor de desarrollo (`npm run dev`).
* Visitar la página principal, llenar el formulario con valores del CSV y verificar que la inferencia no arroje errores matemáticos (NaN).
* Visitar `/resultados` para comprobar que la tabla refleja correctamente las métricas y se actualiza al reentrenar.
