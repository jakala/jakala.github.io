---
title: DDD y el problema de los Value-Objects (1)
layout: post
excerpt_separator: <!--more-->
---

Aplicando DDD podemos encontrarnos con un problema bastante tonto que puede traernos de cabeza: validar el request completo<!--more-->

Cuando te pones a aplicar DDD en un proyecto (en mi caso suele ser con PHP y Symfony), una de las cosas que se suele
empezar a desarrollar es lo que llamamos la capa de **Dominio**.

Sin entrar mucho en DDD, esta forma de desarrollo nos indica, de una forma MUUUUUUYYY resumida, que hay que dividir la aplicación
en distintas capas:

  - **Infraestructura**: Es la parte donde nuestra aplicacion se conecta con el "Exterior". Aqui encontramos los controladores, asi como
las implementaciones de repositorios (acceso a BBDD, api externa) en general, todo aquello de nuestra aplicación que conecta E/S.
  - **Aplicación**: Aqui nos encontramos con distintas "herramientas" (servicios, comandos, DTO de respuesta...). Es donde podemos encontrar
ciertas lógicas de negocio (lo que comunmente se llamaria "Casos de Uso")
  - **Dominio**: Aqui, en un nivel más interno, nos encontramos con ciertas lógicas más específicas de nuestra aplicación. Aqui tenemos
definidos los eventos de dominio, interfaces, ValueObjects...

Y es en los **ValueObjects** donde nos podemos encontrar con "El Problema".

En esencia, un ValueObject es una clase que define conceptos de nuestra aplicación que solo tienen un valor. Son inmutables,
sin identidad. Un ejemplo rápido seria, para un caso de validación de usuarios, el **email** de dicho usuario. Este email, en concepto, 
es una cadena.

Pero al presentarla como un ValueObject, podemos hacer que esa cadena tenga su propia validación.

Por ejemplo, si en nuestra aplicación solo queremos que los usuarios de un dominio en particular puedan validarse, 
podriamos definir un ValueObject de email que analice la parte de la cadena del TLD, y si no coincide, lanzamos una excepción.

Veamos el caso. Definimos un ValueObject para Email, con una validación para el dominio 'pucelaraiders.es':

    ...
    final class Email
    {
        private string $email;
        
        public function __construct(string $value)
        {
            $this->validate($value);
            $this->email = $email;
        }

        private function validate(string $email): void
        {
            if (!str_ends_with($email, 'pucelaraiders.es') {
                throw new Exception('email no valido');
            }
        }
    }

Ahora, cuando queramos crear un usuario, podemos poner en el constructor algo como:

```
class User
{
    private Email $email;
    
    public function __construct(Email $email)
    {
        $this->email = $email;
    }
}
```

Esto es muy interesante, porque al crear el User, tenemos que asegurarnos de pasar un valueObject que sea de tipo Email.
Con ello, estamos forzando al usuario que cumpla nuestra lógica de validación.

# El problema
Llegamos al tema que nos ocupa. Todo el asunto del valueObject está muy bien, pero se nos da lo siguiente:

Si tenemos varios valueObjects en la clase User, y nos da error en uno de ellos, la excepción que se lanza es del primer 
valueObject erroneo. La excepción corta el proceso y devuelve el error, pero SOLO de ese valueObject. 

En un caso de envio de un formulario, nos interesaría devolver TODOS los errores posibles...