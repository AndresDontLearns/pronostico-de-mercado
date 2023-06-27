# 📈 Pronóstico de mercado 
Proyecto de **Data Science** con el objetivo de pronosticar el estado del mercado para las Pymes en los próximos meses, en base a publicaciones en grupos de Facebook. 

La idea de este proyecto es recolectar las publicaciones y sus reacción en un grupo de Facebook, de esta manera se toma el numero de publicaciones como la oferta,aqw y la suma de reacciones sería la demanda en el mercado. Entre estos dos indicadores calculamos también el equilibrio de mercado $(\frac{Demanda}{Oferta})$. Así se se pronostica el estado del mercado en un futuro cercano en base a los datos de la series de oferta y demanda (Forecasting). Considerando lo anterior, este trabajo se divide en cuatro partes.

1. [Recuperar la información desde algún grupo de Facebook representativo.](#1-scraping-de-facebook)
2. [Contrastar los datos con algún indicador económico para definir si existe alguna correlación.](#2-correlación-facebook---iac)
3. Realizar forecast de los datos de Facebook.
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
Como apreciamos en la tabla anterior de los datos extraidos de Facebook, hay algunos meses que se aprecian con un número bajo de Posts, muy alejados del resto de los datos o simplemente que no aparecen post en esos meses. Por lo tanto, antes de realizar el análisis de correlación con el indicador económico se realiza un preprocesamiento en los datos, con el fin de obtener resultados más representativos. Los datos pasan por el siguiente proceso:  
1. Eliminar la los post con fecha 07-2020.
2. Modelar los datos de 07 y 08 del 2022.
3. Escalar los valores de la demanda y oferta.
4. Crear la nueva variable de **equilibrio** y escalarla.

Este proceso se realiza con el siguiente cuadro de código, y lo puedes encontrar en [regresion-lineal.ipynb]([https://github.com/AndresDontLearns/pronostico-de-mercado/blob/main/scraper.ipynb](https://github.com/AndresDontLearns/pronostico-de-mercado/blob/main/regresion-lineal.ipynb)).
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

Para este proyecto se utilza el índice de actividad del comercio (IAC), calculado por el [Insituto Nacional de Estadística (INE)](https://www.ine.gob.cl/estadisticas/economia/comercio-servicios-y-turismo/actividad-mensual-del-comercio). Estamos usando el archivo correspondiente a Mayo-2023, que cuenta con los datos hasta Abril del 2023. Este archivo es [base-de-datos-a-abril-csv-2023.csv](https://github.com/AndresDontLearns/pronostico-de-mercado/blob/main/base-de-datos-a-abril-csv-2023.csv). Del reporte IAC solo consideramos las actividades relacionadas a el mercado al detalle, dado que para efectos del mercado de pequeños emprendimientos enos interesa el comportamiento del consumidor final.



