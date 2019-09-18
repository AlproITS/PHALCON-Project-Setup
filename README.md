# PHALCON-Project-Setup
Repository ini adalah salah satu modul yang digunakan sebagai bahan ajar mata kuliah PBKK di Departemen Informatika ITS, tahun ajaran 2019/2020.

nb. Silahkan ikuti tutorial berikut ini. Jika ada bagian yang tidak dijelaskan, ikuti dulu saja ya :)

## Prasyarat
1. Web Server sudah terinstall (Apache 2, PHP 7.2)
2. Phalcon dan library sudah sesuai dengan versi PHP dan arsitektur yang akan dibuat (dalam kasus ini MVC.)

Sesuaikan nama folder tempat project kali ini akan dibuat. Contoh nama folder kali ini adalah phalcon-init.

## Langkah Project Setup

### 1. Buat folder yang akan dibutuhkan
Buat folder yang dibutuhkan, yaitu : apps, public, tests, dan vendor.

### 2. index.php
Buat file index.php di folder Public, lalu isikan dengan kode ini :
```
<?php

error_reporting(E_ALL); // mengaktifkan semua error reporting PHP
ini_set('display_errors', 1); // menampilkan error
ini_set('memory_limit', '64M'); //membatasi limit eksekusi script
header('Content-Type: text/html; charset=utf-8'); //mengirimkan standar header ke client
mb_internal_encoding('UTF-8');

setlocale(LC_TIME, 'id-ID'); //menetapkan informasi lokal

define('BASE_PATH', dirname(__DIR__)); //mendefinisikan path dasar (BASE PATH) ke variabel BASE_PATH
define('APP_PATH', BASE_PATH . '/apps'); //mendefinisikan path aplikasi (APP PATH) ke variable APP_PATH. APP PATH nanti berfungsi agar ketika kita mau mengakses file di folder /apps, kita tinggal menggunakan APP_PATH.

date_default_timezone_set('Asia/Jakarta'); //standar zona waktu lokal

require __DIR__ . '/../vendor/autoload.php'; //mengaktifkan autoload

require_once APP_PATH . '/Bootstrap.php'; //mengaktifkan file Bootstrap.php

$app = new Bootstrap('dashboard'); 

$app->init(); // inisiasi dashboard
```

### 3. composer.json
Buat file composer.json (setingkat dengan ketiga folder tadi), lalu isikan dengan kode berikut :
```
{
    "require": {
        "vlucas/phpdotenv": "^3.3"
    }
}

```
Setelah itu jalankan command
```
composer install
```
atau
```
composer update
```
Kode ini berfungsi untuk mengaktifkan library / plug-in phpdotenv, agar kita tinggal mengatur file environment project kita di file .env

nb. pastikan Composer sudah terinstall. Jika belum, silahkan install terlebih dahulu.

### 4. .env
Atur file environment seperti ini :
```
APPLICATION_ENV=local

APP_MODE=DEVELOPMENT

BASE_URL=http://phalcon-init.local \\ Pastikan BASE URL sesuai dengan URL yang sesuai di local masing-masing

# DATABASE

DB_ADAPTER=Phalcon\Db\Adapter\Pdo\Mysql
DB_HOST=localhost \\ localhost karena menggunakan DB local
DB_NAME=          \\ Sesuaikan dengan nama Database local yang digunakan
DB_USERNAME=      \\ Sesuaikan dengan username Database local yang digunakan
DB_PASSWORD=      \\ Sesuaikan dengan password Database loacl yang digunakan

```
Letakkan file .env ini didalam folder apps.

