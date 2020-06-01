# Node 
FROM node:lts-alpine

# Instalar bash
RUN apk update && apk add bash

# Instalar @angular/cli
RUN npm install -g @angular/cli

# Directorio de trabajo
WORKDIR /app

# se agrega `/app/node_modules/.bin` al $PATH
ENV PATH /app/node_modules/.bin:$PATH

# Instalacion de dependencias de aplicacion web
COPY package.json /app/package.json
RUN npm install
RUN npm install -g @angular/cli

# se copia todo el proyecto (menos los archivos del .dockerignore)
COPY . /app

# Se inicia la aplicacion web.
CMD ng serve --host 0.0.0.0 --port 2400 --disable-host-check
