# Snorlax

A light-weight RESTful client built on top of [Guzzle](http://docs.guzzlephp.org/en/latest/) that gives you full control of your API's resources. Its based on method definitions and parameters for your URLs. See the usage below.

# Basic Usage

```php
<?php

use Snorlax\Resource\Resource;
use Snorlax\RestClient;

class PokemonResource extends Resource
{
    public function getBaseUri()
    {
        // You don't want a raw value like this, use an environment variable ;)
        return 'http://localhost/api/pokemons';
    }

    public function getActions()
    {
        return [
            'all' => [
                'method' => 'GET',
                'path' => '/'
            ],
            'get' => [
                'method' => 'GET',
                'path' => '/{0}.json'
            ],
            'create' => [
                'method' => 'POST',
                'path' => '/'
            ]
        ];
    }
}

$client = new RestClient([
    'resources' => [
        'pokemons' => PokemonResource::class
    ]
]);

// GET http://localhost/api/pokemons?sort=id:asc
$response = $client->pokemons->all(['
    'query' => [
        'sort' => 'id:asc'
    ]
']);
// GET http://localhost/api/pokemons/143.json?fields=id,name
$response = $client->pokemons->get(143, [
    'query' => ['fields' => 'id,name']
]);
// POST http://localhost/api/pokemons
$response = $client->pokemons->create([
    'body' => [
        'name' => 'Bulbasaur'
    ]
]);
```

As you can see, each action on your resource is defined an array with two keys, `method`, defining the HTTP method for the request and `path`, which defines the path from the base URI returned by the `getBaseUri` method. You can mock URLs using environment variables as you wish.

`Snorlax` assume your API returns JSON, so it already returns an `StdClass` object with the response, decoded by `json_decode`. If you want to get the raw object returned by `Guzzle`, use `$client->resource->getLastResponse()`.

# Sending parameters and headers

As said before, `Snorlax` is built on top of `Guzzle`, so it works basically the same way on passing headers, query strings and request bodies.

```php
<?php

$pokemons = $client->pokemons->all([
    'query' => [
        'sort' => 'name',
        'offset' => 0,
        'limit' => 150
    ],
    'headers' => [
        'X-Foo' => 'Bar'
    ]
]);

$pokemons = $client->pokemons->create([
    'body' => [
        'name' => 'Ivysaur',
        'attacks' => ['Tackle', 'Leer']
    ]
]);
```

# Changing client options

If you want to set default headers for every request you send to the default guzzle client, just use the `headers` options on your `params` config key, just like the [Guzzle docs](http://guzzle.readthedocs.org/en/latest/request-options.html#headers).

```php
<?php

$client = new Snorlax\RestClient([
    'client' => [
        'params' => [
            'headers' => [
                'X-Foo' => 'Bar'
            ],
            'defaults' => [
                'debug' => true
            ],
            'cache' => true
        ]
    ]
]);
```

# Setting a base URI

If all your resources are under the same base URI, you can pass it on the constructor instead of declaring on the `resource` class.

```php
<?php

$client = new Snorlax\RestClient([
    'client' => [
        'params' => [
            'base_uri' => 'http://localhost/api'
        ]
    ]
]);
```

# API Authorization

`Snorlax` supports two types of Authorization for now: `Bearer` and `Basic`. They're pretty easy to use.

```php
<?php

$client = new Snorlax\RestClient([
    // ...
]);
// Basic authorization
$client->setAuthMethod(new Snorlax\Auth\BasicAuth('user', 'password'));
// Bearer authorization
$client->setAuthMethod(new Snorlax\Auth\BearerAuth('your token'));
```

# Using a custom client

If you don't want to use `Guzzle`'s default client (or want to mock one), `Snorlax` accepts any class that implements `GuzzleHttp\ClientInterface`, so just pass your custom client in the constructor. Can be an instance or a callable.

```php
<?php

class MyOwnClient implements GuzzleHttp\ClientInterface
{
    private $config;

    public function __construct(array $params)
    {
        $this->config = $params;
    }
}

// Using a callable to instantiate a new client everytime
$client = new Snorlax\RestClient([
    'client' => [
        'custom' => function(array $params) {
            return new MyOwnClient($params);
        },
        'params' => ['param1' => 'value']
    ]
]);

$client = new Snorlax\RestClient([
    'client' => [
        'custom' => new MyOwnClient(['param1' => 1])
    ]
]);
```
