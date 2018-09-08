# pragmatic-functional
A pragmatic approach to functional javascript in an es6 / node landscape

Pragmatic-functional adapts functional and reactive programming into a whole of applocation approach. It is a set of principles that meets functional programming at the edge - to provide a way to build modern javascript applications.

Principles
----------

1. State should be stored in one place.
1. **immutable** over mutable
1. Use reusable functions over classes.
1. Functions **where possible should be pure**
1. Where functions cannot be pure, they should be written with the injector or DI pattern, so they are predictable, testable, and isolated.
1. Use recursion over looping, and ensure that memory leaks are minimised.
1. When using Observables and other *function-storing-patterns* ensure that memory leaks are minimised.

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

In the above we can see how myNumberV<x> changes over time and if we wanted to, we could even print all of the iterations out in a console.log for comparison.

## So why can't we build our whole application as stateless?

In a whole application approach, an application cannot be a pure stateless function. As soon as a program has multiple **closures**, (for example - event listeners waiting for user input), an application-wide-stateless program fails. It fails due to the way that closures *capture* state at the time that the closure is activated. Closures take a snapshot the point of activation, and so if state updates outside of the closure - it has no concept of this.

At a minimum, a whole application approach must have at least one implementation of state.

## So why not store state in many places?

A developer should have the ability to see how state holistically changes from one iteration to the next. Not all previous states need to be kept, and in production you may only keep the current state in memory, but when in development, the developer should be able to change a setting to trace state changes over time. Having state stored in many places makes this difficult to achieve. For more on this topic - read [this article](https://github.com/attack-monkey/zeron/wiki/Article:-Immutable-State-Stores).

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
Finally, it's ok to have meta-state within a closure, or object. Meta state can for example be a map of callbacks, used in a subscription.

```javascript

/* 
 * An event emitter module that can be subscribed to and unsubscribed from.
 * Meta-state of functionMap and id are stored within the module.
 * Nothing else in the application needs to know about the state of functionMap and id.
 * id and functionMap are mutable in this example - but ideally would be immutable in a real application. 
 */

function eventEmitter() {
  let functionMap = {};
  let id = 0;
  return {
    subscribe: (func) => {
      functionMap[counter] = func;
      id = id + 1;
      return id;
    },
    unsubscribe: (id) => {
      delete functionMap(id);
      return id;
    },
    emit: (value) => {
    	Object.keys(functionMap).forEach(key => functionMap[key](value));
    }
  }
}

```

Reusable functions over classes
-------------------------------

Since functional programming keeps state and function separate, there are very few times when classes (or constructor functions, factories, etc.) are needed. Classes are great for creating instances of objects that share methods with other object of the same class. When functions and state are separate though, objects no longer have methods - they are just data.

While classes may not be as highly prized, reusability still is. Functions are made reusable by being able to perform the same operation on an input, regardless of what the input is (providing that it is at least of compatible type).

For example, our very first increment function doesn't care what number it gets, it simply returns a new number 1 larger than the number passed in.

### When to use classes, constructor functions, and factories...

So just to clear things up classes, constructor functions, and factories all produce new instances of objects, and while have some differences, can be generally grouped together.

Since state is separate to function, they're not going to get used all the time - however they are still important. For example, our event emitter above is a factory function. It produces event emitters. So as soon as some sort of state (even meta-state) is stored with function, then a class, constructor, factory should be used - so it can be reused to create instances of that thing.

Pure functions
--------------

In a functional program, a pure function exhibit the following characteristics...

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

