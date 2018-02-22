# Step 3: Integración de contendores

Como el proyecto NodeJs está listo para ser desplegado sobre contenedores y esta detallado en un Dockerfile con sus procesos de compilación en etapas distintas, solo falta habilitar los microservicios con los que se integra esos son MySQL y Keycloak. Ambos servicios mencionados están contenerizados por las comunidades y/o corporaciones que desarrollan su tecnología y están publicados en repositorios que son públicos en [Docker.io](https://hub.docker.com).

* [Imagen MySQL Oficial](https://hub.docker.com/_/mysql/).
* [Imagen Keycloak](https://hub.docker.com/r/jboss/keycloak/).

Las versiones utilizadas durante las actividades son MySQL 5.7 y Keycloak 3.3.0.Final, las versiones soportados no están definidas.

El proceso de integración entre contenedores es simple, ya que los contenedores están orientados a habitar servicios cuya comunicación, conexión o consumo es dado a través de un protocolo de red, lo que no significa que crear una tarea programada u otro proceso en background sobre contenedores sea imposible. En el caso de este tutorial estamos tratando con una aplicación API REST que almacena datos en una base de datos MySQL y realiza sus procesos de autenticación y autorización a través de un Sigle Sign On Keycloak, dada esta arquitectura para desplegarlo sobre contenedores lo recomendado es separar los servicio en distintos ambientes para su funcionamiento aislado y que en ningún momento estorbaría sobre el ambiente de otro servicio, la separación recomendada bajo esta arquitectura es agrupar funcionalidades y definirlas como un microservicio de esta forma: 

* Servicios de autenticación y autorización.
* Servicios de almacenamiento.
* Servicios REST de integración.

Por lo tanto, para que mi API REST funcione destinaré un contenedor para que la porte, uno para la base de datos MySQL y uno para Keycloak.

### Ejercicio: integrar API REST, MySQL y Keycloak sobre contenedores

**Observación**
Para este ejecicio es necesario disponer de la imagen **poc-docker/api** creada en el [step anterior](../step-2-multistage-build/README.md).

Para partir se crea una base de datos MySQL contenerizada, la imagen a utilizar es la oficial de MySQL que presenta una serie de variables de entorno que se pueden declarar para que al levantar el contenedor se apliquen cambios como crear una base de datos, un usuario con su contraseña, setear la contraseña del usuario administrador, etc.. La combinación de variables de entorno seleccionadas para crear el contenedor MySQL son las siguientes:

* **MYSQL_RANDOM_ROOT_PASSWORD:** auto genera la contraseña del usuario de administrador de la base de datos MySQL. Es opcional y si es utilizada debe tener el valor **yes**.
* **MYSQL_DATABASE:** nombre de base de datos que se creará al iniciar el contenedor. Es opcional.
* **MYSQL_USER:** nombre de usuario que se creará al iniciar el contenedor. Es opcional, pero si se declara, el usuario creado es dueño de la base de datos que se crea al utilizar la variable MYSQL_DATABASE.
* **MYSQL_PASSWORD:** contraseña del usuario creado a través de la variable MYSQL_USER, por ende depende de esa variable para que tenga efecto.

Esta imagen dispone más variables de entorno que permiten trabajar otras configuraciones que no aplican para esta actividad, para tener más información de la imagen oficial verificar su [repositorio público](https://hub.docker.com/_/mysql/).

Entonces, considerando las variables de entorno mencionadas anteriormente, el comando utilizado para generar el contenedor MySQL es el siguiente:

```bash
$ docker run -d --name mysql-api -e MYSQL_RANDOM_ROOT_PASSWORD=true -e MYSQL_DATABASE=happyfoot_db -e MYSQL_USER=happyfoot -e MYSQL_PASSWORD=happyfoot mysql:5.7
## Resultado esperado es el id único del nuevo contenedor:
# 2988dc7ee5e7c90fb5b53e31a78f7f68e67bc09e2dc65b25a495ebb45450ab9f
$ docker ps
## Resultado esperado:
# CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
# 2988dc7ee5e7        mysql               "docker-entrypoint.s…"   50 seconds ago      Up 50 seconds       3306/tcp            mysql-api
```

Para probar que la base de datos MySQL quedó lista y funcional se ejecuta lo siguiente:

```bash
$ docker exec mysql-api mysqladmin ping -uhappyfoot -phappyfoot
## Resultado esperado:
# mysqladmin: [Warning] Using a password on the command line interface can be insecure.
# mysqld is alive
$ docker exec mysql-api mysql -uhappyfoot -phappyfoot -e"show databases;" | grep happyfoot
## Resultado esperado:
# mysql: [Warning] Using a password on the command line interface can be insecure.
# happyfoot_db
```

Lo que sigue es crear el contenedor con Keycloak, para ello se utiliza la imagen del repositorio **jboss**. La imagen Keycloak, al igual que la de MySQL, tambien dispone de una serie de variables de entorno que permiten generar configuraciones iniciales sobre el servicio, en este caso se utiliza las siguientes:

* **KEYCLOAK_USER:** usuario administrador de la plataforma Keycloak.
* **KEYCLOAK_PASSWORD:** contraseña del usuario administrador de la plataforma Keycloak.
* **DB_VENDOR:** proveedor de base de datos que va a utilizar Keycloak.
* **MYSQL_ADDR:** hostname o FQDN del servicio de base de datos MySQL. Aplica solo cuando la variable DB_VENDOR sea igual a MYSQL.
* **MYSQL_PORT:** puerto de escubha del servicio de base de datos MySQL. Aplica solo cuando la variable DB_VENDOR sea igual a MYSQL.
* **MYSQL_DATABASE:** nombre de la base de datos que utilizará Keycloak. Aplica solo cuando la variable DB_VENDOR sea igual a MYSQL.
* **MYSQL_USERNAME:** usuario de base de datos que utilizará Keycloak para conectarse. Aplica solo cuando la variable DB_VENDOR sea igual a MYSQL.
* **MYSQL_PASSWORD:** contraseña de acceso a base de datos que utilizará Keycloak para conectarse. Aplica solo cuando la variable DB_VENDOR sea igual a MYSQL.

Para más detalles de otras variables de entorno verificar el [repositorio público](https://hub.docker.com/r/jboss/keycloak/). Como se puede interpretar, Keycloak también puede estar integrado con una base de datos MySQL (O también PostgreSQL) para así persistir las configuraciones, eso se logra utilizando la variable de entorno DB_VENDOR que soporta 3 valores: h2 (Valor por defecto), mysql o postgres. Para efecto de esta actividad se crea otro contenedor MySQL para entregar servicio de persistencia al Keycloak, se logra ejecutando el siguiente comando:

```bash
$ docker run -d --name mysql-keycloak -e MYSQL_RANDOM_ROOT_PASSWORD=true -e MYSQL_DATABASE=keycloak_db -e MYSQL_USER=keycloak -e MYSQL_PASSWORD=keycloak mysql:5.7
## Resultado esperado es el id único del nuevo contenedor:
# 20eef4740902fdbe8ebc62ee8bcc5b2800283251f601413d67d7ca64aabce2c7
$ docker ps
## Resultado esperado:
# CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
# 20eef4740902        mysql:5.7           "docker-entrypoint.s…"   39 seconds ago      Up 38 seconds       3306/tcp            mysql-keycloak
# 2988dc7ee5e7        mysql               "docker-entrypoint.s…"   About an hour ago   Up About an hour    3306/tcp            mysql-api
```

Para validar que haya quedado bien el nuevo contenedor solo se debe repetir los pasos de validación del anterior pero con los datos del nuevo contenedor. Una vez ya esté funcionando el contenedor de base de datos se procede a crear el de Keycloak:

```bash
$ docker run -d --name keycloak -e KEYCLOAK_USER=admin -e KEYCLOAK_PASSWORD=admin -e DB_VENDOR=MYSQL -e MYSQL_ADDR=mysql -e MYSQL_PORT=3306 -e MYSQL_DATABASE=keycloak_db -e MYSQL_USER=keycloak -e MYSQL_PASSWORD=keycloak --link mysql-keycloak:mysql keycloak:3.3.0.Final
## Resultado esperado es el id único del nuevo contenedor:
# 20eef4740902fdbe8ebc62ee8bcc5b2800283251f601413d67d7ca64aabce2c7
$ docker ps
## Resultado esperado:
# CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS              PORTS               NAMES
# f9d28321b9ba        jboss/keycloak:3.3.0.Final   "/opt/jboss/docker-e…"   34 minutes ago      Up 34 minutes       8080/tcp            keycloak
# 20eef4740902        mysql:5.7                    "docker-entrypoint.s…"   43 minutes ago      Up 43 minutes       3306/tcp            mysql-keycloak
# 2988dc7ee5e7        mysql                        "docker-entrypoint.s…"   2 hours ago         Up 2 hours          3306/tcp            mysql-api
```

Para validar que el servicio Keycloak está funcionando de forma correcta ver los logs del contenedor:

```bash
$ docker logs keycloak
## Resultado esperado:
# ...
# 18:27:41,422 INFO  [org.wildfly.extension.undertow] (ServerService Thread Pool -- 50) WFLYUT0021: Registered web context: '/auth' for server 'default-server'
# 18:27:41,435 INFO  [org.jboss.as.server] (ServerService Thread Pool -- 49) WFLYSRV0010: Deployed "keycloak-server.war" (runtime-name : "keycloak-server.war")
# 18:27:41,476 INFO  [org.jboss.as.server] (Controller Boot Thread) WFLYSRV0212: Resuming server
# 18:27:41,478 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0060: Http management interface listening on http://127.0.0.1:9990/management
# 18:27:41,478 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0051: Admin console listening on http://127.0.0.1:9990
# 18:27:41,479 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0025: Keycloak 3.3.0.Final (WildFly Core 3.0.8.Final) started in 32098ms - Started 535 of 857 services (570 services are lazy, passive or on-demand) 
```

E ingresar a la consola de administración con el usuario configurado en base a veriables de entorno, para ello ver la IP que obtuvo el contenedor Keycloak:

```bash
$ docker inspect keycloak -f "{{ .NetworkSettings.IPAddress }}"
## Resultado esperado:
# 172.17.0.4
```

La función inspect de docker permite obtener la descripción completa de un componente a través de un documento json, si se declara el parametro **-f** o **--format** es posible definir especificamente que dato o valor se desea utilizando un template Go. Los componentes que se pueden inspeccionar son contenedores, imagenes, volumenes y redes.

Con la IP obtenida ingresar desde un explorador al puerto **8080** y al contexto **/auth** para autenticarse con las credenciales de administrador admin/admin (Credenciales definidas por variables de entorno al crear el contenedor). Con los servicios MySQL y Keycloak funcionando ya se puede proceder a ejecutar la creción del contenedor con el API utilizando la imagen generada en los anteriores steps, pero esta vez considerando el link entre contenedores:

```bash
$ docker run -d --name api --link mysql-api:mysql --link keycloak:keycloak poc-docker/api
## Resultado esperado es el id único del nuevo contenedor:
# 4de5cd1f224f97b93a7777318559b1688c2231c06cdf2c3848f033610c36ffff
```

La integración entre contenedores se logra realizando un link entre ellos bajo el estándar cliente-servidor, donde el cliente debe tener configurado el link. Este link altera el contenedor cliente durante su creación agregando una serie de variables de entorno que hacen referencia al contenedor servidor y también agrega a la tabla de hosts del cliente la IP del contenedor servidor con el nombre o fqdn declarado en el link. Para declarar un link en el proceso de creación del contenedor cliente se utiliza el parametro **--link** indicando el ID o nombre del contenedor servidor y, separado por dos puntos (:), el nombre o fqdn que sera utilizado para consumir el contendor servidor.

Luego de haber creado el contenedor API, se revisa el log para ver el resultado:

```bash
$ docker logs api
## Resultado esperado es el id único del nuevo contenedor:
# Tue, 20 Feb 2018 19:12:36 GMT sequelize deprecated String based operators are now deprecated. Please use Symbol based operators for better security, read more at http://docs.sequelizejs.com/manual/tutorial/querying.html#operators at node_modules/sequelize/lib/sequelize.js:242:13
# Declare  auth  route.
# Declare  categorias  route.
# Declare  direcciones  route.
# Declare  profesionales  route.
# Declare  servicios  route.
# Declare  usuarios  route.
# Unhandled rejection SequelizeAccessDeniedError: Access denied for user 'root'@'172.17.0.5' (using password: YES)
#     at Utils.Promise.tap.then.catch.err (/src/node_modules/sequelize/lib/dialects/mysql/connection-manager.js:141:19)
#     at tryCatcher (/src/node_modules/bluebird/js/release/util.js:16:23)
#     at Promise._settlePromiseFromHandler (/src/node_modules/bluebird/js/release/promise.js:512:31)
#     at Promise._settlePromise (/src/node_modules/bluebird/js/release/promise.js:569:18)
#     at Promise._settlePromise0 (/src/node_modules/bluebird/js/release/promise.js:614:10)
#     at Promise._settlePromises (/src/node_modules/bluebird/js/release/promise.js:689:18)
#     at Async._drainQueue (/src/node_modules/bluebird/js/release/async.js:133:16)
#     at Async._drainQueues (/src/node_modules/bluebird/js/release/async.js:143:10)
#     at Immediate.Async.drainQueues [as _onImmediate] (/src/node_modules/bluebird/js/release/async.js:17:14)
#     at runCallback (timers.js:756:18)
#     at tryOnImmediate (timers.js:717:5)
#     at processImmediate [as _immediateCallback] (timers.js:697:5)
# Tue Feb 20 2018 19:12:36 GMT+0000 (UTC): Node server stopped.
```

El contenedor del API iniciará con un error que corresponde a un fallo en integración con la base de datos MySQL destinada para el API, esto es porque el API tiene sus configuraciones de conexión definidas en un archivo json que esta ubicado en una ruta del código fuente. Para resolver este problema hay que cambiar el archivo de configuración para que utilice los datos de conexión que se definieron en la creación del contenedor de MySQL, lo cual nos lleva al próximo step, el [Step 4 - Volumenes y Redes](../step-4-volumes-network/README.md).