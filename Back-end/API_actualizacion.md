API de actualización
========
Se utiliza cuando se realizan cambios en la configuración que afectan a datos almacenados.
- Campos de una entidad.
- Schemas.
- Tipos de datos, valores permitidos o estructura de arreglos para una key de configuración.
- Dependencias. (Ejemplo de un plugin)
- Etc.

#### Para ejecutar los hooks update:
```
$ drush updb -y
```

#### Funciones para actualizar entidades y schemas
```
- getEntityType($entity_type_id)
- installEntityType(EntityTypeInterface $entity_type)
- updateEntityType(EntityTypeInterface $entity_type)
- uninstallEntityType(EntityTypeInterface $entity_type)
- getFieldStorageDefinition($name, $entity_type_id)
- installFieldStorageDefinition($name, $entity_type_id, $provider, FieldStorageDefinitionInterface $storage_definition)
- updateFieldStorageDefinition(FieldStorageDefinitionInterface $storage_definition)
- uninstallFieldStorageDefinition(FieldStorageDefinitionInterface $storage_definition)
```

#### Un nuevo campo ha sido agregado a un tipo de contenido (nodo)
```php
/**
 * Agregar el nuevo campo 'revision_translation_affected' a las entidades de tipo node.
 */
function node_update_8001() {
  // Crear la definición del nuevo campo.
  $storage_definition = BaseFieldDefinition::create('boolean')
      ->setLabel(t('Revision translation affected'))
      ->setDescription(t('Indicates if the last edit of a translation belongs to current revision.'))
      ->setReadOnly(TRUE)
      ->setRevisionable(TRUE)
      ->setTranslatable(TRUE);

  \Drupal::entityDefinitionUpdateManager()
    ->installFieldStorageDefinition('revision_translation_affected', 'node', 'node', $storage_definition);
}
```



