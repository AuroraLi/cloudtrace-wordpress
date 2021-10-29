# Set up Cloud Trace for Wordpress

## Install OpenCensus composer package

```
cd /var/www/html
sudo composer require opencensus/opencensus:~0.2
```

## Configure PHP
Add the following line to your `php.ini` file at `/etc/php/x.x/apache2/`
```
extension=opencensus.so
```

## Configure Wordpress
Add the following lines to your `wp-config.php` file at `/var/www/html/wp-config.php`. Replace <your GCP Project ID> with your project ID.
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use OpenCensus\Trace\Exporter\StackdriverExporter;
use OpenCensus\Trace\Tracer;

OpenCensus\Trace\Integrations\Wordpress::load();


$projectId = '<your GCP Project ID>';

# [START trace_setup_php_exporter_setup]
$exporter = new StackdriverExporter([
    'clientConfig' => [
        'projectId' => $projectId
    ]
]);

//$exporter = new StackdriverExporter();
Tracer::start($exporter);


function trace_callable()
{
    # [START trace_setup_php_span_with_closure]
    Tracer::inSpan(
        ['name' => 'slow_function'],
        function () {
            sleep(1);
        }
    );
    # [END trace_setup_php_span_with_closure]
}
```

Restart PHP
```
sudo systemctl restart apache2
```

## Check GCP Permissions
Go into VM instances Details page, In API and identity management section, make sure `Stackdriver Trace` is set to `Write Only`.

Go to IAM page, make sure your default service account (or the service account the Wordpress VM uses) have `Cloud Trace Agent` role. 

## View traces in GCP Console.
Go to Trace page. New traces will be tracked. 

## Configure more integrations
Refer to [framework integrations](https://cloud.google.com/trace/docs/setup/php#framework_integrations) to set up more integrations with Cloud SQL and other systems