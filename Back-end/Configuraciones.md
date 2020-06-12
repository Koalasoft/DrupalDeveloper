Configuraciones
===

#### Operaciones con el API
```
//Cron
\Drupal::config('system.cron')->get('threshold.autorun')
\Drupal::state()->get('system.cron_last')

//Email
\Drupal::config('user.mail')->get('register_admin_created.subject');
\Drupal::config('user.mail')->get('register_admin_created.body');

//Escribiendo configuraciones
$config = \Drupal::service('config.factory')->getEditable('system.performance');
$config->set('cache.page.enabled', 1); // Set a scalar value.
$page_cache_data = array('enabled' => 1, 'max_age' => 5); // Set an array of values.
$config->set('cache.page', $page_cache_data);
$config->save();

//Servicios
$resource_id = 'product_resource';
$resources = \Drupal::config('rest.settings')->get('resources') ?: array();
$resources[$resource_id] = array(); // reset de resource configuration
$method = "GET";

$resources[$resource_id][$method] = array();
$resources[$resource_id][$method]['supported_formats'] = array("json");
$resources[$resource_id][$method]['supported_auth'] = array("cookie");

\Drupal::configFactory()->getEditable('rest.settings')
  ->set('resources', $resources)
  ->save();

//Sistema
\Drupal::configFactory()->getEditable('system.site')
  ->set('page.404', 'not-found')
  ->save();

```