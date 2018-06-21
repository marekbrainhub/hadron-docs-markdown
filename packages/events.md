## Installation

```bash
npm install @brainhubeu/hadron-events --save
```

[More info about installation](/core/#installation)

## Overview

Event Manager is a tool which allows for manipulating Hadron's default behavior without the need to change the code base. It can be achieved via custom listeners defined by the developer. There are a bunch of extension points spread all over the Hadron framework where listeners can be hooked up.

## Initializing

Pass the package as an argument for Hadron bootstrapping function:

```javascript
const hadronEvents = require('@brainhubeu/hadron-events');
// ... importing and initializing other components

hadron(expressApp, [hadronEvents], config).then(() => {
  console.log('Hadron with eventManager initialized');
});
```

After initialization you can retrieve the event manager from the DI container - it is registered under the key `eventManager`.

## Event Manager methods

### Registering listeners for events

```javascript
eventManager.registerEvents(listeners);
```

* `listeners` - an array of objects which have to follow the convention shown below:

```javascript
{
  name: 'string',  // listener name
  event: 'string', // event to register to
  handler: 'function' // function to handle the event
}
```

Example:

```javascript
const config = {
  events: {
    listeners: [
      {
        name: 'Listener1',
        event: 'createRoutesEvent',
        handler: (callback, ...args) => {
          const myCustomCallback = () => {
            console.log("Hey! I've changed the original hadron function!");
            return callback(...args);
          };
          return myCustomCallback();
        },
      },
      {
        name: 'Listener2',
        event: 'myCustomEvent',
        handler: (callback, ...args) => {
          const myCustomCallback = () => {
            console.log('My custom event!');
            return callback(...args);
          };
          return myCustomCallback();
        },
      },
    ],
  },
};

hadron(app, [hadronEvents], config).then((container) => {
  container.take('eventManager').emitEvent('myCustomEvent'); // "My custom event!"
});
```

### Emitting events

```javascript
eventEmitter.emitEvent(eventName);
```

Calls all listener handlers registered for the event with event name passed to it.

* `eventName` - name of the event which will be fired

## Listeners

You can create your listeners in the main config file.

As the first argument listener's handler method will receive a callback function originally called by hadron, so you can change/override it however you want and then return a call of a newly created function or a call of an existing callback if you don't want to change it.

To be able to receive the callback mentioned above, the first argument should be named exactly `callback`, otherwise, you will not receive the callback.

You can also define your listener's handler without `callback` argument or even without any arguments, which is also a valid way to create listeners, you just won't be able to access the callback.

The second argument of the listener's handler method is `...args`, which can be used as arguments for the callback function.

An example of a listener:

```javascript
{
  name: 'Listener',
  event: 'createRoutesEvent',
  handler: (callback, ...args) => {
    const myCustomCallback = () => {
      console.log("Hey! I've changed the original hadron function!");
      return callback(...args);
    }
    return myCustomCallback();
  }
}
```

## Extension points in Hadron

As said before, there are a couple of extension points in the Hadron framework to which you can hook up your listeners.
The extension depends on the packages that you are using and are listed below:

--- hadron-express

`HANDLE_REQUEST_CALLBACK_EVENT`

Event fires before a route callback function is called. Passes the route callback to the listener.

Example:

```javascript
const ExpressEvent = require('@brainhubeu/hadron-express').Event;
const listeners = [
  {
    name: 'Listener',
    event: ExpressEvent.HANDLE_REQUEST_CALLBACK_EVENT, // or simply event: 'HANDLE_REQUEST_CALLBACK_EVENT'
    handler: (callback, ...args) => {
      console.log('Request Handled!');
      callback(...args);
    },
  },
];
```

---

`HANDLE_TERMINATE_APPLICATION_EVENT`

Event fires when the application is terminated with <kbd>CTRL</kbd> + <kbd>C</kbd>, passes the default Hadron callback to the listener.

```javascript
const Event = require('@brainhubeu/hadron-events').Event;
const listeners = [
  {
    name: 'Listener',
    event: Event.HANDLE_TERMINATE_APPLICATION_EVENT, // or simply event: 'HANDLE_TERMINATE_APPLICATION_EVENT'
    handler: () => {
      console.log('Application is going to close');
    },
  },
];
```
