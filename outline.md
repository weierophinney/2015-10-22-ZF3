ZF3 Presentation
================

## High Level overview

- Emphasis on **components**
- Focus on HTTP, via **PSR-7** and **middleware**
- Optimize for **PHP 7**, but support **PHP 5.5+**

## Components

### The past (ZF1)

- Components are developed within the framework repository, *and*
- are installable only with the entire framework.

### Today (ZF2)

- Components are developed within the framework repository, *but*
- can be installed individually (Composer, git submodules).

#### Notes

- "Individually" is a loose term; most components have several dependencies currently.
- This is accomplished via a semi-manual process (we trigger it for each
  release).
- If a patch lands in a component, you can only test it in your application
  before a release if you add **all** of ZF as a dependency.

### The future (2.5+)

- Components will be developed in their own repositories.
- Components will be installable individually.
- The ZF repository will **depend** on components!

```json
{
  "require": {
    "zendframework/zend-authentication": "^2.5",
    "zendframework/zend-cache": "^2.5",
    "zendframework/zend-captcha": "^2.5",
    "etc": "*"
  }
}
```

### Why?

### Ease maintenance

- Allow more teams with commit rights
- Allow deterministic fate of repositories

#### Notes

- If a repo has no contributors, and isn't receiving issues/pull requests, we
  may determine in the future to abandon it.

### Selective evolution

- Bump major versions as needed
- Framework repo can selectively upgrade components

#### Notes

- Some components need more changes than others: SM, EM, InputFilter, Forms.
- Others are stable, and may not need huge changes in the future: Escaper,
  Config, Paginator, etc.
- Let them evolve on their own timelines; the framework can upgrade them
  selectively.

### Use-case specific skeletons

- Delivering only web services?
- Need performance?
- Need everything?

#### Notes

- If you're only doing SOAP, XML-RPC, or JSON-RPC, you might do a small subset
  of components. Or if you're doing REST, you might omit things like Navigation,
  Form, XmlRpc, Soap, and session.
- If you're focusing on performance, you might do a different subset.

## HTTP, PSR-7, and Middleware

Probably lift some of this from the other PSR-7 talk. Should introduce PSR-7 and
Middleware FIRST, before discussing how it affects ZF MVC.

### HTTP is the foundation of the web

- client sends request
- server returns a response

### HTTP messages

- Request

```http
GET /path HTTP/1.1
Host: example.com
Accept: application/json
```

- Response

```http
HTTP/1.1 200 OK
Content-Type: application/json

{"foo":"bar"}
```

### Frameworks model messages

But every framework does it differently.

### PSR-7: Shared HTTP message interfaces

So that we can write for HTTP, and share our code regardless of the framework.

### Middleware: between the request and response

```php
function (Request $request, Response $response)
{
    // do some work
    return $response; // any response will do
}
```

### Many types of middleware

- Stacks or pipelines:

```php
$pipeline->pipe($middleware1);
$pipeline->pipe($middleware2);
$pipeline->pipe($middleware3);
```

- Onion-style:

```php
class Outer
{
    public $inner;

    public function __invoke(Request $request, Response $response)
    {
        // do some work

        $response = ($this->inner)($request, $response);

        // do some work
        return $response;
    }
}
```

- Lambda-style:

```php
function (Request $request)
{
    // do some work
    return $response;
}
```

### PSR-7 and the ZF MVC

Still under discussion, but...

- we may consume PSR-7 directly
- or create a bridge implementation

#### Notes

- Bridging would happen in order to retain our current API, while manipulating a
  PSR-7 instance underneath. This would give us compatibility with both worlds,
  and make for an easier migration path. PSR-7 users would still be supported.

### Consuming middleware

- ZF will allow dispatching middleware that follows the pattern:

```php
public function (Request $request, Response $response) : Response
```

- ZF controllers will be middleware themselves

#### Notes

- ZF's DispatchableInterface is essentially this middleware interface already.

### Middleware wrappers for ZF applications

```php
$middleware = new MvcMiddlewareWrapper(
    require 'config/application/config.php'
);

class MvcMiddlewareWrapper
{
    public function __invoke($request, $response)
    {
        $app = Application::init($this->config);
        return $app->dispatch($request, $response);
    }
}
```

#### Notes

- Request/response are PSR-7 messages
- We'll likely do something like this for ZF1, too, which can bridge the
  request/response instances.
- Requires we have a new path that returns the response, instead of emitting it.
- Lazy-initialization so that it only loads ZF if the middleware is invoked!

### Middleware as an alternative runtime

Why?

- Performance
- Developer experience
- Reusability across frameworks

#### Notes

- Performance: lazy-loading of resources based on opportunistic path matching;
  slimmer runtime engine (typically). You **can** run into the same
  performance issues of any web application; it's a matter of how you
  architect things.
- DX: simpler patterns; usually do only one thing per middleware; compose
  middleware in order to accomplish complex workflows; etc.
- If you target middleware, and not the framework, you create reusable
  web-facing widgets!

### Example

```php
$app = new Middleware();
$app->pipe('/', $homepage);
$app->pipe('/customer', $zf2Middleware);
$app->pipe('/products', $zf2Middleware);
$app->pipe('/api', $apigility);
$app->pipe('/user', $userMiddleware);

$server->listen($app);
```

## PHP 7 and PHP 5.5

### Upgrade to 5.5

- We get to use traits!
- We get to use short array syntax!
- We get to use the callable type hint!
- We get to use `finally`!
- No more password hashing shims!
- We can use the `::class` magic constant!
- We get to use generators!
- We get faster, more secure PHP!

### Optimize for PHP 7

- Running tests in strict and/or coercive mode.
  - Allowing us to fix bugs for PHP 5, and also forward-proof code for PHP 7.
- Potentially automate release builds that add scalar typehints from docblock
  annotations.
- Profiling in PHP 7 to see where we can optimize architecture for performance.

#### Notes

- The last is important, as a lot of work hsa been done in PHP 7 to streamline
  the object call graph; if we profile early, we can make changes that will
  benefit PHP 7 users specifically.

## Summary

- Welcome to a true **component** library, built with Composer in mind!
- Welcome to an **HTTP-centric** framework, utilizing the best enterprise stack
  alongside an optimized middleware runtime!
- Welcome to the future of PHP, using the latest and greatest features in order
  to deliver you a better, more performant runtime!
