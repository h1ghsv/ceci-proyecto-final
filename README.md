# Ceci · Proyecto Final

Proyecto final de clasificación multiclase sobre un dataset sintético compatible con la estructura de exportación de **Ceci** para operativos territoriales de Formosa.

## Objetivo

Estimar la categoría del **tono observado** durante la interacción a partir de respuestas de encuesta, características del trámite y variables territoriales/operativas.

La variable objetivo es `tono_categoria`, derivada de `tono_nivel`:

| tono_nivel | Clase |
|---|---|
| 1-2 | Amable |
| 3-4 | Neutral |
| 5-6 | Resignado |
| 7-8 | Quejoso |
| 9-10 | Hostil |

`tono_nivel` y `tono_codigo` se excluyen de las variables predictoras para evitar **data leakage**.

## Dataset

Archivo utilizado:

`data/consolidado/ceci_formosa.csv`

Contiene:

- 4.180 registros
- 34 columnas
- 142 operativos
- 37 localidades
- 9 departamentos de Formosa
- 2 relevadores

Los datos son **sintéticos**. Los resultados no deben interpretarse como una estimación de la población real de Formosa.

## Modelado

Se comparan:

- Baseline de clase mayoritaria
- Regresión Logística
- Random Forest
- XGBoost

La división train/test se realiza mediante `GroupShuffleSplit` usando `operativo_id`. De esta manera, un mismo operativo no aparece simultáneamente en entrenamiento y prueba.

XGBoost se optimiza con:

- `GridSearchCV`
- `GroupKFold`
- métrica `f1_macro`

## Resultados de referencia

En la ejecución validada:

| Modelo | Macro F1 | Balanced Accuracy | Accuracy |
|---|---:|---:|---:|
| Baseline | 0,115 | 0,200 | 0,401 |
| Regresión logística | 0,520 | 0,559 | 0,493 |
| Random Forest | ~0,546 | ~0,579 | ~0,522 |
| **XGBoost optimizado** | **0,563** | **0,596** | **0,528** |

Mejor validación cruzada de XGBoost:

`CV Macro F1 ≈ 0,566`

Mejores hiperparámetros encontrados:

```text
learning_rate = 0.05
max_depth = 2
min_child_weight = 3
n_estimators = 120
subsample = 0.8
colsample_bytree = 0.8
```

Pequeñas variaciones numéricas pueden aparecer entre versiones de las librerías.

## Estructura

```text
ceci-proyecto-final/
├── README.md
├── requirements.txt
├── notebooks/
│   └── proyecto_final_ceci.ipynb
├── data/
│   ├── consolidado/
│   │   └── ceci_formosa.csv
│   └── geo/
│       └── departamentos_formosa.geojson   # se descarga en la primera ejecución
├── outputs/
│   ├── graficos/
│   ├── dashboard/
│   └── resultados_modelos.csv
├── models/
│   ├── xgb_tono_pipeline.joblib
│   └── label_encoder_tono.joblib
├── src/
│   └── verificar_dataset.py
└── docs/
    └── Final_DEFINITIVO.docx
```

## Instalación

Se recomienda Python 3.11 o superior.

Crear un entorno virtual es opcional pero recomendado:

```bash
python -m venv .venv
```

En Windows:

```bash
.venv\Scripts\activate
```

Instalar dependencias:

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

## Ejecución

Desde la raíz del repositorio:

```bash
jupyter lab
```

Abrir:

`notebooks/proyecto_final_ceci.ipynb`

y ejecutar **Restart Kernel + Run All**.

También se puede verificar el dataset antes de abrir el notebook:

```bash
python src/verificar_dataset.py
```

## Mapa territorial

El notebook utiliza geometrías de los departamentos de Formosa obtenidas desde el servicio **Georef**.

En la primera ejecución, si `data/geo/departamentos_formosa.geojson` todavía no existe, el notebook intenta descargarlo automáticamente.

Por esta razón, la primera ejecución del mapa requiere acceso a Internet. Si no hay conexión, el modelado continúa normalmente y se informa que el mapa quedó pendiente.

## Archivos generados

El notebook genera:

- `outputs/graficos/estado_respuesta.png`
- `outputs/graficos/target_distribution.png`
- `outputs/graficos/encuestas_departamento.png`
- `outputs/graficos/model_comparison.png`
- `outputs/graficos/confusion_matrix.png`
- `outputs/graficos/shap_importance.png`
- `outputs/graficos/shap_importance.csv`
- `outputs/dashboard/resumen_territorial_formosa.csv`
- `outputs/dashboard/mapa_territorial_formosa.html` cuando hay acceso al GeoJSON
- `outputs/resultados_modelos.csv`
- `models/xgb_tono_pipeline.joblib`
- `models/label_encoder_tono.joblib`

## Interpretación responsable

El porcentaje de **tono crítico** corresponde a la proporción de registros clasificados como `Quejoso` o `Hostil` dentro de cada departamento del dataset sintético.

No representa automáticamente a la población real del departamento.

SHAP se utiliza para explicar cómo usa las variables el modelo, pero no demuestra relaciones causales.

## Reproducibilidad

El proyecto utiliza `RANDOM_STATE = 42`, separación por operativo y controles automáticos de:

- dimensiones del dataset
- IDs únicos
- rangos válidos
- separación entre operativos de train/test
- métricas finales dentro de una tolerancia pequeña

El objetivo es que el notebook pueda ejecutarse de principio a fin con **Run All**.
