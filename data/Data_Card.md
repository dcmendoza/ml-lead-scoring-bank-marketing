# Data card - Bank Marketing

## Fuente

- Dataset: Bank Marketing, UCI Machine Learning Repository.
- Archivo usado: `bank-additional-full.csv`.
- Licencia reportada por UCI: CC BY 4.0.

## Propósito

Construir un modelo de clasificación binaria para estimar si un cliente aceptará una oferta de depósito a término.

## Tamaño

- Filas: 41,188
- Columnas: 21
- Target: `y` (`yes` / `no`)
- Proporción positiva: 11.27%

## Variables

- Cliente: `age`, `job`, `marital`, `education`, `default`, `housing`, `loan`.
- Contacto/campaña: `contact`, `month`, `day_of_week`, `campaign`, `pdays`, `previous`, `poutcome`.
- Macroeconómicas: `emp.var.rate`, `cons.price.idx`, `cons.conf.idx`, `euribor3m`, `nr.employed`.
- Target: `y`.
- Riesgo de leakage: `duration`.

## Riesgos

1. `duration` no debe usarse para scoring pre-contacto.
2. La clase positiva es minoritaria, por lo que accuracy no basta.
3. Algunas categorías tienen valor `unknown`.
4. Variables sociodemográficas pueden requerir análisis de sesgo.
