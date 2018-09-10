# App-Wide-Functional-js

An application wide approach to functional / reactive programming in an es6+ / node landscape

awf.js provides a set of principles, reasons, and patterns that meet functional programming at the edge - to provide a way to build modern javascript applications.

Example stack
-------------

```

frontend

- zeron {
  component-api - for components / views
  state/store-api - for managing state in one place
  router-api - for managing url routing
}
- rxjs - for event driven reactive programming
- ramda - for low level functional programming 

```

Principles
----------

1. State should be stored in one place.
1. **Immutable** over Mutable.
1. Use reusable functions over classes.
1. Functions **where possible should be pure**
1. Where functions cannot be pure, they should be written with the injector or DI pattern, so they are predictable, testable, and isolated.
1. Use recursion over looping, and ensure that memory leaks are minimised.
1. When using Observables and other *function-storing-patterns* ensure that memory leaks are minimised.
1. Use semantic file names to indicate the type of module eg. `home.function.ts`, `thing.factory.ts`.

## Functional vs Object Oriented

The big difference between a functional program and an Object Oriented one is the treatment of state and functions.

In a functional program state and function is separate...

```javascript

const state = 0;

function increment(state) {
  return state + 1;
}

```

In the above, the function is **stateless**. It has no concept of it's own state, other than that which is passed in.

... and in OO state (properties) and functions (methods) tend to reside on the object

```javascript

stateAndFunction = class {
  	constructor() {
    	this.state = 0;
    }
	increment() {
    	this.state++;
    }
}

```

In the above, the class is **stateful**. It stores state.

## Immutability over Mutability

In functional programming the function should not alter state, but rather return a new value - **state 2.0**. 
This keeps things clean as programs become more complex.

*E.g. Using the increment function from above...*

```javascript

const myNumberV1 = 0;
const myNumberV2 = increment(myNumberV1); // 1
const myNumberV3 = increment(myNumberV2); // 2

```

> In the above we can see how myNumberV**x** changes over time and if we wanted to, we could print all of the iterations out in a console.log for comparison. This is the benefit of immutability.

