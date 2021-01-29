---
layout: post
title: Creación de un Pop-up o modal en Drupal 8
tags:
  - drupal
  - modal
  - pop up
categories:
  - drupal
  - back-end
  - custom
  - code
comments: true
visible: 1
---


![Pop-Up](/images/PopUp.jpg)

<span>Photo by <a href="https://unsplash.com/@knarfy?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Ryan Moulton</a> on <a href="https://unsplash.com/?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>

## Creación de un Pop-up o modal en Drupal 8

Es muy frecuente como usuario, la interacción con un Pop-up, ventana emergente o modal. 
Todas ellas son distintas maneras de llamar al mismo tipo de elemento por lo que utilizaremos estos nombres 
indistintamente durante todo el articulo para referirnos a dicho elemento.

En primer lugar definamos brevemente que distingue a este elemento. 
Un Pop-up o modal no es más que una ventana que aparece de forma repentina sobre el contenido de la web que estamos visitando.

Los ejemplos más claros que podemos encontrar son los que se utilizan para informarnos del almacenamiento de cookies
por parte de la web que contienen información sobre nuestra navegación y donde se nos solicita dar nuestro consentimiento.
Pero este elemento es ampliamente utilizado ya sea para mostrar publicidad como para avisarnos sobre la acción 
que estamos a punto de realizar; y que probablemente sea irrecuperable. 

