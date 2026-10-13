# Actividad 5 Ejercicio práctico (Entrenamiento y evaluación de un modelo básico)

Hecho por: Santiago Chacón Serrano

## Introducción

Esta actividad se hizo de nuevo en el laboratorio de AWS, con Amazon SageMaker como pilar: el entorno donde se crean, entrenan y despliegan modelos de Machine Learning.El trabajo se realizo en la rama dev_schacon46, con mínimo tres commits por laboratorio y una subcarpeta por cada etapa (Esto no se solicitaba pero lo hice yo para organizarlo mucho mejor).

Los dos laboratorios trabajan con el conjunto de datos de la columna vertebral de la actividad anterior, donde la clase `1` es **Anormal** y la clase `0` es **Normal**.

## Laboratorio 3.4 - Amazon SageMaker - Formación de un modelo laboratorio 

En este laboratorio se prepara y entrena el modelo que se usara en el laboratorio 3.5. Lo dividí en tres etapas, cada una con su carpeta y su commit.

### Explorar los datos

Cargué el dataset y revisé su tamaño, sus columnas y cómo se reparten las dos clases. Esto es importante antes de entrenar, porque un modelo solo aprende lo que los datos le muestran, y si las clases estuvieran muy desbalanceadas habría que tenerlo en cuenta. Aquí tuve el mismo problema de la actividad pasada, el código usaba librerías que no estaban instaladas (scipy y scikit-learn), así que las instalé con !pip install.

### Preparar los datos

Separé los datos en conjuntos de entrenamiento, validación y prueba, dejé la clase como primera columna y subí los archivos CSV a S3. Es necesario hacerlo de esta forma debido a que XGBoost en SageMaker lee sus datos desde S3 en ese formato, y tener un conjunto de prueba aparte permite medir el modelo con datos que nunca vio.

### Entrenar el modelo

Configuré el estimador de XGBoost con sus parámetros y lo entrené con fit sobre los datos de entrenamiento y validación. Esto permita al modelo ser capaz de decir si una columna es normal o anormal (Con datos de 0 a 1), y es el que se reutiliza en el siguiente laboratorio.

## Laboratorio 3.5 - Amazon SageMaker - Despliegue, predicciones y transformación por lotes

Aquí se usa el modelo entrenado en el anterior laboratorio para hacer predicciones. Igual que antes lo dividí en tres etapas, cada una con su carpeta y commit.

### Importar datos

Evidentemente aqui se rehizo lo de el laboratorio pasado, entonces en vez de explicar lo mismo de antes, mencionare un problema en general de este laboratorio, debido a que por temas de tiempo tuve que cerrar y abrir el laboratorio varias veces, creia que con desgargar y subir el notebook ANTES de cerrar el trabajo para asi trabajar desde el nuevo laboratorio serviria, sin embargo no sirvio por temas del Bucket, el cual cambiaba de numero a cada rato, tocaria buscar el numero de bucket (cosa que aws no permite) y por ello decidi simplemente re hacer el trabajo cuando ocurria (Cosa que no era grave mas vale la pena mencionar). 

### Endpoint y predicciones

Desplegué el modelo en un endpoint, que es un servicio que está siempre disponible para responder predicciones al instante. El endpoint no responde simplemente con normal o anormal, sino una probabilidad de que la columna sea anormal (0 = Normal y 1 = Anormal). Para convertirla en una clase se usa un umbral, que en el laboratorio es 0,65: si la probabilidad lo supera, se clasifica como anormal.

En la primera fila del conjunto de prueba el modelo dio una probabilidad de 0,9966, o sea clase 1 o mejor dicho, anormal, y la predicción fue correcta. En el reto de comparar cuatro predicciones con sus clases reales acertó 3 de 4. El fallo fue una fila cuya probabilidad dio errada mas se acerco bastante al umbral puesto por el laboratorio.

#### Validación del endpoint (Lo que se solicita en la actividad)

Para comprobar que el endpoint funcionaba, le envié una entrada nueva:

```python
entrada = "63.0,12.5,45.0,50.5,125.0,3.0"
resultado = xgb_predictor.predict(entrada)
print(resultado)   # b'0.01694011501967907'
```

El modelo estima una probabilidad de aproximadamente 1,7 % (0.169) de que esa columna sea anormal. Como está muy por debajo del umbral de 0,65 (y de cualquier umbral razonable), se clasifica como Normal (clase 0). Esto confirma que el endpoint quedó bien desplegado y responde de forma coherente.

Despues de esto, se desactivo, apago y borro el endpoint para que no consuma datos (No tengo pruebas, pero creo que en el semestre pasado deje algo asi abierto, pues incluso fuera de aws algo me consumio TODOS los creditos)

### Transformación por lotes

Por último usé un transformer para predecir todo el conjunto de prueba de una sola vez y comparar los resultados con las clases reales. Es la opción adecuada cuando hay muchos registros, porque consultar el endpoint uno por uno sería lento e innecesario.

## ¿Por qué hay un archivo .gitignore?

Contiene una sola línea, .ipynb_checkpoints/. JupyterLab crea esa carpeta automáticamente para guardar copias de respaldo de los notebooks. No forman parte del trabajo, cambian cada vez que se guarda y llenarían el repositorio de archivos que no sirven. Con el .gitignore, Git las ignora y solo quedan en el repositorio los archivos que realmente se entregan.

## Evidencias

Las evidencias esta vez se encuentran en la carpeta correspondiente para que sean visualizadas, evidentemente no hay evidencias de como subir las evidencias mas se entiende como se hizo el proceso (Y re hizo cuando me liaba con los endpoints)
