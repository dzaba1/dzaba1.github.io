# JavaScript

## What are data types in JavaScript?

Primitives:
- null
- undefined
- boolean
- number
- bigint
- string
- symbol

And object

## What is the difference between double equals operator (==) and triple equals operator (===, also called strict equals)?

The `==` first tries to convert types and then comparison. It may lead to some unexpected results.

The `===` also checks types before checking values.

## What is the result of the following expression?

"2" + 5 + 6 = "256"

5 + 6 + "2" = "112"

Because of casting.

## `this` key word

`this` is not a pointer to method owner object. The value of `this` is determined by how a function is called.

TODO

## What is the difference between `null` and `undefined`?

`undefined` is used when some variable is not yet assigned.

`null` has to be assigned. It means that someone or something assigned 'no value' to a variable.

## Is it possible to execute asynchronous code in JavaScript?

By default JS code is single threaded by we can use:
- setTimeout, setInterval
- promises and callbacks