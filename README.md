# 📈 Pronóstico de mercado 
Proyecto de **Data Science** con el objetivo de pronosticar el estado del mercado para las Pymes en los próximos meses, en base a publicaciones en grupos de Facebook. 

La idea de este proyecto es recolectar las publicaciones y sus reacción en un grupo de Facebook, de esta manera se toma el numero de publicaciones como la oferta,aqw y la suma de reacciones sería la demanda en el mercado. Entre estos dos indicadores calculamos también el equilibrio de mercado $(\frac{Demanda}{Oferta})$. Así se se pronostica el estado del mercado en un futuro cercano en base a los datos de la series de oferta y demanda (Forecasting). Considerando lo anterior, este trabajo se divide en cuatro partes.

1. [Recuperar la información desde algún grupo de Facebook representativo.](#1-scraping-de-facebook)
2. [Contrastar los datos con algún indicador económico para definir si existe alguna correlación.](#2-correlación-facebook---iac)
3. [Realizar forecast de los datos de Facebook.](#3-forecast-del-mercado)
4. Desarrollar un panel donde visualizar la información.

En este repositorio revisaremos a fondo el código ejecutado en el desarrollo del proyecto, además de los comentarios y conclusiones de los resultados obtenidos.  

## 1. Scraping de Facebook
El scraping es una técnica utilizada para extraer información desde sitios web. En este caso se extraen datos desde Facebook, tomando el supuesto de que funciona como representación de la demanda y oferta de los pequeños negocios del pais. Esto dado que los emprendimientos suelen utilizar los medios digitales para encontrar clientes, y entre estos medios digitales encontramos las redes sociales y los grupos de Facebook.  

El código se encuentra en el archivo [scraper.ipynb](https://github.com/AndresDontLearns/pronostico-de-mercado/blob/main/scraper.ipynb)  

```python
  PPP = 100
  postID = []
  fecha = []
  coments = []
  likes = []
  reactions =[]
  count_r = []
  shares = []
  
  #El grupo 23369... es Compra y venta Santiago de Chile
  #La variable page_limit y PPP determinan la cantidad de publicaciones que se obtienen
  #En este caso se descargó un año de publicaciones
  for post in get_posts(group=2336974279945613, page_limit = 2400,extra_info = True,options={'allow_extra_requests':False,'posts_per_page': PPP}):
      postID.append(post['post_id'])
      time = post['time']
      fecha.append(time)
      coments.append(post['comments'])
      likes.append(post['likes'])
      reactions.append(post['reactions'])
      count_r.append(post['reaction_count'])
      shares.append(post['shares'])
``` 

Para extraer datos de Facebook se utilizó la libreria **Facebook-scraper** con su método correspodiente para obtener los post de la web. En este trabajo se rescataron los datos desde el grupo 'Compra y Venta Santiago de Chile', de un total de 2400 páginas y 100 post por cada página. De cada post extraido se guarda su Id, fecha de publicación, número de comentarios y reacciones, los datos se encuentran en el archivo [facebook-grupo.csv](https://github.com/AndresDontLearns/pronostico-de-mercado/blob/main/facebook-grupo.csvhttps://github.com/AndresDontLearns/pronostico-de-mercado/blob/main/facebook-grupo.csv). En el siguiente cuadro resumen vemos los datos recolectados, agrupados por el mes en el que fueron publicados los post:  

|**Mes**|**Nº de Posts**|**Comentarios + Reacciones**|
|-------|---------------|----------------------------|
|07-2020|1|1|
|06-2022|1|13|
|09-2022|3|150|
|10-2022|3|65|
|11-2022|1.442|1.478|
|12-2022|4.905|4.775|
|01-2023|9.322|13.628|
|02-2023|9.265|16.023|
|03-2023|10.791|63.831|
|04-2023|11.232|17.715|
|05-2023|3.223|6.977|
|06-2023|182|145|

De ahora en adelante se consideran los terminos **Oferta** como en número de post y **Demanda** como la cantidad de comentarios + reacciones. En la siguiente sección se veremos si estos datos estan relacionados a la economía de Chile 🇨🇱.  

## 2. Correlación Facebook - IAC
Como apreciamos en la tabla anterior de los datos extraidos de Facebook, hay algunos meses que se aprecian con un número bajo de Posts, muy alejados del resto de los meses o simplemente que no aparecen post. Por lo tanto, antes de realizar el análisis de correlación con el indicador económico se realiza un preprocesamiento en los datos, con el fin de obtener resultados más representativos. Los datos pasan por el siguiente proceso:  
1. Eliminar la los post con fecha 07-2020.
2. Modelar los datos de 07 y 08 del 2022.
3. Escalar los valores de la demanda y oferta.
4. Crear la nueva variable de **equilibrio** y escalarla.

Este proceso se realiza con el siguiente cuadro de código, y lo puedes encontrar en [regresion-lineal.ipynb](https://github.com/AndresDontLearns/pronostico-de-mercado/blob/main/regresion-lineal.ipynb).
```Python
#Eliminar valor atípico
pdata.drop(datetime(2020,7,1),inplace=True)

#modelar datos vacios
idx = pd.date_range(start=pdata.index.min(),end = pdata.index.max(),freq = 'MS')
pdata = pdata.reindex(idx)
pdata['demanda'].interpolate(method='cubic',inplace=True)
pdata['oferta'].interpolate(method='cubic',inplace=True)

#definir scaler y escalar demanda, oferta y equilibrio
scaler = preprocessing.StandardScaler()
pdata[['demanda','oferta']]= scaler.fit_transform(pdata[['demanda','oferta']])
pdata['equi'] = pdata['demanda']/pdata['oferta']
pdata[['equi']]= scaler.fit_transform(pdata[['equi']])
```  

El escalado de los datos se realiza para obtener un análisis comprensible a simple vista, ya que al trabajar con números cercanos cercanos al rango entre 0 y 1 es más fácil comprender la correlación que existen entre las variables independientes (en este caso oferta, demanda y equilibrio) y las dependeintes (el indicador económico).  

Para este proyecto se utilza el índice de actividad del comercio (IAC), calculado por el [Insituto Nacional de Estadística (INE)](https://www.ine.gob.cl/estadisticas/economia/comercio-servicios-y-turismo/actividad-mensual-del-comercio). Estamos usando el archivo correspondiente a Mayo-2023, que cuenta con los datos hasta Abril del 2023. Este archivo es [base-de-datos-a-abril-csv-2023.csv](https://github.com/AndresDontLearns/pronostico-de-mercado/blob/main/base-de-datos-a-abril-csv-2023.csv). Del reporte IAC solo consideramos las actividades relacionadas con el mercado al detalle, dado que para efectos de los pequeños emprendimientos nos interesa el comportamiento del consumidor final. Con los datos ya definidos se realiza un preprocesamiento similar al de los extrasidos desde Facebook:  

1. Se iguala el formato de las fechas con el de las variables independientes.
2. Se eliminan los datos que no corresponden con actividades al detalle.
3. Se escalan los valores de los datos a analizar. 

Ya con los datos preparados se realiza el análisis usando la función regresión lineal, de la libreria sklearn, para determinar los coeficientes de correlación de las variables y el valor de $r^2$ de los modelos aplicados a cada actividad económica estudiada. Los resultados son los siguientes

|Clase |   coef_dem |   coef_ofe |   coef_equi |       $r^2$ |
|--------|-----------|-----------|------------|---------|
|    4711 |  0.65 |  -0.37 |  -0.59  | 0.79 |
|    4719 |  0.88  |  -0.78 |  -0.91  | 0.84 |
|    4720 |  0.65  |  -0.42 |  -0.51  | 0.76 |
|    4730 |  0.08 |  -0.44  |  -0.07 | 0.67 |
|    4740 |  1.01   |  -0.68 |  -0.84  | 0.70 |
|    4752 |  0.23  |  -0.11 |  -0.05 | 0.25 |
|    4759 |  1.05   |  -0.87 |  -0.93  | 0.89  |
|    4760 |  1.16   |  -0.66 |  -0.90  | 0.80 |
|    4771 |  1.04   |  -0.73 |  -1.12   | 0.90 |
|    4772 |  0.81  |  -0.43 |  -0.55  | 0.63 |
|    4773 |  0.76  |  -0.65 |  -0.64   | 0.80 |  

De la tabla anterior se comprende que, cada clase corresponde a un modelo de regresión lineal aplicado a los datos de la actividad económica que representa. Las columnas de "coef_" determinan cuanto peso tiene la variable en la explicación del modelo, mientras más alejado de 0 sea este valor más será su importancia. En el caso del indicador $r^2$, muestra que tanto se ajusta el modelo a los datos que busca representar, siendo 0 nada y 1 perfectamente.  

Del cuadro podemos inferir que la importancia de las variables independientes esta en el siguiente orden: **demanda**, **equilibrio** y **oferta**. Y el modelo que mejor se ajusta a los datos es el de la clase *4771*, que corresponde a "venta al por menor de prendas de vestir, calzado y artículos de cuero en comercios especializados". Si deseas saber la definición de las demás clases te invito a revisar le link del INE que deje más arriba.  

Si bien los indicadores de correlación parecen altos y con ellos podriamos concluir que encontramos una directa relación entre las publicaciones del grupo de Facebook y la actividad comercial, la verdad es que la cantidad de datos que tenemos no es suficientemente representativa como para indicar si el modelo realmente puede predecir la actividad economica. Sin embargo, este aálisis se continuará realizando en los meses futuros con el fin de entregarles el prónostico a los emprendimientos que visiten [Pymetools](https://www.pymestools.com) y a medida que se acumulen los datos con el tiempo se podrá verificar si los indicadores actuales tenían peso realmente.

## 3. Forecast del mercado
Esta es la última parte que se realiza en Python, el pronóstico de las variables de oferta, demanda y equilibrio del mercado según la actividad en el grupo de Facebook. Para lograrlo es necesario realizar un Forecasting sobre los datos de oferta y demanda, esto significa que se ajustarán modelos para predecir los datos de estas variables de forma autoregresiva, es decir, pronosticar datos futuros en base a sus datos pasados.  

Este modelo se construye con las libreria Skforecast, que funciona de la mano con la librería Sklearn. Para trabajar con los datos se realiza el mismo preprocesamiento mostrado anteriormente, y para predecir los valores se utiliza una función Ridge, que es una regresión lineal utilizada para evitar el sobreajuste de las variables en el modelo.  

``` python
#definir series de datos para el forecast, dejando el ultimo dato como evaluador
d_train = pd.Series(pdata['demanda'])[:-1]
o_train = pd.Series(pdata['oferta'])[:-1]
d_test = pd.Series(pdata['demanda'])[-1:]
o_test = pd.Series(pdata['oferta'])[-1:]

#Definición de la función de predicción y ajuste para la demanda
d_forecaster = ForecasterAutoreg(
                regressor = Ridge(alpha= 4),
                lags=8)
d_forecaster.fit(y= pd.Series(d_train))
print(d_forecaster.get_feature_importances())

#Predicción de los dos pasos futuros de la demanda
d_pred = d_forecaster.predict(steps=2)

error_mse = mean_squared_error(
                y_true = d_test,
                y_pred = d_pred[:1]
            )

print(f"Error de test (mse): {error_mse}")

#Graficar
fig, ax = plt.subplots(figsize=(7, 2.5))
d_train.plot(ax=ax, label='train')
d_test.plot(ax=ax, label='test')
d_pred.plot(ax=ax, label='predicciones')
ax.set_title('Demanda')
ax.legend()
```

En el código anterior vemos la creación del modelo de Forecast y la predicción realizada, puedes ver este código completo en el archivo [forecast-ridge.ipynb](https://github.com/AndresDontLearns/pronostico-de-mercado/blob/main/forecast-ridge.ipynb). En este modelo, a diferencia de la práctica usual, solo queda un dato para probar el modelo y no es representativo. Esto se debe a que es la información recolectada del mes en curso, por lo que no tiene el total de publicaciones del mes. Otro aspecto importante a destacar es el número de pasos que se pronostican, que es igual a los meses, actualmente está configurado a 2 meses en el futuro conssiderando el mes en curso y el siguiente. Finalmente, el último aspecto del modelo que me parece relevante destacar es que la demanda tiene un factor anti-ajuste mayor que la oferta dada su mayor disperción.  

Ya realizadas las predicciones de los datos solo falta calcular la variable de **equilibrio**. Los datos resultantes los puedes encontrar en 



## 4. Looker Studio
