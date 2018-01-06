Pandas para trabajar con tablas
===============================

Para todo lo que tiene que ver con datos que podrian ir en tablas (al estilo MS Excel) el modulo 'pandas' es una herramienta muy util. Nos permite combinar tablas, hacer operaciones en las filas y columnas, agrupar segun los valores en algunas columnas y hacer operaciones. En fin, nos permite hacer y deshacer lo que se nos ocurra. 

Vamos a bajar y extraer la data `datos.zip <../datos/datos.zip>`. 
El archivo `PERSONA-P03.csv` contiene informacion recopilada del Censo Nac. de Hogares y viviendas 2010. En este caso, poder tener la informacion en un formato tan conveniente se lo debemos a Manuel Aristaran, que `colgo <http://dump.jazzido.com/CNPHV2010-RADIO/>` los resultados de queries a la base de datos REDATAM en donde estan registrados los resultados del ultimo censo en Argentina.

En particular el archivo que tenemos aca tiene una columna con los codigos que identifican los radios censales, con el nombre 'link'. El resto de las columnas corresponden a cada edad posible en anios, de forma que cada fila puede tomarse como informacion de la piramide poblacional de un radio censal. Con este dataset podemos saber esencialmente cuanta gente de cada edad habia en cada lugar en octubre de 2010.

Vamos a cargarlo en un notebook de IPython:

.. ipython:: python

    import pandas as pd

    persona = pd.read_csv('datos/PERSONA-P03.csv')

    #Ahora ya tenemos guardado el dataset como un DataFrame de pandas.
    #Un dataframe tiene muchos metodos que nos sirven para interactuar con el. Por ejemplo, para ver una muestra de la data:
    persona.sample(5)


Geopandas
---------

Los ejemplos que voy a mostrar explotan no solo la data estadistica sino tambien la geografica. Notar que la clave del dataset persona que acabamos de cargar es que tenga un codigo identificatorio del area censal. Esto es esencial porque nos permite la combinacion con otros dataset que compartan esta columna, o bien que tengan una columna, por ejemplo 'provincia' o 'departamento' que se pueda combinar con la de los radios censales.

Hay un detalle que tambien hay que tener en cuenta. Cuando estamos en la capa geografica, mucha informacion puede naturalmente corresponder a puntos, lineas o poligonos (areas) en el espacio. Estamos habalndo por ejemplo de la ubicacion de edificios, el recorrido de lineas de transporte u otra infraestructura, o los limites de una provincia o distrito. La calidad de la informacion y de los analisis que podamos hacer va a ser infinitamente mayor si contamos con esta informacion de manera exacta.

Para este tipo de informacion, hay formatos especiales de archivos. Las alternativas son muchas, pero una bastante standard es usar archivos ShapeFile (.shp). El archivo .shp viene siempre acompaniado de otros archivos con informacion necesaria como aclaraciones de los sistemas de coordenadas, entre otras cosas.

En una terminal (ubicarse en el directorio donde)

.. code:: bash

   $ unzip datos.zip
   $ cd datos/link_areas/.shp
   $ ls
   metadatos shape pxlocdatos.pdf  nota aclaratoria.pdf  pxlocdatos.cpg  
   pxlocdatos.dbf  pxlocdatos.prj  pxlocdatos.qpj  pxlocdatos.shp  pxlocdatos.shx  Thumbs.db

Ahora vamos a usar el modulo geopandas, que esencialmente es lo mismo que pandas, con la posibilidad de incluir formas geometricas (puntos, lineas, poligonos) y hacer operaciones con ellos.

Coordinate reference system (CRS)
---------------------------------

Un GeoDataFrame que se lee de un ShapeFile contiene por lo general informacion sobre el sistema de coordenadas en el cual esta proyectada la data.

Empecemos leyendo la data del archivo ``pxlocdatos.shp``.

.. ipython:: python

    import geopandas as gpd
    
    # Leer data
    pxlocdatos = gpd.read_file("datos/link_areas/pxlocdatos.shp")
    
    # Muestra de la data
    pxlocdatos.sample(3)

Al igual que en un DataFrame corriente de pandas, podemos por ejemplo preguntar cuales son las columnas de esta tabla:


.. ipython:: python

    pxlocdatos.columns

We can see the current coordinate reference system from ``.crs``
attribute:

.. ipython:: python

    pxlocdatos.crs

Informacion sobre los sistemas de coordenadas se puede encontrar en:

  - `www.spatialreference.org <http://spatialreference.org/>`__
  - `www.proj4.org <http://proj4.org/projections/index.html>`__
  - `www.mapref.org <http://mapref.org/CollectionofCRSinEurope.html>`__

