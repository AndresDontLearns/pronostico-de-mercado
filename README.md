# 📈 Pronóstico de mercado 
Proyecto de **Data Science** con el objetivo de pronosticar el estado del mercado para las Pymes en los próximos meses, en base a publicaciones en grupos de Facebook. 

La idea de este proyecto es recolectar las publicaciones y sus reacción en un grupo de Facebook, de esta manera se toma el numero de publicaciones como la oferta,aqw y la suma de reacciones sería la demanda en el mercado. Entre estos dos indicadores calculamos también el equilibrio de mercado $(\frac{Demanda}{Oferta})$. Así se se pronostica el estado del mercado en un futuro cercano en base a los datos de la series de oferta y demanda (Forecasting). Considerando lo anterior, este trabajo se divide en cuatro partes.

1. Recuperar la información desde algún grupo de Facebook representativo.
2. Contrastar los datos con algún indicador económico para definir si existe alguna correlación.
3. Realizar forecast de los datos de Facebook.
4. Desarrollar un panel donde visualizar la información.

En este repositorio revisaremos a fondo el código ejecutado en el desarrollo del proyecto, además de los comentarios y conclusiones de los resultados obtenidos.  

## 1. Scraping de Facebook
El scraping es una técnica utilizada para extraer información desde sitios web. En este caso se extraen datos desde Facebook, tomando el supuesto de que funciona como representación de la demanda y oferta de los pequeños negocios del pais. Esto dado que los emprendimientos suelen utilizar los medios digitales para encontrar clientes, y entre estos medios digitales encontramos las redes sociales y los grupos de Facebook.  

El código se encuentra en el archivo [scraper.ipynb](https://github.com/AndresDontLearns/pronostico-de-mercado/blob/main/scraper.ipynb)
(Agregar codigo de la libreria utilizada)  
Para extraer datos de Facebook se utilizó la libreria **Facebook-scraper** con su metodo correspodiente para obtener los post de la web. En este trabajo se rescataron los datos desde el grupo 'Compra y Venta Santiago de Chile', de un total de 2400 páginas y 100 post por cada página. De cada post extraido se guarda su Id, fecha de publicación, número de comentarios y reacciones, los datos se encuentran en el archivo [facebook-grupo.csv](https://github.com/AndresDontLearns/pronostico-de-mercado/blob/main/facebook-grupo.csvhttps://github.com/AndresDontLearns/pronostico-de-mercado/blob/main/facebook-grupo.csv). En el siguiente cuadro resumen vemos los datos recolectados agrupados por mes en el que fueron publicados los post:  

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




