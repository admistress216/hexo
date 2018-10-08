---
title: dingo总结
date: 2018-10-08 11:07:48
tags:
---
### 1. api request示例
```php
// 1.1header
Accept: application/vnd.lumen.v1+json

// 1.2.env文件
API_PREFIX=api
API_STANDARDS_TREE=vnd
API_SUBTYPE=lumen
API_VERSION=v1
API_STRICT=false
API_DEFAULT_FORMAT=json
API_DEBUG=true

// 1.3route:/routes/api/v1.php
$api = app('Dingo\Api\Routing\Router');

	// v1 version API
	// add in header    Accept:application/vnd.lumen.v1+json
$api->version('v1', [
    'namespace' => 'App\Http\Controllers\Api\V1',
    'middleware' => [
        'cors',
        'serializer',
         //'serializer:array', // if you want to remove data wrap
        'api.throttle',
    ],
    // each route have a limit of 20 of 1 minutes
    'limit' => 20, 'expires' => 1,
], function ($api) {
    // Auth
    // login
    $api->post('authorizations', [
        'as' => 'authorizations.store',
        'uses' => 'AuthController@store',
    ]);

        $api->get('test', [
                'as' => 'test.index',
                'uses' => 'TestController@index',
        ]);
});

// 1.4controller:/app/Http/Controllers/Api/V1/TestController.php
<?php

namespace App\Http\Controllers\Api\V1;

use Illuminate\Http\Request;
use App\Models\Authorization;
use App\Transformers\AuthorizationTransformer;

class TestController extends BaseController
{

        public function index() {
                echo 'index';
        }
}
```