Para obtener datos de las formas geometricas de las localidades vamos a cargar los archivos shape pertenecientes a la Provincia de Buenos Aires y la Ciudad de Buenos Aires (CABA).

.. ipython:: python

    Buenos_Aires_datos = gpd.read_file("datos/Buenos Aires/Buenos_Aires_con_datos.shp")
    
    Buenos_Aires_datos.sample(5)

Fijense que hay una columna que se llama ``geometry``. En general la informacion especial de los objetos va a ir a para a esta columna. en el caso de la tabla 'pxlocdatos' los elementos son instancias de shapely.Point. Estan describiendo probablemente un centroide del radio censal. El dataset de la Provincia si tiene formas geometricas, ver por ejemplo lo que pasa cuando hacemos:

.. ipython:: python

    Buenos_Aires_datos['geometry'][10]

Que nos grafica el area de la fila 10 con un dibujito.

Los GeoDataFrames permiten usar toda la funcionalidad de los DataFrames de pandas. Por ejemplo, podemos crear nuevas columnas con codigos de provincia y departamento (partido/comuna) y otra columna que se va a llamar 'dpto_link' que es una concatenacion del codigo de provincia y departamento, de forma de tener un codigo de departamento util a nivel nacional.


.. ipython:: python

    Buenos_Aires_datos['geometry'][10]
    
Como ultimo ejemplo podemos graficar las localidades en el espacio. Las coloreamos segun la provincia, para ilustrar una de las posibilidades.

.. code:: python
    
    #import the standard plotting module
    import matplotlib.pyplot as plt
    %matplotlib inline

    # create subplots
    f, ax = plt.subplots(1, figsize=(3, 5))

    pxlocdatos.plot(axes = ax, column = 'codpcia', edgecolor = 'None', marker = '.')
       
    # Add title
    plt.title('Localidades y provincias');

    # Remove empty white space around the plot
    plt.tight_layout()
    
    plt.show()
    
Los ejemplos mostrados aqui estan en el notebook Parte_I.ipynb 

Geolocalizacion
===============

Muchas veces nos va a interesar relacionar un dato como por ejemplo el nombre de un edificio, una direccion, u algun monumento con un objeto en una base de datos, especialmente si esta base de datos tiene coordenadas geograficas.
Dicho de otro modo, una cosa es saber el nombre de un lugar y su direccion, y tenerlo anotado. Otra cosa (por lo general, mejor) es conocer sus coordenadas geograficas, ya que pasamos de una frase que debe ser interpretada (la direccion) a un numero que describe una posicion univocamente y puede ser usado para cualquier calculo de ser necesario.

Este pasaje se llama **geolocalizacion** y en general, no se reduce solo a encontrar las coordenadas geograficas para una 'frase' que uno pregunte, sino que relaciona esta frase con un objeto en la base de datos de consulta y todos sus atributos. Es decir, nos puede ademas brindar otra informacion acerca del objeto por el cual preguntamos, por ejemplo que tipo de edificio se ubica en el lugar y que funciones cumple, cual es el distrito, provincia o pais correspondiente, su codigo postal, entre otras informaciones.

Hay varios servicios de geolocalizacion. Lo mas comodo para mi es usar la **API de Google Maps**, que viene a ser un canal de comunicacion con Google Maps de forma que obtenemos resultados de busqueda pero sin necesidad de hacer trabajo manual. 

Para los que deseen, pueden ir directamente a la pagina en donde se dan las instrucciones <https://developers.google.com/maps/documentation/geocoding/start> de uso. Es muy facil, en la parte inferior de dicha pagina hay un boton con la opcion de generar una API key. Esta es una clave que se pide cada vez que hacemos una consulta. Una vez que entregan la API key, ya podemos empezar a consultar. Por ejemplo, en la barra de busqueda se puede escribir:

https://maps.googleapis.com/maps/api/geocode/json?address=Instituto+Santa+Maria+de+los+Angeles,+Coghlan&key=TU API KEY

y vas ver que viene dada una respuesta con informacion en formato json. Mas adelante vamos a ver como consultar todas las direcciones que podemos tener en una de las columnas de nuestra table de datos. Ahi es cuando se empieza a poner bueno.

Antes de continuar, queria comentar algunos de los inconvenientes que hay con este metodo. 

Como siempre, es posible que la direccion que busquemos no se encuentre. En esta situacion hay varias opciones que considerar. Basicamente hay que averiguar porque exactamente no se encontro la direccion, y tambien evaluar que tan importante es en el marco de lo que estamos haciendo. 

