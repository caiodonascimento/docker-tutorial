# Step 1: Hacer una imagen base y probarla

Una imagen base considera una o más funcionalidades utiles para ser reutilizadas en otras imagenes, lo que conlleva que por si sola no represente un servicio o una aplicación. Ejemplos de imagenes base son las de sistema operativo, clientes de interfaz de comando específicos, plataformas, entre otros.

Para crear una imagen de contenedor de software con técnologia docker existe dos formas:

1. *A través de Dockerfile*: es el método más popular y el recomendado tanto por la comunidad como por la empresa. Consta de un archivo de texto simple que debe tener el nombre **Dockerfile** *(Sin extensión)*, este archivo contiene las capas o layers de la imagen final a través de los tags permitidos.

2. *Con un contenedor en ejecución*: este método considera un contenedor en ejecución que por cada cambio que tenga, ya sea cambios de configuración, filesystem o de sistema, se generan layers al realizar el commit del contenedor, al finalizar los commits se puede generar una imagen. No se recomienda el uso de este proceso ya que le entrega mayor tamaño a la imagen final debido a que considera todos los archivos de logs generados y hasta los cambios disnecesarios que fueron realizados.

## Dockerfile

El archivo Dockerfile permite definir las capas o layers de una imagen de contenedor, cada layer es un binario identificado por un hashcode encriptado con el protocolo sha256. Este archivo permite establecer una imagen como base, ejecutar comandos sobre esta, copiar archivos, exponer volumenes y puertos, y definir los comandos que se ejecutarán al iniciar el contendor con la imagen que generará.

El Dockerfile si bien es suficiente para la creación de una imagen, también es necesario entender el contexto de compilación de una imagen. El contexto de compilación es la ruta o carpeta donde están los archivos y componentes que son utilizados en el momento de compilación, tales como archivos de configuración, binarios de plataformas, scripts bash, entre otros. Esto se debe a que cuando en nuestro Dockerfile ejecutamos un comando que copia o agrega algo externo a la compilación, lo busca en el contexto de compilación. Si no se indica el archivo que representa el Dockerfile también lo busca en el contexto con el nombre estándar.

### Ejercicio: Hacer una imagen base con cliente git

Para ello es necesario crear un archivo Dockerfile en una ruta en específico:

```bash
$ touch Dockerfile
```

Luego, editar el archivo Dockerfile para agregar la imagen base:

```bash
$ vi Dockerfile
# Contenido del archivo:
#
# FROM alpine
```

Se utiliza el tag **FROM** para definir como imagen base la distribución mínima de linux para contenedores, alpine. La imagen base debe ser el primer TAG del archivo Dockerfile.

Finalmente se cierra la imagen agregando el comando alpine utilizado para instalar git:

```bash
$ vi Dockerfile
# Contenido del archivo:
#
# FROM alpine
#
# RUN apk --no-cache add git
```

Ya esta listo para la compilación, para ello ejecutar el comando:

```bash
$ docker build -t poc-docker/git .
```

Para ejecutar este comando es necesario tener acceso a internet, ya que será necesario bajar la imagen de contenedor alpine y se instalará los paquetes de git en el contendor de compilación para generar una imagen de alpine con git. La imagen resultado de la compilación se llamará: poc-docker/git, que es el valor entregado al parámetro **-t**. El formato del comando de compilación indica el tag o nombre de la imagen resultado a través del parámetro *-t*, y define el contexto de compilación al final que mediante un punto (**.**) representa la ruta donde se desea compilar la imagen. Caso el archivo Dockerfile no tiene el nombre estándar, el anterior comando de compilación no lo encontrará en el contexto de compilación, por ende, en ese caso se puede utilizar el parámetro **-f** para indicar cuál es el archivo Dockerfile que se desea utilizar. En el caso de que el proceso necesite variables de entorno en el build, tales como las necesarias para definir un servidor proxy, la forma de declararlo es a través del tag **--build-arg** que espera recibir una entrada de llave valor; siguiendo el mismo ejemplo del proxy, para definir un servidor proxy para un proceso de compilación agrego las configuraciones de la siguiente forma: 

```bash
$ docker build -t poc-docker/git --build-arg http_proxy=http://ip:prueto --build-arg https_proxy=http://ip:prueto .
```

Para probar nuestra imagen compilada:

```bash
# Listar imagenes locales
$ docker images
# REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
# poc-docker/git      latest              4a91bbe58691        13 hours ago        18.9MB

# Probar de forma interactiva el cli de git
$ docker run --rm -it poc-docker/git
/ # git
# usage: git [--version] [--help] [-C <path>] [-c name=value] ...
```

### Posibles errores durante la actividad

#### Error con el comando apk add

El problema está relacionado con el acceso a los repositorios de paquetes de Alpine Linux, por ende si debes utilizar proxy para consumir a recursos públicos debes considerar agregar tu proxy como variable de entorno en el script de compilación:

```bash
$ docker build -t poc-docker/git --build-arg http_proxy=http://ip:prueto --build-arg https_proxy=http://ip:prueto .
```

Caso no funcione debes verificar si el proxy que estás utilizando permite el acceso a los recursos bajo el dominio: **alpinelinux.org**.