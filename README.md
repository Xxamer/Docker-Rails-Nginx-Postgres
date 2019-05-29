# NGINX + RAILS PRODUCTION + PUMA + DOCKER
## 1. Pasos Iniciales

En la aplicación de Rails vamos al Gemfile.lock y eliminamos en la parte final del archivo las líneas de `BUNDLE WITH {Versión del Bundler }.`

![](https://i.imgur.com/iLvqjve.png)

Añadimos los ficheros de docker que crearán un contenedor de postgresql, uno de Rails con nuestra aplicación en modo producción y uno de Nginx que  redigirá las peticiones al servidor de rails por el puerto 80 por defecto o el que nosotros designemos.

La ruta por defecto es:
`docker-compose.yml` en la raíz del proyecto de Rails
Una carpeta docker con dos carpetas “`web`” y “`app`” en su interior, app para la configuración de la aplicación de Rails y Web para Nginx.

Una vez hecho esto podemos configurar los ficheros primero o mover la aplicación al servidor y editarlo mediante ftp.

| ![Imgur ](https://i.imgur.com/1Oa1pRF.png ) | ![Imgur ](https://i.imgur.com/L890aHO.png ) ||||
| -------------------------------------------| -------------------------------------------| --- | --- | --- |

## 2. Edición de los archivos de Docker
### Docker-compose.yml

En el fichero ‘docker-compose.yml’ definimos el nombre de los contenedores y las dependencias de estos, tambien en el contenedor de nginx en “ports” definimos el puerto por el que vamos a poder conectarnos de manera externa.

Nota: Los nombres de los contenedores y el puerto de salida deben ser distintos para poder tener múltiples servidores en paralelo.

![Imgur](https://i.imgur.com/uOVuAMg.png)

### app/dockerfile

En este fichero definimos la instalación de Ruby con la que hemos creado el proyecto de Rails, en mi caso es la versión 2.5.1. La versión Alpine significa que es una versión “light” para reducir el tamaño del contenedor.

Luego instalamos las dependencias necesarias para arrancar el proyecto de Rails y el bundler para poder instalar las gemas. 

Creamos una variable llamada “RAILS_ROOT” en donde definiremos la ruta donde está alojado el proyecto de Rails 

Luego se define el entorno de la aplicación como en producción y añadimos las gemas al directorio de trabajo y ejecutamos el bundle para poner en marcha el servidor puma.

![Imgur](https://i.imgur.com/dYqqRY3.png)

#### Nota: Si la aplicación de Rails está desarrollada en modo api deberemos comentar la línea de ‘RUN bundle exec rake assets:precompile’

### web/dockerfile

Aquí descargamos la imagen de nginx e instalamos las actualizaciones y herramientas necesarias para ponerlo en marcha, luego establecemos la ruta donde Nginx va a buscar ficheros y guardar las configuraciones y demás.

![Imgur](https://i.imgur.com/24kLVid.png)

### web/nginx.conf

Al principio del archivo definimos cual es el servidor de Rails, poniendo el nombre del contenedor y el puerto que por defecto es 3000.

Luego definimos cual va a ser el server, en mi caso es la ip del servidor pero si tenemos un DNS sería el nombre (p.j www.example.com)

Denegamos las peticiones a archivos a los que que no se debería poder acceder, como los logs y los ficheros con extensión ‘.rb’

Para el resto habilitamos el CORS para poder acceder desde cualquier dirección exterior.

Definimos los tiempos de espera por si el servidor tiene mucha carga o tiene que resolver muchos datos

![Imgur](https://i.imgur.com/dFiRS4v.png)

Una vez hecho esto, desde el servidor nos vamos a la carpeta donde está el proyecto de Rails y los ficheros de Docker y ahí ejecutamos "docker-compose build" para comenzar la descarga y creación del as imágenes. Cuando estén descargadas ejecutamos "docker-compose up" para iniciar las imágenes de Docker y ya tendremos conexión a la aplicación de Rails a través de nginx.

![Imgur](https://i.imgur.com/oPe7NV6.png)



## Conexión de la aplicación Rails con la base de datos

Por defecto la imagen de Postgres trae el usuario “postgres” con contraseña vacía así que lo dejaremos así a no ser que definamos otro usuario en el Docker-compose, en el “host” especificaremos el nombre del container

![Imgur](https://i.imgur.com/Un8LrJI.png)



## Posibles errores

En caso de que la aplicación de Rails no se encuentre disponible, el servidor de Nginx mandará un error “502”

![Imgur](https://i.imgur.com/5FGmgHC.png)


## Error: Missing ‘secret key base’ for ‘production’ environment
Este error se produce al iniciar una aplicación en Rails en modo producción y no encontrar la “secret key”, para arreglarlo se usa el comando `rails rake secret`, la string que nos devuelva la pondremos en el fichero config/application.rb en un campo denominado `config.secret_key_base= string de rake secret`

![Imgur](https://i.imgur.com/GTUl0uD.png)

![Imgur](https://i.imgur.com/aeDaE5N.png)