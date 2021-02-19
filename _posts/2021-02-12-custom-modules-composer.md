---
layout: post
title: Gestión de módulos custom mediante Composer y GitHub
tags:
  - git
  - composer
categories:
  - git
  - back-end
  - custom
  - contrib
  - composer
  - github
comments: true
visible: 1
---

![Git&Composer](/images/Git-Composer.jpg)

<span>Photo by <a href="https://unsplash.com/@stefany_andrade?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Stefany Andrade</a> on <a href="https://unsplash.com/?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>

## Introducción

En varias ocasiones tendremos que crear un único módulo dentro de un repositorio, que puede ser utilizado por varios de nuestros proyectos (transversal) o por únicamente uno de ellos.

Pero, ¿que sucede cuando este módulo necesita una actualización?
Por lo general, tendriamos que sustituir en cada proyecto la versión existente de dicho módulo por la versión actual. 
Pero a veces esa actualización solo es útil para ciertos proyectos, quedando el resto tal y como están. 
Como se puede preveer, esta no es para nada una gestión óptima nuestro módulo custom. 

Una posible solución es gestionar nuestro módulo custom mediante composer del mismo modo que si fuese un módulo contrib de los que podemos encontrar en drupal.org

# GitHub - GitLab

Para poder gestionar nuestro módulo mediante composer necesitaremos crear una release del mismo. 
Esto anteriormente solo era posible utilizando **GitHub**; pero a partir de **GitLab 11.7.** también tenemos la posibilidad de generar releases en esta plataforma. 

Independientemente de que plataforma de almacenamiento utilicemos para el código de nuestro módulo, tendremos que  seguir los siguientes pasos: 

Para hacerlo, comenzaremos accediendo a la sección de **Releases** dentro del repositorio de nuestro módulo en la url: "/releases" como se ve en la imagen. 
Yen el caso de utilizar GitLab a la sección **Versiones** (si tenemos nuestra cuenta configurada en castellano) en la url: "/-/releases".

![GitHub_release](/images/GitHub_release.jpg)

En este apartado añadiremos una nueva release seleccionando **Draft a new release** o accediendo a la url: "/releases/new". 
Si nuestro repositorio se encuentra alojado en GitLab seleccionaremos **Nueva versión** o accederemos a la url: "/-/releases/new".

![GitLab_release](/images/GitLab_release.jpg)


Como se puede observar son soluciones muy similares.

Esto nos servirá para que composer pueda gestionar las distintas versiones de nuestro módulo. 

# Composer

Debemos añadir nuestro módulo (my_module) en el archivo composer.json, en el apartado **"repositories"** como puede verse en el siguiente ejemplo de un site Drupal:

```
    "repositories": {
        "0": {
            "type": "composer",
            "url": "https://packages.drupal.org/8"
        },
        "mymodule": {
            "type": "vcs",
            "url": "https://github.com/my-user/my-module.git"
        }
    }    
```

En el apartado "require" debemos añadir la versión (release) correspondiente de nuestro módulo como a continuación:

```
    "require": {
        "composer/installers": "^1.0.24",
        "wikimedia/composer-merge-plugin": "^1.4",
        "drush/drush": "^9.5",
        "my_package/my_module": "^1.0"
    },
```

De este forma en caso de necesitar actualizar nuestro módulo solo tendriamos que introducir:

```
composer update my_module
```

# Para mas información...

* *Composer y Drush en el contexto de Drupal* - [https://medium.com/drupal-y-yo/composer-y-drush-en-el-contexto-de-drupal-9883d2cfb007  ](https://medium.com/drupal-y-yo/composer-y-drush-en-el-contexto-de-drupal-9883d2cfb007)
* *Administrar lanzamientos en un repositorio* - [https://docs.github.com/es/github/administering-a-repository/managing-releases-in-a-repository](https://docs.github.com/es/github/administering-a-repository/managing-releases-in-a-repository)
* *Repositories - Composer* - [https://getcomposer.org/doc/05-repositories.md#git-alternatives](https://getcomposer.org/doc/05-repositories.md#git-alternatives)
* *Composer: How to use Git repositories* - [https://getcomposer.org/doc/05-repositories.md#git-alternatives](https://getcomposer.org/doc/05-repositories.md#git-alternatives)