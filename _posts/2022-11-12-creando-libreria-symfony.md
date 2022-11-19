---
title: DDD y el problema de los Value-Objects (2)
layout: post
excerpt_separator: <!--more-->
---

Aplicando DDD podemos encontrarnos con un problema bastante tonto que puede traernos de cabeza: validar el request completo<!--more-->

El asunto del problema que presentaba en el post anterior, es un poco conflictivo.

  - Por una parte, tiene todo el sentido que la lógica de validación de cada campo esté contemplada en el ValueObject correspondiente.

  - Por otro lado, esto nos dificulta devolver todos los errores en el controlador, pues en cuando ocurre uno, se para el proceso.

Asi pues, ¿como solucionamos esto?

En mi opinión, el problema presentado NO existe. Es un problema más bien del punto de vista. 

Y es que, el problema de devolver todos los parámetros que den error, ocurre porque hemos permitido "entrar" en nuestra aplicacion
cierta información que NO es correcta. Es decir, en la parte de INFRAESTRUCTURA (nuestro controlador) dejamos que la petición tenga valores
que no cumplen con lo esperado.

Es cierto que la validación la hemos puesto en el dominio, puesto que la especificidad del error tiende a ir hacia el dominio
(lo cual es correcto). 

Desde mi punto de vista, los casos de uso (que se cubren en Aplicacion y Dominio), Deben tener sus propias validaciones. Y
por su propio "peso", dichas logicas acaban en Dominio. 

Sin embargo, el controlador (que recibe la petición) está "fuera" de dicho ámbito, y deberíamos "proteger" nuestros casos
de uso de entradas "maliciosas". Y esa "protección" de datos maliciosos deberían tener su propia validación. En general, 
la entrada será un Json, un Xml, un post de formulario...

Esto, se suele conocer como PAYLOAD.

Y en mi opinión, este payload deberia validarse en su capa correspondiente (Infraestructura). Si no es correcta, no pasa. Punto.

## ¿y como validas en infraestructura?

La forma más "correcta" que le veo a este tipo de problema, es aplicar una validación al request, para que cumpla el Payload.
Normalmente, en las aplicaciones Symfony, obtenemos todos los datos como un objeto especifico de tipo **Request**. 

Pero al final,
tanto venga como Json, Xml, Post, o parametros... dichos datos se pueden pasar como un array. Con lo que podriamos utilizar 
herramientas como [justinrainbow/json-schema](https://github.com/justinrainbow/json-schema), que nos provee de una clase 
para validar dichos datos a través de un estandar json-schema.

Estoy planteandome aplicar el validador anterior en el [esqueleto](https://github.com/jakala/symfony6-ddd-skeleton) de 
symfony con DDD que tengo en mi repositorio github... (A ver como lo hago) 

