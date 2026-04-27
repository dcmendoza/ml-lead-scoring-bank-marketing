# ML Lead Scoring - Bank Marketing

Repositorio sugerido para la Entrega 1 del proyecto de Machine Learning.

## Problema

Estimar la probabilidad de que un cliente acepte una **oferta de depósito a término** en una campaña de telemarketing bancario. La salida del modelo se usa como score para priorizar llamadas.

## Dataset

- Fuente: UCI Machine Learning Repository - Bank Marketing.
- Archivo usado: `bank-additional-full.csv`.
- Tamaño observado: **41,188 filas** y **21 columnas**.
- Target: `y` (`yes` si aceptó, `no` si no aceptó).
- Prevalencia positiva observada: **11.27%**.

## Métrica principal propuesta

**Average Precision / PR-AUC**, porque el problema está desbalanceado y el objetivo práctico es priorizar clientes con mayor probabilidad de aceptar, no solo maximizar accuracy.

## Baseline de Entrega 1

1. `DummyClassifier(strategy="prior")`: baseline mínimo por prevalencia.
2. Regresión logística con pipeline de preprocesamiento: baseline simple e interpretable.

## Estructura

```text
ml-lead-scoring-bank-marketing/
├── README.md
├── requirements.txt
├── data/
│   ├── raw/bank+marketing.zip
│   └── README.md
├── notebooks/Entrega_1_Bank_Marketing.ipynb
├── figures/
├── report/
│   ├── data_card_bank_marketing.md
├── src/README.md
```

## Reproducir

```bash
pip install -r requirements.txt
```

Luego abrir `notebooks/Entrega_1_Bank_Marketing.ipynb` en Google Colab o Jupyter y ejecutar las celdas.

## Repositorio

- [ml-lead-scoring-bank-marketing](https://github.com/dcmendoza/ml-lead-scoring-bank-marketing)

## Respuestas

- ¿Qué problema intenta resolver?

Respuesta: Estimar la probabilidad de que un cliente acepte una oferta de depósito a término en una campaña de telemarketing bancario, para priorizar llamadas.

- ¿Por qué este conjunto de datos es adecuado?

Respuesta: El dataset Bank Marketing de UCI es un benchmark clásico para clasificación binaria con variables sociodemográficas, de contacto y macroeconómicas, lo que lo hace ideal para practicar técnicas de ML aplicadas a problemas reales de marketing.

- ¿Qué métrica es razonable y por qué?

Respuesta: Average Precision (PR-AUC) es razonable porque el problema está desbalanceado (11.27% positivos) y el objetivo es priorizar clientes con mayor probabilidad de aceptar, no solo maximizar accuracy.

- ¿Cuál es el baseline y qué tan difícil parece el problema?

Respuesta: El baseline mínimo es un `DummyClassifier` que predice siempre la clase mayoritaria (no acepta). Un baseline más informativo es una regresión logística simple con preprocesamiento. Dado el desbalance y la complejidad de las variables, el problema parece desafiante pero abordable con técnicas estándar de clasificación.