Writing immutable functions are easy when dealing with strings and numbers, a little trickier when dealing with arrays, but  much harder when dealing with objects. Using a utility like [Zeron's](https://www.npmjs.com/package/zeron) `iu` (immutable update) function makes dealing with objects (and whole state a breaze).

## Stateful vs Stateless

Pure functions, including the `increment` function above are stateless. They don't hold any state.
It stands to reason that if we were to apply an application-wide functional approach, that it too would be sttaeless - but in short in can't be, and in fact there are advantages to holding some state.

### So why can't we build our whole application as stateless?

In a whole application approach, an application cannot be a pure stateless function. As soon as a program has multiple **closures** (for example - event listeners waiting for user input), an application-wide-stateless program fails. It fails due to the way that closures *capture* state at the time that the closure is activated. Closures take a snapshot at the point of activation, so the closure has no idea of whether or not state has changed outside of its immediate scope.

At a minimum, a whole application approach must have at least one implementation of state.

## So why not store state in many places?

A developer should have the ability to see how state holistically changes from one iteration to the next. Not all previous states need to be kept, and in production you may only keep the current state in memory, but when in development, the developer should be able to trace state changes over time. Having state stored in many places makes this difficult to achieve. For more on this topic - read [this article](https://github.com/attack-monkey/zeron/wiki/Article:-Immutable-State-Stores).

## One Store

### Immutable State Store

To store current state and previous states in one variable, an Immutable State Store can be used. It's a fancy way of saying *An array of states* where `stateStore[0]` is current state, `stateStore[1]` is previous state and so on.

### Working State
When changing states it may be beneficial to perform several operations on the current state, before actually saving it as the new current state. This is known as *working state*.

Eg.

```javascript

const workingState1 = getCurrentState();
const workingState2 = addNewThingToState(workingState1);
saveNewCurrentState(workingState2);

```

This is totally acceptable :smile:

### Partial State
It might also be beneficial to get part of the currentState, perform operations on that smaller **partial state** and then immutably merge it back into a new current state. Again totally acceptable :smile:

### Static and Psuedo-static files
- It's also fine to have static config files for an application.
- It's also fine to dynamically build a static file at the start of application run time. So long as the application then treats the result as static.

### Meta-state
Finally, it's ok to have meta-state within a closure, or object. Meta state can for example be a map of callbacks, used in a subscription, and in the following example an id counter as well. Note that real state isn't actually stored in the object.

```javascript

/* 
 * An event emitter module that can be subscribed to and unsubscribed from.
 * Meta-state of functionMap and id are stored within the module.
 * Nothing else in the application needs to know about the state of functionMap and id.
 * id and functionMap are mutable in this example - but ideally would be immutable in a real application. 
 */

function eventEmitter() {
  const proto = {
    subscribe: function (func) {
      this.functionMap[this.id] = func;
      this.id = this.id + 1;
      return this.id;
    },
    unsubscribe: function (inputId) {
      delete this.functionMap(inputId);
      return inputId;
    },
    emit: function (value) {
    	Object.keys(this.functionMap).forEach(key => this.functionMap[key](value));
    }
  };
   
  const instance = Object.create(proto);
  instance.id = 0;
  instance.functionMap = {};
  return instance;
}

```

Reusable functions over classes
-------------------------------

Since functional programming keeps state and function separate, there are very few times when classes (or constructor functions, factories, etc.) are needed. Classes are great for creating instances of objects that share methods with other objects of the same class. When functions and state are separate though, objects no longer have methods - they are just data.

While classes may not be as highly prized in functional programming, reusability still is. If obeying the separate state from function rule, it is easy to make functions reusable.

For example, our very first increment function doesn't care what number it gets, it simply returns a new number 1 larger than the number passed in.

### When to use classes, constructor functions, and factories...

So just to clear things up classes, constructor functions, and factories all produce new instances of objects, and while have some differences, can be generally grouped together. From here on they will be referred to as factories.

Since state is separate to function, factories are not going to get used all the time - however they are still important. For example, our event emitter above is a factory function. It produces event emitter instances with properties and methods.

### Singletons

Singletons are essentially factories that only returns a single object instance. Each time the singleton is called, the **same** object is returned.

They're very important as they let the same object be imported anywhere in the application.

```javascript

const protoObject = {thing: 'new object'}

function myFactory() {
  return Object.create(protoObject);
}

const singletonObject = {
  thing: 'cat'
};

function mySingleton() {
  return singletonObject;
}

// Even though this factory produces objects that look identical, they are not the
// same instance... and therefore not equal.
const myInstance1 = myFactory();
const myInstance2 = myFactory();
console.log(myInstance1 === myInstance2); // false

// A singleton produces the sameobject instance.
const myInstance3 = mySingleton();
const myInstance4 = mySingleton();

console.log(myInstance3 === myInstance4); // true

```

Pure functions
--------------

In a functional program, a pure function exhibits the following characteristics...

- They only refer to paramaters that were passed into the function - no outside variables
- They don't alter any paramaters that is passed in
- They always return a value
- If the same arguments are passed in, the function will always return the same response
- They don't produce any side-effects (eg. print, send an email, change a global variable, etc.)

Functions **where possible should be pure** - however their are times when they simply can't be.

Functions that are predictable, testable, and isolated
------------------------------------------------------

Impure functions are not bad. They have to exist. If nothing else, to produce side-effects like displaying an on-screen view, sending an email, calling an api, etc.

In these situations though, it's important to have a strategy for testing that they are doing what they are meant to be doing - in a predictable, repeatable way. 

TODO: Explain further


Looping vs Recursion
----------------------------

A looping function mutates state (eg. i changes state)

```javascript

program();
function program() {
  var i;
  for (i = 0; i <=10; i++) {
      console.log(i);
  } 
}

```

A recursive function doesn't mutate state

```javascript

program(0);
function program(state) {
  if (state <=10) {
    console.log(state);
  	program(state + 1);
  }
}

```

However, if we were to set this to a really high number - we'd blow the memory stack. For every unreturned function call, that function remains in memory and eventually the system will run out.

To prevent this - make sure that your program is in strict mode and you return a result. This tells garbage collection that the function is no longer required in memory.

```javascript

"use strict"

program(0);
function program(state) {
  if (state <=10) {
    console.log(state);
  	return program(state + 1);
  } else {
    return;
  }
}

```

Observables and memory leaks
----------------------------

TODO: Write me

Semantic file names
-------------------

To help developers know what type of file they are working on it is best to:
a) limit the number of things that any one file exports to 1.
and
b) Semantically name the file based on what the export is

If it's a singleton, end the file name in .singleton.js
If it's a factory -> .factory.js
If it's just a function -> .function.js

If it's multiple of one thing -> .functions.js (Though usually we only want one file to one export)
If it's multiple different things being exported -> module.js (Generally not good practice to have a mixed bag, but there may be reasons)
