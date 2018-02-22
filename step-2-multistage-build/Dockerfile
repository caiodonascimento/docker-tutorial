# Desplegar proyecto Node.JS

# Se inicia la primera etapa donde se realiza la descarga del código fuente
# Imagen base creada por nosostros, solo tiene git
FROM poc-docker/git as git

# Declaramos variables de entorno
ENV GIT_REPO=https://github.com/caiodonascimento/backend-services.git \
    SOURCE_DIR=source-dir

# Descarga repositorio que indica la variable de entorno GIT_REPO en la ruta que indica la variable de entorno SOURCE_DIR
RUN git clone $GIT_REPO $SOURCE_DIR

# Se inicia la segunta etapa con otra imagen base que tiene NPM instalado, lo que genera un nuevo contenedor con la imagen base definida a continuación. El contenedor anterior queda en memoria para ser utilizado en lo que queda de ejecución del build
FROM ymedlop/npm-cache-resource as npm

# Se define la variable de entorno SOURCE_DIR que indica la ruta donde esta el código fuente del contenedor anterior
ENV SOURCE_DIR=source-dir

# Crear una carpeta src 
RUN mkdir src && echo "Directorio del otro contenedor: $SOURCE_DIR"

# Define la ruta /src como carpeta de trabajo
WORKDIR src

# Copiar desde la primera etapa de compilación el contenido de la carpeta /source-dir/
COPY --from=git /$SOURCE_DIR/ ./

# Ejecutar comando de descarga de dependencias
RUN npm install --production

# Se inicia la última etapa donde se selecciona la mínima expresión de nodeJs para contenedores 
FROM node:alpine

# Define la ruta /src como carpeta de trabajo, si no existe la crea 
WORKDIR src

# Copia desde la segunda etapa el contenido de la carpeta /src
COPY --from=npm /src/ ./

# Comando que se debe ejecutar al crear el contenedor
ENTRYPOINT ["node", "server.js"]