Hay diversas maneras de añadir un Pop-up a nuestra web Drupal, utilizando HTML, Javascript o utilizando módulos contrib 
como [Simple Popup Blocks]( https://www.drupal.org/project/simple_popup_blocks).

Pero nosotros vamos a centrarnos en la creación de un modal a partir de un módulo custom y la utilización de un bloque. 

## Añadir un Pop-up a nuestro módulo custom

Para ello debemos contar con un módulo custom previamente creado. 
En el siguiente enlace podéis obtener más información sobre la creación de módulos custom en Drupal 8.

En primer lugar definiremos una ruta; para ello editaremos/crearemos el fichero "routing.yml" de nuestro módulo
(Por ejemplo; si nuestro módulo se llama "my_modal" tendremos que buscar y editar/crear el fichero "my_modal.routing.yml")

Añadiremos las siguientes líneas:

```
my_modal.modal:
  path: 'modal-element/modal'
    defaults:
      _title: 'My Modal'
      _controller: '\Drupal\my_modal\Controller\CustomModalController::modal'
    requirements:
      _permission: 'access content'
```    

Este código crea la ruta nombrada como my_modal.modal donde se indica que cuando un usuario acceda a la url
 **/modal-element/modal** dentro de nuestro site se lanzará nuestro modal. 
 La línea: 
 ```_controller: '\Drupal\my_modal\Controller\CustomModalController::modal'``` hace referencia al controlador 
 encargado de mostrar nuestro pop-up.
 
## Creación del Controlador
 
 A continuación vamos a definir nuestro controlador; para ello deberemos crear la siguiente estructura de directorios 
 en nuestro módulo:
 
  * my_modal
    * src/
        * Controller/
          * CustomModalController.php
    * my_modal.info.yml        
    * my_modal.module
    * my_modal.routing.yml     
    
Dentro del fichero CustomModalController.php añadiremos el siguiente código:
   
```php
<?php

/**
     * @file
     * CustomModalController class.
     */

namespace Drupal\my_modal\Controller;

use Drupal\Core\Ajax\AjaxResponse;
    use Drupal\Core\Ajax\OpenModalDialogCommand;
    use Drupal\Core\Controller\ControllerBase;

class CustomModalController extends ControllerBase {

  public function modal() {
    $options = [
      'dialogClass' => 'popup-dialog-class',
      'width' => '50%',
    ];
    $response = new AjaxResponse();
    $response->addCommand(new OpenModalDialogCommand(t('Título del Modal'), t('Texto del modal'), $options));

    return $response;
  }
}    
```   

Este código abrirá un pop-up como el que aparece en la siguiente imagen:

![Modal](/images/modal_example.jpg) 

Este modal utiliza un comando Ajax para abrir un **dialog modal**. 
Drupal utiliza Ajax para hacer que una página ya cargada siga comunicándose con el servidor de forma asíncrona, 
sin necesidad de recargar la página completa. 
Al final de este post incluiremos algunos enlaces para aportar más información sobre Ajax y "dialog modal".

Es posible utilizar este pop-up para mostrar cualquier cosa en su interior; puede definirse un formulario, 
un bloque de texto, o el resultado de una vista (como veremos más adelante).


## Creación del bloque

Todo modal necesita una acción previa que lo haga aparecer y esta acción puede variar según nuestras necesidades 
pudiendo aparecer al interactuar sobre un botón, un enlace, al finalizar la carga del site o cualquier otra acción que 
se nos ocurra.

Para nuestro pop-up vamos a crear un bloque que contendrá el botón/enlace que abrirá nuestro modal.
En nuestro módulo crearemos dentro de **src/** el directorio **Plugin/** y en su interior el directorio **Block/** que 
almacenará el código de nuestro bloque. La estructura de directorios de nuestro módulo quedará tal que así:

  * my_modal
    * src/
        * Controller/
          * CustomModalController.php
        * Plugin/
          * Block/
            * ModalBlock.php
    * my_modal.info.yml        
    * my_modal.module
    * my_modal.routing.yml  
 
Dentro de ModalBlock.php añadiremos el siguiente código:

```php
<?php

/**
        * @file
        * Contains \Drupal\my_modal\Plugin\Block\ModalBlock.
        */
       
namespace Drupal\my_modal\Plugin\Block;

use Drupal\Core\Block\BlockBase;
use Drupal\Core\Url;
use Drupal\Core\Link;
use Drupal\Component\Serialization\Json;
       
/**
 * Provides a 'Modal' Block
 *
 * @Block(
 *   id = "modal_block",
 *   admin_label = @Translation("Modal block"),
 * )
 */
class ModalBlock extends BlockBase {
  /**
   * {@inheritdoc}
   */
  public function build() {
    $link_url = Url::fromRoute('my_modal.modal');
    $link_url->setOptions([
      'attributes' => [
        'class' => ['use-ajax', 'button', 'button--small'],
        'data-dialog-type' => 'modal',
        'data-dialog-options' => Json::encode(['width' => 400]),
      ]
    ]);
   
    return array(
      '#type' => 'markup',
      '#markup' => Link::fromTextAndUrl(t('Mostrar modal'), $link_url)->toString(),
      '#attached' => ['library' => ['core/drupal.dialog.ajax']]
    );
  }
}        
```
    
Este código crea un enlace en el bloque que llama directamente a la url que creamos previamente en nuestro fichero routing.yml.

    $link_url = Url::fromRoute('my_modal.modal'); 

Se crea el enlace a partir de **$link_url** en esta línea:

    '#markup' => Link::fromTextAndUrl(t('Mostrar modal'), $link_url)->toString(),

Por último incorporamos la librería encargada de gestionar los dialogs en Drupal. 
Para ello utilizamos la propiedad **#attached** sobre la que hablaremos con más detenimiento en próximos artículos. 

    '#attached' => ['library' => ['core/drupal.dialog.ajax']]

## Añadir el bloque a una región 

Para que nuestro bloque sea visible dentro de nuestro site debemos añadirlo a una región dentro de **Block Layout**.
Accederemos a "/admin/structure/block" elegiremos la región donde deseamos que aparezca nuestro bloque y lo añadiremos 
pulsando en **Place block**; quedando la disposición de bloques de esta forma:

![Block Layout](/images/modal_block.jpg)

En nuestro caso el bloque elegido se encuentra localizado en la parte superior de la web. 
Una vez que pulsemos en **Mostrar modal** aparecerá nuestro pop-up.
 
 
## EXTRA - Mostrar una vista en Pop-up   

A modo de aportación adicional añadimos un pequeño ejemplo con en el que mostraremos los resultados de una vista en nuestro pop-up.

Las vistas son componentes incluidos en el core de Drupal que son realmente útiles y que cuentan con una gran potencia. 
Estos componentes se utilizan para mostrar listados de elementos existentes en nuestro sitio web (nodos, términos de taxonomía, etc).
Estos listado puedes ser editados de multiples formas mostrando tanto los elementos completos como de forma parcial, 
permite la reescritura de resultados y utilizar presentaciones como una página o un bloque.

En primer lugar necesitaremos tener una vista creada que nos sirva como base para mostrar sus resultados. 
Para más información sobre la creación y uso de vistas en Drupal 8 al final de este post podréis encontrar varios enlaces que pueden ser de utilidad. 

Para poder utilizar dicha vista en nuestro pop-up tan solo tendremos que modificar nuestro controlador para que 
la función modal() tenga el siguiente aspecto:

```php
public function modal() {
  $view = Views::getView('my_view');
  $view->preview('page_1');
  $options = [
    'dialogClass' => ['popup-dialog-class', 'language-selector'],
    'width' => '40%',
    'display' => 'none'
  ];

  $response = new AjaxResponse();
  $response->addCommand(new OpenModalDialogCommand(t('Available in: '),$view->render(), $options));

  return $response;
}      
```

En este código obtenemos la vista utilizando su id o machine name e indicamos que presentación deseamos utilizar; 
en nuestro caso la presentación **page_1** de la vista **my_view**.

    $view = Views::getView('my_view');
    $view->preview('page_1');

A continuación utilizamos el método [render()](https://api.drupal.org/api/drupal/core%21includes%21common.inc/function/render/8.2.x) 
para mostrar la vista dentro de nuestro modal.     


## Otros enlaces de interés

* *Ajax Basic Concepts* - [https://www.drupal.org/docs/drupal-apis/ajax-api/basic-concepts](https://www.drupal.org/docs/drupal-apis/ajax-api/basic-concepts)
 
* *Ajax Dialog Boxes* - [https://www.drupal.org/docs/drupal-apis/ajax-api/ajax-dialog-boxes](https://www.drupal.org/docs/drupal-apis/ajax-api/ajax-dialog-boxes)
 
* *How to Create Views in Drupal 8* - [https://hostadvice.com/how-to/how-to-create-views-in-drupal-8/](https://hostadvice.com/how-to/how-to-create-views-in-drupal-8/)
 
* Charla [Views 101](https://asociaciondrupal.es/video/views-101) de Jose Luis Bellido Rojas en la Drupal Camp Spain de 2019 en Conil