# Guía del notebook

El notebook conserva el flujo de trabajo original y está organizado en estas secciones:

1. **Carga y preparación:** importa las bibliotecas, lee `train.csv`, combina variables laborales o académicas, codifica categorías y limpia las filas.
2. **Modelos de demostración:** ajusta una regresión logística con tres variables sobre las primeras 20 filas limpias y muestra predicciones de ejemplo.
3. **Modelo de diez variables:** ajusta otro modelo con hasta las primeras 100.000 filas y calcula probabilidades para las filas posteriores.
4. **Resumen estadístico:** estima una regresión logística con `statsmodels` sobre todos los datos limpios y muestra coeficientes, errores estándar y valores p.
5. **Evaluación y gráficos:** representa probabilidades, convierte las predicciones en clases con umbrales de 0,5 y 0,2 y dibuja las matrices de confusión.

Ejecuta las celdas en orden. Las variables se crean en celdas anteriores y algunas vuelven a utilizarse más adelante. Para conocer el alcance de cada resultado, consulta el [análisis del problema](analisis-del-problema.md).
