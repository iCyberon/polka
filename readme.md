<div align="center">
  <img src="/polka.png" alt="Polka" width="400" height="300" />
</div>

<h1 align="center">Polka</h1>

<div align="center">
  <a href="https://npmjs.org/package/polka">
    <img src="https://img.shields.io/npm/v/polka.svg" alt="version" />
  </a>
  <a href="https://travis-ci.org/lukeed/polka">
    <img src="https://img.shields.io/travis/lukeed/polka.svg" alt="travis" />
  </a>
  <a href="https://npmjs.org/package/polka">
    <img src="https://img.shields.io/npm/dm/polka.svg" alt="downloads" />
  </a>
</div>

<div align="center">A micro web server so fast, it'll make you dance! :dancers:</div>

<br />

Polka is an extremely minimal, highly performant Express.js alternative. Yes, you're right, Express is _already_ super fast & not _that_ big :thinking: --- but Polka shows that there was (somehow) room for improvement!

Essentially, Polka is just a [native HTTP server](https://nodejs.org/dist/latest-v9.x/docs/api/http.html#http_class_http_server) with added support for routing, middlewares, and sub-applications (TODO). That's it! :tada:

And, of course, in mandatory bullet-point format:

* 33-50% faster than Express for simple applications
* Middleware support, including Express middleware you already know & love
* Nearly identical application API & route pattern definitions
* 70 LOC for Polka, 105 including [its router](https://github.com/lukeed/trouter)


## Install

```
$ npm install --save polka
```

## Usage

```js
const polka = require('polka');

polka()
  .use(one, two)
  .get('/users/:id', (req, res) => {
    res.end(`User: ${req.params.id}`);
  })
  .listen(3000);
```

## API

### use(...fn)

Attach middleware(s) and/or sub-application(s) (TODO) to the server. These will execute _before_ your routes' handlers.

#### fn
Type: `Function|Array`

You may pass one or more function(s) at a time. Each function must have the standardized `(req, res, next)` signature.

Please see [`handler`](#handler) and [Express' middleware examples](http://expressjs.com/en/4x/api.html#middleware-callback-function-examples) for more info.

### parse(req)

Returns: `Object`

This is an alias of the awesome [`parseurl`](https://github.com/pillarjs/parseurl#api) module. There are no Polka-specific changes.

### start(port, hostname)

Returns: `Promise`

Wraps the native [`server.listen`](https://nodejs.org/dist/latest-v9.x/docs/api/http.html#http_server_listen) with a Promise, rejecting on any error.

### listen(port, hostname)

Returns: `Promise`

This is an alias of [`start`](#start).

### send(res, code, body, type)

A minimal helper that terminates the [`ServerResponse`](https://nodejs.org/dist/latest-v9.x/docs/api/http.html#http_class_http_serverresponse) with desired values.

#### res
Type: `ServerResponse`

#### code
Type: `Number`<br>
Default: `200`

#### body
Type: `String`<br>
Default: `http.STATUS_CODES[code]`

Returns the default `statusText` for a given [`code`](#code).

#### type
Type: `String`<br>
Default: 'text/plain'

The `Content-Type` header value for the response.

### handler(req, res, next)

The main Polka [`ClientRequest`](https://nodejs.org/dist/latest-v9.x/docs/api/http.html#http_class_http_clientrequest) handler. It receives all requests and tries to match the incoming URL against known routes.

If the `req.url` is not matched, a `(501) Not Implemented` response is returned. Otherwise, all middlewares will be called & then, finally, the route handler (user-defined) will be called --- assuming that a middleware hasn't already returned a response or thrown an error!

#### req
Type: `ClientRequest`

#### res
Type: `ServerResponse`

#### next
Type: `Function`

An optional function (with the same signature) to call at the end of the response cycle.


## Routing

Routes are used to define how an application responds to varying HTTP methods and endpoints.

> If you're coming from Express, there's nothing new here!<br> However, do check out [Comparisons](#comparisons) for some pattern changes.

### Basics

Each route is compromised of a pathname, a HTTP method, and a handler (aka, what you want to do).

In code, this looks like:

```js
app.METHOD(pattern, handler);
```

wherein:

* `app` is an instance of `polka`
* `METHOD` is any valid HTTP method, lowercased
* `pattern` is a [routing pattern](#patterns) string
* `handler` is the function executed when the `pattern` is matched

Also, a single pathname (or `pattern`) may be reused with multiple METHODs.

The following examples demonstrate some simple routes.

```js
const app = polka();

app.get('/', (req, res) => {
  res.end('Hello world!');
});

app.get('/users', (req, res) => {
  res.end('Get all users!');
});

app.post('/users', (req, res) => {
  res.end('Create a new User!');
});

app.put('/users/:id', (req, res) => {
  res.end(`Update User with ID of ${req.params.id}`);
});

app.delete('/users/:id', (req, res) => {
  res.end(`CY@ User ${req.params.id}!`);
});
```

### Patterns

Unlike the very popular [`path-to-regexp`](https://github.com/pillarjs/path-to-regexp), Polka uses string comparison to locate route matches. While [faster](https://github.com/lukeed/matchit#benchmarks) & more memory efficient, this does also prevent complex pattern matching.

However, have no fear! :boom: All the basic and most commonly used patterns are supported. You probably only ever used these patterns in the first place :wink:

> See [Comparisons](#comparisons) for the list of `RegExp`-based patterns that Polka does not support.

The supported pattern types are:

* static (`/users`)
* named parameters (`/users/:id`)
* nested parameters (`/users/:id/books/:title`)
* optional parameters (`/users/:id?/books/:title?`)
* any match / wildcards (`/users/*`)

### Parameters

Any named parameters included within your route [`pattern`](#patterns) will be automatically added to your incoming `req` object. All parameters will be found within `req.params` under the same name they were given.

> **Important:** Your parameter names should be unique, as shared names will overwrite each other!

```js
app.get('/users/:id/books/:title', (req, res) => {
  let { id, title } = req.params;
  res.end(`User: ${id} && Book: ${title}`);
});
```

```sh
$ curl /users/123/books/Narnia
#=> User: 123 && Book: Narnia
```

### Methods

Any valid HTTP method is supported! However, only the most common methods are used throughout this documentation for demo purposes.

> **Note:** For a full list of valid METHODs, please see [this list](http://expressjs.com/en/4x/api.html#routing-methods).

### Handlers

Request handlers accept the incoming [`ClientRequest`](https://nodejs.org/dist/latest-v9.x/docs/api/http.html#http_class_http_clientrequest) and the formulating [`ServerResponse`](https://nodejs.org/dist/latest-v9.x/docs/api/http.html#http_class_http_serverresponse).

Every route definition must contain a valid `handler` function, or else an error will be thrown at runtime.

> **Important:** You must _always_ terminate a `ServerResponse`!

It's a **very good** practice to _always_ terminate your response ([`res.end`](https://nodejs.org/api/http.html#http_request_end_data_encoding_callback)) inside a handler, even if you expect a [middleware](#middlewares) to do it for you. In the event a response is/was not terminated, the server will hang & eventually exit with a `TIMEOUT` error.

> **Note:** This is a native `http` behavior.

#### Async Handlers

If using Node 7.4 or later, you may leverage native `async` and `await` syntax! :heart_eyes_cat:

No special preparation is needed &mdash; simply add the appropriate keywords.

```js
const app = polka();

const sleep = ms => new Promise(r, setTimeout(r, ms));

async function authenticate(req, res, next) {
  let token = req.getHeader('authorization');
  if (!token) return app.send(res, 401);
  req.user = await Users.find(token); // <== fake
  next(); // done, woot!
}

app
  .use(authenticate)
  .get('/', async (req, res) => {
    // log middleware's findings
    console.log('~> current user', req.user);
    // force sleep, because we can~!
    await sleep(500);
    // send greeting
    res.end(`Hello, ${req.user.name}`);
  });
```


## Middleware

Middleware are functions that run in between (hence "middle") receiving the request & executing your route's [`handler`](#handlers) response.

> Coming from Express? Use any middleware you already know & love! :tada:

The middleware signature receives the request (`req`), the response (`res`), and a callback (`next`).

These can apply mutations to the `req` and `res` objects, and unlike Express, have access to `req.params`, `req.pathname`, `req.search`, and `req.query`!

Most importantly, a middleware ***must*** either call `next()` or terminate the response (`res.end`). Failure to do this will result in a never-ending response, which will eventually crash the `http.Server`.

```js
// Log every request
function logger(req, res, next) {
  console.log(`~> Received ${req.method} on ${req.url}`);
  next(); // move on
}

function authorize(req, res, next) {
  // mutate req; available later
  req.token = req.getHeader('authorization');
  req.token ? next() : ((res.statusCode=401) && res.end('No token!'));
}

polka().use(logger, authorize).get('*', (req, res) => {
  console.log(`~> user token: ${req.token}`);
  res.end('Hello, valid user');
});
```

```sh
$ curl /
# ~> Received GET on /
#=> (401) No token!

$ curl -H "authorization: secret" /foobar
# ~> Received GET on /foobar
# ~> user token: secret
#=> (200) Hello, valid user
```

In Polka, middleware functions are mounted globally, which means that they'll run on every request (see [Comparisons](#comparisons). Instead, you'll have to apply internal filters to determine when & where your middleware should run.

> **Note:** This might change in Polka 1.0 :thinking:

```js
function foobar(req, res, next) {
  if (req.pathname.startsWith('/users')) {
    // do something magical
  }
  next();
}
```

### Middleware Errors

If an error arises within a middleware, the loop will be exited. This means that no other middleware will execute & neither will the route handler.

Similarly, regardless of `statusCode`, an early response termination will also exit the loop & prevent the route handler from running.

There are three ways to "throw" an error from within a middleware function.

> **Hint:** None of them use `throw` :joy_cat:

1. **Pass any string to `next()`**

    This will exit the loop & send a `500` status code, with your error string as the response body.

    ```js
    polka()
      .use((req, res, next) => next('💩'))
      .get('*', (req, res) => res.end('wont run'));
    ```

    ```sh
    $ curl /
    #=> (500) 💩
    ```

2. **Pass an `Error` to `next()`**

    This is similar to the above option, but gives you an window in changing the `statusCode` to something other than the `500` default.

    ```js
    function oopsies(req, res, next) {
      let err = new Error('Try again');
      err.code = 422;
      next(err);
    }
    ```

    ```sh
    $ curl /
    #=> (422) Try again
    ```

3. **Terminate the response early**

    Once the response has been ended, there's no reason to continue the loop!

    This approach is the most versatile as it allows to control every aspect of the outgoing `res`.

    ```js
    function oopsies(req, res, next) {
      if (true) {
        // something bad happened~
        res.writeHead(400, {
          'Content-Type': 'application/json',
          'X-Error-Code': 'Please dont do this IRL'
        });
        let json = JSON.stringify({ error:'Missing CSRF token' });
        res.end(json);
      } else {
        next(); // never called FYI
      }
    }
    ```

    ```sh
    $ curl /
    #=> (400) {"error":"Missing CSRF token"}
    ```


## Benchmarks

A round of Polka-vs-Express benchmarks across varying Node versions can be [found here](/bench).

> **Important:** Time is mostly spent in _your application code_ rather than Express or Polka code!<br> Switching from Express to Polka will (likely) not show such drastic performance gains.

```
# Node v9.1.0

#=> POLKA
Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     2.34ms  198.33us   8.98ms   92.70%
    Req/Sec     5.15k   152.73     6.93k    85.70%
  1233025 requests in 30.10s, 127.00MB read
Requests/sec:  40962.46
Transfer/sec:      4.22MB

#=> EXPRESS
Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     3.22ms  406.21us  10.57ms   92.21%
    Req/Sec     3.87k   248.19    10.42k    98.00%
  924698 requests in 30.10s, 115.52MB read
Requests/sec:  30716.24
Transfer/sec:      3.84MB
```


## Comparisons

Polka's API aims to be _very_ similar to Express since most Node.js developers are already familiar with it. If you know Express, you already know Polka! :dancer:

There are, however, a few main differences. Polka does not support or offer:

1) Any built-in view/rendering engines.

    Most templating engines can be incorporated into middleware functions or used directly within route handler.

2) The ability to `throw new Error`s from within middleware.

    However, all other forms of middleware-errors are supported.

    ```js
    function middleware(res, res, next) {
      // pass an error message to next()
      next('uh oh');

      // pass an Error to next()
      next(new Error('🙀'));

      // send an early, customized error response
      res.statusCode = 401;
      res.end('Who are you?');
    }
    ```

3) Response helpers... yet!

    Express has a nice set of [response helpers](http://expressjs.com/en/4x/api.html#res.append). While Polka relies on the [native Node.js response methods](https://nodejs.org/dist/latest-v9.x/docs/api/http.html#http_class_http_serverresponse), it would be very easy/possible to attach a global middleware that contained a similar set of helpers. (TODO)

4) The `.use()` method does not accept a `pathname` filter.

5) `RegExp`-based route patterns.

    Polka's router uses string comparison to match paths against patterns. It's a lot quicker & more efficient.

    The following routing patterns **are not** supported:

    ```js
    app.get('/ab?cd', _ => {});
    app.get('/ab+cd', _ => {});
    app.get('/ab*cd', _ => {});
    app.get('/ab(cd)?e', _ => {});
    app.get(/a/, _ => {});
    app.get(/.*fly$/, _ => {});
    ```

    The following routing patterns **are** supported:

    ```js
    app.get('/users', _ => {});
    app.get('/users/:id', _ => {});
    app.get('/users/:id?', _ => {});
    app.get('/users/:id/books/:title', _ => {});
    app.get('/users/*', _ => {});
    ```

6) Sub-applications...yet! (TODO 1.0)


## License

MIT © [Luke Edwards](https://lukeed.com)
