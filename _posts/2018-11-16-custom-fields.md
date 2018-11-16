---
layout: post
title: Creacion de custom Fields en Drupal 8
categories:
  - github
  - drupal
  - custom
  - backend
  - field
  - compound
  - module
  - backend
comments: true
visible: 1
---

![Custom Fields](/images/drupal_compound.jpg)

<p>A medida que comenzamos a desarrollar en Drupal nos vamos encontrando con la necesidad de reutilizar grupos de campos simples.
Por ejemplo, a la hora almacenar la información personal de N personas.
No tendremos problema si el formulario almacena solo la información de 1 persona en cada iteración; pero, ¿Y si nuestro sistema es un registro de agrupaciones?

Para explicar como crear y utilizar campos compuestos (custom fields) utilizaremos el siguiente caso de ejemplo.</p>

Nuestro Drupal almacenará el registro de asociaciones de vecinos de nuestro municipio. Para cada Asociación deberemos almacenar la siguiente información:

  * Nombre Asociación
  * Nº Miembros
  * Datos Personales de Miembro

Obviamente, los datos personales contendrán información de diferente tipo; pero deseamos almacenar esta información como un solo conjunto. Este conjunto nos identificará a una persona y contendrá la siguiente información:

  * Nombre
  * Apellidos
  * Cargo
  * DNI
  * Telefono
  * E-mail

<p>Una de las posibles soluciones, sobre todo para aquellos que se enfocan más hacia el site-building en Drupal 8, es el uso del **módulo "Field Collection"**. Este módulo nos permite agrupar varios campos a traves de la interfaz de Drupal de modo que podamos utilizar esta agrupación (colección) al igual que si se tratase de un único campo.

El problema de este módulo es que presenta ciertas difilcultades a menudo y no resulta una solución realmente robusta.
Pero es una solución a tener en cuenta si no vas a utilizar muchos campos compuestos y/o no te apetece arremangarte para codificarlos a mano.</p>

