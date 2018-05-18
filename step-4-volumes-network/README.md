# Step 4: Volumenes y Redes

Los contenedores utilizan volumenes efímeros para cada instancia que se crea por lo que si un contenedor es eliminado toda la data se pierde. Es de lo más común que ciertos archivos son necesarios persistirlos, tales como archivos de log, de configuración, entre otros, para cubrir esa necesidad de persistir datos utilizados o generados por los contenedores en ejecución es que Docker permite administrar volumenes para vincularlos con un punto de montaje del SO principal que aloja la instancia de contenedor Docker.

## Administración de Volumenes para Contenedores

Lo más importante, antes de ver el cómo se hace, es saber identificar y definir los volumenes que una imagen permite montar. Si es una imagen existente la forma de acceder al listado de volumenes que dispone es a través del comando:

```bash
$ docker inspect <nombre-de-imagen-o-contenedor> --format="{{json .ContainerConfig.Volumes}}" | python -m json.tool
``` 

Ejemplo, inspeccionar la imagen `mysql`:

```bash
$ docker inspect mysql --format="{{json .ContainerConfig.Volumes}}" | python -m json.tool
## Respuesta esperada:
# {
#   "/var/lib/mysql": {}
# }
```

Por otro lado, también hay que saber como generar esa configuración si es el caso de que se está creando una imagen desde un Dockerfile. Para definir uno o más volumenes expuestos para una imagen se utiliza el tag VOLUME en el Dockerfile indicando los directorios o rutas que estarán destinados a ser utilizados como volumenes persistentes una vez se compile y quede generada.

```dockerfile
...
VOLUME /src/config /src/data /src/logs
...
```



### Ejercicio: montar volumenes y habilitar puertos

Para continuar con el proyecto del api, ahora es necesario persistir 