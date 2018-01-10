
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

