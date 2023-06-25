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

(Agregar codigo de la libreria utilizada)  
Para extraer datos de Facebook se utilizó la libreria **Facebook-scraper** con su metodo correspodiente para obtener los post de la web. En este trabajo se rescataron los datos desde el grupo 'Compra y Venta Santiago de Chile', de un total de 2400 páginas y 100 post por cada página. De cada post extraido se guarda su Id, fecha de publicación, número de comentarios y reacciones. En el siguiente cuadro resumen vemos los datos recolectados agrupados por mes en el que fueron publicados los post:  

|**Mes**|**Nº de Posts**|**Comentarios**|**Reacciones**|