### 5. Bootstrap.php
Buat file Bootstrap.php di folder apps, lalu isikan dengan kode berikut :
```
<?php

use Phalcon\Mvc\Application;
use Phalcon\Debug;
use Phalcon\DI\FactoryDefault;

class Bootstrap extends Application
{
	private $modules;
	private $defaultModule;

	public function __construct($defaultModule)
	{
		$this->modules = require APP_PATH . '/config/modules.php';
		$this->defaultModule= $defaultModule;
	}

	public function init()
	{
		$this->_registerServices();

		$config = $this->getDI()['config'];

		if ($config->mode == 'DEVELOPMENT') {
			$debug = new Debug();
			$debug->listen();
		}
		
		/**
		 * Load modules
		 */
		$this->registerModules($this->modules);

		echo $this->handle()->getContent();
	}

	private function _registerServices()
	{
		$defaultModule = $this->defaultModule;

		if (getenv('APPLICATION_ENV') !== 'production') {
			$envFile = ((getenv('APPLICATION_ENV') === 'testing') ? '.env.test' : '.env');
			$dotEnv = Dotenv\Dotenv::create(APP_PATH, $envFile);
			$dotEnv->load();
		}
		$env = getenv('APPLICATION_ENV');

		//setup config
		if (!$env) {
			echo "Application environment not set";
			exit;
		} else {
			$config = require APP_PATH . '/config/config.php';
		}

		$di = new FactoryDefault();
		$config = require APP_PATH . '/config/config.php';
		$modules = $this->modules;

		include_once APP_PATH . '/config/loader.php';
		include_once APP_PATH . '/config/services.php';
		include_once APP_PATH . '/config/routing.php';

		$this->setDI($di);
	}
}
```
Kode diatas berfungsi untuk mengatur environment project. Apabila environment (.env) belum di-set, maka akan menampilkan error.