Podrás encontrar el Módulo Field Collection en el siguiente enlace:
[Field Collection](https://www.drupal.org/project/field_collection)

Si nos decantamos por codificar nosotros mismos nuestro "custom field" necesitaremos en primer lugar crear un módulo.
Para crear un módulo en Drupal 8 podemos seguir el siguiente árticulo de José Antonio Rodríguez: [Como crear un módulo en Drupal 8](https://www.ladrupalera.com/es/drupal/desarrollo/drupal8/crear-un-modulo-en-drupal-8)

</br>
Una vez contamos con nuestro módulo vamos a generar la estructura de nuestro campo custom "Persona". Al lío!!
</br>

<p>En primer lugar necesitaremos crear un plugin field type para que nuestro campo custom sea visible para la Field API de Drupal.
De esta forma podremos seleccionarlo al añadir un nuevo campo a cualquier entidad.

Para ello, una vez desplegado nuestro módulo; accederemos a al directorio "nuestro_modulo/Plugin/Field". Y dentro de Field, encontraremos otros 3 directorios: "FieldFormatter, FieldType, FieldWidget".</p>

Con la siguiente estructura:
  nuestro_modulo/Plugin/Field

  * persona.info.yml
  * src/
    * Plugin/
      * Field/
        * FieldFormatter/
          * PersonaFormatter.php
        * FieldType/
          * PersonaField.php
        * FieldWidget/
          * PersonaWidget.php

</br>

## FieldType
Comenzaremos creando un nuevo fichero "PersonaField" dentro de la carpeta FieldType, donde definiremos la clase que corresponde con nuestro campo.
Antes de comenzar a describir nuestro campo debemos añadir a este fichero el namespace al que pertenece además de las clases que utiliza:

```php
<?php

namespace Drupal\nuestro_modulo\Plugin\Field\FieldType;

use Drupal\Core\Field\FieldItemBase;
use Drupal\Core\TypedData\DataDefinition;
use Drupal\Core\Field\FieldStorageDefinitionInterface as StorageDefinition;
```
Posteriormene añadiremos unas anotaciones que identificarán nuestro tipo de campo o "field type".
En estas anotaciones encontraremos los siguientes datos:

  * **id:** que utilizaremos como identificador de nuestro campo.
  * **label:** Nombre del campo.
  * **description:** Descripcion del campo.
  * **defualt_widget:** Widget usado por defecto para insertar los datos de este campo.
  * **default_formatter:** Formatter usado por defecto para representar los datos de este campo.
  * **category:** donde definiremos la categoría donde se englobará dentro de los campos de la Field API de Drupal.

A continuación veremos el código correspondiente a nuestro ejemplo:

```php
/**
 * @FieldType(
 *   id = "Persona",
 *   label = @Translation("Persona"),
 *   category = @Translation("Nuestro Modulo"),
 *   description = @Translation("Añade los campos relacionados con los datos personales de una persona."),
 *   default_widget = "PersonaWidget",
 *   default_formatter = "PersonaFormatter"
 * )
 */
class PersonaField extends FieldItemBase {
```

A continuacion implementaremos lo siguientes métodos:

  * **propertyDefinition():** Define las propiedades del campo mediante Data Definitions para que drupal pueda manejar el campo y así poder establecer sus reglas de validación.
  * **schema():** Define la estructura del campo en BD
  * **isEmpty():** Define las condiciones a tener en cuenta para considerar que el campo está vacio. Esto se usará para decidir si guardar/borrar o no una entrada de nuestro campo custom.

    (Tambien existen los metodos: storageSettingsForm, fieldSettingsForm; entre otros)

En el siguiente ejemplo observamos como se utilizarían cada uno de dichos métodos para incluir la variable "nombre" a nuestro custom field "Persona":

```php
  public static function propertyDefinitions(StorageDefinition $storage) {

    $properties['nombre'] = DataDefinition::create('string')
      ->setLabel(t('Nombre persona'));
    $properties['apellidos'] = DataDefinition::create('string')
      ->setLabel(t('Apellidos persona'));
    $properties['cargo'] = DataDefinition::create('string')
      ->setLabel(t('Cargo persona'));
    $properties['dni'] = DataDefinition::create('string')
      ->setLabel(t('DNI persona'));
    $properties['telefono'] = DataDefinition::create('integer')
      ->setLabel(t('Telefono personal'));
    $properties['email'] = DataDefinition::create('string')
      ->setLabel(t('Correo Electrónico persona'));

    return $properties;
   }

  public static function schema(StorageDefinition $storage) {
	  $columns = [];

    $columns['nombre'] = [
      'type' => 'char',
      'length' => '255',
    ];
    $columns['apellidos'] = [
      'type' => 'char',
      'length' => '255',
    ];
    $columns['cargo'] = [
      'type' => 'char',
      'length' => '255',
    ];
    $columns['dni'] = [
      'type' => 'char',
      'length' => '255',
    ];
    $columns['telefono'] = [
      'type' => 'int',
      'size' => 'medium',
    ];
    $columns['email'] = [
      'type' => 'char',
      'length' => '255',
    ];

    return [
      'columns' => $columns,
      'indexes' => [],
    ];
  }

 public function isEmpty() {
    $isEmpty =
      empty($this->get('nombre')->getValue()) &&
      empty($this->get('apellidos')->getValue()) &&
      empty($this->get('cargo')->getValue()) &&
      empty($this->get('dni')->getValue()) &&
      empty($this->get('telefono')->getValue()) &&
      empty($this->get('email')->getValue());

    return $isEmpty;
  }
}
```
Los diferentes tipos de datos que podremos utilizar para la variable $columns están disponibles en la siguiente url: [Tipos de Datos](https://www.drupal.org/node/159605)
</br>

## FieldWidget
El siguiente paso será crear un widget para nuestro campo custom que nos generará un nuevo elemento en el formulario.
De modo que crearemos un archivo en el directorio /src/Plugin/FieldWidget bajo el nombre "PersonaWidget".

```php
<?php

namespace Drupal\plan_local\Plugin\Field\FieldWidget;

use Drupal;
use Drupal\Core\Field\FieldItemListInterface;
use Drupal\Core\Field\WidgetBase;
use Drupal\Core\Form\FormStateInterface;

/**
 * Plugin implementation of the 'PersonanWidget' widget.
 *
 * @FieldWidget(
 *   id = "PersonaWidget",
 *   label = @Translation("Cargar Persona"),
 *   field_types = {
 *     "Persona"
 *   }
 * )
 */
```

Sobre estas anotaciones cabe destacar que **field_types** define con que campos se podrá usar el widget. De modo que podremos crear Widget genericos que puedan ser utlizado para varios campos.

A continuacion pasamos a definir la clase que contendrá el formulario para nuestro campo compuesto utilizando el método formElement.
Para nuestro campo "Persona" el widget asociado nos quedará similiar al siguiente:

```php
class PersonaWidget extends WidgetBase {

  public function formElement(FieldItemListInterface $items, $delta, Array $element, Array &$form, FormStateInterface $formState) {

    $element['nombre'] = [
      '#title' => t('Nombre'),
      '#type' => 'textfield',
      '#maxlength' => 50,
      '#default_value' => isset($items[$delta]->nombre) ?
        $items[$delta]->nombre : NULL,
    ];
    $element['apellidos'] = [
      '#title' => t('Apellidos'),
      '#type' => 'textfield',
      '#maxlength' => 50,
      '#default_value' => isset($items[$delta]->apellidos) ?
        $items[$delta]->apellidos : NULL,
    ];
    $element['cargo'] = [
      '#title' => t('Cargo en la Asociacion'),
      '#type' => 'textfield',
      '#default_value' => isset($items[$delta]->cargo) ?
        $items[$delta]->cargo : NULL,
    ];
    $element['dni'] = [
      '#title' => t('Documento Nacional de Identidad (DNI)'),
      '#type' => 'textfield',
      '#default_value' => isset($items[$delta]->dni) ?
        $items[$delta]->dni : NULL,
    ];
    $element['telefono'] = [
      '#title' => t('Telefono'),
      '#type' => 'number',
      '#default_value' => isset($items[$delta]->telefono) ?
        $items[$delta]->telefono : NULL,
    ];
    $element['email'] = [
      '#title' => t('Correo Electronico'),
      '#type' => 'textfield',
      '#maxlength' => 50,
      '#default_value' => isset($items[$delta]->email) ?
        $items[$delta]->email : NULL,
    ];

    return $element;
  }
}
```

En el siguiente enlace se encuentra el listado de tipos de elementos permitidos: [Tipos de Elementos](https://api.drupal.org/api/drupal/elements)
</br>


##FieldFormatter
Con los formatters definiremos como se mostrará nuestro campo.
Para ello crearemos un fichero "PersonaFormatter.php" en la carpeta src/Plugin/FieldFormatter con las siguiente estructura:

```php
/**
 * Plugin implementation of the 'PersonaFormatter' formatter.
 *
 * @FieldFormatter(
 *   id = "PersonaFormatter",
 *   label = @Translation("Persona"),
 *   field_types = {
 *     "Persona"
 *   }
 * )
 */

 class PersonaFormatter extends FormatterBase {

 public function viewElements(FieldItemListInterface $items, $langcode) {
    $elements = [];
    foreach ($items as $delta => $item) {
      $elements[$delta] = [
        '#type' => 'markup',
        '#markup' => 'Registro de Personas : ' . $item->persona,
      ];
    }
    return $elements;
  }

}
```

Los métodos con mas relevancia dentro de field formatter son:

  * **defaultSettings:** Configuración por defecto
  * **settingsForm:** Formulario de configuración del formatter
  * **settingsSummary:** Descripción de los ajustes realizados en el formatter
  * **viewElements:** El él único obligatorio y renderiza el valor de nuestro campo.

</br>
</br>

<p>Una vez finalizado todo este proceso tan solo tendremos que limpiar la caché de nuestro Drupal y ya podremos seleccionar nuestro nuevo tipo de campo en nuestro Drupal mediante sitebuilding.

Se trata de una solución realmente potente para obtener un campo compuesto y no supone gran dificultad para alguien que lleve poco tiempo trabajando con Drupal 8.</p>
</br>
Ánimo y a Drupalear!!