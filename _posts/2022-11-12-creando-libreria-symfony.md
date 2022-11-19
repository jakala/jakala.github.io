---
title: Como crear una libreria symonfy
layout: post
excerpt_separator: <!--more-->
---

Encontré un manual para crear un repositorio-libreria de symfony<!--more-->

A raiz del tema de los valueObjects, me puse a mirar como podia crear una libreria para symfony. El repositorio de Seldaek
donde implementa json-schema para php [justinrainbow/json-schema](https://github.com/justinrainbow/json-schema) define una
clase que hay que instanciar para poder hacer la validación correspondiente.

He querido aportar mis dos cents añadiendo una libreria que publica un servicio en el contenedor de inyeccion de
dependencias de symfony, basado en la libreria de json-schema. Podeis encontrarla en
[jakala/json-schema-validator](https://github.com/jakala/JsonSchemaValidator).

## el archivo composer.json
Aunque podeis encontrarlo en el repositorio, lo pongo aqui para comentar un par de cosas:
```
{
    "name": "jakala/json-schema-validator",
    "description": "Symfony service to use schema json validator",
    "type": "symfony-bundle",
    "autoload": {
        "psr-4": {
            "Jakala\\Validator\\": "src"
        }
    },
    "require": {
        "justinrainbow/json-schema": "^5.2"
    },
    "require-dev": {
        "symfony/dependency-injection": "^6.1",
        "symfony/config": "^6.1",
        "symfony/http-kernel": "^6.1",
        "symfony/yaml": "^6.1"
    },
    "authors": [
        {
            "name": "Angel Cid",
            "email": "angel.cid@gmail.com"
        }
    ]
}
```
Del composer.json hay que destacar dos cosas importantes:

  - El `type` de la libreria debe ser **symfony-bundle**
  - debeis añadir la parte de autoload con el nombre de vuestro namespace. En mi caso es `Jakala\Validator` y hace referencia
a la carpeta `src`
  - los vendor de la sección `require-dev` son importantes, pero deben estar en esta sección, ya que la aplicacion
principal que utilice esta librería ya deberia tenerlos instalados.

## Estructura de carpetas
podemos crearla con:

    mkdir -p config src/DependencyInjection

y nos debe quedar una estructura como esta:
```
JsonSchemaValidator
├─ config
│  └─ services.yaml
└─ src
   ├─ JsonSchemaValidator.php
   ├─ JsonSchemaValidatorBundle.php
   └─ DependencyInjection
     ├─ ConfigurationSchema.php
     └─ JsonSchemaValidatorExtension.php
```
En mi caso solo tengo definida la clase `JsonSchemaValidator` para que quede definida como servicio. Pero aqui somos libres de
añadir más archivos que ayuden a nuestro desarrollo.




El manual en el que me he basado lo podeis encontrar en este [enlace](https://macrini.medium.com/how-to-create-service-bundles-for-a-symfony-application-f266ecf01fca).

