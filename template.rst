Introduccion
============

Cada dia escuchamos mas seguido hablar de como la data crece en cantidad y calidad, al punto que es un signo de nuestra epoca. Los usos que se le pueden dar dependen de las intenciones e intereses de cada uno. Es muy comun que los negocios usen los datos para 'vender mas', o que algunos los usen para buscar estrategias financieras rentables. Hay tantos usos posibles de los datos como personas y organizaciones en el mundo.

Lo que mas me interesa a mi es el uso de los datos para informar al estado. Es decir, conocer con mayor detalle las dinamicas y situaciones que atraviesan los actores y asi poder diseniar y evaluar politicas, y en general lograr que el estado funcione de forma mas eficiente y racional. 

Me atrapa este tema porque creo que puede ayudar a mejorar la calidad de vida de muchos. El caso argentino para mi es muy emocionante, no solo porque de ahi vengo, sino porque ademas hay mucho espacio para mejorar y avanzar, tanto en la calidad de vida y el desarrollo economico en general, como en las practicas de recoleccion y registro de datos. 

Con estos tutoriales, tengo varios objetivos:
-Por un lado quisiera aportar a concientizar sobre la importancia de que nuestro pais tenga buenas practicas en cuanto al manejo de datos. Sobretodo en la medida en que nos pueden permitir mejorar muchisimo la calidad de nuestras politicas informando y evaluando realidades.
-Por otro lado, muchas veces el hecho de que no contemos con informacion de buena calidad va de la mano de la formacion de los recursos humanos. La estadistica existe desde hace rato, pero las posibles explotaciones de los datos y las tecnologias de innovacion actual todavia no se ensenian en ningun lado. Tenemos que agarrarlas al vuelo y aprovecharlas muchas veces depende de nuestra inventiva. En este proceso es muy importante compartir ideas y cosas que probamos para construir conocimiento y experiencia colectivamente. Ojala le sea util a los que quieran empezar a estudiar y trabajar con la data argentina explotando algunas de las nuevas tecnicas disponibles.

Espero hacer el esfuerzo de acompaniar de forma didactica, pero este siempre fue mi punto debil, asi es que espero sus comentarios para poder adaptar y aclarar la exposicion de los puntos complicados.

Instalar Python + packages
==========================

**Las herramientas que vamos a usar**

Los codigos que incluyo aca son en lenguage Python. Nunca es tarde para empezar a usar Python, sirve para hacer que la computadora haga cosas increibles! Se puede instalar directo de `su pagina <https://www.python.org/>`_, pero **recomiendo** usar `Anaconda <https://www.continuum.io/anaconda-overview>`_ que es una distribucion open-source de los lenguajes Python y R para procesamiento de data, predictive analytics, y computacion cientifica, que intenta simplificar el manejo de los packages o modulos. Es decir, nos simplifica la vida.

El codigo en Python esta pensado para ser escrito en un archivo y correrlo todo de una vez. Muchas veces es mas deseable correr solamente los ultimos renglones, sobretodo si estamos con un codigo que demora en correr y donde toda la primera parte no tiene ningun problema. Ademas en el caso que hayamos calculado algunas variables, quisieramos que sean recordadas por el momento y poder volver a usarlas facilmente. Por este tipo de cuestiones practicas, y otras mas, se desarrollaron las Jupyter notebooks, que nos permiten correr los codigos Python con esta modalidad mas practica. Para instalar `ver aca. <https://jupyter.readthedocs.io/en/latest/install.html#new-to-python-and-jupyter>`

Install Python + packages en Windows
------------------------------------

Estos pasos se testearon en Windows 7 and 10 con Anaconda3 64 bit, usando conda v4.3.29. (Octubre 2017)

`Download Anaconda installer (64 bit) <https://www.continuum.io/downloads>`_ para Windows.

