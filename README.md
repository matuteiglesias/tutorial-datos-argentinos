## tutorial-datos-argentinos

Este es el repositorio asociado a la pagina ([tutoriales con datos argentinos] (https://matuteiglesias.github.io/tutorial-datos-argentinos/index.html)). Aqui se puede escribir el contenido.

#### Lineamientos

En este repositorio la comunidad puede acumular:

 - Material para el aprendizaje colectivo sobre herramientas tecnicas y teoricas de procesamiento de datos.
 - Fuentes y datasets economicos, sociales y culturales argentinos.
 - Links a ideas y puntos de vista sobre la importancia y los desafios de la recopilacion y estudio de datos para el desarrollo del pais.

### ¿Como contribuir?

#### Aporta tu contenido al sitio web

Tutoriales, datasets, links y otros materiales se pueden anadir facilmente escribiendo texto (como .rst) e integrandolo al repositorio principal a traves de GitHub.

Sphinx usa **archivos .rst** ([reStucturedText] (https://en.wikipedia.org/wiki/ReStructuredText)). Por lo tanto, las paginas deben escribirse en tales archivos. **Es fácil, intuitivo y conveniente!**. Es similar a Markdown (casi como escribir en el bloc de notas).
Todos los archivos .rst deben colocarse en la carpeta [/source] (/source) que es el directorio donde Sphinx busca la documentación. **Esos archivos .rst son los que modificas para cambiar la pagina web**.

#### Proponer los cambios a traves de GitHub

Es decir
 - Clonar
 - Hacer cambios
 - Push request
 - Listo. El sitio web se actualizara.

El resto de este readme explica estos pasos en detalle:

###### Clonar una copia en tu computadora

   Primero necesitas un fork local del proyecto. Hace click el el boton "fork" en GitHub. Esto crea una copia del repositorio en tu propia cuanta de GitHub y vas a ver la anotacion de que se trata de un fork debajo del nombre del proyecto. 
   Ahora necesitas una copia local en tu disco. Vas a una carpeta en tu PC y haces (asume git instalado)
   
~~~~
$ git init

$ git clone https://github.com/matuteiglesias/tutorial-datos-argentinos
~~~~

Vas a la carpeta del nuevo proyecto:

~~~~
$ cd tutorial-datos-argentinos
~~~~

Finalmente, en esta etapa, necesitamos configurar un nuevo *remote* que apunte al proyecto original para que pueda hallar cualquier cambio y traerlo a la copia local. Primero veni al repositorio original (que está etiquetado como "Forked from" en la parte superior de la página de GitHub, para que veas la "SSH clone URL" y usarla para crear el nuevo *remote*, que llamaremos *upstream*.

~~~~
$ git remote add upstream https://github.com/matuteiglesias/tutorial-datos-argentinos
~~~~

Ahora tenes dos remotos para este proyecto en el disco:

- *origin* que apunta a tu fork del proyecto. Podes leer y escribir en este remoto.
- *upstream* que apunta al repositorio GitHub del proyecto principal. Solo podes leer desde este control remoto.

###### Hace tus cambios

Arriba encontras info sobre editar y crear tus paginas para anadir. Ademas del contenido, puede haber otros aspectos que necesiten mantencion.

Para empezar, creamos una nueva *branch* desde *master*:

~~~~
$ git checkout master
$ git pull upstream master && git push origin master
$ git checkout branch-de-trabajo
~~~~

En primer lugar, nos aseguramos de que estamos en la rama principal. Luego, el comando *git pull* sincroniza nuestra copia local con el proyecto ascendente y git push lo sincroniza con nuestro proyecto bifurcado GitHub. 

Ahora puedes hacer los cambios que quieras. 

Podes consultar el status con el comando
~~~~
$ git status
~~~~

que siempre es muy util. Nos va a decir cuales son los archivos con cambios. Elegi aquellos que quieras combinar con el proyecto central y agregalos con el comando `add`, o bien anadi todos ellos haciendo:

~~~~
$ git add -A
~~~~
~~~~
$ git commit -m "el commit message describe los cambios que se proponen"
~~~~

###### Push request

Para eso necesitas enviar tu rama al origen y algunos clicks en GitHub.

Para impulsar una nueva sucursal:

$ git push origin branch-de-trabajo

Esto crea la rama (branch) en tu proyecto GitHub. Volve a la pagina web de la bifurcación del proyecto (https://github.com/tu-usuario/tutorial-datos-argentinos) y verás la nueva rama con un botón de "Compare & pull request". Click.

Provee un pequeno titulo para la *pull request* y alguna descripcion opcional. Si esta todo OK, click en "Create pull request" y listo!

###### Listo, se chequean los cambios y se recompila la pagina web.
