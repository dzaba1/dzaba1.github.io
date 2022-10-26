# TypeScript

## What is the major advantage of using typescript instead of javascript?

Optional strong typing with classes and interfaces.

Maybe for home-made, one person projects it doesn't bring much values but for corporate projects it does a lot. Like intellisense, compile-time type validation during CI/CD, better indexing by search engines.

## Is TypeScript strongly typed?

Nope, there's the `any` keyword and then type check will be omitted during compilation.
Example:
```
var a: any = 1;
a = "potato";
a = true;
```

## What is tsconfig.json file?

When the file is present in some directory then the directory is a root one for the whole project.

It contains compile options including source files.

## Does TypeScript fixes the `this` issue?

Nope. Everything stays the same as in JavaScript.

## How the `private` access modifier is compiled?

It's not compiled at all. It is used only by the compiler to check access.

## How to make method parameter optional?

We can use the `?` character. Example:
```
function Do(myArg?: number) {}
```

## Is it possible to debug typescript code directly?

Yes, it is possible but first you need to have source map.

## What `export` keyword does? What is a difference from `export default`?

Import changes then. With `export default` you can import later a type with a different name.

Having `export` you must import directly type from a file later.

TODO