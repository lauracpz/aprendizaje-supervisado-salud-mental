# Análisis del problema de clasificación de depresión

## 1. Objetivo y alcance

El objetivo académico es estimar la probabilidad de que una observación tenga la etiqueta `Depression` y clasificarla como 0 o 1 mediante regresión logística. El notebook explora asociaciones entre variables personales, académicas o laborales y la etiqueta. No constituye una evaluación clínica; los resultados no demuestran causalidad.

## 2. Datos y variables

El notebook lee `train.csv` del conjunto [Exploring Mental Health Data, Kaggle Playground Series S4E11](https://www.kaggle.com/competitions/playground-series-s4e11). La etiqueta objetivo es `Depression`.

| Variable | Significado y codificación |
| --- | --- |
| `Gender` | Género; Female = 0, Male = 1. |
| `Age` | Edad, convertida a valor numérico por el conjunto de datos. |
| `Working Professional or Student` | Situación laboral o académica; Student = 0, Working Professional = 1. |
| `Have you ever had suicidal thoughts ?` | Pensamientos suicidas declarados; No = 0, Yes = 1. |
| `Work/Study Hours` | Horas diarias de trabajo o estudio. |
| `Financial Stress` | Nivel numérico de estrés financiero. |
| `Family History of Mental Illness` | Antecedentes familiares; No = 0, Yes = 1. |
| `Sleep Duration` | Duración del sueño, aproximada a partir del texto. |
| `Work/Study Pressure` | Presión académica o laboral combinada. |
| `Work/Study Satisfaction` | Satisfacción académica o laboral combinada. |

Se eliminan campos que el notebook no usa en el modelo, como nombre, ciudad, titulación, profesión, CGPA y hábitos alimentarios.

## 3. Preparación de los datos

- Se combinan las columnas de presión académica y laboral mediante el primer valor no ausente. Se aplica el mismo criterio a satisfacción académica y laboral.
- Se codifican género, situación laboral o académica, pensamientos suicidas e historial familiar con valores 0 y 1.
- Para la duración del sueño, un rango `N-M hours` se convierte en `N`; `more than N hours` se convierte en `N + 1` y `less than N hours` en `N - 1`.
- Las opciones de sueño que no reconoce la función se convierten en ausentes. Después se eliminan las filas incompletas.
- El notebook conserva observaciones con sueño de hasta 10 horas, trabajo o estudio de hasta 12 horas y edad de hasta 60 años.

Estas reglas forman parte del código original y condicionan qué filas entran en los modelos y en los resultados.

## 4. Modelos y evaluación

El notebook contiene varios ejercicios relacionados, con muestras y propósitos diferentes:

- **Demostración de tres variables:** usa pensamientos suicidas, duración del sueño y estrés financiero, y ajusta el modelo sobre las primeras 20 filas limpias. Por el tamaño reducido, es solo una demostración.
- **Modelo de diez variables:** ajusta la regresión logística sobre las primeras 100.000 filas limpias. Usa las diez variables de la tabla anterior.
- **Predicciones posteriores:** si quedan filas después de la fila 100.000, el notebook usa esas filas para calcular probabilidades con el modelo de diez variables.
- **Resumen de `statsmodels`:** ajusta una regresión logística separada sobre todos los datos limpios para mostrar coeficientes, errores estándar y valores p. Este resumen no es una evaluación independiente de rendimiento.
- **Umbrales:** convierte las probabilidades del modelo de diez variables a clases con umbrales de 0,5 y 0,2, y calcula matrices de confusión y accuracy para las filas posteriores a las primeras 100.000.

La partición se basa en la posición de las filas del DataFrame: el notebook no aleatoriza ni estratifica los registros. Por ello, las cifras no deben interpretarse como una validación independiente representativa. El modelo de tres variables y el de diez variables también se ajustan con tamaños de muestra distintos, así que no son una comparación controlada.

## 5. Lectura de los resultados

### Asociaciones estimadas

El resumen de `statsmodels` del notebook muestra estos coeficientes para el ajuste con diez variables en sus escalas originales:

| Variable | Coeficiente | Dirección de la asociación en ese ajuste |
| --- | ---: | --- |
| Gender | 0,0590 | Positiva para la categoría codificada como 1. |
| Age | -0,1683 | Negativa. |
| Working Professional or Student | -1,9039 | Negativa para Working Professional (1) frente a Student (0). |
| Have you ever had suicidal thoughts ? | 2,4817 | Positiva. |
| Work/Study Hours | 0,1329 | Positiva. |
| Financial Stress | 0,5563 | Positiva. |
| Family History of Mental Illness | 0,2075 | Positiva. |
| Sleep Duration | -0,1302 | Negativa. |
| Work/Study Pressure | 0,7345 | Positiva. |
| Work/Study Satisfaction | -0,4015 | Negativa. |

Un coeficiente positivo indica mayores log-odds estimados al aumentar el predictor o pasar a la categoría 1; uno negativo indica menores log-odds, manteniendo los demás predictores constantes. Estas magnitudes dependen de la escala y del ajuste. Los valores p describen la evidencia estadística dentro de ese modelo, no importancia clínica ni causalidad.

### Umbral de clasificación

El notebook compara 0,5 y 0,2 para convertir probabilidades en clases. Bajar el umbral suele detectar más casos positivos y también puede aumentar los falsos positivos. La matriz de confusión permite ver ambas clases de error; la accuracy sola no muestra el equilibrio entre ellas. La elección de umbral depende del uso previsto y de los costes relativos de esos errores.

## 6. Limitaciones

- El modelo de tres variables usa solo 20 observaciones y no sirve para sacar conclusiones de rendimiento.
- El modelo de diez variables toma las primeras 100.000 filas sin una partición aleatoria y estratificada; el resultado depende del orden del archivo.
- Las predicciones posteriores proceden del mismo archivo de entrenamiento de Kaggle, no de una cohorte externa.
- El resumen de `statsmodels` usa todos los datos limpios y debe leerse como un análisis estadístico exploratorio, no como validación.
- No se realiza validación cruzada, calibración de probabilidades ni selección del umbral con un coste de error explícito.
- El desbalance de clases puede hacer que accuracy oculte un rendimiento bajo para una clase.
- La representatividad y procedencia de los datos limitan la generalización a otras muestras o poblaciones.
- La variable de pensamientos suicidas es sensible y muy cercana al resultado estudiado. Su capacidad predictiva no la convierte en una herramienta de diagnóstico.
- Los datos observacionales pueden mostrar asociaciones sin permitir inferir causalidad.

## 7. Conclusión

El notebook ilustra un flujo de clasificación binaria con limpieza y codificación de variables, regresión logística, probabilidades predichas y análisis de errores para dos umbrales. Las asociaciones más marcadas en el resumen estadístico aparecen en pensamientos suicidas, situación laboral o académica, presión laboral o académica y estrés financiero. Deben interpretarse como resultados exploratorios de este conjunto y este procedimiento, no como relaciones clínicas o causales.
