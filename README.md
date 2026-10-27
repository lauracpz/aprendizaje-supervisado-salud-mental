# Clasificación de salud mental con aprendizaje supervisado

Proyecto académico que estudia la clasificación de la etiqueta `Depression` mediante regresión logística sobre el conjunto *Exploring Mental Health Data* (Kaggle Playground Series S4E11). El trabajo compara predictores relacionados con salud mental, edad, sueño y actividad laboral o académica, y analiza las probabilidades y los errores de clasificación.

El código del proyecto está en el notebook original [codigo aprendizaje supervisado.ipynb](codigo%20aprendizaje%20supervisado.ipynb). El [análisis del problema](docs/analisis-del-problema.md) explica los datos, el procedimiento y sus limitaciones; la [guía del notebook](docs/guia-del-notebook.md) resume sus secciones.

> **Uso responsable:** este proyecto es académico. Las predicciones no son diagnósticos ni deben usarse para tomar decisiones clínicas o personales. Las asociaciones estimadas no demuestran causalidad.

## Qué hace

- Carga `train.csv` y revisa sus columnas.
- Combina presión y satisfacción académica o laboral en variables comunes.
- Codifica algunas categorías y transforma los rangos de duración del sueño a valores aproximados.
- Elimina filas incompletas y filtra edades, horas de trabajo o estudio y duración del sueño fuera de los límites definidos en el notebook.
- Ajusta una regresión logística de tres variables con las primeras 20 filas limpias y otra de diez variables con hasta las primeras 100.000.
- Ajusta además un modelo de `statsmodels` sobre todos los datos limpios para mostrar coeficientes y valores p.
- Genera probabilidades para las filas posteriores a las primeras 100.000 y compara matrices de confusión con umbrales de 0,5 y 0,2.

El análisis documenta cómo interpretar estos pasos y por qué los resultados deben leerse con cautela.

## Datos

El notebook usa `train.csv` de [Exploring Mental Health Data, Kaggle Playground Series S4E11](https://www.kaggle.com/competitions/playground-series-s4e11). Descarga el archivo desde Kaggle y colócalo en la raíz de este repositorio, junto al notebook. El CSV no se incluye.

## Instalación y ejecución

Se requiere Python 3.10 o posterior. Desde la raíz del repositorio:

~~~powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
py -m pip install -r requirements.txt
jupyter notebook
~~~

Abre `codigo aprendizaje supervisado.ipynb` y ejecuta las celdas en orden. También puedes abrir el notebook en Google Colab; en ese caso, carga `train.csv` en el entorno antes de ejecutar las celdas.

## Estructura

~~~text
clasificacion-salud-mental-notebook/
├── codigo aprendizaje supervisado.ipynb  # notebook original
├── docs/
│   ├── analisis-del-problema.md
│   └── guia-del-notebook.md
├── requirements.txt
├── .gitignore
└── README.md
~~~

## Interpretación y limitaciones

El signo de un coeficiente describe una asociación dentro del modelo y depende de las variables incluidas y de su escala. El notebook no es una herramienta clínica. Su protocolo no usa una partición aleatoria y estratificada: los primeros 100.000 registros se usan para el modelo de diez variables y las filas posteriores para ciertas predicciones. La sección de `statsmodels` ajusta otro modelo sobre todos los datos limpios. El [análisis](docs/analisis-del-problema.md) detalla estas diferencias y otras limitaciones.
