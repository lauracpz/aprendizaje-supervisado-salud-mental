# Análisis del problema de clasificación de depresión

## 1. Objetivo y alcance

El objetivo académico es estimar la probabilidad de que una observación tenga la etiqueta `Depression` y clasificarla como 0 o 1 mediante regresión logística. El análisis explora asociaciones entre variables personales, de salud mental y de actividad laboral o académica.

La etiqueta procede del conjunto de datos de Kaggle; no equivale a un diagnóstico clínico. Los coeficientes y las predicciones describen patrones del conjunto analizado y no demuestran que una variable cause depresión.

## 2. Datos y variables

El notebook lee `train.csv` del conjunto [Exploring Mental Health Data, Kaggle Playground Series S4E11](https://www.kaggle.com/competitions/playground-series-s4e11). La variable objetivo es `Depression`. El archivo no se incluye en este repositorio y debe colocarse junto al notebook.

| Variable | Significado y codificación |
| --- | --- |
| `Gender` | Género; Female = 0, Male = 1. |
| `Age` | Edad en años. |
| `Working Professional or Student` | Situación laboral o académica; Student = 0, Working Professional = 1. |
| `Have you ever had suicidal thoughts ?` | Pensamientos suicidas declarados; No = 0, Yes = 1. |
| `Work/Study Hours` | Horas diarias de trabajo o estudio. |
| `Financial Stress` | Nivel numérico de estrés financiero. |
| `Family History of Mental Illness` | Antecedentes familiares; No = 0, Yes = 1. |
| `Sleep Duration` | Duración del sueño aproximada en horas. |
| `Work/Study Pressure` | Presión académica o laboral combinada. |
| `Work/Study Satisfaction` | Satisfacción académica o laboral combinada. |

El notebook descarta campos que no utiliza en los modelos, como nombre, ciudad, titulación, profesión, CGPA, identificador y hábitos alimentarios.

## 3. Preparación y selección de observaciones

El notebook aplica estas operaciones:

- Combina `Academic Pressure` con `Work Pressure`, y `Study Satisfaction` con `Job Satisfaction`, tomando en cada caso el primer valor no ausente.
- Codifica género, situación laboral o académica, pensamientos suicidas e historial familiar con valores 0 y 1.
- Convierte la duración del sueño: un rango `N-M hours` se aproxima por `N`; `more than N hours`, por `N + 1`; y `less than N hours`, por `N - 1`.
- Las categorías de duración del sueño que la función no reconoce se convierten en ausentes. Después elimina las filas incompletas.
- Conserva observaciones con duración del sueño de hasta 10 horas, trabajo o estudio de hasta 12 horas y edad de hasta 60 años.

La conversión de los rangos de sueño es una aproximación, no una medición exacta. Las sustituciones categóricas usan las etiquetas previstas en el CSV; el notebook no normaliza de forma general mayúsculas, espacios ni categorías desconocidas. Si el archivo contiene categorías distintas, podrían permanecer sin codificar y provocar errores en el ajuste.

Tras la limpieza y los filtros, la ejecución guardada en el notebook contiene 140.645 observaciones. Las cifras de las secciones siguientes corresponden a esa ejecución; pueden variar con otro archivo o con cambios en el procedimiento.

## 4. Modelos y protocolo del notebook

La regresión logística modela la probabilidad de la clase positiva mediante la función logística:

~~~text
P(Depression = 1 | X) = 1 / (1 + exp(-(beta_0 + beta_1*x_1 + ... + beta_k*x_k)))
~~~

Cada coeficiente expresa el cambio en los log-odds estimados al variar un predictor, manteniendo los demás constantes.

El notebook contiene varios ejercicios con muestras distintas:

- **Demostración con tres variables:** pensamientos suicidas, duración del sueño y estrés financiero. El modelo se ajusta con las primeras 20 filas limpias; esta muestra solo ilustra el código y no permite evaluar el rendimiento de forma fiable.
- **Modelo de diez variables:** se ajusta con las primeras 100.000 filas limpias. Sus diez predictores son las variables de la tabla anterior.
- **Predicciones y evaluación por umbral:** las filas limpias posteriores a la posición 100.000 —40.645 en la ejecución guardada— se usan para calcular probabilidades con el modelo de diez variables. El notebook las convierte en clases con umbrales de 0,5 y 0,2 y muestra matrices de confusión y accuracy.
- **Resumen estadístico de `statsmodels`:** se ajusta una regresión logística distinta sobre las 140.645 filas limpias. Este resumen sirve para examinar coeficientes e incertidumbre, pero no es una evaluación independiente: incluye también las filas usadas para la evaluación por umbral.

La separación del modelo de diez variables se basa en el orden de las filas (`head(100000)` frente al resto): equivale aproximadamente a 71,1 % para entrenamiento y 28,9 % para evaluación en la ejecución guardada. No es una partición aleatoria ni estratificada. Por eso, estas cifras no deben describirse como una partición aleatoria 70/30. El modelo de tres variables y el de diez variables tampoco forman una comparación controlada: se ajustan con tamaños de muestra muy diferentes.

## 5. Resultados de la ejecución guardada

### 5.1 Resumen estadístico y coeficientes

El ajuste de `statsmodels` convergió en 9 iteraciones con 140.645 observaciones y 10 predictores. El resumen informa un pseudo-R² de McFadden de 0,6610, log-verosimilitud de −22.594 y una prueba de razón de verosimilitudes con `p < 0,001`.

| Variable | Coeficiente | Error estándar | z | Valor p | IC del 95 % |
| --- | ---: | ---: | ---: | ---: | ---: |
| Constante | 0,2327 | 0,082 | 2,855 | 0,004 | [0,073; 0,392] |
| Gender | 0,0591 | 0,025 | 2,408 | 0,016 | [0,011; 0,107] |
| Age | -0,1683 | 0,002 | -106,382 | < 0,001 | [-0,171; -0,165] |
| Working Professional or Student | -1,9038 | 0,028 | -68,033 | < 0,001 | [-1,959; -1,849] |
| Sleep Duration | -0,1334 | 0,006 | -20,523 | < 0,001 | [-0,146; -0,121] |
| Have you ever had suicidal thoughts ? | 2,4815 | 0,028 | 87,156 | < 0,001 | [2,426; 2,537] |
| Work/Study Hours | 0,1329 | 0,003 | 39,992 | < 0,001 | [0,126; 0,139] |
| Financial Stress | 0,5562 | 0,009 | 61,149 | < 0,001 | [0,538; 0,574] |
| Family History of Mental Illness | 0,2069 | 0,024 | 8,463 | < 0,001 | [0,159; 0,255] |
| Work/Study Pressure | 0,7343 | 0,010 | 74,513 | < 0,001 | [0,715; 0,754] |
| Work/Study Satisfaction | -0,4017 | 0,009 | -44,438 | < 0,001 | [-0,419; -0,384] |

En ese ajuste, todos los predictores tienen `p < 0,05`; para `Gender`, `p = 0,016`. Los coeficientes positivos se asocian con mayores log-odds estimados y los negativos, con menores log-odds, manteniendo los demás predictores constantes. Por ejemplo, la categoría `Working Professional` (1) tiene un coeficiente negativo respecto a `Student` (0), mientras que los pensamientos suicidas declarados tienen un coeficiente positivo. La significación estadística no mide importancia clínica y puede aumentar con tamaños de muestra grandes.

Estos coeficientes son del resumen de `statsmodels`, con predictores en sus escalas originales. No deben compararse directamente con coeficientes de una versión estandarizada del modelo.

### 5.2 Efecto del umbral

La matriz de confusión del notebook está normalizada por la clase real. En la tabla, el acierto de la clase 0 corresponde a la tasa de verdaderos negativos y el acierto de la clase 1, a la tasa de verdaderos positivos o recall de esa clase.

| Umbral | Acierto clase 0 (TN) | Falsos positivos | Falsos negativos | Acierto clase 1 (TP/recall) | Accuracy | Balanced accuracy* |
| ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 0,5 | 96,75 % | 3,25 % | 20,34 % | 79,66 % | 93,59 % | 88,21 % |
| 0,2 | 91,70 % | 8,30 % | 8,18 % | 91,82 % | 91,72 % | 91,76 % |

Al bajar el umbral de 0,5 a 0,2, el recall de la clase 1 sube 12,16 puntos porcentuales y el acierto de la clase 0 baja 5,05 puntos. La accuracy baja 1,87 puntos. La balanced accuracy, calculada como promedio de las tasas de acierto de ambas clases, sube de 88,21 % a 91,76 %. La elección del umbral depende del coste de falsos negativos frente al de falsos positivos.

\* La balanced accuracy de esta tabla está calculada a partir de las dos tasas por clase de la matriz; el notebook no la imprime como métrica separada.

### 5.3 Qué expresan las métricas

- **Accuracy:** proporción total de predicciones correctas; puede ocultar un rendimiento bajo en la clase minoritaria.
- **Balanced accuracy:** promedio de la tasa de acierto de cada clase.
- **Precisión:** proporción de predicciones positivas que fueron correctas.
- **Recall:** proporción de casos positivos reales detectados por el modelo.
- **F1:** media armónica de precisión y recall.
- **ROC AUC:** capacidad de ordenar casos positivos por encima de negativos a través de distintos umbrales; se calcula con probabilidades, no con clases ya umbralizadas.

El notebook muestra accuracy y matrices de confusión para los dos umbrales. No calcula precisión, F1 ni ROC AUC en esas celdas.

## 6. Contexto de la implementación modular anterior

El otro repositorio documentaba una implementación modular distinta. Se incluye aquí como contexto metodológico, no como protocolo ni resultado del notebook:

- Comparaba el modelo base de tres variables y el modelo completo de diez variables sobre la misma partición.
- Reservaba 20 % de los datos para validación y usaba una partición estratificada con semilla 42.
- Ajustaba el escalado dentro de una tubería y lo aprendía solo con los datos de entrenamiento.
- Usaba umbral 0,5 y calculaba accuracy, balanced accuracy, precisión, recall, F1, ROC AUC y matrices de confusión.
- Guardaba las métricas, los coeficientes y las matrices en `outputs/`.

Ese protocolo no es intercambiable con el notebook: los tamaños de muestra, la separación de los datos y el escalado son distintos. Las cifras de coeficientes y de umbrales de las secciones anteriores corresponden a la ejecución del notebook, no a resultados generados por esa implementación modular.

## 7. Limitaciones y uso responsable

- La demostración con tres variables usa solo 20 observaciones.
- El modelo de diez variables separa por posición de fila; el resultado puede depender del orden del CSV y no conserva explícitamente la proporción de clases.
- Las filas de evaluación pertenecen al mismo archivo de entrenamiento de Kaggle; no constituyen validación externa.
- El resumen de `statsmodels` se ajusta con todos los datos limpios, incluidas las filas de evaluación, por lo que sus valores p y coeficientes no deben interpretarse como evaluación fuera de muestra.
- No se realiza validación cruzada, calibración de probabilidades ni selección del umbral con un coste de error previamente definido.
- La regresión logística de `scikit-learn` del notebook se ajusta sin una tubería de estandarización; las escalas de los predictores son distintas y los coeficientes no tienen magnitudes directamente comparables.
- La aproximación de rangos de sueño y los filtros de edad, horas y sueño cambian la población analizada.
- El desbalance entre clases puede hacer que accuracy parezca alta aunque el recall de una clase sea menor.
- La procedencia y representatividad del conjunto limitan la generalización a otras poblaciones.
- Los pensamientos suicidas son una variable sensible y cercana al resultado estudiado. Su capacidad predictiva no la convierte en instrumento de diagnóstico.
- Los datos observacionales permiten describir asociaciones, pero no establecer causalidad.

## 8. Conclusión

El notebook presenta un flujo de clasificación binaria con limpieza de datos, codificación de categorías, regresión logística, análisis estadístico y comparación de dos umbrales. En la ejecución guardada, bajar el umbral aumentó el recall de la clase 1 y la balanced accuracy, a costa de más falsos positivos y menor accuracy. Las asociaciones más marcadas aparecen en pensamientos suicidas, situación laboral o académica, presión laboral o académica y estrés financiero. Son resultados exploratorios de un conjunto y un procedimiento concretos, no conclusiones clínicas o causales.
