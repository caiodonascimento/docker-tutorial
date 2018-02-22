# Step 2: Compilación de imagen multietapa

Como ya se mencionó en el step anterior las imagenes de contenedor se crean a través de la compilación de un archivo Dockerfile, lo interesante es como mejorar el proceso y lograr generar la mejor imagen de contenedor mediante un proceso estándarizado y automono en contempla un solo Dockerfile. Para ello Docker desarrolló la función de compilación multietapa.

Entender como funciona una compilación multietapa de Docker es tan simple como vislumbrar el flujo completo del despliegue que se desea hacer y separarlo en fases que se distinguen por sus requerimientos de software. Un simple ejemplo sería el despliegue de una aplicación Java, un típico "Hello World", que esta versionado en un repositorio Git, si relatamos el proceso se acerca a lo siguiente: "Para empezar es necesario bajar el código fuente del repositorio y para ello necesitamos un cliente Git, luego viene bajar las dependencias y lo que nos obliga a tener Maven instalado, para finalizar necesitamos dejar el binario de la aplicación en un servidor de aplicaciones para finalmente ejecutar este último para así visualizar la aplicación". El flujo relatado se podría hacer con una imagen base que tenga las tres tecnologías que se mencionaron (Git, Maven y un Servidor de Aplicaciones), o puedo utilizar una compilación multietapa donde cada etapa del proceso realiza un paso específico haciendo transferencia del material necesario entre las etapas, para así lograr no tener librerías y archivos disnecesario en la imagen final.

Para empezar a utilizar esta metodología de compilación de imagenes de contenedores solo es necesario un Dockerfile y tener claro las etapas, cada etapa debe utilizar su propria imagen base lo que significa que cada una incia en su tag **"FROM"** y finaliza en el **"FROM"** que sigue o en el cierre del Dockerfile. Cada etapa puede tener un alias que se configura indicando a través del parametro **as** después de definir el nombre de la imagen base, este alias sirve de referencia entre las etapas del proceso de compilación y en el caso de que una etapa no tenga alias la referencia se hace a través de su posición en el flujo partiendo del valor 0. Al hacer referencia a una etapa es posible copiar archivos desde una específica hacia la actual para así continuar el flujo.

Para transferir archivos de una etapa a otra se utiliza el tag **COPY** con el parametro **--from** que espera la referencia a la etapa deseada, para luego indicar el archivo o ruta de origen, ruta cual debe existir en la etapa seleccionada, y finalmente el destino en la etapa actual:

```Dockerfile
FROM ejemplo/git as git

RUN git clone https://github.com/link-de-ejemplo/app-ejemplo /ruta-destino

FROM ejemplo/maven

COPY --from=git /ruta-destino/ ./

RUN mvn package
```
### Ejercicio: desplegar API REST sobre contenedores con compilación multietapa

Se debe crear un nuevo archivo Dockerfile

```bash
$ touch Dockerfile-multistage
```

Empezamos creando el primer stage o etapa

```bash
$ vi Dockerfile-multistage
## Contenido del archivo
# FROM poc-docker/git as git
# ENV GIT_REPO=https://github.com/caiodonascimento/backend-services.git \
#     SOURCE_DIR=source-dir
# RUN git clone $GIT_REPO $SOURCE_DIR
```

Donde como imagen base se utiliza la imagen git creada en el [Step 1: Hacer una imagen base y probarla](../step-1-make-image/README.md) que recibe el alias de **git**, luego se declaran variables de entorno (Las variables de entorno no son persistentes a través de las etapas, para variables globales agregarlas como argumento de compilación). Las variables de entorno GIT\_REPO y SOURCE\_DIR son utilizadas para el proceso de clonación del repositorio donde se ubica el código fuente.

Para esta actividad se toma un proyecto API Rest NodeJs como aplicación que se debe desplegar para su funcionamiento, esta aplicación tiene integración con una base de datos relacional MySql y la instancia de autenticación centralizada Keycloak.

Una vez el archivo Dockerfile contenga la primera etapa procedemos a probar

```bash
$ docker build -t poc-docker/api -f Dockerfile-multistage .
## Resultado esperado:
# Sending build context to Docker daemon  9.728kB
# Step 1/3 : FROM poc-docker/git as git
#  ---> 9b43fe6c56f7
# Step 2/3 : ENV GIT_REPO=https://github.com/caiodonascimento/backend-services.git     SOURCE_DIR=source-dir
#  ---> Running in e8d791e83191
# Removing intermediate container e8d791e83191
#  ---> e18541b9c9bd
# Step 3/3 : RUN git clone $GIT_REPO $SOURCE_DIR
#  ---> Running in d1250f00d67e
# Cloning into 'source-dir'...
# Removing intermediate container d1250f00d67e
#  ---> d9ef2f0ae1c1
# Successfully built d9ef2f0ae1c1
# Successfully tagged poc-docker/api:latest
```