## 6. modules.php
Buat folder config di dalam apps, lalu buat file modules.php dengan isi sebagai berikut :
```
<?php

return array(
    'dashboard' => [
        'namespace' => 'Phalcon\Init\Dashboard',
        'webControllerNamespace' => 'Phalcon\Init\Dashboard\Controllers\Web',
        'apiControllerNamespace' => 'Phalcon\Init\Dashboard\Controllers\Api',
        'className' => 'Phalcon\Init\Dashboard\Module',
        'path' => APP_PATH . '/modules/dashboard/Module.php',
        'defaultRouting' => true,
        'defaultController' => 'dashboard',
        'defaultAction' => 'index'
    ],

    'backoffice' => [
        'namespace' => 'Phalcon\Init\BackOffice',
        'webControllerNamespace' => 'Phalcon\Init\BackOffice\Controllers\Web',
        'apiControllerNamespace' => 'Phalcon\Init\BackOffice\Controllers\Api',
        'className' => 'Phalcon\Init\BackOffice\Module',
        'path' => APP_PATH . '/modules/backoffice/Module.php',
        'defaultRouting' => true,
        'defaultController' => 'index',
        'defaultAction' => 'index'
    ],

);
```
Penjelasan terkait masing masing syntax :
#### namespace
```
Mari kita bayangkan namespace sebagai sebuah kabinet. Masing-masing laci kabinet 
tersebut sudah ada pemiliknya. Kita dapat menaruh apapun ke dalam laci kabinet 
tersebut; pensil, penghapus, apapun. Namun orang lain juga dapat menaruh barang 
yang identik di lacinya sendiri. Lalu bagaimana cara kita dapat membedakan kalau 
barang ini milik kita atau milik orang lain ? Caranya adalah kita memberi label 
untuk masing-masing laci yang ada di kabinet tersebut untuk menandai bahwa barang 
yang ada di laci tersebut milik siapa.
```
-[Penjelasan Lengkap tentang Namespace di PHP](https://medium.com/koding-kala-weekend/penjelasan-lengkap-tentang-namespace-di-php-ab356b44c34)
#### webControllerNamespace dan apiControllerNamespace
Deklarasi namespace untuk route web dan route api
#### className
Deklarasi namespace untuk tiap module
#### path
Alamat file tiap module
#### defaultRouting
Mengaktifkan default routing
#### defaultController
Mengatur controller default tiap module
#### defaultAction
Mengatur fungsi default di controller default

## 7. config.php
Didalam folder apps/config, buat file config.php dengan kode seperti berikut :
```
<?php

use Phalcon\Config;

return new Config(
    [
        'mode' => getenv('APP_MODE'), //DEVELOPMENT, PRODUCTION, DEMO

        'database' => [
            'adapter' => getenv('DB_ADAPTER'),
            'host' => getenv('DB_HOST'),
            'username' => getenv('DB_USERNAME'),
            'password' => getenv('DB_PASSWORD'),
            'dbname' => getenv('DB_NAME'),
        ],   
        
        'url' => [
            'baseUrl' => getenv('BASE_URL'),
        ],
        
        'application' => [
            'libraryDir' => APP_PATH . "/lib/",
            'cacheDir' => APP_PATH . "/cache/",
        ],

        'version' => '0.1',
    ]
);
```
Kode ini berfungsi mengaktifkan pengaturan environment di file .env

## 8. loader, service, routing
Tambahkan file loader, service, dan routing di folder apps/config

### loader.php
```
<?php

$loader = new \Phalcon\Loader();

/**
  * Load library namespace
  */
$loader->registerNamespaces(array(
	/**
	 * Load SQL server db adapter namespace
	 */
	//'Phalcon\Db\Adapter\Pdo' => APP_PATH . '/lib/Phalcon/Db/Adapter/Pdo',
	//'Phalcon\Db\Dialect' => APP_PATH . '/lib/Phalcon/Db/Dialect',
	//'Phalcon\Db\Result' => APP_PATH . '/lib/Phalcon/Db/Result',

));

$loader->register();
```

### services.php
```
<?php

use Phalcon\Logger\Adapter\File as Logger;
use Phalcon\Session\Adapter\Files as Session;
use Phalcon\Http\Response\Cookies;
use Phalcon\Security;
use Phalcon\Mvc\Dispatcher;
use Phalcon\Mvc\View;
use Phalcon\Flash\Direct as FlashDirect;
use Phalcon\Flash\Session as FlashSession;

$di['config'] = function() use ($config) {
	return $config;
};

$di->setShared('session', function() {
    $session = new Session();
	$session->start();

	return $session;
});

$di['dispatcher'] = function() use ($di, $defaultModule) {

    $eventsManager = $di->getShared('eventsManager');
    $dispatcher = new Dispatcher();
    $dispatcher->setEventsManager($eventsManager);

    return $dispatcher;
};

$di['url'] = function() use ($config, $di) {
	$url = new \Phalcon\Mvc\Url();

    $url->setBaseUri($config->url['baseUrl']);

	return $url;
};

$di['voltService'] = function($view, $di) use ($config) {
    $volt = new \Phalcon\Mvc\View\Engine\Volt($view, $di);
    if (!is_dir($config->application->cacheDir)) {
        mkdir($config->application->cacheDir);
    }

    $compileAlways = $config->mode == 'DEVELOPMENT' ? true : false;

    $volt->setOptions(array(
        "compiledPath" => $config->application->cacheDir,
        "compiledExtension" => ".compiled",
        "compileAlways" => $compileAlways
    ));
    return $volt;
};

$di['view'] = function () {
    $view = new View();
    $view->setViewsDir(APP_PATH . '/common/views/');

    $view->registerEngines(
        [
            ".volt" => "voltService",
        ]
    );

    return $view;
};

$di->set(
    'security',
    function () {
        $security = new Security();
        $security->setWorkFactor(12);

        return $security;
    },
    true
);

$di->set(
    'flash',
    function () {
        $flash = new FlashDirect(
            [
                'error'   => 'alert alert-danger',
                'success' => 'alert alert-success',
                'notice'  => 'alert alert-info',
                'warning' => 'alert alert-warning',
            ]
        );

        return $flash;
    }
);

$di->set(
    'flashSession',
    function () {
        $flash = new FlashSession(
            [
                'error'   => 'alert alert-danger',
                'success' => 'alert alert-success',
                'notice'  => 'alert alert-info',
                'warning' => 'alert alert-warning',
            ]
        );

        $flash->setAutoescape(false);
        
        return $flash;
    }
);

$di['db'] = function () use ($config) {

    $dbAdapter = $config->database->adapter;

    return new $dbAdapter([
        "host" => $config->database->host,
        "username" => $config->database->username,
        "password" => $config->database->password,
        "dbname" => $config->database->dbname
    ]);
};
```

### routing.php
```
<?php

$di['router'] = function() use ($defaultModule, $modules, $di, $config) {

	$router = new \Phalcon\Mvc\Router(false);
	$router->clear();

	/**
	 * Default Routing
	 */
	$router->add('/', [
	    'namespace' => $modules[$defaultModule]['webControllerNamespace'],
		'module' => $defaultModule,
	    'controller' => isset($modules[$defaultModule]['defaultController']) ? $modules[$defaultModule]['defaultController'] : 'index',
	    'action' => isset($modules[$defaultModule]['defaultAction']) ? $modules[$defaultModule]['defaultAction'] : 'index'
	]);
	
	/**
	 * Not Found Routing
	 */
	$router->notFound(
		[
			'namespace' => 'Phalcon\Init\Common\Controllers',
			'controller' => 'base',
			'action'     => 'route404',
		]
	);

	/**
	 * Module Routing
	 */
	foreach ($modules as $moduleName => $module) {

		if ($module['defaultRouting'] == true) {
			/**
			 * Default Module routing
			 */
			$router->add('/'. $moduleName . '/:params', array(
				'namespace' => $module['webControllerNamespace'],
				'module' => $moduleName,
				'controller' => isset($module['defaultController']) ? $module['defaultController'] : 'index',
				'action' => isset($module['defaultAction']) ? $module['defaultAction'] : 'index',
				'params'=> 1
			));
			
			$router->add('/'. $moduleName . '/:controller/:params', array(
				'namespace' => $module['webControllerNamespace'],
				'module' => $moduleName,
				'controller' => 1,
				'action' => isset($module['defaultAction']) ? $module['defaultAction'] : 'index',
				'params' => 2
			));

			$router->add('/'. $moduleName . '/:controller/:action/:params', array(
				'namespace' => $module['webControllerNamespace'],
				'module' => $moduleName,
				'controller' => 1,
				'action' => 2,
				'params' => 3
			));	

			/**
			 * Default API Module routing
			 */
			$router->add('/'. $moduleName . '/api/{version:^(\d+\.)?(\d+\.)?(\*|\d+)$}/:params', array(
				'namespace' => $module['apiControllerNamespace'] . "\\" . 1,
				'module' => $moduleName,
				'version' => 1,
				'controller' => isset($module['defaultController']) ? $module['defaultController'] : 'index',
				'action' => isset($module['defaultAction']) ? $module['defaultAction'] : 'index',
				'params'=> 2
			));
			
			$router->add('/'. $moduleName . '/api/{version:^(\d+\.)?(\d+\.)?(\*|\d+)$}/:controller/:params', array(
				'namespace' => $module['apiControllerNamespace'] . "\\" . 1,
				'module' => $moduleName,
				'version' => 1,
				'controller' => 2,
				'action' => isset($module['defaultAction']) ? $module['defaultAction'] : 'index',
				'params' => 3
			));

			$router->add('/'. $moduleName . '/api/{version:^(\d+\.)?(\d+\.)?(\*|\d+)$}/:controller/:action/:params', array(
				'namespace' => $module['apiControllerNamespace'] . "\\" . 1,
				'module' => $moduleName,
				'version' => 1,
				'controller' => 2,
				'action' => 3,
				'params' => 4
			));	
		} else {
			
			$webModuleRouting = APP_PATH . '/modules/'. $moduleName .'/config/routes/web.php';
			
			if (file_exists($webModuleRouting) && is_file($webModuleRouting)) {
				include $webModuleRouting;
			}

			$apiModuleRouting = APP_PATH . '/modules/'. $moduleName .'/config/routes/api.php';
			
			if (file_exists($apiModuleRouting) && is_file($apiModuleRouting)) {
				include $apiModuleRouting;
			}
		}
	}
	
    $router->removeExtraSlashes(true);
    
	return $router;
};
```

Ketiga file ini langsung di copy-paste saja :)

## 9. modules
Kali ini, buat folder modules di bawah folder apps (apps/modules) yang akan berisi view dari tiap modul kita. Buat dua modul, yaitu backoffice dan dashboard. Kedua folder bisa dilihat di github ini dan langsung copy paste isinya.


# Selesai
Demikian modul PHALCON Project Setup. Jika ada pertanyaan silahkan ditanyakan di kelas. Terimakasih !
