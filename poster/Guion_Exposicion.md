# Guión de exposición — Bank Marketing Lead Scoring

## Aprendizaje de Máquina Aplicado · EAFIT · 10 minutos · 2 expositores

## BLOQUE 1 — Notebook Entrega 1 · 2 minutos · 🟦 A

> **Mostrar en pantalla:** notebook Entrega 1, ir bajando por las secciones que se citan.

**[0:00–0:30] Problema** *(sección 1, celda 1)*

"Buenos días. Nuestro proyecto parte de un banco portugués que hace campañas de telemarketing para vender depósitos a término. El problema de negocio es simple: llamar a todos los clientes es caro e ineficiente, porque **solo el 11% suscribe**. La pregunta central que guía todo el proyecto es: *¿podemos predecir si un cliente va a suscribir ANTES de llamarlo?* Es una clasificación binaria, y la salida del modelo se usa como score para priorizar a quién llamar primero."

**[0:30–1:00] Métrica y desbalance** *(sección 2 + sección 5, celda 14)*

"Lo primero que define el proyecto es el desbalance: **88.7% no suscribe, 11.3% sí** [celda de balance del target]. Esto descarta usar accuracy, porque un modelo que diga 'no' a todos acierta el 89% sin aprender nada. Por eso elegimos como métrica principal el **Average Precision**, que se concentra en qué tan bien rankeamos la clase positiva, que es la que nos interesa."

**[1:00–1:30] EDA y leakage** *(sección 8, celda 23 + sección 9)*

"En el EDA tomamos dos decisiones clave. Primera: **excluimos la variable `duration`** —la duración de la llamada— porque no se conoce antes de llamar; usarla sería leakage, haría trampa. Segunda: identificamos que las **variables macroeconómicas** son las más correlacionadas con el target, más que el perfil del cliente. Eso será un hilo conductor de todo el proyecto."

**[1:30–2:00] Baseline** *(sección 10, celdas 36–37)*

"Cerramos la Entrega 1 con dos baselines [celda de comparación]. Un Dummy que predice la clase mayoritaria da **ROC-AUC 0.50** —el piso absoluto— y una regresión logística simple ya alcanza **0.795**. Esto confirma dos cosas: hay señal predictiva real en los datos, y tenemos una vara contra la cual medir todo lo que venga después. Con eso, le paso a [B]."

> **Transición:** A entrega a B mientras se cambia al notebook de Entrega 2.

---

## BLOQUE 2 — Notebook Entrega 2 · 2 minutos · 🟧 B

> **Mostrar en pantalla:** notebook Entrega 2.

**[2:00–2:30] Qué se comparó y cómo** *(sección 3 + sección 4, celda 8)*

"Gracias. En la Entrega 2 comparamos cuatro familias de modelos: regresión logística, árbol de decisión, random forest y SVM. Todo bajo el mismo protocolo: **validación cruzada estratificada de 5 folds**, con el preprocesamiento dentro de un pipeline que se ajusta solo con train en cada fold —así no hay leakage—. Reportamos media y desviación estándar, no un solo número, para distinguir señal de ruido."

**[2:30–3:10] Resultados de la comparación** *(sección 10, celda 31)*

"Los resultados en validación cruzada [mostrar tabla de comparación]: Random Forest con Optuna lidera con **AP 0.459**, seguido muy de cerca por Random Forest normal (0.458) y la regresión logística (0.445). El SVM quedó último con 0.351, además de ser el más lento. El hallazgo honesto aquí es que **las diferencias entre los mejores son pequeñas** —caben dentro de una desviación estándar—, lo que ya nos anticipa que el problema tiene un techo de señal."

**[3:10–3:40] Threshold y manejo del desbalance** *(sección 8 SVM + función de evaluación, celda 8)*

"Un punto técnico importante: con datos desbalanceados, el threshold por defecto de 0.5 es malo. Por ejemplo, el SVM al threshold 0.5 daba un F1 de apenas 0.04 [celda 25], pero ajustando el threshold con out-of-fold predictions subía a 0.49. Por eso seleccionamos el threshold sobre train con OOF, **sin tocar validation ni test**."

**[3:40–4:00] Decisión provisional** *(sección 13, celda 41)*

"Cerramos la Entrega 2 con una decisión provisional: Random Forest con Optuna como candidato más prometedor, pero dejando claro que estaba **empatado estadísticamente con la regresión logística**. Eso nos dejó la pregunta abierta para la entrega final: ¿vale la pena un modelo más complejo? Le paso a [A]."

> **Transición:** B continúa (mantiene la voz en el modelado) y cambia al notebook de Entrega Final.

---

## BLOQUE 3 — Notebook Entrega 3 · 2 minutos · 🟧 B

> **Mostrar en pantalla:** notebook Entrega Final.

**[4:00–4:30] Boosting y selección final** *(sección 4 + sección 5, celda 17)*

"Sigo yo con la entrega final, donde agregamos la familia que faltaba: **boosting**, con XGBoost y LightGBM. La tabla final de candidatos [celda 17]: **LightGBM lidera en AP con 0.463**, por encima de Random Forest y la baseline. La ventaja sigue siendo modesta —dentro de una desviación estándar— pero LightGBM lidera de forma consistente y, al ser modelo de árboles, nos da interpretabilidad. Por eso lo elegimos como modelo final."

**[4:30–5:00] Función de costo de negocio** *(sección 6, celda 20)*