Si no hace la diferencia, podemos 'tirar' las direcciones que no hayan dado resultado y seguir con el analisis. Si queremos encontrar la informacion del dato de alguna forma, lo primero que hay que chequear es que no haya errores de tipeo. Luego, muchas veces si tenemos direcciones que son largas, puede ser que algunas de las palabras confundan al buscador. Puede ser mejor evitar informacion extra del tipo 'Departamento de Quimica Organica' o 'Barrio el Jaguel'. En el primer caso porque los datos son de edificios enteros, y en general incluir nombres de divisiones institucionales confunde mas de lo que aporta. En el segundo caso, puede tratarse de nombres populares de zonas, pero por ejemplo en el caso de Buenos Aires, la direccion ya es unica a nivel partido, es decir que barrios o localidades en realidad no hacen la diferencia.

Haciendo mapas con shapely y geopandas
======================================

A continuacion, vamos a aprovechar la funcionalidad de geopandas y los poligonos de shapely para construir mapas. Primero, vamos a cargar algunas tablas con informacion de poblacion y poligonos de distritos, junto con sus perimetros en formato de poligono shapely.

El INDEC ofrece tablas con la informacion necesaria (https://www.indec.gov.ar/codgeo.asp), sin embargo aparece separado en datasets para cada provincia, y los criterios en cada dataset son 'parecidos' pero nollegan a ser 100% compatibles como deberian. Me refiero a que los nombres de columnas pueden variar ligeramente de provincia a provincia, o que las codificaciones de los distritos pueden llegar a estar en un formato ligeramente diverso, de forma que combinar las tablas no es automatico y tenemos que hacer algunos retoques. Aprovecho para llamar la atencion sobre este tipo de desinteligencias en el registro de datos que nos complican la vida. En el codigo, las van a notar cuando escribo (!!!). Eso significa que es algo que no tendriamos que estar haciendo si los datos estuvieran registrados mas sistematicamente. De todos modos nos sirve para ver como tratar en python pandas estas situaciones que son muy comunes.

En este caso vamos a combinar las tablas de radios censales (junto con algunos datos de poblacion) de la Ciudad de Buenos Aires (CABA) y la Provincia.


.. ipython:: python

    # Cargar archivo
    CABA_datos = gpd.read_file("datos/CABA/cabaxrdatos.shp")
    # Seleccionar columnas deseadas
    CABA_datos = CABA_datos[[u'PROV', u'DEPTO',u'LINK',u'VARONES',u'MUJERES',u'TOT_POB', u'HOGARES', u'VIV_PART', u'VIV_PART_H', u'geometry']]
    # Tenemos que renombrarlas para que coincidan con las de la tabla de Buenos Aires (!!!)
    CABA_datos.columns = [u'prov', u'depto',u'link',u'varon',u'mujer',u'totalpobl', u'hogares', u'viviendasp', u'viv_part_h', u'geometry']

    # Cargar archivo
    Buenos_Aires_datos = gpd.read_file("datos/Buenos Aires/Buenos_Aires_con_datos.shp")
    # Le faltan columnas con codigo de provincia y departamento (!!!)
    Buenos_Aires_datos['prov'] = Buenos_Aires_datos['link'].str[:2]
    Buenos_Aires_datos['depto'] = Buenos_Aires_datos['link'].str[2:5]
    Buenos_Aires_datos['dpto_link'] = Buenos_Aires_datos['link'].str[:5]
    Buenos_Aires_datos = Buenos_Aires_datos[[u'prov', u'depto', u'link', u'varon', u'mujer', u'totalpobl',
           u'hogares', u'viviendasp', u'viv_part_h', u'geometry']]

Bueno, ya tenemos dos tablas con los radios censales de la Ciudad y de Buenos Aires. Que podemos hacer con data en este formato?
Veamos algunos ejemplos:

.. ipython:: python
    CABA_polygon = CABA_datos.dissolve(by = 'prov')['geometry']
    partidos = Buenos_Aires_datos.dissolve(by = 'depto')[['geometry']]

    
En pandas, una de las operaciones mas importante es `groupby`. Lo que hace es, tomando una(s) columna(s) de referencia, reunir a todas las observaciones que coinciden en dicha columna y devuelven un 'objeto agrupado'. A este objeto le podemos poner una operacion, por ejemplo suma, promedio, tomar muestra, etc. Por ejemplo podriamos tener informacion de poblacion a nivel de departamentos, y con `pandas.groupby()` agregarla a nivel de provincias y mostrar el total de poblacion por provincia. 

El metodo `geopandas.dissolve` hace esencialmente lo mismo pero combinando las formas geometricas con union. Es decir que siguiendo el ejemplo, junto con la suma de poblacion nos va a crear el agregado de departamentos en una provincia. Para todas las provincias.

Las formas geometricas de shapely tienen varios atributos que las describen. Por ejemplo, podemos calcular el centroide:

.. ipython:: python
    CABA_centroid = CABA_polygon.centroid.iloc[0]
    
El centroide es un objeto punto de shapely. Podemos usarlo para calcular la distancia de los partidos de Buenos Aires al centroide de la capital. Anotemoslo como una columna del GeoDataFrame 'partidos'

.. ipython:: python
    partidos['distancia_CABA'] = [CABA_centroid.distance(m) for m in partidos.geometry]
    
Podemos extraer los codigos de los partidos mas cercanos a la Ciudad y usarlos para filtrar en el dataset de radios censales de Provincia de Buenos Aires aquellos que pertenecen a este autodefinido 'GBA'. Uso 36 partidos porque si, la idea es nomas dar ejemplos de lo que se puede hacer. Tambien podriamos haber metido directamente la lista de partidos del conurbano, las alternativas quedan a gusto de cada uno! 

.. ipython:: python
    partidos_GBA = partidos.sort_values(by = 'distancia_CABA')[:36].index
    GBA_datos = Buenos_Aires_datos.loc[Buenos_Aires_datos.depto.isin(partidos_GBA)]
    AMBA_datos = pd.concat([CABA_datos, GBA_datos])
    
usamos aca el metodo `pd.concat`, que nos pone una tabla a continuacion de la otra. En un paso anterior escogimos las mismas columnas en ambos DataFrames, ademas de asignarles nombres identicos a sus columnas. De esta forma el DataFrame 'AMBA_datos' que generamos queda uniforme como debe ser.

Ejemplo: cantidad de chicos de 13 a 18 por radio censal
-------------------------------------------------------

Carguemos data de poblacion por radio censal. Basicamente esto nos permite conocer a donde vive la gente. Usamos el dataset 'PERSONA-P03.csv' creado a partir de queries a la base REDATAM de INDEC (mas info). Como dijimos al principio, este dataset nos informa la 'piramide poblacional' de cada radio censal. Buen nivel, no? ;)

