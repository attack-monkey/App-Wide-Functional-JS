# App-Wide-Functional-js

An application wide approach to functional programming in an es6+ / node landscape

AWF.js provides a set of principles, reasons, and patterns that make functional programming in js simple.

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
1. Use semantic file names to indicate the type of module eg. `home.function.ts` or the shorter `home.fn.ts`.

```

## Functional vs Object Oriented

The big difference between a Functional Program and an Object Oriented one is the treatment of state and functions.

In a functional program state and function is separate...

```javascript

const state = 0;

function increment(state) {
  return state + 1;
}

```

In the above, the function is **stateless**. It has no concept of it's own state, other than that which is passed in.

In an Object Oriented Program state (properties) and functions (methods) tend to reside on the object. Objects are usually constructed from either classes (es6+), constructor functions (the old school), and factories (Another pattern that also generates objects).

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

In the above, the class creates a **stateful** object. It stores state.

## Why Functional?

State is represented by pure data-objects (no methods).
This reduces the need for classes, since objects don't have methods and don't require extending.
Less classes and extensions means less complexity.
There is much less of a need for objects to reference other objects, meaning less errors, and easier tracking of changes to state.
State changes are immutable ... `e.g. newState = myFunction(state)`, again meaning less errors.

## The Functional Frontend

The Functional Frontend separates State from Function.

The function in this case is that which generates the **view** in entirity - let's call it `mainComponent`.

This could be expressed with something like...

```

const state = { ... };

const view = mainComponent(state);

display(view);

```

The `mainComponent` calls child component functions, passing in `state`. Those components in turn calls the next and so on. 
Each component returns a view, which finally returns the whole view.

Of course our application is not static. Events trigger changes in state, which in turn trigger a change in view.

Functions that alter state (or produce other side-effects) are generally known as **actions**. 

Our application can now be expressed in the following way...

```

const initialState = { ... };

app(initialState, mainComponent, actions);

```

or even `app({ ... }, mainComponent, actions);`

There is no `displayView`. Instead `app` runs the `displayView` from within. `app` also manages the functions used to generate the new state. Whenever the state is updated, `app` reruns the `displayView` passing in the new state.

In the above, `app` is responsible for rendering the components into the view, and also re-rendering the view each time the state changes. Note that `initialState` is passed into `app`, and then `app` immutably manages the state-injection process from there on.

### Example Frontend Frameworks

React is a good example of a framework that can be used in both a functional AND an OO way.

React comes with both Functional Components (they don't store state)  
and  
Class Components (which do store state)

When React is combined with something like React-Fn (Coming soon), it can be used in a completely functional way.

https://github.com/jorgebucaran/hyperapp is another example of a Functional Framework - built functional from the ground up.

https://maquettejs.org/ is another.

## The Functional Backend

TBA

## Immutability over Mutability

In functional programming the function should not alter the passed in state, but rather return a new value - **state 2.0**. 
This keeps things clean as programs become more complex.

*E.g.*

```javascript

const myNumberV1 = 0;
const myNumberV2 = increment(myNumberV1); // 1
const myNumberV3 = increment(myNumberV2); // 2

```

> In the above we can see how myNumberV**x** changes over time and if we wanted to, we could print all of the iterations out in a console.log for comparison. This is the benefit of immutability.

Writing immutable functions is easy when dealing with strings and numbers, a little trickier when dealing with arrays, but  much harder when dealing with objects.

Using [https://github.com/attack-monkey/immutable-update](Immutable Update (AKA iu) ) or [https://github.com/attack-monkey/iu-ts](Immutable Update for Typecript (iu-ts)) simplifies the immutable updating of objects greatly.

## State in one place

An initial state should be created, representing the entire application state.
From there on, only actions should modify state.

#### Working State
When changing states it may be beneficial to perform several operations on the current state, before actually saving it as the new current state. This is known as *working state*.

Eg.

```javascript

const workingState1 = getCurrentState();
const workingState2 = addNewThingToState(workingState1);
saveNewCurrentState(workingState2);

```

This is totally acceptable :smile:

#### Partial State
It might also be beneficial to get part of the currentState, perform operations on that smaller **partial state** and then immutably merge it back into a new current state. Again totally acceptable :smile:

Reusable functions over classes
-------------------------------

Since functional programming keeps state and function separate, there are very few times when classes (or constructor functions, factories, etc.) are needed. Classes are great for creating instances of objects that share methods with other objects of the same class. When functions and state are separate though, objects no longer have methods - they are just data.

While classes may not be as highly prized in functional programming, reusability still is. If obeying the separate state from function rule, it is easy to make functions reusable.

For example, in our very first increment function, it doesn't care what number it gets, it simply returns a new number 1 larger than the number passed in. This type of function can be reused throughout the entire app.

Pure functions
--------------

In a functional program, a pure function exhibits the following characteristics...

- They only refer to paramaters that were passed into the function - no outside variables
- They don't alter any paramaters that is passed in
- They always return a value
- If the same arguments are passed in, the function will always return the same response
- They don't produce any side-effects (eg. print, send an email, change a global variable, etc.)

Functions **where possible should be pure** - however their are times when they simply can't be.

Actions for example 


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
