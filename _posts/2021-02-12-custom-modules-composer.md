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

Una posible solución es gestionar nuestro módulo custom mediante composer del mismo modo que si fuese un módulo contrib de los que podemos encontrar en [drupal.org](https://www.drupal.org/).

# Repositorio GitHub/GitLab

Para poder gestionar nuestro módulo mediante composer necesitaremos dos cosas.
En primer lugar, añadiremos un fichero composer.json que incluiremos en el repositorio de nuestro módulo donde especificaremos la información relacionada con el mismo.

Dos de las propiedades más importantes que define este fichero son el **name** y el **type**. 
Composer permite utilizar muchos tipos de paquetes como librerías, módulos (drupal-module), temas (drupal-theme), etc. 
En este archivo tambien debemos incluir las dependencias que necesitemos.
Un ejemplo básico de este fichero composer.json podría ser el siguiente:

```
{
    "name": "my_package_/my_module",
    "description": "My Custom Module",
    "type": "drupal-module",
    "homepage": "https://github.com/my-user/my_module",
    "authors": [
        {
            "name": "my name",
            "email": "my_email@mail.com"
        }
    ],
    "repositories": {
        "drupal": {
            "type": "composer",
            "url": "https://packages.drupal.org/8"
        },
        "asset-packagist": {
            "type": "composer",
            "url": "https://asset-packagist.org"
        }
    }
}
```

A continuación tendremos que crear una release del nuestro módulo. Esta release tendrá asociado un tag que nos servirá para controlar las distintas versiones del código de nuestro  módulo.

Esto anteriormente solo era posible utilizando **GitHub**; pero a partir de **GitLab 11.7.** también tenemos la posibilidad de generar releases en esta plataforma. 

Independientemente de que plataforma de almacenamiento utilicemos para el código de nuestro módulo, tendremos que  seguir los siguientes pasos: 

## GitHub

Para hacerlo, comenzaremos accediendo a la sección de **Releases** dentro del repositorio de nuestro módulo en la url: "/releases" como se ve en la imagen. 
En este apartado añadiremos una nueva release seleccionando **Draft a new release** o accediendo a la url: "/releases/new". 

![GitHub_release](/images/GitHub_release.jpg)

## GitLab

En el caso de utilizar GitLab accederemos a la sección **Versiones** (si tenemos nuestra cuenta configurada en castellano) en la url: "/-/releases".
Una vez aquí, seleccionaremos **Nueva versión** o accederemos a la url: "/-/releases/new". 
Como se puede observar son soluciones muy similares.

![GitLab_release](/images/GitLab_release.jpg)

Esto nos servirá para que composer pueda gestionar las distintas versiones de nuestro módulo. 

# Añadir a proyecto usando Composer

Debemos añadir nuestro módulo (my_module) en el archivo composer.json del proyecto donde deseemos utilizarlo; en el apartado **"repositories"** como puede verse en el siguiente ejemplo de un site Drupal:

```
    "repositories": {
        "0": {
            "type": "composer",
            "url": "https://packages.drupal.org/8"
        },
        "my-module": {
            "type": "vcs",
            "url": "https://github.com/my-github-user/my-module.git"
        }
        "my-second-module": {
            "type": "vcs",
            "url": "https://gitlab.com/my-gitlab-user/my-second-module.git"
        }
    }    
```

En la propiedad **type** utilizamos **vcs** (version control system) ya que composer detecta el driver del sistema de control de versiones automáticamente. 
Pero si deseamos especificarlo por alguna razón podemos utilizar **bit-bitbucket**, **hg-bitbucket**, **github**, **gitlab**, **perforce**, **fossil**, **git**, **svn** or **hg** en lugar de vcs.

Posteriormente, añadiremos la versión (release) correspondiente de nuestro módulo en el apartado "require" como puede verse a continuación:

```
    "require": {
        "composer/installers": "^1.0.24",
        "wikimedia/composer-merge-plugin": "^1.4",
        "drush/drush": "^9.5",
        "my_package/my_module": "^1.0",
        "package/my_second_module": "^1.0"
    },
```

La nomenclatura utilizada para indicarle a composer que versión necesitamos para nuestro proyecto sigue la siguiente estructura:

`          "Nombre del paquete/Nombre del elemento": "version"`

Si utilizamos el simbolo "**^**" al comienzo de la versión seleccionada, le estamos indicando a composer que necesitamos una versión **igual o superior** a la indicada.

Por último para incorporar al proyecto los cambios añadidos deberemos ejecutar:

```
composer install
```

Por otro lado, cuando composer revise si existen actualizaciones para todos los elementos del proyecto tambien añadirá cualquier actualización de nuestro módulo.

```
composer update
```

En caso de necesitar actualizar únicamente nuestro módulo tan solo tendriamos que introducir:

```
composer update my_module
```


# Para mas información...

* *Composer y Drush en el contexto de Drupal* - [https://medium.com/drupal-y-yo/composer-y-drush-en-el-contexto-de-drupal-9883d2cfb007  ](https://medium.com/drupal-y-yo/composer-y-drush-en-el-contexto-de-drupal-9883d2cfb007)
* *Composer: How to use Git repositories* - [https://daggerhartlab.com/composer-how-to-use-git-repositories](https://daggerhartlab.com/composer-how-to-use-git-repositories)
* *Administrar lanzamientos en un repositorio* - [https://docs.github.com/es/github/administering-a-repository/managing-releases-in-a-repository](https://docs.github.com/es/github/administering-a-repository/managing-releases-in-a-repository)
* *Repositories - Composer* - [https://getcomposer.org/doc/05-repositories.md#git-alternatives](https://getcomposer.org/doc/05-repositories.md#git-alternatives)
* *Composer - Loading a package from a VCS repository* - [https://getcomposer.org/doc/05-repositories.md#vcs](https://getcomposer.org/doc/05-repositories.md#vcs)