Instalar Anaconda a tu computadora clickeando el instalador, e instalarlo en el directorio que quieras (necesita administrador).
Para **todos los usuarios** y con configuraciones por default.

Fijate que ``conda`` funcione `abriendo una consola como administrador <http://www.howtogeek.com/194041/how-to-open-the-command-prompt-as-administrator-in-windows-8.1/>`_ y corriendo ``conda --version``.

A medida que sea necesario, instala estos packages con conda (y pip) corriendo en una consola estos comandos (el orden importa):

.. code::

    # Install numpy (v 1.13.1)
    conda install numpy

    # Install pandas (v 0.20.3) --> bundled with python-dateutil (v 2.6.1) and pytz (v 2017.2)
    conda install pandas

    # Install scipy (v 0.19.1)
    conda install scipy

    # Install matplotlib (v 2.0.2) --> bundled with cycler, freetype, icu, jpeg, libpng, pyqt, qt, sip, sqlite, tornado, zlib
    conda install matplotlib

    # Install Geopandas (v 0.3.0) --> bundled with click, click-plugins, cligj, curl, descartes, expat, fiona, freexl, gdal, geos, hdf4, hdf5, kealib, krb5, libiconv, libnetcdf, libpq, libspatialindex, libspatialite, libtiff, libxml2, munch, openjpeg, pcre, proj4, psycopg2, pyproj, pysal, rtree, shapely, sqlalchemy, xerces-c
    conda install -c conda-forge geopandas


Chequea que funciono
~~~~~~~~~~~~~~~~~~~~

Se puede testear que las instalaciones funcionaron corriendo en la consola IPython (es decir en un Jupyter notebook).

.. code:: python

     import numpy as np
     import pandas as pd
     import geopandas as gpd
     import scipy
     import shapely
     import matplotlib.pyplot as plt
     import pysal

Si no recibis errores, deberia estar funcionando!

Install Python + packages en Linux / Mac
----------------------------------------

Esto funciona en Ubuntu 16.04 y posiblemente tambien en Mac.

**Instalar Anaconda 3 y agregarlo al system path**

.. code::

    # Download and install Anaconda
    sudo wget https://repo.continuum.io/archive/Anaconda3-4.1.1-Linux-x86_64.sh
    sudo bash Anaconda3-4.1.1-Linux-x86_64.sh

    # Add Anaconda installation permanently to PATH variable
    nano ~/.bashrc

    # Add following line at the end of the file and save (EDIT ACCORDING YOUR INSTALLATION PATH)
    export PATH=$PATH:/PATH_TO_ANACONDA/anaconda3/bin:/PATH_TO_ANACONDA/anaconda3/lib/python3.5/site-packages

**Instalar packages de Python**

Instalar packages con conda (y pip) corriendo en una terminal los siguientes comandos (el orden importa):

.. code::

    # Install numpy (v 1.13.1)
    conda install numpy

    # Install pandas (v 0.20.3) --> bundled with python-dateutil (v 2.6.1) and pytz (v 2017.2)
    conda install pandas

    # Install scipy (v 0.19.1)
    conda install scipy

    # Install matplotlib (v 2.0.2) --> bundled with cycler, freetype, icu, jpeg, libpng, pyqt, qt, sip, sqlite, tornado, zlib
    conda install matplotlib

    # Install Geopandas (v 0.3.0) --> bundled with click, click-plugins, cligj, curl, descartes, expat, fiona, freexl, gdal, geos, hdf4, hdf5, kealib, krb5, libiconv, libnetcdf, libpq, libspatialindex, libspatialite, libtiff, libxml2, munch, openjpeg, pcre, proj4, psycopg2, pyproj, pysal, rtree, shapely, sqlalchemy, xerces-c
    conda install -c conda-forge geopandas

Si no funcionara alguna instalacion
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Si llegara a faltar algun otro package, ya te imaginas como instalarlo, igualmente lo mas recomendable **buscar en Google** como hacer cualquier cosa que necesites.

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