.. ipython:: python

    persona = gpd.GeoDataFrame(pd.read_csv('datos/PERSONA-P03.csv'))
    persona.rename(columns={'radio': 'link', 'TOTAL': 'totalpobl'}, inplace=True)
    # Aca tenemos que prestar atencion y completar con un cero por delante, en caso de que haya sido removido automaticamente en algun paso intermedio.
    persona['link'] = persona['link'].astype(str).str.zfill(9)

La informacion de las edades de la gente por radio censal puede ser un nivel de desagregacion excesivo para algunas aplicaciones. Aca sin embargo, vamos a aprovechar esta informacion, y ya que estamos hablando de escuelas secundarias, vamos a calcular cuantos jovenes de entre 13 y 18 anios hay por radio censal. Del dataset 'persona' nos vamos a quedar entonces solo con dos columnas: 'link' y 'persona_13_18'.

.. ipython:: python
    # Sumar personas en edades 13 a 18.
    persona['persona_13_18'] = persona.iloc[:, 14:19].sum(axis = 1)

    # Combinar con la tabla AMBA_datos, que contiene los poligonos de los radios censales. 
    AMBA_datos_persona_13_18 = AMBA_datos.merge(persona[['link', 'persona_13_18']], on = 'link')
    # Calcular area en km2
    AMBA_datos_persona_13_18["area_km2"] = 10**-6 * AMBA_datos_persona_13_18['geometry'].area
    # Con cantidad de chicos y area, calcular densidad
    AMBA_datos_persona_13_18['densidad'] = AMBA_datos_persona_13_18['persona_13_18'] / AMBA_datos_persona_13_18["area_km2"]

Combinamos la informacion de cantidad de chicos con la tabla que tiene los poligonos de cada radio censal. Para eso es necesario tener una columna compartida que identifique los radios en ambas tablas. En este caso, la columna 'link'. Creemos un mapa para poder echar un vistazo.

.. ipython:: python
    f, ax = plt.subplots(1, figsize=(10, 10))
    AMBA_datos_persona_13_18.plot(axes = ax, column = 'densidad', cmap='gray', edgecolors = 'None', vmin = 0, vmax = 2300, alpha = 0.5)
    # Graficamos tambien los partidos (sin pintar y con borde blanco) como referencia visual.
    partidos.plot(axes=ax, color = 'None', edgecolor = 'w')
    plt.xlim(4140000, 4220000)
    plt.ylim(6100000, 6180000)
    plt.show()
    
Ejemplo: zonas en las que cada escuela es la mas cercana
--------------------------------------------------------

