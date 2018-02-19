# Tutorial Docker
## El futuro esta desplegado sobre contenedores 

**Autor:** Caio Medeiros Pinto do Nascimento.
**Duración:** 6 horas.
**Nivel:** Intermedio.

## Introducción

Los contenidos detallados y las actividades prácticas de este tutorial son inductorias a la técnologia de contenedores de software basado en Docker. Todo los conceptos son explicados en palabras breves para facilitar la asimilación del conjunto completo de componentes que se utiliza para trabajar con contenedores.

Para complementar el conocimiento adquirido mediante este material favor consultar la documentación oficial de Docker.

## Qué es Docker?

Plataforma que permite compilar, administrar y compartir contenedores de software.
Conceptos cruciales:

* **Imagen de Contenedor**: es el molde del contenedor. Define la base, las configuraciones y que se ejecutará al iniciar el contenedor.
* **Registro de imágenes**: permite almacenar imágenes, y puede ser público o privado. El más popular es el oficial de Docker: [Docker.io](https://hub.docker.com).
* **Contenedor de Software**: ambiente aislado para el funcionamiento de una aplicación o servicio.

### Instalar docker

Para efectos de este curso se debe utilizar la versión comunitaria de Docker, igual o superior al release 17.5.x-ce. La utilizada para desarrollar este tutorial fue la versión 17.12.0-ce (build c97c6d6).

Para proceder a instalar Docker en su host favor seguir la [guía oficial del producto](https://docs.docker.com/install/linux/docker-ce/centos/). Si para acceder a internet debes pasar por un servidor proxy, debes seguir los pasos declarados en la esta [guía](https://docs.docker.com/config/daemon/systemd/#httphttps-proxy) para configurar las salidas de Docker a través del proxy.

### Comandos básicos

Estado del servicio docker engine

```bash
$ docker info
```

Autenticarse en un registro de imágenes

```bash
# Por defecto hace login en docker.io (hub.docker.com), si desea autenticarse en otro registro debe indicar la URI de este. Lo mismo aplica para el cierre de sesión
$ docker login
$ docker logout
```

Inspeccionar componente docker

```bash
$ docker inspect <nombre-o-id-componente>
```

#### Para imágenes

Crear imagen desde un archivo Dockerfile

```bash
$ docker build -t <nombre-de-imagen> .
```

Este comando será abordado en detalle más adelante en la actividad [1 - Hacer una imagen base](step-1-make-image/README.md).

Listar imágenes almacenadas de forma local

```bash
$ docker images
```

Buscar imágenes en un registro

```bash
# Por defecto busca en docker.io (hub.docker.com), para buscar en otro registry es necesario configurar el servicio docker engine o declarar en la busqueda <uri-repositorio>/<nombre-imagen>
$ docker search <nombre-imagen>
```

Bajar imágenes en un registro

```bash
# Por defecto busca la imagen en docker.io (hub.docker.com), para bajar desde otro registry es necesario configurar el servicio docker engine o declarar la descarga con <uri-repositorio>/<nombre-imagen>
$ docker pull <nombre-imagen>
```

Renombrar imagen

```bash
$ docker tag <imagen-a-renombrar> <nuevo-nombre>
```

Subir imagen a registro

```bash
# Require estar autenticado en el registro de destino normalmente
$ docker push <nombre-imagen>
```

Eliminar imagen cargada localmente

```bash
# La imagen no puede estar en uso por un contenedor
$ docker rmi <nombre-o-id-imagen>
```

#### Para contenedores

Listar contenedores

```bash
# Lista solo los contenedores activos, al agregar -a se listan también los contenedores que no están en funcionamiento
$ docker ps
```

Ejecutar contenedores

```bash
$ docker start <nombre-o-id-contenedor>
```

Ejecutar contenedores

```bash
$ docker start <nombre-o-id-contenedor>
```

Parar contenedores

```bash
$ docker stop <nombre-o-id-contenedor>
```

Matar contenedores

```bash
$ docker kill <nombre-o-id-contenedor>
```

Eliminar contenedores

```bash
# El contenedor debe estar detenido
$ docker rm <nombre-o-id-contenedor>
```

Crear y ejecutar contenedores

```bash
$ docker run <nombre-imagen>
```

Al iniciar un contenedor con el anterior comando, significa que se ejecutará en primer plano y estará en ejecución mientras la sesión sobre ese contenedor este activa. Para que el contenedor se ejecute como servicio, o sea en segundo plano, se debe agregar el input **-d** en el comando (*el nombre de la imagen de contenedor debe ser siempre el último input*).

## Ejercicios de aprendizaje

Para aprender hay que practicar, para ello utiliza lso siguientes ejercicios:

1. [Hacer una imagen base](step-1-make-image/README.md).
2. [Compilación de imagen multi-etapa](step-2-multistage-build/README.md).
3. [Integración de contenedores](step-3-link-containers/README.md).
4. [Volumenes y Redes](step-4-volumes-network/README.md).
5. [Docker Compose](step-5-docker-compose/README.md).

Trás haber terminado las actividades prácticas proponte un desafio y toma una aplicación o servicio con todas sus integraciones y desplagalo sobre contenedores. 

## ¿Qué sigue?

Ahora que ya conoces los contenedores de docker conoce los proyectos más populares que lo integran como base:

- [Orquestador de contenedores más popular](https://kubernetes.io)
- [Projecto Atomic](https://www.projectatomic.io).
- [PaaS Openshift](https://www.openshift.com).

## Licencia

MIT