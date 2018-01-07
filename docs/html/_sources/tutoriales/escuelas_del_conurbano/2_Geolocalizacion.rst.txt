Geolocalizacion
===============

Muchas veces nos va a interesar relacionar un dato como por ejemplo el nombre de un edificio, una direccion, u algun monumento con un objeto en una base de datos, especialmente si esta base de datos tiene coordenadas geograficas.
Dicho de otro modo, una cosa es saber el nombre de un lugar y su direccion, y tenerlo anotado. Otra cosa (por lo general, mejor) es conocer sus coordenadas geograficas, ya que pasamos de una frase que debe ser interpretada (la direccion) a un numero que describe una posicion univocamente y puede ser usado para cualquier calculo de ser necesario.

Este pasaje se llama **geolocalizacion** y en general, no se reduce solo a encontrar las coordenadas geograficas para una 'frase' que uno pregunte, sino que relaciona esta frase con un objeto en la base de datos de consulta y todos sus atributos. Es decir, nos puede ademas brindar otra informacion acerca del objeto por el cual preguntamos, por ejemplo que tipo de edificio se ubica en el lugar y que funciones cumple, cual es el distrito, provincia o pais correspondiente, su codigo postal, entre otras informaciones.

Hay varios servicios de geolocalizacion. Lo mas comodo para mi es usar la **API de Google Maps**, que viene a ser un canal de comunicacion con Google Maps de forma que obtenemos resultados de busqueda pero sin necesidad de hacer trabajo manual. 

Para los que deseen, pueden ir directamente a la pagina en donde se dan las instrucciones <https://developers.google.com/maps/documentation/geocoding/start> de uso. Es muy facil, en la parte inferior de dicha pagina hay un boton con la opcion de generar una API key. Esta es una clave que se pide cada vez que hacemos una consulta. Una vez que entregan la API key, ya podemos empezar a consultar. Por ejemplo, en la barra de busqueda se puede escribir:

https://maps.googleapis.com/maps/api/geocode/json?address=Instituto+Santa+Maria+de+los+Angeles,+Coghlan

y vas ver que viene dada una respuesta con informacion en formato json. Mas adelante vamos a ver como consultar todas las direcciones que podemos tener en una de las columnas de nuestra table de datos. Ahi es cuando se empieza a poner bueno.

Antes de continuar, queria comentar algunos de los inconvenientes que hay con este metodo. 

Como siempre, es posible que la direccion que busquemos no se encuentre. En esta situacion hay varias opciones que considerar. Basicamente hay que averiguar porque exactamente no se encontro la direccion, y tambien evaluar que tan importante es en el marco de lo que estamos haciendo. 

Si no hace la diferencia, podemos 'tirar' las direcciones que no hayan dado resultado y seguir con el analisis. Si queremos encontrar la informacion del dato de alguna forma, lo primero que hay que chequear es que no haya errores de tipeo. Luego, muchas veces si tenemos direcciones que son largas, puede ser que algunas de las palabras confundan al buscador. Puede ser mejor evitar informacion extra del tipo 'Departamento de Quimica Organica' o 'Barrio el Jaguel'. En el primer caso porque los datos son de edificios enteros, y en general incluir nombres de divisiones institucionales confunde mas de lo que aporta. En el segundo caso, puede tratarse de nombres populares de zonas, pero por ejemplo en el caso de Buenos Aires, la direccion ya es unica a nivel partido, es decir que barrios o localidades en realidad no hacen la diferencia.