Ahora, carguemos un dataset con ubicaciones de edificios. Para este ejemplo vamos a usar el dataset de escuelas secundarias del AMBA geolocalizadas que obtuvimos en el tutorial anterior. 

.. ipython:: python
    # Cargar csv, y tirar observaciones sin localizar.
    edificios = pd.read_csv('datos/esc_sec_AMBA_geoloc_full.csv').dropna()

    # Pasar la data de latitud y longitud a objetos shapely.Point
    geometry = [Point(xy) for xy in zip(edificios.lng, edificios.lat)]
    edificios.drop(['lng', 'lat'], axis=1, inplace=True)

    # Lo reescribimos como GeoDataFrame, indicando el sistema de coordenadas.
    edificios = GeoDataFrame(edificios, crs = {'init': u'epsg:4326'}, geometry=geometry)

    # Pero pasamos al sistema de coordenadas (CRS) de 'partidos'. Todos los otros datos ya estan en este CRS.
    edificios['geometry'] = edificios['geometry'].to_crs(crs=partidos.crs)

Tenemos la data de los edificios geolocalizados como un objeto GeoDataFrame. Nomas para mostrar algunas de las posibilidades que ofrece este tipo de data, vamos a calcular para esta coleccion de puntos, lo que son los poligonos de Voronoi. Ojo, no es nada complejo, simplemente cada escuela tendra a su alrededor un poligono, y este poligono contiene todos los puntos para los cuales la escuela es la mas cercana. Es como demarcar las areas para las cuales cada escuela es la mas cercana. 

.. ipython:: python

    from scipy.spatial import Voronoi, voronoi_plot_2d
    import shapely.geometry
    import shapely.ops
    
    # Necesitaremos tener una columna con el valor de 'x' e 'y'.
    def getPointCoords(row, geom, coord_type):
    
        """Calculates coordinates ('x' or 'y') of a Point geometry"""
        if coord_type == 'x':
            return row[geom].x
        elif coord_type == 'y':
            return row[geom].y
    edificios['x'] = edificios.apply(getPointCoords, geom='geometry', coord_type='x', axis=1)
    edificios['y'] = edificios.apply(getPointCoords, geom='geometry', coord_type='y', axis=1)
    
    # Lista de puntos (ubicaciones de las escuelas en este caso) y calculo de los poligonos de Voronoi.
    points = edificios[['x','y']].as_matrix()
    vor = Voronoi(points)

    # Crear mapa
    f, ax = plt.subplots(1, figsize=(10, 10))
    # Incluimos partidos (GBA) y comunas (CABA), demarcados con lineas rojas...
    partidos['geometry'].plot(axes=ax, color = 'None', edgecolor = 'red')
    CABA_datos.dissolve(by = 'depto')['geometry'].plot(axes=ax, color = 'None', edgecolor = 'red')
    # y los poligonos.
    voronoi_plot_2d(vor, ax=ax)

    # Limites del mapa
    plt.xlim(4160000, 4200000)
    plt.ylim(6120000, 6160000)
    plt.show()

    # Recoger todas las lineas y armarlas en poligonos.
    lines = [shapely.geometry.LineString(vor.vertices[line]) for line in vor.ridge_vertices]
    p = []
    for poly in shapely.ops.polygonize(lines):
        p += [poly]

    # Creamos un dataframe con la informacion de los poligonos
    voronoi = gpd.GeoDataFrame(p, columns = ['geometry'])

Listo, tenemos un GeoDataframe con poligonos que marcan la zona en la cual cada escuela es la mas cercana. En la proxima seccion vamos a darle un aplicacion combinando informacion sobre la poblacion por radio censal. El objetivo de este pequenio ejemplo es mostrar una operacion que podemos hacer que relaciona puntos con una vecindad correspondiente. Recordemos que los puntos de este ejemplo pueden ser reemplazados por cualquier otra informacion de ubicaciones dependiendo de la aplicacion que le queramos dar. Lo importante de estos ejemplos no son los resultados particulares que tengamos aca, sino que cada uno puede adaptar el codigo de acuerdo a lo que le interese en el momento.

Guardamos las tablas utiles
---------------------------
Por ultimo vamos a guardar la informacion util, de forma de poder cargarla directamente para usar en los ejemplos siguientes.

.. ipython:: python
    voronoi.to_file('datos/voronoi_escuelas_secundarias_AMBA.shp')
    AMBA_datos_persona_13_18.to_file('datos/AMBA_datos_persona_13_18.shp')
    partidos.to_file('datos/partidos.shp')
    CABA_datos.to_file('datos/CABA_datos.shp')
