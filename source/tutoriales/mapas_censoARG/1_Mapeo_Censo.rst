# Cómo crear mapas interactivos

En este tutorial usamos un poco de python para crear paginas muy lindas donde podemos visualizar datos que tengamos en mapas. De paso cargamos los datos del Censo argentino 2010 que de cualquier forma son interesantes y vale la pena tener. [CLICK PARA VER DE QUE SE TRATA!](http://matiasdice.com/maps/hogares.html "ejemplo mapas interactivos") 

#### Los códigos los encontrás en [GitHub](https://github.com/matuteiglesias/mapas_censoARG "repo GitHub") 

Los mapas interactivos tienen dos partes fundamentales: la web (lo que tiene que ver con crear una pagina que responda interactivamente) y los datos, o sea la informacion que hace a los objetos del mapa, ya sea la info geometrica o el resto de los atributos. En este notebook les muestro uno de los caminos posibles para procesar la informacion que sirve de fuente a los mapas.

Las librerias que vamos a usar para procesar los datos son pandas y geopandas. Al momento de subir la informacion a las herramientas de mapeo web, vamos a usar los modulos os y json, ademas del Uploader de mapbox.

### Datos del censo ARG 2010

La informacion con las respuestas a las preguntas por radio censal la tenemos de los [archivos csv](http://blog.jazzido.com/2014/02/24/resultados-censo-2010-radio-censal/ "datos censo") que el Sr. Aristaran ha puesto a disposicion scrapeando el sistema REDATAM. Estrictamente es una fuente no'oficial, aunque no existe (corrijanme si me equivoco) fuentes oficiales para las tablas con los resultados del Censo argentino 2010 al nivel de radio censal. 

El censo en su cuestionario básico tiene preguntas sobre la calidad de vivienda y necesidades básicas insatisfechas, niveles educativos, la calidad de conexión a servicios como el gas o agua de red, o el uso de garrafa o leña, los desagues a cloacas, entre otras cosas. 

### ¿Y como se podrán mapear?

Una posibilidad que tenemos con los mapas es colorear para tener una impresion directa de lo que dicen los numeros. Sin embargo, si queremos colorear segun la respuesta a cada una de las preguntas del censo, tenemos que hacer mas de 20 mapas separados! Una forma de implementar esto seria incluir todas las capas en una sola aplicacion y habilitar una opción para elegir según cuál cpaa queremos colorear. Si quisieramos ver la informacion, tendriamos que imprimir todas las respuestas al censo al clicker en un radio censal. Esto podría funcionar, aunque corremos el riesgo de estar sobrecargando la experiencia con tanta información. Hay que tener en cuenta que algunas variables se cuentan a nivel de persona, otras a nivel de hogares o de vivienda. Además, algunas son respondidas sólo por mayores de cierta edad, o sólo cuando hayan respondido afirmativamente en una pregunta anterior (por ej. si no tenés baño, no te voy a preguntar si tiene cadena). sumado a esto, a veces las preguntas no tienen mucho que ver unas con otras, con lo cual termina quedando un cambalache de información que quizas abruma. 

Lo que me pareció mas piola de hacer es agrupar la variables en temáticas que las une y usar un solo mapa de colores sintetizando a todas ellas. En este caso vamos a hacer el mapa de las variables de EDUCACION y lo vamos a colorear con un indice que basicamente pondera qué fracción de la población llegó al nivel primario, secundario, etc.

En la carpeta 'Preguntas' esta de todos modos toda la info, con la cual uds pueden elegir la parte del censo que les guste. Al inicio tenemos un diccionario con  los nombres de los archivos y los nombres que he puesto para identificar a las preguntas.

.. ipython:: python

    from IPython.display import display, HTML

    pregs_dict = {'names': ['PERSONA-CONDACT', 'PERSONA-P01', 'PERSONA-P02', 'PERSONA-P05',
			     ...
                           'VIVIENDA-V01', 'VIVIENDA-V02'],
                  'prefix': ['Actividad ', 'Relacion con jefe ', 'Sexo ', 'Nacionalidad ', 
			     ...
                            'Tipo de vivienda particular ', 'Ocupacion ']}

Se necesita la informacion geometrica de cada radio censal, cada departamento provincial o cada provincia, para poder dibujarlos en el mapa. Es algo sencillo, se trata de polígonos que demarcan los distritos. Cada distrito tiene ademas una serie de variables asociadas. Por ejemplo, pueden tener un nombre, una provincia en donde se encuentran, pueden tener una cantidad de poblacion mayor de 18 años, etc. Geopandas es la libreria que sirve para manejar esta doble cara de la informacion geografica. Los objetos  geopandas.GeoDataFrame son como un pandas.DataFrame, es decir una tabla común y corriente, pero admiten una columna llamada 'geometry' para asociar info geográfica, ya sea coordenadas de un lugar, o un polígono delimitante.

Hay distintos tipos de archivos que sirven para este tipo de informacion. Acá vamos a usar dos: 'shapefiles' y 'geojson'. Por ahora nos alcanza con el primero de ellos. Los archivos shapefiles vienen en grupo, en general podemos bajarlos sencillamente de internet cuando se trata de informacion pública (por ejemplo limites administrativos). Muchas veces tienen sus pequeños truquitos, errores y bugs. A veces es casi inevitable, en el caso de Argentina, por ejemplo, hay mas de 50000 radios censales. Es casi inevitable que alguno tenga un error o se haya perdido. En general es responsabilidad del Instituto Geográfico, del INDEC o de las oficinas de datos y tecnología mantenerlos, y para ser buenos, en lineas generales los archivos estan en buen estado. Otros truquitos que nos pueden dar algun dolor de cabeza tienen que ver con el encoding del archivo. Esto es, el reconocimiento de caracteres especiales como las tildes o la Ñ. Para cumplir con todo eso, es que terminamos importando los archivos asi:

.. ipython:: python

    radios_gdf = gpd.GeoDataFrame.from_file('./radios_censales/radios_w_geometry.shp').rename(columns = {'LINK': 'radio'})
    deptos_gdf = gpd.GeoDataFrame.from_file('./departamentos/pxdptodatosok.shp', driver = 'ESRI Shapefile')

### Identificar radios censales a sus departamentos y provincias

Algo que es clave y que vamos a querer tener siempre bien ordenado es una columna que nos va a permitir identificar a los radios censales (o a los deparmantos). Se trata de un códico, que hoy por hoy en Argentina esta bastante unificado, por el cual podemos identificar los lugares con un número, evitando confusiones, pero no sólo eso, las primeras cifras de los codigos corresponden a la unidad mayor en jerarquía, de forma que agregar las variables por departamento o por provincia a partir de radios censales se vuelve muy facil. Por eso tenemos la columna 'ĺink' que es basicamente el codigo de departamento. Prestar atencion a que sea de mismo tipo ('string') en todos los datasets, sino los mergings van a fallar.

.. ipython:: python

    radios_gdf['link'] = radios_gdf['radio'].astype(str).str[:5]

### Integrar la info de preguntas

Bueno ahora si nos ponemos en acción. En la notebook uso un tipo de loop que me gusta mucho, se trata simplemente de armar una lista de dataframes que podemos concatenar al final. Cada vuelta de loop es para una pregunta. Se carga el csv, se le modifica como a nosotros nos interese. En este caso lo que hago es tomar porcentajes y agrupar lo que es EGB y polimodal en primaria y secundaria. 

.. ipython:: python

    for name in ['PERSONA-P07', 'PERSONA-P08', 'PERSONA-P09', 'PERSONA-P10', 'PERSONA-P12']:

	# Cargo el csv de preguntas, acomodo el cero a la izquierda para que los codigos sean idénticos en todas las tablas.
    
        df_pregunta = pd.read_csv("./../Datos_censo/Preguntas/merged/"+name+".csv", encoding = 'utf-8')
        df_pregunta['radio'] = df_pregunta['radio'].astype(str).str.zfill(9)

	# Elijo las ultimas columnas, que son las que contienen respuesta a las preguntas.
        info = df_pregunta.iloc[:, 5:].set_index('radio')
        info = info.add_prefix(name[-3:]+'_')

	# Para agregar este df que hicimos por departamentos, tenemos que mergear un df radio-link, agrupar y sumar por link.
        info_dptos = info.reset_index().merge(radios_gdf[['radio','link']]).groupby('link').sum()

	# Tomamos porcentaje y redondeamos para que no se nos llene la tabla de decimales.
        info = pd.concat([info, 100*info.div(info.iloc[:, -1], axis = 0).add_prefix('%_').round(3)], axis = 1)
        info_dptos = pd.concat([info_dptos, 100*info_dptos.div(info_dptos.iloc[:, -1], axis = 0).add_prefix('%_').round(3)], axis = 1)

El resultado son varias tablas del tipo: 

.. image:: ../info_radios.png


##### Listo, ya tenemos la data que vamos a querer mostrar en el mapa. Mergeamos la info geometrica:

.. ipython:: python

    radios_info_gdf = gpd.GeoDataFrame(info_df.reset_index().merge(radios_gdf[['radio', 'geometry']]))
    deptos_info_gdf = gpd.GeoDataFrame(info_dptos_df.reset_index().merge(deptos_gdf[['link', 'departamen', 'provincia', 'geometry']]))

### Ahora a Mapbox

Y ahora vamos a subirlo a mapbox. La funcion save_geojson la van a encontrar definida en el notebook. Lo que hace es muy simple, guarda la informacion de un GeoDataFrame como un archivo json, que aunque suene raro no es mas que un archivo de texto como este que escribe cada fila de nuestro dataframe como un objeto entre llaves ({}). Se llama geojson porque tiene un formato muy especifico, un FeatureCollection, que como su numbre lo sugiere es una colección de features que pueden ser polígonos, puntos o líneas con informacion adjunta. 

.. ipython:: python

    save_geojson(deptos_info_gdf, 'deptos_info.geojson')
    save_geojson(radios_info_gdf, 'radios_info.geojson')

En si no nos tenemos que hacer mucho problema por el geojson. Lo que si les aviso es que es muy probable pifiar con algun detalle minimo, como que nos falte una coma, o que tengamos una palabra con tilde sin en encoding correcto. Estos detalles nimios nos pueden frenar todo el trabajo hasta que los logramos corregir. Lo mejor es googlear los errores, revisar los archivos con paciencia hasta ver cuál es el detalle que no nos esta dejando hacer las cosas normalmente. Atender que python acepta comillas simples miestras que para json tienen que ser todas dobles.

Ya tenemos un archivo geojson. Estamos como queremos!
Vamos a usar Mapbox que es un servicio que nos permite crear mapas. Ellos se van a encargar de darnos un mapa background que nos guste (satelite o neutral) nos va a permitir elegir los ingredientes que queremos que se vean en el mapa. Caminos, relieve, terrenos, lo que nos guste. Esto es lo que se llaman los 'estilos'. Tenemos la posibilidad de elegir estilos creados por otros usuarios, algunos de los cuales son muy lindos en lo estetico. Con lo cual ya de entrada podemos tener un buen mapa para mostrar. Ahora, obvio, nos faltan nuestros datos. Los tenemos como archivo geojson que creamos en el paso anterior, y la funcion que sigue se va a encargar de que en nuestro Mapbox studio nos aparezcan los datos mapeados.

.. ipython:: python

    def upload_file(data, name, username = 'matuteiglesias', token = 'TU_TOKEN'):
        # Dump into file for upload
        with open('./upload_data.geojson', 'w') as outfile:
            json.dump(data, outfile)

        service = Uploader(access_token=token)
        with open('./upload_data.geojson', 'r') as src:
            # Acquisition of credentials, staging of data, and upload
            # finalization is done by a single method in the Python SDK.
            upload_resp = service.upload(src, username+'.'+name)


En las lineas que siguen, lo que hacemos es armar una lista con los nombres de los geojson que queremos subir y les aplicamos la funcion upload_file.
Paso siguiente, ya podemos ir al Mapbox studio y usar la interfaz de puro botoncitos y opciones para crear nuestro mapa. Convertirlo o embeberlo en una pagina web va a requerir que usemos un poquito de html o front end. Hasta aca lo que hicimos fue procesar los datos y pasarlos al servicio que nos permite mapear.

.. ipython:: python

    files = os.listdir('./geojson/')
    files = ['provs_info.geojson', 'deptos_info.geojson', 'radios_info.geojson']
    names = [name.split('.')[0] for name in files]
    
    for i in range(len(files)):
        print names[i]
        data = json.load(open('./geojson/'+files[i]))
    
        try:
            upload_file(data, names[i])
        except:
            pass

Desde Mapbox ya con copiar un link tenemos un mapa en una pagina que podemos compartir. 

te sirvió este tutorial? Te funciona? Tenes dudas? Animate a editar esta pagina y proponer cambios o corregir errores desde los repos de GitHub.




