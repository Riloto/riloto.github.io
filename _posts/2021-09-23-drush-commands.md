---
layout: post
title: Drush. Introducción y Creación de comandos Drush
tags:
  - drupal
  - drush
  - composer
categories:
  - back-end
  - custom
comments: true
visible: 1
---

![Drush](/images/drush_header.png)

## ¿Que es Drush?

Drush es una interfaz de línea de comandos (CLI) para Drupal que utiliza Unix como base para su funcionamiento. 
Esta herramienta nos permite interactuar con el código de módulos y temas, realizar migraciones de bases de datos; y otras operaciones propias de Drupal como ejecutar cron o limpiar la caché. 
Y puede ser extendido por archivos de comandos de terceros, como veremos con detalle más adelante.
Para más información puedes visitar la página oficial de [Drush](https://www.drush.org/latest/).


## Instalación

La instalación de Drush mediante composer es muy simple como vemos a continuación:

```
  composer require drush/drush
```

Una vez instalado, podemos consultar la version de drush que tenemos usando el comando:

```
  drush --version
```

## Comandos Drush

A veces es realmente útil contar con un comando drush que nos permita realizar acciones directamente desde nuestro terminal sin tener que acceder a nuestro site Drupal.

A continuación vamos a repasar algunos de los comandos más comunes además de algunos que bajo mi humilde pueden ser de los más útiles. 
Para esta lista he recopilado los comandos para Drush en la versión 9/10. 
La mayoría de los comandos tienen los mismos alias y son muy similares de una versión a otra, pero cuentan con pequeñas variaciones. 
En el siguiente link puedes encontrar recopilados los distintos comandos drush Agrupados por su correspondiente versión: [Drush Commands](https://drushcommands.com/).


  - [**Comando (alias)**]
  - - - - - -
  - **drush cache:rebuild (drush cr)**: Limpia todas las cachés del site.

  - **drush config:import (drush cim)**: Importa la configuración del directorio de configuración. 
    Tras listar todos los cambios que se aplicarán nos solicita confirmación para su ejecución. Incorpora la opción "-y" para que automáticamente acepte todos los cambios sugeridos.

  - **drush config:export (drush cex)**: Exporta la configuración actual al directorio de configuración. 
    Del mismo modo que "config-import" nos solicitará confirmación, y tambien cuenta con la opción "-y".

  - **drush core:cron (drush cron)**: Ejecuta todos los procedimientos cron para todos los módulos activos del site.

  - **drush entity:updates (drush entup)**: Aplica las actualizaciones pendientes del entity schema.

  - **drush pm:list (drush pml)**: Muestra una lista de extensiones (módulos y temas) disponibles en el site.

  - **drush pm:enable (drush en)**: Habilita uno o más módulos.

  - **drush pm:uninstall (drush pmu)**: Desinstala uno o más módulos y sus módulos dependientes.

  - **drush sql:drop (drush sql-drop)**: Elimina todas las tablas de la base de datos especificada.

    *Ejemplo:* drush sql:drop "database" 

  - **drush sql:cli (drush sqlc)**: Abre una interfaz de lina de comandos SQL utilizando las credenciales del site. 
    Tambien puede utilizarse para importar un dump de BD como vemos en el siguiente ejemplo.

    Ejemplo: drush sql-cli < ~/my-sql-dump-file-name.sql   

  - **drush sql:dump**: Exporta la BD a un fichero .sql utilizando mysqldump o alguna herramienta equivalente.
  
    Ejemplo: drush sql-dump > ~/my-sql-dump-file-name.sql

  - **drush theme:enable (drush then)**: Habilita uno o más temas.

  - **drush theme:uninstall (drush thun)**: Desinstala uno o más temas.

  - **drush updatedb**: Aplica las actualizaciones de BD pendientes.

  - **drush user-login (drush uli)**: Muestra un enlace de login de 1 solo uso para el usuario ID 1, u otro usuario.

  - **drush user:create (drush ucrt)**: Crea una nueva cuenta de usuario.

    Ejemplo: drush user:create username --mail="user@mail.com" --password="userpassword"

  - **drush user:role:add (drush urol)**: Asigna un rol a un usuario.

    *Ejemplo:* drush user:role:add "administrator" username 


## Creación de un comando Drush custom

Para añadir nuestro comando drush personalizado deberemos crear un nuevo módulo o añadirlo a algún módulo custom que ya tengamos creado.

En primer lugar debemos crear el fichero **drush.services.yml**

```yml
services:
  my_module.custom_rebuild:
    class: \Drupal\my_module\Commands\CustomCacheRebuild
    tags:
      - { name: drush.command }

```

Dentro de nuestro módulo crearemos un fichero "MyDrushCommand.php" dentro de /src/Commands. Es probable que no contemos con alguno de estos directorios en nuestro módulo, de ser así, los generaremos y añadiremos dentro nuestro fichero.

Además de nuestro namespace debemos incluir:  use Drush\Commands\DrushCommands

Del mismo modo añadiremos la clase que contendrá el código de nuestro comando extendiendo la clase DrushCommands. 

A continuación, crearemos un método public con anotación. 

¿Que son las Anotaciones o Annotations?

Las anotaciones son bloques de comentarios en PHP con un formato especial que se utilizan para la descripción de metadatos. Aunque técnicamente es posible utilizar las anotaciones para otros fines, por el momento Drupal sólo las utiliza para el sistema de plugins. Al final de este artículo se encuentran varios links donde podrás encontrar más información sobre Annotations en Drupal 8.

Algunas de las anotaciones disponibles para utilizar son:

  - **@command** – Indica la definición del comando, debe seguir la siguiente estructura =>  nombre_modulo:comando.
  - **@aliases** – Alias disponible para tu comando. El funcionamiento es idéntico al utilizado en los comandos que trae drush por defecto como explicamos anteriormente.
  - **@param** – Indica los parámetros de entrada del comando.
  - **@option** – Define las opciones disponibles para el comando, deben estar indicadas en un array asociativo.
  - **@usage** – Añade un ejemplo de cómo debe utilizarse el comando.


Dentro de este método incorporaremos la lógica que deseemos implementar mediante nuestro comando.
Para ver en datalle como se realizaría, analizaremos el siguiente ejemplo de comando Drush custom:

```php
<?php

namespace Drupal\my_module\Commands;

use Drush\Commands\DrushCommands;
use Drupal\Component\Datetime\TimeInterface;

/**
 * Class CustomCacheRebuild
 */
class CustomCacheRebuild extends DrushCommands {

  /**
   * Rebuild caché showing the time on message
   *
   * @command custom:rebuild
   *
   * @aliases custom
   */
  public function customCacheRebuild() {
    $time = \Drupal::time()->getCurrentTime();
    $time = date("H:i:s", $time);
    drupal_flush_all_caches();

    $text = 'Cache rebuild completed at ' . $time;
    $this->output()->write(" ", TRUE);
    $this->output()->writeln($text);
  }

}

```

Como podemos apreciar, hemos utilizado el método **drupal_flush_all_caches()** para limpiar todas las cachés del sistema. 
Y a traves del servicio **\Drupal::time()->getCurrentTime()** y el método **date()** de php hemos obtenido la hora justo antes de limpiar las cachés para incluirla en el mensaje devuelto al usuario.

Por último probaremos la ejecución de nuestro comando de Drush tras instalar (en caso de que sea necesario) nuestro módulo.
Si todo ha ido bien nos devolverá una salida como la siguiente:

![Terminal](/images/salida_terminal.jpg)

# Quizá te interese:

* *Drush 9 Commands* - [https://drushcommands.com/drush-9x/](https://drushcommands.com/drush-9x/)
* *Composer y Drush en el contexto de Drupal* (@davidjguru) - [https://medium.com/drupal-y-yo/composer-y-drush-en-el-contexto-de-drupal-9883d2cfb007](https://medium.com/drupal-y-yo/composer-y-drush-en-el-contexto-de-drupal-9883d2cfb007)
* *Introduction of Annotations un Drupal 8* (Blair Wadman) - [https://befused.com/drupal/annotations](https://befused.com/drupal/annotations)
* *Drupal 8 Annotations* (Shemsedin Callaki) - [https://oxwebs.com/drupal8/drupal8-annotations/](https://oxwebs.com/drupal8/drupal8-annotations/)
* *Creating Custom Drush 9 Commands in Drupal 8* (Dominika Szydłowska) - [https://www.droptica.com/blog/creating-custom-drush-9-commands-drupal-8/](https://www.droptica.com/blog/creating-custom-drush-9-commands-drupal-8/)
