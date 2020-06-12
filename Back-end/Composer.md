Composer
========

#### Resolver dependencias de librerias js de módulos contribuidos

Resolver dependencias haciendo un merge. Útil para proyectos en github
```
$ composer require wikimedia/composer-merge-plugin
```

Editar composer.json agregar  en la sección "extra"
```
"merge-plugin": {
  "include": [
    "docroot/modules/contrib/webform/composer.libraries.json"
  ]
},
```

From now on, every time the "composer.json" file is updated, it will also read the content of "composer.libraries.json" file located at web/modules/contrib/webform/ and update accordingly.

3) In order for the "composer.json" file to install all the libraries mentioned inside the "composer.libraries.json", from the Git bash composer update --lock


#### Requerir un nuevo componente

```
// Versión en producción
$ composer require drupal/adminimal_theme:^1.3

// Alfa
$ composer require drupal/domain_entity:^1.0@alpha

// Beta
$ composer require drupal/better_normalizers:^1@beta

// Rama en desarrollo
$ composer require drupal/flexslider:^dev

```

#### Problemas comunes
```
// La memoria especificada para ejecución de php no es suficiente
php -d memory_limit=-1 composer require 
```




ENLACES Y FUENTES
=================
Resolver dependencias de webform
https://www.drupal.org/docs/8/modules/webform/webform-frequently-asked-questions/how-to-use-composer-to-install-libraries