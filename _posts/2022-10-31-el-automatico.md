---
title: El automático
featured: /assets/images/elAutomatico.jpg
layout: post
excerpt_separator: <!--more-->
---

He estado experimentando con el despliegue automatico de github. Este es el resultado <!--more-->

Hace relativamente poco que he empezado con este blog. La idea de utilizar las páginas de github no es nueva, pero si el 
diseño y el ir publicando posts cada cierto tiempo. Para ello estoy usando [Jekyll](https://jekyllrb.com), que es un 
sistema de blog bastante sencillo. 

El tema es que este framework (si se puede llamar asi) genera una página web estática a partir de los datos que encuentra
en la carpeta ___posts__ (también los includes de assets y demás). Esto hace que lo que funciona realmente es la carpeta ___site__,
y debe rehacerse cada vez que quieras publicar algo.

Inicialmente, el sistema de github pages ya prepara una action para que, al subir a la rama master, se lance el "build" del 
framework y se reconstruya la carpeta ___site__. 

Por otra parte, resulta que los post se deben indicar con la fecha de publicación, en formato __YYYY-MM-DD-titulo-del-post__.

Y aqui llega el "problema":

Si quieres publicar un post para una fecha en concreto (el clásico ejemplo es un viaje o tu dia de cumpleaños) puedes indicar una fecha
futura, e inicialmente hasta ese dia no se publica. Pero como hemos dicho antes, el build se encarga de crear esa carpeta estatica.
Y esto hace que, si no se lanza el build de nuevo el dia de la publicación, dicho post no aparece.

Vamos a ver, que todo esto es totalmente lógico. No es un "problema" como tal. Es, simplemente, que el sistema funciona asi.

Y entonces, como podemos hacer que se lance el build el dia del post futuro?

La solución es bastante sencilla, teniendo en cuenta que ya se lanza una acción en el evento "on Push" de una rama. Asi que
si buscamos por internet, podemos ver distintas maneras de añadir un nuevo action para que se haga este build.

En concreto, yo estoy usando esta action. El archivo os lo podeis bajar de [aqui](https://raw.githubusercontent.com/jakala/jakala.github.io/master/.github/workflows/github-pages.yml).

Debeis ponerlo en la carpeta __.github/workflows__ de vuestro proyecto. 

El contenido del archivo es:

```
name: Build and deploy Jekyll site to GitHub Pages

on:
  schedule:
    - cron: "40 * * * *"
    
jobs:
  github-pages:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/cache@v2
        with:
          path: vendor/bundle
          key: ${{ runner.os }}-gems-${{ hashFiles('**/Gemfile') }}
          restore-keys: |
            ${{ runner.os }}-gems-
      - uses: helaili/jekyll-action@v2.4    # Choose any one of the Jekyll Actions
        with:                              # Some relative inputs of your action
          token: ${{ secrets.GITHUB_TOKEN }}
          target_branch: 'gh-pages'
```

Lo importante de este action es que se lanza desde un schedule (lo que normalmente se indica con un cron). En mi caso,
lo tengo puesto para que se lance a cada hora, a los 40 min. Lo he puesto asi porque te indican en algunos sitios que puede
haber retraso de despliegue (hay muchos proyectos en github, y supongo que se comparten muchos cron). He estado haciendo
experimentos y en mi caso es la mejor hora para lanzar el despliegue.

Lo suyo sería ponerlo únicamente una vez al dia. Pero dado que podría ser que no se lance, me he decidido por esta opción.
Más adelante iré mirando si puedo bajarlo de ratio, porque con que se lance una vez al dia (o como mucho 2) debería estar 
resuelto el problema.



