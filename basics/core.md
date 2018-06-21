## Installation

* Install Node.js. We recommend using the latest version, installation details on [nodejs.org](https://nodejs.org)

* Install following modules from npm:

```bash
npm install @brainhubeu/hadron-core @brainhubeu/hadron-express express --save
```

## Hello World app

Let's start with a simple Hello World app. It will give you a quick grasp of the framework.

```javascript
const hadron = require('@brainhubeu/hadron-core').default;
const express = require('express');

const port = 8080;
const expressApp = express();

const config = {
  routes: {
    helloWorldRoute: {
      path: '/',
      callback: () => 'Hello world!',
      methods: ['get'],
    },
  },
};

hadron(expressApp, [require('@brainhubeu/hadron-express')], config).then(() => {
  expressApp.listen(port, () =>
    console.log(`Listening on http://localhost:${port}`),
  );
});
```

In the sections below, we will describe step by step what just happened.

## Bootstrapping an app

The main hadron-core function is responsible for bootstrapping the app. It registers packages based on passed config and server instance:

```javascript
const hadron = require('hadron-core').default;

hadron(serverInstance, [...packages], config);
```

The purpose of the main function is to initialize the dependency injection (DI) container and register package dependencies according to corresponding sections in the config object (described in detail in the next chapters).

The main function returns a promise that resolves to the created DI container instance. In the promise's `.then()` method, aside from performing operations on the container instance, we can actually start our server, by calling the Express `listen` method:

```javascript
hadron(serverInstance, ...rest).then((container) => {
  // do some things on container...

  serverInstance.listen(PORT, callback);
});
```

Now, let's move to DI container itself.

## Dependency Injection

The whole framework is built around the concept of a DI container. Its purpose is to automatically supply proper arguments for routes' callbacks and other building blocks of the framework.

DI container instance is created and used internally by the bootstrapping function, it is also returned (as a promise) from the bootstrapping function, as mentioned in the previous section.

### Container methods

#### Registering items

```javascript
container.register(key, item, lifetime);
```

* `key` - item name with which it's going to be registered inside the container
* `item` - any value (primitive, data structure, function, class, etc.)
* `lifetime` - the type of the item's life-span

Lifetime options:

* `'value'` - the container returns registered item as is [default]
* `'singleton'` - the container always returns the same instance of the registered class / constructor function
* `'transient'` - the container always returns a new instance of the registered class / constructor function

#### Retrieving items

```javascript
container.take(key);
```

* `key` - item name (same as provided during registration)

The method returns an item or an item instance according to the item type and lifetime option.

#### Example usage in the bootstrapping function

```javascript
const { default: hadron, Lifetime } = require('hadron-core');

hadron(...args).then((container) => {
  container.register('foo', 123);
  container.register('bar', class Bar {}, Lifetime.Singleton);
  container.register('baz', class Baz {}, Lifetime.Transient);

  // other stuff...
});
```

### Accessing container items from route callbacks

To access container items from callbacks, you can just set arguments' names to match container keys, and required dependency will be provided.

See an example [here](../routing/#retrieving-items-from-container-in-callback)
