---
layout: post
title: Añadir Validación Custom sobre Webform mediante Handler
tags:
  - drupal
  - webform
categories:
  - drupal
  - code
comments: true
visible: 1
---


![Webform](/images/webform.jpg)

(Photo by Green Chameleon on Unsplash)

## Webform para creación de formularios

A la hora de implementar un formulario en Drupal, una de las soluciones más utilizadas (aunque no siempre de forma acertada) es Webform.
Para quien no sepa que es Webform, se trata de un módulo contribuido utilizado para la generación de formularios.
Esta herramienta tiene una gran potencia de cara a la gestión de formularios mediante SiteBuilding.


## Validación Personalizada 

Webform aporta algún mecanismo de validación de campos, pero puede que necesitemos definir una validación propia (custom) para un campo específico. 
Para aplicar una validación personalizada para un campo, como por ejemplo para que un campo tenga la structura DNI para almacenarse, deberíamos aplicar una validación propia para dicho campo.

Para esto debemos utilizar un módulo custom creado previamente al que añadiremos la siguiente estructura de directorios:

  * nuestro_modulo
    * src/
        * Plugin/
        * WebformHandler/
            * CustomWebformHandler.php
    * nuestro_modulo.info.yml        
    * nuestro_modulo.module        

```php
<?php

namespace Drupal\nuestro_modulo\Plugin\WebformHandler;

use Drupal\Core\Form\FormStateInterface;
use Drupal\webform\Plugin\WebformHandlerBase;
use Drupal\Component\Utility\Html;
use Drupal\webform\WebformSubmissionInterface;
use Drupal\Core\StringTranslation\StringTranslationTrait;

/**
 * Webform validate handler.
 *
 * @WebformHandler(
 *   id = "my_custom_module_validator",
 *   label = @Translation("Alter webform with custom validation"),
 *   category = @Translation("Settings"),
 *   description = @Translation("Form alter to add custom validation."),
 *   cardinality = \Drupal\webform\Plugin\WebformHandlerInterface::CARDINALITY_SINGLE,
 *   results = \Drupal\webform\Plugin\WebformHandlerInterface::RESULTS_PROCESSED,
 *   submission = \Drupal\webform\Plugin\WebformHandlerInterface::SUBMISSION_OPTIONAL,
 * )
 */
class DammWebformHandler extends WebformHandlerBase {

    use StringTranslationTrait;

  /**
   * {@inheritdoc}
   */
  public function validateForm(array &$form, FormStateInterface $form_state, WebformSubmissionInterface $webform_submission) {
    $this->validateIDNumber($form_state);
  }
  

   /**
   * Validate ID number (Spanish format).
   */
  private function validateIDNumber(FormStateInterface $formState) {
    $value = !empty($formState->getValue('dni')) ? Html::escape($formState->getValue('dni')) : NULL;

    // Skip empty unique fields or arrays (aka #multiple).
    if (empty($value) || is_array($value)) {
      return;
    }

 
    if (preg_match("/\b[0-9]{8}-\D\b/i", $value)) {  
      $formState->setValue('dni', $value);
    }
    else {
        $formState->setErrorByName('dni', $this->t('ID Number not valid.'));
    }
  }
}  

```

Por último debemos añadir nuestro handler al webform deseado en nuestro sitio, para ello accederemos a la siguiente url:

"/admin/structure/webform/manage/[your_webform_name]/handler"

Pulsando en Add handler y seleccionando "Alter webform with custom validation" como muestra la siguiente imagen.

![Custom handler](/images/handler.png)

De esta forma el Webform seleccionado comprobará que se cumpla nuestro criterio de validación antes de almacenar el formulario.
