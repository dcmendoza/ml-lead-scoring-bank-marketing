# Reproducibility Checklist — Bank Marketing Lead Scoring

**Proyecto:** ML Lead Scoring — Bank Marketing
**Curso:** Aprendizaje de Máquina Aplicado — EAFIT
**Autores:** Alejandro Osorno y Danna Mendoza
**Fecha:** Mayo 2026
**Repositorio:** https://github.com/dcmendoza/ml-lead-scoring-bank-marketing

Este documento permite reproducir los resultados de las tres entregas de principio
a fin, obteniendo las mismas métricas reportadas. Cada ítem marcable (`[ ]`) es una
condición verificable de reproducibilidad.

---

## 1. Datos

| Ítem | Detalle |
|---|---|
| Fuente | UCI Machine Learning Repository — Bank Marketing |
| Archivo | `bank-additional-full.csv` |
| Ruta esperada | `data/raw/bank-additional-full.csv` |
| Filas × columnas | 41,188 × 21 |
| Variable objetivo | `y` (`yes`/`no`) → codificada 1/0 |
| Prevalencia positiva | 11.27 % |
| Licencia | CC BY 4.0 (reportada por UCI) |

- [ ] El archivo está en `data/raw/` sin modificaciones respecto al original de UCI.
- [ ] No se filtraron ni limpiaron filas manualmente antes del split.
- [ ] El separador de lectura coincide con el archivo (`,` en `bank-additional-full.csv`).

---

## 2. Entorno y dependencias

| Componente | Detalle |
|---|---|
| Lenguaje | Python 3.11+ |
| Núcleo | numpy, pandas, matplotlib, seaborn |
| Modelado | scikit-learn, xgboost, lightgbm |
| Optimización | optuna |
| Desbalance | imbalanced-learn |
| Serialización | joblib |

- [ ] Dependencias listadas en `requirements.txt`.
- [ ] Instalación reproducible: `pip install -r requirements.txt`.
- [ ] Los tres notebooks corren completos en Colab o Jupyter sin intervención manual.

```bash
pip install numpy pandas matplotlib seaborn scikit-learn xgboost lightgbm optuna imbalanced-learn joblib
```

---

## 3. Semilla y determinismo

| Ítem | Valor |
|---|---|
| Semilla global | `SEED = 42` |
| Fijación | `np.random.seed(SEED)` + `random_state=SEED` en todos los estimadores y splits |
| Sampler de Optuna | `TPESampler(seed=SEED)` |
| Validación cruzada | `StratifiedKFold(n_splits=5, shuffle=True, random_state=SEED)` |

- [ ] Todos los componentes con aleatoriedad reciben `random_state=SEED`.
- [ ] Los estudios de Optuna usan sampler con semilla fija.
- [ ] **Nota:** los hiperparámetros hallados por Optuna pueden variar levemente entre
      versiones de librería; los valores finales usados quedan fijados explícitamente
      en el notebook (ver sección 6).

---

## 4. Particiones de datos (idénticas en las 3 entregas)

| Partición | Tamaño | % | Hash de índices | Uso |
|---|---|---|---|---|
| Train | 26,360 | 64 % | `34959e7af23a` | Ajuste y selección vía CV |
| Validation | 6,590 | 16 % | `f3a6afbe3a7d` | Reporte intermedio (E2) |
| Test | 8,238 | 20 % | `fb1445d245c0` | Evaluación final, una sola vez (E3) |

- [ ] Split en dos pasos con `train_test_split`, `stratify=y` en ambos, `random_state=SEED`.
- [ ] Estratificación verificada: las tres particiones conservan ~11.3 % de positivos.
- [ ] **Hash de índices** impreso en cada entrega; debe coincidir con la tabla anterior.
- [ ] El test set NO se tocó durante el diseño (modelo, hiperparámetros, threshold).

---

## 5. Preprocesamiento (sin leakage)

| Tipo | Tratamiento |
|---|---|
| Numéricas | Imputación por mediana + `StandardScaler` |
| Nominales | Imputación por moda + `OneHotEncoder(handle_unknown="ignore")` |
| Ordinal (`education`) | Orden educativo explícito + `OrdinalEncoder` |
| `pdays = 999` | Variable binaria `nunca_contactado` + 999 → NaN |
| `duration` | **Excluida** desde el inicio (target leakage) |
| `"unknown"` | Conservado como categoría válida (no se imputa) |

- [ ] Todo el preprocesamiento está en un `Pipeline` / `ColumnTransformer`.
- [ ] El preprocesador se ajusta SOLO con el fold de train en cada CV.
- [ ] `duration` se elimina antes de cualquier análisis (`LEAKAGE_VARS = ["duration"]`).
- [ ] La transformación de `pdays` usa una regla del diccionario, no estadísticas observadas.
- [ ] SMOTE (cuando se evaluó) va dentro de `imblearn.pipeline.Pipeline` → solo afecta train.

