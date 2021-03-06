# Cache Component

[![Build Status](https://secure.travis-ci.org/reactphp/cache.png?branch=master)](http://travis-ci.org/reactphp/cache) [![Code Climate](https://codeclimate.com/github/reactphp/cache/badges/gpa.svg)](https://codeclimate.com/github/reactphp/cache)

Async, [Promise](https://github.com/reactphp/promise)-based cache interface.

The cache component provides a
[Promise](https://github.com/reactphp/promise)-based
[`CacheInterface`](#cacheinterface) and an in-memory [`ArrayCache`](#arraycache)
implementation of that.
This allows consumers to type hint against the interface and third parties to
provide alternate implementations.

**Table of Contents**

* [Usage](#usage)
  * [CacheInterface](#cacheinterface)
    * [get()](#get)
    * [set()](#set)
    * [remove()](#remove)
  * [ArrayCache](#arraycache)
* [Common usage](#common-usage)
  * [Fallback get](#fallback-get)
  * [Fallback-get-and-set](#fallback-get-and-set)
* [Install](#install)
* [License](#license)

## Usage

### CacheInterface

The `CacheInterface` describes the main interface of this component.
This allows consumers to type hint against the interface and third parties to
provide alternate implementations.

#### get()

```php
$cache
    ->get('foo')
    ->then('var_dump');
```

This example fetches the value of the key `foo` and passes it to the
`var_dump` function. You can use any of the composition provided by
[promises](https://github.com/reactphp/promise).

If the key `foo` does not exist, the promise will be rejected.

#### set()

```php
$cache->set('foo', 'bar');
```

This example eventually sets the value of the key `foo` to `bar`. If it
already exists, it is overridden. No guarantees are made as to when the cache
value is set. If the cache implementation has to go over the network to store
it, it may take a while.

#### remove()

```php
$cache->remove('foo');
```

This example eventually removes the key `foo` from the cache. As with `set`,
this may not happen instantly.

### ArrayCache

The `ArrayCache` provides an in-memory implementation of the
[`CacheInterface`](#cacheinterface).

```php
$cache = new ArrayCache();

$cache->set('foo', 'bar');
```

## Common usage

### Fallback get

A common use case of caches is to attempt fetching a cached value and as a
fallback retrieve it from the original data source if not found. Here is an
example of that:

```php
$cache
    ->get('foo')
    ->then(null, 'getFooFromDb')
    ->then('var_dump');
```

First an attempt is made to retrieve the value of `foo`. A promise rejection
handler of the function `getFooFromDb` is registered. `getFooFromDb` is a
function (can be any PHP callable) that will be called if the key does not
exist in the cache.

`getFooFromDb` can handle the missing key by returning a promise for the
actual value from the database (or any other data source). As a result, this
chain will correctly fall back, and provide the value in both cases.

### Fallback get and set

To expand on the fallback get example, often you want to set the value on the
cache after fetching it from the data source.

```php
$cache
    ->get('foo')
    ->then(null, array($this, 'getAndCacheFooFromDb'))
    ->then('var_dump');

public function getAndCacheFooFromDb()
{
    return $this->db
        ->get('foo')
        ->then(array($this, 'cacheFooFromDb'));
}

public function cacheFooFromDb($foo)
{
    $this->cache->set('foo', $foo);

    return $foo;
}
```

By using chaining you can easily conditionally cache the value if it is
fetched from the database.

## Install

The recommended way to install this library is [through Composer](http://getcomposer.org).
[New to Composer?](http://getcomposer.org/doc/00-intro.md)

This will install the latest supported version:

```bash
$ composer require react/cache:~0.4.0
```

If you care a lot about BC, you may also want to look into supporting legacy versions:

```bash
$ composer require "react/cache:~0.4.0|~0.3.0"
```

More details and upgrade guides can be found in the [CHANGELOG](CHANGELOG.md).

## License

MIT, see [LICENSE file](LICENSE).
