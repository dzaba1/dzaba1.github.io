## OOP principles

- Encapsulation - The internal state of objects should be hidden. So it means that only accessible methods should change it.
- Hierarchization - Classes can be grouped into some tree hierarchies. At the top, there are relatively big and generic classes and at the bottom relatively small and specialized ones.
- Componentization - Objects can be made of other objects.
- Abstraction - Generalization of a particular problem/solution independently of concrete implementation. In some other word, objects can communicate with each other without revealing internal implementation and state.

## What's the interface?

Set of signatures with unique identifier.

## Differences between abstract class and interface

An abstract class has some implementation so it has some state.

An interface has only signatures.

## Why is it a bad practice to call virtual methods in the constructor?

Because the ctor chain can be interrupted by using some non-initiated fields.

## SOLID

- Single responsibility principle - Classes, modules, packages, test suites, test fixtures, tests, etc. should be relatively small and solve one, particular problem
- Opened-closed principle - The code should be closed for modifications but opened for extensions.
- Liskov (Barbara) substitution principle - It means that a base class reference can point to derived class object just like that. So polymorphism. Also the derived class shouldn't change drastically the base class assumptions.
- Interface segregation principle - A lot of small interface are better then a big one. I would say it shouldn't be now about interfaces only but also about classes, packages, test suites, etc.
- Dependency inversion principle - The code should be based on Abstraction instead of concrete implementation.

## DRY

Don't repeat yourself.

## YAGNI

You aren't gonna need it. So focus on your current task and problem and don't make useful things for future. Also make your code simple, don't complicate it.

## KISS

Keep it simple stupid. Again, make your code simple, don't complicate it.

## What's the high cohesion?

Classes, packages, etc. do exactly what they should do.

## What's the loose coupling?

Classes, packages, etc. are independently to each other so they can be developed independently and in parallel.

## What's the Dependency Injection?

Inversion of Control realization. It looses coupling between object for Abstraction and interfaces.
Be dependency we usually mean some service class.
Caller provides that class to some another class so it injects a dependency.

There are 2 major ways of DI:
- Constructor injection
- Method, setter, interface injection

Something like this implements the Bridge pattern.

Usually there's some outside, independent mechanism which knows what to inject and when.
Usually it's an Inversion of Control container.
There are initiated usually at some process start via code or some external configuration.

## Differences between Service Locator and Dependency Injection.

Service Locator as the name says it's like a static IoC container. It keeps services.
It's easy to access it from every place.
But it hides dependencies. It's easy to make a mistake by requesting some not initialized service.
Or it initializes services during runtime so you can add service in any time.

In DI every module is passed separately at the beginning. Sometimes it makes more coding but every dependency is visible during compile time. So if one dependency changes then there should be some compile error.
Usually you don't have access to initialized container.
Here the container doesn't only keep services but it keeps methods of creating them. So it works more like a factory.
And usually you only request services when the container is initialized. You shouldn't add new ones in the middle of a process.

## What types of tests do you know?

- Structural tests
- Functional tests - If an operation makes what it should make for end user.

- Compilation - code
- Unit test - logic
- Integration tests (including e2e) - architecture
- Acceptance tests (UAT) - requirements
	- GUI tests
	- Env tests

Regression testing - Test if existing functionality works after some change or extension.

## What is the difference between mock and stub?

Stub is an object which contains minimal implementation of some requested interface. They usually do nothing and don't take part in testing.

Mock is more complicated implementation of some requested interface. For example it tracks number of calls, parameters, etc.
They take part in testing as we setup it at the begging of a test.

## AAA

Arrange, Act, Assert so first setup everything what you need in a test then invoke something and then check.
