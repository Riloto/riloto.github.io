---
layout: post
title: Analogía Docker para Dummies
tags:
  - docker
  - virtualization
categories:
  - docker
comments: true
visible: 1
---

![Docker](/../images/docker_dummies.jpg)

<span>Photo by <a href="https://unsplash.com/@tkolman?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Trevor Kolman</a> on <a href="https://unsplash.com/s/photos/whale-pool?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>

# Docker para Dummies

Hace poco me encontré en la tesitura de tener que explicar los beneficios de utilizar Docker a una persona sin 
conocimientos técnicos sobre desarrollo web. 
Por un lado fue un reto, pero lo cierto es que me ayudo a interiorizar más la filosofía que hay tras Docker.
Me impresionó tanto este hecho que decidí escribir este post por si tiene utilidad para cualquiera que se aproxime a 
Docker desde un punto de vista ya sea técnico o no. 
Si cuentas con estos conocimientos técnicos y lo que buscas es una idea general de lo que es Docker y como funciona 
te recomendamos leas nuestro artículo anterior: [Aproximación a Docker para Desarrollo en Drupal](https://riloto.github.io/docker/).

# Introducción

Como punto de partida para nuestro ejemplo, pensemos que nuestro objetivo es crear un restaurante como modelo de negocio.

Y vamos a comparar el esfuerzo que tendremos que realizar si creamos nuestro restaurante por nuestra cuenta o si 
creamos un restaurante dentro de una franquicia. 
Esto nos servirá como ejemplo para comparar la complejidad de configurar por completo nuestro entorno de desarrollo 
(instalar y configurar Apache, MySQL, PHP, ...) frente a utilizar Docker a fin de delegar muchas de esas tediosas tareas.

![Cocina](/../images\rohan-g--mwNJswDlXE-unsplash.jpg)

<span>Photo by <a href="https://unsplash.com/@rohan_g?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Rohan G</a> on <a href="https://unsplash.com/s/photos/kitchen-restaurant?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>

En primer lugar, debemos obviar ciertos aspectos ya que el funcionamiento de una franquicia difiere sustancialmente 
a como funciona realmente Docker.
Nos centraremos en las herramientas que nos proporciona una franquicia dentro de su estructura frente a la creación 
desde cero de dicha estructura en el caso de la creación por nuestra cuenta. 
No vamos a entrar en las imposiciones y otras implicaciones relacionadas con este modelo de negocio; por lo tanto 
pensemos en una franquicia ideal en la que todo son ventajas ;D.

# Docker

Docker en este escenario tomaría el rol de la marca franquicia; pensemos en cualquier franquicia de comida rápida.
Utilizar Docker nos proporcionará acceso a toda la estructura necesaria para montar nuestro entorno de desarrollo 
al igual que la marca proporciona todo lo necesario a sus franquiciados para arrancar su negocio.

Docker es una herramienta que automatiza la creación de aplicaciones () mediante contenedores software que 
utilizan virtualización de Sistema Operativo.

# DockerHub

En primer lugar todo franquiciado necesita una guía o algún método de referencia para arrancar con cada uno de los 
apartados/secciones del negocio. 
En este catálogo la marca le indica al franquiciado todos los apartados y donde o como puede obtener los mismos.  

Para un usuario de Docker esto sería DockerHub donde se pueden encontrar todas las imágenes disponibles que podremos utilizar.  
DockerHub es un servicio proporcionado por Docker donde buscar y compartir imágenes de contenedores. 
Al igual que otros servicios como GitHub, que permite compartir repositorios de proyectos, DockerHub ofrece la 
posibilidad de mantener dichas imágenes de forma pública o de manera privada.

En el caso de que seamos nosotros mismos quienes nos encarguemos de instalar y configurar todo nuestro entorno; 
seremos nosotros quien deberemos buscar el instalador adecuado para nuestro entorno y del mismo modo configurarlo.
Podremos descargar el software que necesitamos desde un sitio web, un gestor de paquetes (como npm, apt o composer) o desde 
cualquier otra fuente que encontremos.
Por supuesto también debemos contemplar las futuras actualizaciones de ese software, por lo que cualquier herramienta 
nos proporcionará una gran ayuda.  

# Imagen de Contenedor

La imagen de un contenedor es un fichero que almacena el código fuentes, las librerías, las dependencias y el resto de 
herramientas necesarias para que la aplicación (contenedor) funcione correctamente.
Pero esta imagen debe ser ejecutada para obtener algún resultado.

Esta imagen jugaría el papel del apartado relacionado con cada sección dentro del catálogo que hemos mencionado 
anteriormente. En la imagen viene toda la información necesaria para arrancar cada apartado del negocio pero las 
instrucciones por si solas no son suficientes para que este se encuentre en funcionamiento.

En el caso de alguien que quiera arrancar desde cero un negocio esto se traduce en una cantidad enorme de trabajo por delante,
en primer lugar para escoger dentro de los diferentes contratistas o proveedores con quien va a trabajar para poner en marcha 
el negocio. 

Estos apartados necesitan llevarse a cabo siguiendo el "plan" establecido; en el caso de un restaurante sería realizar 
las obras pertinentes, contratar a los empleados necesarios, comprar el mobiliario, etc.
Lo que se corresponde con montar esa imagen para obtener un contenedor Docker en ejecución.  

# Contenedor Docker

El contenedor Docker es la aplicación virtualizada en ejecución; se corresponde con el apartado de negocio en funcionamiento. 
Es decir, en nuestra franquicia equivaldría a la sección del restaurante encargada del reparto a domicilio, 
la sección encargada de anotar los pedidos o incluso un apartado de la cocina (como podrían ser la zona de freidoras).  

Cada sección puede funcionar aunque alguna del resto de secciones no funcione, del mismo modo que cada contenedor es 
independiente del resto.

Los contenedores Docker pueden estar en ejecución o no al igual que una sección de un restaurante puede no estar en 
funcionamiento; y eso no quiere decir que no exista dicha sección o que ese contenedor no se encuentre "montado". 

Para quien monta el negocio por su cuenta una vez ha elegido a los contratistas, decoradores, proveedores, y demás 
con los que va a trabajar, ahora comienza la fase de ejecución donde se lleva a cabo todo lo necesario para la puesta 
en marcha del restaurante.
En esta fase no hay apenas diferencias con respecto a la solución propuesta por la franquicia.

# Docker Compose

Docker Compose se encarga de gestionar de manera conjunta todos los contenedores. 
Este papel lo llevaría a cabo el responsable del restaurante (que puede ser el propio franquiciado, un grupo de accionistas,etc)
y representa al restaurante en conjunto.
Por lo tanto cualquier medida que afecte a todo el restaurante debe dirigirse a este responsable. 

 
# Resumen

El restaurante tiene unas particularidades propias pero todos los aspectos que tiene en común con el resto de 
restaurantes son los que utiliza la franquicia para ofrecer una ventaja.

Por ejemplo; El restaurante de la calle xxxxx de la franquicia X no tiene servicio a domicilio a pesar de que 
la franquicia lo ofrece.

De esta forma la franquicia permite que el negocio esté en funcionamiento mucho más rápido que si el interesado en 
arrancar el restaurante tuviera que obtener todo lo necesario por su cuenta; contactando con proveedores, contratistas, etc. 

Creo que la idea principal que nos serviría resumen es que:

    Cuesta menos transportar una estructura existente 
    que crear una nueva estructura desde cero.