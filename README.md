# express-dinja

[![Build Status][travis-image]][travis-url] [![Coverage Status][coveralls-image]][coveralls-url]

Use dependency injection pattern for Express 4.x applications. Inspired by [express-di](https://github.com/luin/express-di).

## Usage

```js
var express = require('express');
var app = express();
var inject = require('express-dinja')(app);

inject('injected', function (req, res, next) {
    next(null, 'injected');
});

inject('dependency', function (injected, req, res, next) {
    next(null, 'dependency ' + injected);
});

app.get('/', function (dependency, req, res) {
    res.json({
        dinja: dependency
    });
});

require('http').createServer(app).listen(8080);
```

On [localhost:8080](http://localhost:8080) you should see:

```json
{
    "dinja": "dependency injected"
}
```

## Why

Suppose you have this graph of middleware dependencies:

![middlewares](https://cloud.githubusercontent.com/assets/365089/2589017/c0292b1a-ba45-11e3-9a1b-57e63d5cdcd2.png)


In express there is no built-in way to start execution of middlewares parallel, so you have two choices:

 1. Linearize middlewares tree and launch them one after one &mdash; and drop performance of app
 2. Write meta-middleware, that will launch independent middlewares side-by-side &mdash; and write boilerplate code

To reduce boilerplate code (you can see it in statusTodos function below) dependency injection pattern was added to express route function.
Here is example how would applications look in plain express and with express-dinja:

![plain-express-vs-dinja](https://cloud.githubusercontent.com/assets/365089/4331364/d3630bca-3fc3-11e4-98cf-4e3ab8b1e6da.png)

## Difference from express-di

This module is heavily based on code from `express-di`, but has additional features, that I think is necessary for full dependency injection.

 * Dependency resolving in injected dependencies
 * No `express.Route.prototype` patching
 * No hardcoded cache
 * No dependency inheritance in mounted apps (todo)

## Benchmark

I used benchmark from [`express-di`](https://github.com/luin/express-di/tree/master/benchmarks) to compare bare express application performance with patched version. Benchmark takes application with one middleware, that uses dependency injection and after middleware is done - sends 'Hello world' response.

Middleware is faked database connection, which will response in predefined time (horizontal bar) and requests/sec is the vertical bar:

![Performance chart](https://cloud.githubusercontent.com/assets/365089/2590257/9d323e76-ba59-11e3-8ea9-66bae5854c46.png)

## API

### dinja(app)

Returns `Function`, that can inject dependencies into express application `app`. It will patch `route` method to enable injections in route-specific methods like `use`, `get`, `post` and etc.

### inject(name, fn)

Injects dependency with name `name` and dependent express middlewares `fn`.

`fn` almost identically inherits express middleware signature: `function([dependencies], req, res, next)` but with two differences:

 1. It can have own dependencies, that will be resolved, when this middleware is called.
 2. `next(err, value)` function accepts two arguments: `err` and `value` of dependency.

`req`, `res` and `next` names are pre-defined to corresponding arguments of express middleware.

### inject.resolve(name, callback)

Resolves dependency and calls callback:

```js
function resolve(name, cb) {
    var resolved = this.dependencies[name];
    if (!resolved) {
        return cb(new Error('Unknown dependency: ' + name));
    }
    return cb(null, resolved);
}
```

You can override it to add caching.

### inject.declare(name, fn)

Stores dependency in `inject.dependencies` object:

```js
function declare(name, fn) {
    this.dependencies[name] = fn;
}
```

If you want to use shared singleton as storage, you can override this (do not forget to override `inject.resolve` as well).


## License

The MIT License (MIT) © [Vsevolod Strukchinsky](mailto:floatdrop@gmail.com)

[travis-url]: https://travis-ci.org/floatdrop/express-dinja
[travis-image]: http://img.shields.io/travis/floatdrop/express-dinja.svg?style=flat

[coveralls-url]: https://coveralls.io/r/floatdrop/express-dinja
[coveralls-image]: http://img.shields.io/coveralls/floatdrop/express-dinja.svg?style=flat
