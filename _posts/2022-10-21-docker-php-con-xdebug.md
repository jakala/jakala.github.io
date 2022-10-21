---
title: Docker con php y xdebug
featured: /assets/images/docker.jpg
layout: post
excerpt_separator: <!--more-->
---

Xdebug siempre ha sido una de mis debilidades en cuanto a mi desarrollo del dia a dia como programador.
Porque, seamos claros por una vez: ¿quien no prefiere usar un dump o un die en su codigo php? <!--more-->

quien responda "yo prefiero xdebug" MIENTE :smile:

Está claro, que siempre hay que ir aprendiendo en el dia a dia. Pero a veces no tenemos tiempo y tienes que 
buscar una herramienta o un post de alguien que te diga como se resuelve algo, y ya. 

O no consigues configurar tu editor para que funcione tal o cual herramienta de apoyo, y aunque tengas a gente
a tu alrededor para ayudarte, puede ser (como mi caso) que trabajes con Linux y el resto del personal con Mac,
y ahi tenemos ciertas discrepancias en las instalaciones y configuraciones de herramientas...

Bueno, pues buscando la manera de instalar xdebug en un docker, me he encontrado con este [video](https://www.youtube.com/watch?v=IwMYpr0I2m4), 
en el que Pau (muchas gracias por el video) explica de una manera muy sencilla como tener configurado el xdebug dentro de un docker.

Puede que hasta aqui digas "vale, yo tambien he instalado X-debug en mis dockers, que merito tiene esto?". Pues te invito a 
ver su video, y verás por que.

Si bien la solución es muy correcta, se ve que SIEMPRE instala en la imagen la extensión X-debug, con lo que, actives o desactives
la variable de entorno, el xdebug esta ahi, ocupando "espacio".

Asi que poniendo mis 2 cents, yo me planteo el caso de la siguiente manera:

## Script de instalación fuera del docker, en un fichero
Esto es, todas las instrucciones del dockerfile que ejecutan instalacion y creacion de archivos, salen fuera a un archivo
de instalacion. Voy a llamarlo `xdebug-enable.sh` y lo pongo en el raiz del proyecto.

el contenido del archivo sera algo como:

```
#!/bin/sh

if [ $XDEBUG_ENABLED == 1 ]; then
    apk update && apk add php7-dev g++ make
    pecl install xdebug && docker-php-ext-enable xdebug
    echo 'xdebug.remote_enable = ${XDEBUG_ENABLED}' >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
    echo 'xdebug.remote_host = host.docker.internal' >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
fi
```

## 2 Dockerfile copia este archivo dentro del contenedor
Ahora, el archivo Dockerfile tiene que copiar este archivo dentro del contenedor, para poder utilizarlo.
Dentro del dockerfile ponemos:

```
COPY ./xdebug-enable.sh /app/xdebug-enable.sh
```

## docker-compose ejecuta el script
por ultimo, el docker-compose del proyecto debe llamar a este archivo. Dado que tambien tendra definida la variable de 
entorno, este script se ejecuta, pero no siempre instalará la extension.

```
version: 3.7

services:
  app:
    build: .
    volumes:
      - ./:/app
    working_dir: /app
    environment:
      - XDEBUG_ENABLED=${XDEBUG_ENABLED:-0}
    command: /app/xdebug-enable.sh
```