Para probar que la imagen quedó como debería validamos que los archivos del repositorio se descagaron:

```bash
$ docker run --rm poc-docker/api ls source-dir/
## Resultado esperado:
# README.md             generate-entities.sh  routes
# config                migrations            seeders
# data                  models                server.js
# docker-services.sh    package.json          sso
# enums                 realm-export.json
```

El comando anterior permite crear un contenedor de la imagen _poc-docker/api_ y ejecutar el comando bash `ls source-dir/`, el cual, con el parámetros `--rm`, una vez el termine la ejecución, el contenedor creado será eliminado. Para reproducir este proceso debes seguir el formato: docker run [OPCIONES] <nombre-de-imagen>(:<tag>) [COMANDO BASH].

Si no obtuviste el resultado esperado entonces verifica el contenido de tu Dokcerfile. El proximo paso es bajar las dependencias productivas las cuales están administradas por NPM (Node Package Manager), en consecuencia es necesario disponer de una imagen que provea de las librerías NPM. La imagen utilizada para esta actividad es obtenida desde [Docker.io](https://hub.docker.com) la cual responde al nombre [ymedlop/npm-cache-resource](https://hub.docker.com/r/ymedlop/npm-cache-resource/) y permite ejecutar el comando que descarga las dependencias del proyecto en cuestión, para continuar se utiliza la imagen NPM para generar una etapa nueva:

```bash
$ vi Dockerfile-multistage
## Agregar al final del archivo Dockerfile:
# ...
# FROM ymedlop/npm-cache-resource as npm
# ENV SOURCE_DIR=source-dir
# RUN mkdir src && echo "Directorio del otro contenedor: $SOURCE_DIR"
# WORKDIR src
# COPY --from=git /$SOURCE_DIR/ ./
# RUN npm install --production
```

Y se compila con la nueva etapa agregada:

```bash
$ docker build -t poc-docker/api -f Dockerfile-multistage .
## Resultado esperado:
# ...
# Step 4/9 : FROM ymedlop/npm-cache-resource as npm
# latest: Pulling from ymedlop/npm-cache-resource
#  ---> 694ee2e8d9d7
# Step 5/9 : ENV SOURCE_DIR=source-dir
#  ---> Running in 2b8dc1a2e86f
# Removing intermediate container 2b8dc1a2e86f
#  ---> 7390603244de
# Step 6/9 : RUN mkdir src && echo "Directorio del otro contenedor: $SOURCE_DIR"
#  ---> Running in 910ab4903412
# Directorio del otro contenedor: source-dir
# Removing intermediate container 910ab4903412
#  ---> ce683b2e02d5
# Step 7/9 : WORKDIR src
# Removing intermediate container a947d7cef5c7
#  ---> bbfa2914fe8c
# Step 8/9 : COPY --from=git /$SOURCE_DIR/ ./
#  ---> ecc50d4b0490
# Step 9/9 : RUN npm install --production
#  ---> Running in c864f27a2aad
# npm notice created a lockfile as package-lock.json. You should commit this file.
# npm WARN backend-service@1.0.0 No repository field.
# added 261 packages in 18.989s
# Removing intermediate container c864f27a2aad
#  ---> 65c6e766ea3e
# Successfully built 65c6e766ea3e
# Successfully tagged poc-docker/api:latest
```

Una vez terminada la compilación ya podemos probar que el proceso funcionó como se esperaba validando que exista la carpeta `node_modules` con las dependencias del proyecto, para ello ejecutar el siguiente comando:

```bash
$ docker run --rm poc-docker/api ls | grep node_modules
## Resultado esperado:
# node_modules
```

Listo para agregar la etapa final que es desplegar el proyecto sobre su runtime, para este proyecto las librerías necesarias las trae la tecnología NodeJs que permite ejecutar código Javascript del lado del servidor. Para optimizar la imagen final se utiliza una imagen de NodeJs que está basada en el expresión mínima de Linux, o sea una imagen NodeJs con Alpine. Agregar entonces al archivo Dockerfile lo siguiente:

```bash
vi Dockerfile-multistage
## Agregar al final del archivo Dockerfile:
# ...
# FROM node:alpine
# WORKDIR src
# COPY --from=npm /src/ ./
# ENTRYPOINT ["node", "server.js"]
```

Con ello ya tendrás el proceso completo de despliegue del proyecto sobre en el archivo Dockerfile separado por etapas o fases, lo cual al final generará una imagen con el proyecto NodeJs más liviano y unicamente con los componentes necesarios para que funcione, es decir que la imagen final no tendrá archivos de logs u otros generados de las etapas ejecutadas para que el proyecto estuviera listo para su funcionamiento. Finalmente se compila el Dockerfile final y se ejecuta para validar su funcionamiento:

```bash
$ docker build -t poc-docker/api -f Dockerfile-multistage .
## Resultado esperado:
# ...
# Step 10/13 : FROM node:alpine
#  ---> 34a20378e745
# Step 11/13 : WORKDIR src
# Removing intermediate container 2fa19a6b0d88
#  ---> 068d3d959889
# Step 12/13 : COPY --from=npm /src/ ./
#  ---> 4d7437f6378a
# Step 13/13 : ENTRYPOINT ["node", "server.js"]
#  ---> Running in 771892bc6abe
# Removing intermediate container 771892bc6abe
#  ---> 4eaacf905846
# Successfully built 4eaacf905846
# Successfully tagged poc-docker/api:latest
```

Para probarlo ejecutar:

```bash
$ docker run -d --name test-del-api poc-docker/api
## Resultado esperado:
# 5ce2945095cf37d31b1296d63b4d29746a88702d51552a27f3928f02a4a5ab25
```

El comando anterior crea un contenedor en base a la imagen **poc-docker/api**, con los parámetros **-d** se indica a docker que el contenedor debe ejecutar su proceso como servicio en _"background"_, y con el parámetro **--name** se indica el nombre único que tendrá el contendor, nombre con el cual es posible realizar los siguientes comandos:

```bash
# Ver los logs del contenedor
$ docker logs test-del-api
# Parar y eliminar el contenedor
$ docker stop test-del-api; docker rm test-del-api
```

Al ver los logs del contenedor creado anteriormente el resulta esperado es un error con el siguiente mensaje:

```
Mon, 19 Feb 2018 15:32:25 GMT sequelize deprecated String based operators are now deprecated. Please use Symbol based operators for better security, read more at http://docs.sequelizejs.com/manual/tutorial/querying.html#operators at node_modules/sequelize/lib/sequelize.js:242:13
Declare  auth  route.
Declare  categorias  route.
Declare  direcciones  route.
Declare  profesionales  route.
Declare  servicios  route.
Declare  usuarios  route.
Unhandled rejection SequelizeConnectionError: getaddrinfo EAI_AGAIN mysql:3306
    at Utils.Promise.tap.then.catch.err (/src/node_modules/sequelize/lib/dialects/mysql/connection-manager.js:149:19)
    at tryCatcher (/src/node_modules/bluebird/js/release/util.js:16:23)
    at Promise._settlePromiseFromHandler (/src/node_modules/bluebird/js/release/promise.js:512:31)
    at Promise._settlePromise (/src/node_modules/bluebird/js/release/promise.js:569:18)
    at Promise._settlePromise0 (/src/node_modules/bluebird/js/release/promise.js:614:10)
    at Promise._settlePromises (/src/node_modules/bluebird/js/release/promise.js:689:18)
    at Async._drainQueue (/src/node_modules/bluebird/js/release/async.js:133:16)
    at Async._drainQueues (/src/node_modules/bluebird/js/release/async.js:143:10)
    at Immediate.Async.drainQueues [as _onImmediate] (/src/node_modules/bluebird/js/release/async.js:17:14)
    at runCallback (timers.js:756:18)
    at tryOnImmediate (timers.js:717:5)
    at processImmediate [as _immediateCallback] (timers.js:697:5)
```

Lo que revela la dependencia del proyecto a una base de datos MySQL, que como consecuencia dá paso al [próximo step](../step-3-link-containers/README.md).

### Posibles errores durante la actividad

#### Pull access denied: poc-docker/git

Si te ocurre que al compilar tu Dockerfile te muestra el siguiente error:

```
pull access denied for poc-docker/git, repository does not exist or may require 'docker login'
```

Entonces es porque no encontró la imagen **poc-docker/git** localmente ni en los repositorios donde has autenticado, es probable que no hayas realizado la actividad del [Step 1: Hacer una imagen base y probarla](../step-1-make-image/README.md) que genera esta imagen. Para validar que dispones de esta imagen ejecuta:

```bash
$ docker images | grep poc-docker/git
```

Si no te muestra nada debes volver a las actividades del [Step 1: Hacer una imagen base y probarla](../step-1-make-image/README.md), pero si te muestra una imagen entonces ejecuta los siguientes comandos:

```bash
$ docker run --rm -it poc-docker/git git --version
## Resultado esperado:
# git version X.xx.x
```

Si no te presenta la respusta esperada entonces debes eliminar la imagen y volver a compilarla verificando que el proceso no presente errores:

```bash
$ docker rmi poc-docker/git -f
$ cd /path/where/git/Dockerfile/is/
$ docker build -t poc-docker/git -f Dockerfile . # Si debes utilizar proxy no olvides agregarlo como argumento de compilación (--build-arg).
```

Finalmente si el error persiste lo mejor es optar por cambiar la imagen base git por una que ofrezca la comunidad en [Docker.io](https://hub.docker.com).