---

## 6. Modelo final y configuración exacta

| Ítem | Valor |
|---|---|
| Familias comparadas | Logística, Árbol, Random Forest, SVM, XGBoost, LightGBM |
| Métrica de selección | Average Precision (AP / PR-AUC) |
| Validación | CV estratificada 5-fold, media ± std |
| Búsqueda | Optuna (TPE), 30 trials por modelo de boosting |
| **Modelo final** | **LightGBM (Optuna)** con `class_weight="balanced"` |

**Hiperparámetros exactos del modelo final (LightGBM):**
```python
LGBMClassifier(
    n_estimators=400,
    max_depth=12,
    learning_rate=0.01351182947645082,
    num_leaves=31,
    subsample=0.6180909155642152,
    colsample_bytree=0.7301321323053057,
    class_weight="balanced",
    random_state=42,
)
```

- [ ] La regla de selección se definió antes de tocar el test.
- [ ] Se documenta que la ventaja sobre la baseline es modesta (< 1 std combinada).
- [ ] La tabla comparativa se guarda en `report/candidatos_e3.csv`.

---

## 7. Threshold y función de costo

| Ítem | Valor |
|---|---|
| Criterio | Minimización del costo de negocio esperado |
| Costo Falso Negativo (FN) | 120 unidades (venta perdida) |
| Costo Falso Positivo (FP) | 10 unidades (costo de llamada) |
| Ratio FN:FP | 12:1 |
| Threshold óptimo | 0.41 |
| Selección | OOF de train (`cross_val_predict`), sin tocar val/test |

- [ ] El threshold se eligió con OOF de train, sin tocar validation ni test.
- [ ] Se incluye análisis de sensibilidad del threshold a distintos ratios de costo.
- [ ] El threshold se fijó antes del reentrenamiento final sobre train+validation.

---

## 8. Resultados a reproducir (test set, E3)

| Métrica | Valor esperado |
|---|---|
| ROC-AUC | 0.816 |
| Average Precision | 0.495 |
| F1 (clase yes) | 0.447 |
| IC 95 % de AP (bootstrap, n=1000) | [0.462, 0.531] |
| Brier score | 0.149 |
| Matriz de confusión | TP=654 · FP=1345 · FN=274 · TN=5965 |
| % clientes llamados | 24.3 % |
| % suscriptores capturados | 70.5 % |
| Ahorro vs. llamar a todos | 36.6 % |

**Variable más importante (permutation importance):** `nr.employed` (0.134), seguida de
`nunca_contactado` (0.036) y `euribor3m` (0.029).

- [ ] La evaluación en test se realiza una sola vez, sin reajustar nada.
- [ ] El modelo se reentrena sobre train+validation antes de evaluar en test.
- [ ] La incertidumbre se cuantifica vía bootstrap (1,000 remuestreos).

---

## 9. Artefactos generados

| Archivo | Contenido |
|---|---|
| `report/modelo_final_e3.joblib` | Pipeline final (preprocesador + LightGBM) |
| `report/resumen_final_e3.csv` | Métricas finales y configuración |
| `report/candidatos_e3.csv` | Tabla comparativa de modelos en CV |
| `report/sensibilidad_costos.csv` | Threshold óptimo por ratio de costo |
| `report/mejor_pipeline_e2.joblib` | Pipeline candidato de Entrega 2 |
| `report/baseline_metrics.csv` | Métricas de baselines de Entrega 1 |
| `figures/*.png` | Todas las figuras del análisis (150 dpi) |

- [ ] El pipeline final se serializa con `joblib`.
- [ ] Las figuras se exportan a `figures/`.
- [ ] Los CSV de resultados quedan en `report/` para trazabilidad.

---

## 10. Orden de ejecución y trazabilidad

- [ ] Cada notebook se ejecuta de arriba a abajo (Cell → Run All), sin saltar celdas.
- [ ] Las tres entregas comparten split idéntico (verificable por hash, sección 4).
- [ ] El repositorio Git tiene historial de commits coherente con cada entrega.
- [ ] Tiempo aproximado de ejecución de la Entrega 3: ~15 min (dominado por Optuna).

---

## Resumen de garantías

| Garantía | Estado |
|---|---|
| Semilla fija en todos los componentes aleatorios | ✅ |
| Split estratificado verificable por hash | ✅ |
| Preprocesamiento encapsulado en pipeline (sin leakage) | ✅ |
| `duration` excluida (target leakage) | ✅ |
| Test set intacto hasta la evaluación final | ✅ |
| Threshold seleccionado sobre OOF de train | ✅ |
| Hiperparámetros del modelo final documentados | ✅ |
| Dependencias y versiones documentadas | ✅ |
| Artefactos y figuras exportados | ✅ |
| Resultados esperados documentados para verificación | ✅ |
