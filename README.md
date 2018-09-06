# pragmatic-functional
A pragmatic approach to functional javascript in an es6 / node landscape

Pragmatic-functional adapts functional (and reactive) programming principles to a whole of applocation approach. It is a set of principles that meets functional programming at the edge - to provide a way to build applications.

Principles
----------

1 An application should have application stored in one place.
2 An application should at any time be able to log the current state as well as x previous states for holistic comparision of state changes.
3

Functional vs Object Oriented
-----------------------------

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

In functional programming the function should not alter state, but rather return a new value - **state 2.0**. 
This keeps things clean as programs become more complex.

*E.g. Using the functional and OO examples from above...*

```javascript

const myNumberV1 = 0;
const myNumberV2 = increment(myNumberV1); // 1
const myNumberV3 = increment(myNumberV2); // 2

```

In a pragmatic-whole application approach, a program cannot be a stateless function. As soon as a program has multiple event listeners, an application-wide-stateless program fails due to the way that closures *capture* state at the time that the closure is activated. A closure will capture the state and wait until an event causes the listener to fire. When the listener fires though it only has the state that it knows about. The state may have evolved since that point.

At a minimum, a whole application approach must have at least one implementation of state. 





However, in OO - things are mutable ...

```javascript

let myNumber = new stateAndFunction();
myNumber.increment(); // myNumber.state === 1
myNumber.increment(); // myNumber.state === 2

```

OO has controls on what can be mutated and what can't, where as **immutability** is a core tenant of what functional programming is.

Pure functions
--------------

In a functional program, a pure function exhibit the following characteristics...

- They only refer to paramaters that were passed into the function - no outside variables
- They don't alter any paramaters that is passed in
- They always return a value
- If the same arguments are passed in, the function will always return the same response
- They don't produce any side-effects (eg. print, send an email, change a global variable, etc.)

Functions **where possible should be pure** - however their are times when they simply can't be, 
since the whole point of a program is to change state and produce side-effects - AND we'll get to that soon.

But first...

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

## Impure functions

### Event driven javascript cannot be stateless

ToDO

###

ToDO