```
/**
 * Implementions of hook_update_N().
 */
function nombre_modulo_update_8001(&$sandbox) {
  $field_storage = FieldStorageConfig::loadByName('block_content', 'placement');
  $allowed_values = $field_storage->getSetting('allowed_values');
  if (!isset($allowed_values['cashback_checkout'])) {
    $allowed_values['cashback_checkout'] = 'Cashback checkout';
    $field_storage->setSetting('allowed_values', $allowed_values);
    $field_storage->save();
  }
}

/**
 * Implements hook_update_n() for new fields in node type Event.
 */
function real_extra_fields_update_8001() {
  $storage_definitions = [
    'audience_type' => BaseFieldDefinition::create('entity_reference')
      ->setLabel(t('Audience type'))
      ->setSetting('target_type', 'taxonomy_term')
      ->setCardinality(3),

    'date' => BaseFieldDefinition::create('datetime')
      ->setLabel(t('Date'))
      ->setSetting('datetime_type', 'date')
      ->setCardinality(1),

    'details_participation_fee' => BaseFieldDefinition::create('text_long')
      ->setLabel(t('Participation fees (details)'))
      ->setTranslatable(TRUE)
      ->setDisplayOptions('form', [
        'type' => 'text_long',
        'weight' => 0,
      ])
      ->setCardinality(1),

    'repl' => BaseFieldDefinition::create('link')
      ->setLabel(t('Replay'))
      ->setSettings([
        'link_type' => LinkItemInterface::LINK_GENERIC,
        'title' => 1,
      ])
      ->setDisplayOptions('form', [
        'type' => 'link_default',
      ])
      ->setCardinality(1),
  ];

  $entity_definition_update_manager = \Drupal::entityDefinitionUpdateManager();
  foreach ($storage_definitions as $field_name => $storage_definition) {
    $entity_definition_update_manager->installFieldStorageDefinition($field_name, 'node', 'node', $storage_definition);
  }
}

/**
 * Implements hook_update_N().
 *
 * Update date_of_stay values from datetime to date.
 */
function siblu_field_update_8004() {
  $database = \Drupal::database();

  $fields = ['end_date_of_stay_value', 'start_date_of_stay_value'];
  $tables = ['paragraph__end_date_of_stay', 'paragraph__start_date_of_stay'];
  $revision_tables = ['paragraph_revision__end_date_of_stay', 'paragraph_revision__start_date_of_stay'];

  $i = 0;
  foreach ($fields as $field) {
    $sql = "SELECT entity_id, revision_id, $field FROM {$tables[$i]}";
    $result = $database->query($sql, []);
    if ($result) {
      while ($row = $result->fetchAssoc()) {
        $date_long = $row[$field];
        $timestamp = strtotime($date_long);
        $date_short = date('Y-m-d', $timestamp);
        $id = $row['entity_id'];
        // Update field table value.
        $database->query("UPDATE {$tables[$i]} SET {$field} = '$date_short' WHERE {$tables[$i]}.entity_id = :id", [":id" => $id]);
        // Update revision field table value.
        $database->query("UPDATE {$revision_tables[$i]} SET {$field} = '$date_short' WHERE {$revision_tables[$i]}.entity_id = :id", [":id" => $id]);
      }
    }
    $i++;
  }
}

/**
 * Update unused layouts 'layout_x' and 'layout_y'.
 */
function asf_layout_builder_ui_post_update_unused_layouts(&$sandbox = NULL) {
  // Need to update existing nodes' layout sections in following way:
  // 'layout_x' => 'new_layout_x' and
  // 'layout_y' => 'new_layout_y'.
  $field_name = OverridesSectionStorage::FIELD_NAME;
  $layout_ids = [
    'layout_x' => 'new_layout_x',
    'layout_y' => 'new_layout_y',
  ];
  $node_storage = \Drupal::entityTypeManager()->getStorage('node');
  if (!isset($sandbox['ids'])) {
    $sandbox['ids'] = $node_storage->getQuery()->exists($field_name)->execute();
    $sandbox['count'] = count($sandbox['ids']);
  }

  for ($i = 0; $i < 10 && count($sandbox['ids']); $i++) {
    $id = array_shift($sandbox['ids']);
    $save = FALSE;
    $node = $node_storage->load($id);
    $sections = $node->get($field_name)->getSections();
    foreach ($sections as $delta => $section) {
      $layout_id = $section->getLayoutId();
      if (in_array($layout_id, array_keys($layout_ids))) {
        $new_section = Section::fromArray([
          'layout_id' => $layout_ids[$layout_id],
          'components' => array_values(array_map(
            function (SectionComponent $component) {
              return $component->toArray();
            }, $section->getComponents())),
        ]);
        $sections[$delta] = $new_section;
        $save = TRUE;
      }
    }
    if ($save) {
      $node->get($field_name)->setValue($sections);
      $node->save();
    }
  }

  $sandbox['#finished'] = empty($sandbox['ids']) ? 1 : ($sandbox['count'] - count($sandbox['ids'])) / $sandbox['count'];
}

/**
 * Update used sandbox and at file nombre_modulo.post_update.php 
 * function hook_post_update_NAME
 */

function nombre_modulo_post_update_nombre_de_la_funcion(&$sandbox) {
  $storage = \Drupal::entityTypeManager()->getStorage('commerce_product');
  // Initialize some variables during the first pass through.
  if (!isset($sandbox['total'])) {
    $ids = $storage->getQuery()
      ->condition('webform_ads', 'contact_project', '<>')
      ->count()
      ->execute();
    if ($ids == 0) {
      $sandbox['#finished'] = 1;
      return;
    }
    $sandbox['total'] = $ids;
    $sandbox['current'] = 0;
  }
  // Handle one pass through.
  $ids = $storage->getQuery()
    ->condition('webform_ads', 'contact_project', '<>')
    ->range($sandbox['current'], $sandbox['current'] + 25)
    ->execute();
  $products = $storage->loadMultiple($ids);
  if (empty($products)) {
    $sandbox['#finished'] = 1;
    return;
  }
  foreach ($products as $product) {
    $product->webform_ads->target_id = 'contact_project';
    $product->save();
    $sandbox['current']++;
  }
  \Drupal::messenger()->addStatus($sandbox['current'] . ' products processed.');
  $sandbox['#finished'] = ($sandbox['current'] / $sandbox['total']);
}

```