"Acá hicimos algo distinto: en vez de elegir el threshold para maximizar una métrica abstracta, lo elegimos para **minimizar el costo real del negocio**. Asumimos que perder un cliente —un falso negativo— cuesta 12 veces más que una llamada perdida. Con ese criterio, el threshold óptimo bajó a **0.41** [celda 20]."

**[5:00–5:40] Evaluación final en test** *(sección 7, celda 26 + sección 8.3, celda 37)*

"Y aquí viene el momento clave: tocamos el **test set por primera y única vez**. Los resultados [celda 26]: **ROC-AUC 0.816, AP 0.495**. Traducido a negocio [celda 37]: el modelo captura el **70.5% de los suscriptores reales llamando solo al 24.3% de la base**. Frente a llamar a todos, eso ahorra un **36.6%** del costo de campaña."

**[5:40–6:00] Confiabilidad** *(sección 8.2, celda 34 + sección 9.2, celda 45)*

"Para no quedarnos con un solo número, calculamos un intervalo de confianza por bootstrap: el AP en test está entre **0.46 y 0.53** con 95% de confianza. Y el modelo está razonablemente calibrado, con un Brier de 0.149. Con esto cerramos los notebooks; [A] arranca la síntesis en las diapositivas."

> **Transición:** se cambia a las diapositivas en modo presentación (botón ⤢). Arranca A.

---

## BLOQUE 4 — Diapositivas · 4 minutos · 🟦🟧 AMBOS

> **Mostrar en pantalla:** deck HTML en modo presentación (botón ⤢).

**[6:00–6:30] 🟦 A — Slides 1 y 7 (resultado headline)**
*(Saltar directo al impacto; las slides 1–6 ya se cubrieron en los notebooks)*

"Para sintetizar lo que acaban de ver. *(Slide 7, la oscura con los 4 números)* Este es el resultado que importa: con LightGBM y el threshold de negocio, **llamando a uno de cada cuatro clientes capturamos a siete de cada diez** que sí habrían suscrito. ROC-AUC de 0.816 en test, y un ahorro del 36.6% frente a la estrategia de llamar a todos."

**[6:30–7:15] 🟧 B — Slide 8 (análisis de errores)**

"*(Slide 8)* ¿Dónde acierta y dónde falla el modelo? De los 928 suscriptores reales en test, detecta 654 y pierde 274. Genera 1.345 llamadas que no convierten. Esto es **intencional**: como un falso negativo cuesta 12 veces más que una llamada, preferimos errar por exceso de llamadas antes que perder un cliente. Y los falsos negativos se concentran en clientes sin historial previo, donde el modelo tiene menos información."

**[7:15–8:00] 🟦 A — Slide 4 (por qué la macro manda)**

"*(Slide 4 o slide de importancia)* Un hallazgo que nos pareció el más interesante del proyecto: la variable más importante con diferencia es **`nr.employed`, el nivel de empleo** —un indicador macroeconómico—, no la edad ni el trabajo del cliente. Esto significa que el modelo predice más *cuándo* es buen momento para la campaña que *a quién* llamar. Para el banco, eso sugiere intensificar campañas en ventanas económicas favorables."

**[8:00–8:45] 🟧 B — Slide 9 (conclusiones y límites)**

"*(Slide 9)* En conclusión: técnicamente, LightGBM lidera pero apenas supera a un baseline lineal —el problema tiene un techo de señal—. En negocio, reducimos la base de contacto a un cuarto manteniendo el 70% de conversiones. Y somos honestos con los límites: el SVM se entrenó sobre una submuestra, los datos son de 2008–2010 así que la macro puede no generalizar a hoy, y no analizamos fairness, que sería un siguiente paso obligado antes de desplegar."

**[8:45–9:30] 🟦 A — Cierre y cómo se desplegaría** *(Slide 10)*

"*(Slide 10)* Para cerrar: lo que entregamos no es solo un modelo, es un **sistema de scoring reproducible y sin leakage** que el banco podría usar para priorizar llamadas. Para llevarlo a producción haría falta un pipeline que actualice las variables macroeconómicas en tiempo real y un análisis de sesgo. Pero la base metodológica —validación honesta, decisión basada en costos, evaluación única en test— ya está construida."

**[9:30–10:00] 🟧 B — Pregunta-respuesta / remate**

"Gracias. En resumen, el proyecto responde su pregunta original: sí se puede predecir quién suscribirá antes de llamar, con suficiente precisión para ahorrar más de un tercio del costo de campaña. Quedamos atentos a sus preguntas."

---

## Notas de ensayo

- **Ritmo:** 10 minutos es ajustado. Practica con cronómetro; si te pasas, el bloque más comprimible es el 2 (Entrega 2) — puedes resumir el punto del threshold.
- **Quién muestra qué:** la persona que habla debe controlar la pantalla, o coordinen que uno conduce el cursor y otro habla.
- **Transiciones:** las tres entregas se cuentan sobre los notebooks; las slides son SOLO síntesis (no repitas, destaca). Por eso en el bloque 4 saltamos directo a slides 7, 8, 4, 9, 10.
- **Si preguntan "¿por qué LightGBM y no la logística si empatan?":** respuesta corta — "LightGBM lidera en AP de forma consistente y nos da interpretabilidad de variables; reconocemos que la ventaja es modesta, por eso lo documentamos como empate técnico, no como victoria clara".
- **Si preguntan por el ahorro de solo 1.2% vs threshold 0.5:** "El threshold de negocio quedó cerca de 0.5 para este ratio de costos; el ahorro real grande es frente a llamar a todos, que es 36.6%".
