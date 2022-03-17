---
layout: post
title: Plugin API (II) - Views Plugins
tags:
  - drupal
  - plugin
  - view
categories:
  - back-end
  - custom
  - plugins
comments: true
visible: 1
---

![Plugins](/images/plugins-II.jpg)

<span>Photo by <a href="https://unsplash.com/@mourimoto?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Mourizal Zativa</a> on <a href="https://unsplash.com/s/photos/lego-bricks?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a><span>

En el artículo anterior realizamos una breve introducción a los plugins en Drupal y vimos un ejemplo de creación de un **Plugin Block** mediante código. Puedes acceder al mismo aquí: [https://riloto.github.io/plugins-I/](https://riloto.github.io/plugins-I/)


En esta ocasión nos centraremos en los **Views Plugins**; estos se clasifican bajo diferentes categorias: 

- La mayor categoria es "handlers" (manejadores) que son plugins especiales usados para manejar diferentes partes de el query process (filters, sorts, joins, arguments) y elementos de renderizado de vistas (views render) como fields y areas.
- El resto de plugins que no entran en la categoría anterior son responsables de las comprobaciones de acceso, la caché, los displays (página, bloque, REST), las wizard options, los query objects y los estilos (tabla, render entities, RSS).

Los distintos tipos de View plugins son: **@ViewsDisplay**, **@ViewsArgument**, **@ViewsField**, **@ViewsFilter**, **@ViewsArea**, **@ViewsStyle**

En esta ocasión nos centraremos en analizar 2 de ellos, el tipo @ViewsArea y @ViewsField.


---


### @ViewsArea 

Para este artículo comenzaremos con un ejemplo de un plugin view de tipo **@ViewsArea**. 
Estos plugins nos permiten añadir un apartado independiente a los resultados de la vista que podremos colocar a modo de Header, como Footer o como opción cuando no se encuentren resultados para nuestra vista (No results Behaviour).

A modo de eejemplo mostraremos un texto simple y un enlace a la home de nuestro site:

```php
<?php

/**
 * @file
 * Contains \Drupal\my_module\Plugin\views\area\MyCustomSiteArea.
 */

namespace Drupal\my_module\Plugin\views\area;

use Drupal\Core\Form\FormStateInterface;
use Drupal\Core\Url;
use Drupal\views\Plugin\views\area\AreaPluginBase;

/**
 * Views area MyCustomSiteArea handler.
 *
 * @ingroup views_area_handlers
 *
 * @ViewsArea("my_custom_site_area")
 */
class MyCustomSiteArea extends AreaPluginBase {
```

Como puede apreciarse utilizaremos las clases FormStateInterface, Url y AreaPluginBase a partir de la cual extenderemos nuestra clase MyCustomSiteArea.

En el Annotation indicamos el tipo de nuestro plugin view **@ViewsArea** añadiendo su nombre entre parentesís y donde se encontrará agrupado dentro de los manejadores (handlers); en este caso **views_area_handlers**.

```php
  /**
   * {@inheritdoc}
   */
  protected function defineOptions() {
    $options = parent::defineOptions();
    return $options;
  }

  /**
   * {@inheritdoc}
   */
  public function buildOptionsForm(&$form, FormStateInterface $form_state) {
    parent::buildOptionsForm($form, $form_state);
  }

  /**
   * {@inheritdoc}
   */
  public function render($empty = FALSE) {
    //Comprobamos si la vista tiene resultados, o si en la configuración de nuestra area 
    //se fuerza a que se muestre a pesarde no existir resultados
    if (!$empty || !empty($this->options['empty'])) {
      $output = array();
      $output['text'] = [
        '#type' => 'processed_text',
        '#text' => '<p>' . $this->t('My custom text') . '</p>',
        '#format' => 'full_html',
      ];
      //Nuestro botón para volver a la pagina principal con clases custom.
      $output['link'] = [
        '#title' => $this->t('< Back to front'),
        '#type' => 'link',
        '#url' => Url::fromRoute('<front>'),
        '#attributes' => [
          'class' => ['button', 'secondary']
        ]
      ];

      return $output;
    }

    return array();
  }

}
```

Los métodos **defineOptions()** y **buildOptionsForm()** se utilizan para la configuración de nuestro plugin; en este caso...
En el siguiente ejemplo de plugin view entraremos en detalle en estos métodos.

Para terminar nos encontramos el método render() que es el encargado de mostrar (renderizar) nuestra custom area en la vista.
En nuestro ejemplo, antes de renderizar nuestra área comprobamos si la vista tiene resultados o si a traves de configuración se fuerza su renderizado a pesar de que la vista no muestre ningún resultado.

Añadimos al array $output un elemento text de tipo "processed_text" con formato "full_html" y un enlace que nos devuelva a la home utilizando el método **fromRoute()** de la clase Url que indicamos al inicio del fichero.


A continuación necesitaremos implementar el **hook_views_data()** en un fichero **my_module.views.inc** como se describe a continuación:

### my_module.views.inc

```php
<?php
/**
 * @file
 * Provide views data for our custom module_name.
 */

/**
 * Implements hook_views_data().
 */
function my_module_views_data() {

  $data['views']['my_custom_site_area'] = array(
    'title' => t('My custom site area'),
    'help' => t('Provide a custom text and a return button to the front page of the site.'),
    'area' => array(
      'id' => 'my_custom_site_area',
    ),
  );

  return $data;
}
```

En este fichero incluiremos la información relacionada con nuestro plugin que obtendremos al buscar nuestro plugin en la administración de la vista dentro de nuestro site Drupal.

![Plugin Area Info](/images/plugin_area_info.jpg)

En este momento ya podremos añadir nuestro plugin view area a las secciones antes comentadas de cualquier vista.

![View Area plugin](/images/view_plugin1.jpg)

Por último, veamos el resultado final que produce nuestro plugin:

![Vista Resultado con Area](/images/area_plugin.jpg)


---


### @ViewsField

A continuación vamos a analizar como se realizaría la implementación de un plugin de tipo **@ViewsField**; o lo que es lo mismo, como crear un campo dentro de una vista.

Para el siguiente ejemplo vamos a añadir un campo a la vista a modo de contador que nos muestre que posición ocupa cada resultado dentro del conjunto global de resultados de la vista.

```php
<?php

namespace Drupal\my_module\Plugin\views\field;

use Drupal\Core\Form\FormStateInterface;
use Drupal\views\Plugin\views\field\FieldPluginBase;
use Drupal\views\ResultRow;

/**
 * A handler to provide a field that is completely custom by the administrator.
 *
 * @ingroup views_field_handlers
 *
 * @ViewsField("counter_views_field")
 */

```

Para ello indicaremos nuestro namespace, las clases que utilizaremos y en nuestro Annotation indicaremos que se trata de un plugin **@ViewsField** y que se agrupa dentro de los manejadores **views_field_handlers**.

```php
class CounterViewsField extends FieldPluginBase
{
  /**
   * {@inheritdoc}
   */
  public function usesGroupBy()
  {
    return FALSE;
  }

  /**
   * {@inheritdoc}
   */
  public function query()
  {
    // No haremos nada -- para sobrescribir la query padre.
  }

  /**
   * {@inheritdoc}
   */
  protected function defineOptions()
  {
    $options = parent::defineOptions();
    $options['counter_start'] = ['default' => 1];
    $options['hide_alter_empty'] = ['default' => FALSE];
    return $options;
  }

  /**
   * {@inheritdoc}
   */
  public function buildOptionsForm(&$form, FormStateInterface $form_state)
  {
    $form['counter_start'] = [
      '#type' => 'textfield',
      '#title' => $this->t('Starting value'),
      '#default_value' => $this->options['counter_start'],
      '#description' => $this->t('Specify the number the counter should start at.'),
      '#size' => 2,
    ];
    parent::buildOptionsForm($form, $form_state);
  }

  /**
   * {@inheritdoc}
   */
  public function render(ResultRow $values)
  {
    // Nota: Restamos 1 al inicio del contador ya que el valor del contador se incrementa en 1 al final de esta función
    $count = is_numeric($this->options['counter_start']) ? $this->options['counter_start'] - 1 : 0;
    $pager = $this->view->pager;

    // Si la vista tiene paginador lo utiliza para calcular el valor del elemento actual. 
    if ($pager->usePager()) {
      $count += ($pager->getItemsPerPage() * $pager->getCurrentPage() + $pager->getOffset());
    }
    // Añade el valor actual del contador 
    $count += $this->view->row_index + 1;
    return $count;
  }
}
```

Como vimos en el ejemplo anterior volvemos a utilizar los métodos **defineOptions()**, **buildOptionsForm()** y **render()**.

Vamos a centrarnos en los 2 primeros; en el primer ejemplo dejamos que fuese la clase padre (parent) la que se encargase de este procesamiento ya que no necesitabamos definir nada a traves de configuración.

El método defineOptions() nos permite definir los valores por defecto que traerá nuestro campo y el método buildOptionsForm() nos permite modificar el formulario de configuración de nuestro campo.

En este caso vamos a añadir una opción a traves de la cual el usuario podrá especificar el número por el cual comenzará a contar nuestro campo. 
Esta opción viene identificada por **counter_start** y definimos sus caracteríticas en buildOptionsForm(); tales como que será de tipo texto, su descriptión y que tendrá un tamaño igual a 2. Por otro lado le hemos añadido como valor por defecto el 1 en defineOptions().

Tambien indicamos la opción **hide_alter_empty** por defecto desactivada que nos ocultaría nuestro campo en el caso de que estuviese vacío (empty).

![View Field Form](/images/field_form.jpg)

Por otro lado, el método render() utiliza el paginador de la vista (pager) y la propiedad row_index para calcular el valor que le corresponde a cada elemento. Necesitamos añadir la clase **Drupal\views\ResultRow** para poder manejar los resultados de la vista a traves de la variable $values.

Esto nos produce el siguiente resultado en nuestra vista:

![View Field](/images/view_field.jpg)


Aunque solo sea a modo de introducción espero que este artículo sea de utilidad para todos aquellos que quieran comenzar con la creación y/o modificación de views de manera programátrica.

# Quizá te interese:

* *Annotations-based plugins* (drupal.org) - [https://www.drupal.org/docs/drupal-apis/plugin-api/annotations-based-plugins](https://www.drupal.org/docs/drupal-apis/plugin-api/annotations-based-plugins)
* *Annotation-Based Plugins in Views* (drupal.org) - [https://www.drupal.org/docs/8/api/plugin-api/annotation-based-plugins-in-views](https://www.drupal.org/docs/8/api/plugin-api/annotation-based-plugins-in-views)
* *Views plugins* (drupal.org) - [https://api.drupal.org/api/drupal/core!modules!views!views.api.php/group/views_plugins/](https://api.drupal.org/api/drupal/core!modules!views!views.api.php/group/views_plugins/)

## Otros Views Plugins
* *How to Create a Custom Views Argument Plugin in Drupal 8* (by Jigar Mehta)- [https://evolvingweb.ca/blog/how-create-custom-views-argument-plugin-drupal-8](https://evolvingweb.ca/blog/how-create-custom-views-argument-plugin-drupal-8)
* *Building a Views display style plugin for Drupal* (drupal.org) - [https://www.drupal.org/docs/creating-custom-modules/building-a-views-display-style-plugin-for-drupal](https://www.drupal.org/docs/creating-custom-modules/building-a-views-display-style-plugin-for-drupal)
* *Creating a custom Views filter in Drupal 8* (webomelette.com) - [https://www.webomelette.com/creating-custom-views-filter-drupal-8](https://www.webomelette.com/creating-custom-views-filter-drupal-8)
* *[Drupal 8] - Creating Views Display Plugin* (osseed.com) - [https://www.osseed.com/blog/drupal-8-creating-views-display-plugin](https://www.osseed.com/blog/drupal-8-creating-views-display-plugin)
* *Cómo crear nuestro propio Plugin de Exposed Form para Views en Drupal 8* (ateneatech.com/blog) - [https://ateneatech.com/blog/como-crear-nuestro-propio-plugin-de-exposed-form-para-views-en-drupal-8](https://ateneatech.com/blog/como-crear-nuestro-propio-plugin-de-exposed-form-para-views-en-drupal-8)