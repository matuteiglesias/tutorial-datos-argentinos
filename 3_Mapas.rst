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