### Cargar nuevas configuraciones
```
/**
 * Create 'Duration', 'Model' vocabularies.
 */
function siblu_field_update_8004() {
  $configs = [
    'taxonomy.vocabulary.model',
    'taxonomy.vocabulary.duration',
    'language.content_settings.taxonomy_term.model',
    'language.content_settings.taxonomy_term.duration',
  ];
  $module_installer = \Drupal::service('module_installer');
  $module_installer->install([
    'config_import',
  ], TRUE);
  /* @var \Drupal\config_import\ConfigImporterServiceInterface $config_importer */
  $config_importer = \Drupal::service('config_import.importer');
  $config_importer->importConfigs($configs);
  $module_installer->uninstall([
    'config_import',
  ], TRUE);
}
```

### Alterar los links de menús, campos de tipo link
```
/**
 * Implements hook_link_alter().
 *
 * Add target="_blank" to all external links.
 */
function mi_modulo_link_alter(&$variables) {
  if ($variables['url']->isExternal()) {
    $webUri = (isset($_SERVER['HTTPS']) && $_SERVER['HTTPS'] === 'on' ? "https" : "http") . "://$_SERVER[HTTP_HOST]";
    /* @var $url Drupal\Core\Url */
    $url = $variables['url'];
    $linkUri = $url->toString();
    if (str_contains($linkUri, $webUri) === FALSE) {
      $variables['options']['attributes']['target'] = '_blank';
      $variables['options']['attributes']['rel'] = 'noopener';
    }
  }
}
```
### Alterar el modo de vista de un nodo
```
/**
 * Implements hook_node_view().
 */
function nombre_modulo_node_view(array &$build, EntityInterface $entity, EntityViewDisplayInterface $display, $view_mode) {
  if ($entity->bundle() == 'landing_page' && $view_mode == "teaser") {
    if (empty($build['hea'][0])) {
      $id_empty_image = 39;
      $media_entity = Media::load($id_empty_image);

      $build['hea']['#theme'] = 'field';
      $build['hea']['#title'] = 'Header image';
      $build['hea']['#label_display'] = 'hidden';
      $build['hea']['#view_mode'] = 'teaser';
      $build['hea']['#language'] = $entity->language()->getId();
      $build['hea']['#field_name'] = 'hea';
      $build['hea']['#field_type'] = 'entity_reference';
      $build['hea']['#field_translatable'] = FALSE;
      $build['hea']['#entity_type'] = 'node';
      $build['hea']['#bundle'] = 'landing_page';
      $build['hea']['#object'] = $entity;
      $build['hea']['#formatter'] = 'entity_reference_entity_view';
      $build['hea']['#is_multiple'] = FALSE;
      $build['hea']['#third_party_settings'] = [];
      $build['hea'][0] = [
        '#media' => $media_entity,
        '#view_mode' => 'card',
        '#cache' => [
          'tags' => [
            '0' => "media:" . $id_empty_image,
            '1' => "media_view",
          ],
          'contexts' => [
            '0' => "route.name.is_layout_builder_ui",
            '1' => "user.permissions",
          ],
          'max-age' => -1,
          'keys' => [
            '0' => "entity_view",
            '1' => "media",
            '2' => $id_empty_image,
            '3' => "card",
          ],
          'bin' => "render",
        ],
        '#theme' => 'media',
        '#weight' => 0,
        '#pre_render' => [
          '0' => [
            '0' => \Drupal::entityTypeManager()->getViewBuilder('media'),
            '1' => 'build',
          ],
        ],
      ];
      $build['hea']['#weight'] = 0;
    }
  }
}
```

ENLACES Y FUENTES
=================
APi de actualización
- https://www.drupal.org/docs/drupal-apis/update-api/introduction-to-update-api-for-drupal-8

Lista de hooks
- https://api.drupal.org/api/drupal/core!core.api.php/group/hooks/8.2.x


Que son los hooks
- https://drupalize.me/tutorial/what-are-hooks?p=2766
- https://www.drupal.org/docs/creating-custom-modules/understanding-hooks

Ejemplos Hooks update
- https://www.drupal.org/docs/8/api/update-api/updating-database-schema-andor-data-in-drupal-8
- https://www.drupal.org/docs/7/creating-custom-modules/howtos/examples-for-database-update-scripts-using-hook_update_n-how

Hooks post update
- https://api.drupal.org/api/drupal/core%21lib%21Drupal%21Core%21Extension%21module.api.php/function/hook_post_update_NAME/8.2.x