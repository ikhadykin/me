## Notes for Chapter 1 Programming With Objects: A Primer - [Object Design Style Guide](../)

In this chapter author covers fundamental aspects of programming:
* Classes and objects
* State
* Behavior
* Dependencies
* Inheritance
* Polymorphism
* Composition
* Return statements and exceptions
* Unit testing
* Dynamic arrays

### Classes and objects
Runtime behavior of an object is defined by its class definition. 

There are three types of class methods:
* **instance** or **object** methods - can be called only on an instance of a class
* **static** methods - can be called on a class itself
* **constructor** methods - this special method will be called before a reference to the object gets returned

```js
class Foo
{
    public function __construct()
    {
        // Prepare the object
    }
}

object1 = new Foo();  // __construct() will be implicitly called before a Foo instance gets assigned to objectl.
```

The standard way to instantiate a class is using the `new` operator. It’s also possible to define a static `factory method` on the class itself, which returns a new instance of the class.
```js
class Foo
{
    public static function create(): Foo
    {
        return new Foo();
    }
}

object1 = Foo.create();
object2 = Foo.create();
```

Common rules:
* two instances of the same class should not be considered the same
* to prevent an object from being fully instantiated throw an exception inside the constructor

### State
An object can contain data. This data will be stored in **properties**. A property will have a **name** and a **type**, and it can be populated at any moment after instantiation. A common place for assigning values to properties is inside the constructor.
```js
class Foo
{
    private int someNumber;
    private string someString;

    public function __construct()
    {
        this.someNumber = 10;
        this.someString = 'Hello, world!';
    }
}

object1 = new Foo(); // After instantiation, someNumber and someString will contain 10 and ‘Hello, world!’ respectively.
```

```js
class Foo
{
    private int someNumber;

    public function __construct(int initialNumber)
    {
        this.someNumber = initialNumber;
    }
}

object1 = new Foo(); // It won’t be possible to instantiate Foo without providing a value for initialNumber.
object2 = new Foo(20); // This should work; it assigns the initial value of 20 to the someNumber property of the new Foo instance.              
```

The data contained in an object is also known as its **state**.

When the value of an object’s property can change during the lifetime of the object, it’s considered a **mutable** object. If none of an object’s properties can be modified after instantiation, the object is considered an **immutable** object. The following listing shows examples of both cases.

```js
class Mutable
{
    private int someNumber;

    public function __construct(int initialNumber)
    {
        this.someNumber = initialNumber;
    }

    public function increase(): void
    {
        this.someNumber = this.someNumber + 1;
    }
}

class Immutable
{
    private int someNumber;

    public function __construct(int initialNumber)
    {
        this.someNumber = initialNumber;
    }

    public function increase(): Immutable
    {
        return new Immutable(someNumber + 1);
    }
}

object1 = new Mutable(10);
object1.increase(); // Calling increase() on Mutable will change the state of objectl by changing the value of its someNumber property.
 
object2 = new Immutable(10);
object2 = object2.increase(); // Calling increase() on Immutable doesn’t change the state of object2. Instead, we receive a new instance with the value of someNumber increased.
```

### Behavior
Besides state, an object also has behaviors that its clients can make use of. These behaviors are defined as methods on the object’s class. The **public** methods are the ones accessible to clients of the object. They can be called any time after the object has been created.

Some methods will return something to the caller. In that case an explicit type will be declared as the **return type**. Some methods will return nothing. In that case the return type will be `void`.

Usually though, private methods are used to represent smaller steps in a larger process.

```js
class Foo
{
    public function someMethod(): int
    {
        value = this.stepOne();

        return this.stepTwo(value);
    }

    private function stepOne(): int
    {
        // ...
    }

    private function stepTwo(int value): int
    {
        // ...
    }
}
```

Common rules:
* **Private should be your default scope** - in general, a **private** scope is preferable and should be your default choice. Limiting access to object data helps the object keep its implementation details to itself. It ensures that clients won’t rely on any specific piece of data owned by the object, and that they will always talk to the object through explicitly defined public methods
** **Check for null arguments** - to avoid `NullPointerExceptions`

### Dependencies
If object `Foo` needs object `Bar` to perform part of its job, `Bar` is called a dependency of `Foo`. 

There are different ways to make sure that `Foo` has access to the `Bar` dependency;
* It could instantiate `Bar` itself.
* It could fetch a `Bar` instance from a known location.
* It could get a `Bar` instance injected upon construction.

```js
class Foo
{
    public function someMethod(): void
    {
        logger = new Logger(); // Foo instantiates a Logger when needed.
        logger.debug('...');
    }
}

class Foo
{
    public function someMethod(): void
    {
        logger = ServiceLocator.getLogger(); // Foo fetches a Logger instance from a known location.
        logger.debug('...');
    }
}

class Foo
{
    private Logger logger;

    public function __construct(Logger logger)
    {
        this.logger = logger; // Foo has an instance of Logger provided to it as a constructor argument.
    }

    public function someMethod(): void
    {
        this.logger.debug('...');
    }
}
```

Fetching dependencies from a known location is called **service location**, and that retrieving dependencies as constructor arguments is called **dependency injection**.

### Inheritance
It’s possible to define only part of a class and let others expand on it. For instance, you can have a class with no properties and no methods, but only method signatures. Such a class is usually called an **interface**, and many object-oriented languages allow you to define it as such. A class can then implement the interface and provide the actual implementations of the methods that were defined in the interface.
```js
interface Foo
{
    public function foo(): void; // The Foo interface declares a foo() method but doesn’t provide an implementation.
}

class Bar implements Foo // Doesn't work: Bar is an incorrect implementation of Foo, because it doesn’t have an implementation for the foo() method.
{
}

class Baz implements Foo  // Baz is a correct implementation of Foo, because it provides an implementation for the foo() method.
{
    public foo(): void
    {
        // ...
    }
}
```

An interface doesn’t define any implementation, but an **abstract class** does. It allows you to provide the implementation for some methods and only the signatures for some other methods. An abstract class can’t be instantiated, but first has to be extended by a class that provides implementations for the abstract methods.

Classes that extend from another class have access to `public` and `protected` methods of the parent class.

Subclasses can only override `protected` and `public` methods of a parent class too.

Author argues that in practice, using **inheritance mostly leads to a confusing design**. In his book, he'll use *inheritance* mainly in two situations:

* When defining interfaces for dependencies
* When defining a hierarchy of objects, such as when defining custom exceptions that extend from built-in exception classes

In most other cases we’d want to actively prevent developers to extend from our classes. You can do so by adding the `final` keyword in front of the class.

### Polymorphism
**Polymorphism** is one of the foundations of object-oriented programming. Polymorphism means that if a parameter has a certain class as its type, any object that is an instance of that class can be provided as a valid argument.

In most situations it’s better to use polymorphism with an interface parameter type. 
```js
interface Foo
{
    // ...
}

final class Bar
{
    public function bar(Foo foo): void
    {
        foo.someMethod();
    }
}
```

### Composition
Assigning an object to another object’s property is called **object composition**. You are building up a more complicated object out of simpler objects. Object composition can be combined with polymorphism to compose your object out of other objects, whose (interface) type you know, but not the actual class.


### Unit testing
A Unit-testing framework will look for classes of a specific type, also called **test** classes. It will then instantiate each test class, and call each of the methods that are marked as a test (methods with the `@test` annotation).

The basic structure of each test method is Arrange-Act-Assert:

1. **Arrange** — Bring the object that we’re testing (also known as the SUT, or Subject Under Test) into a certain known state.
1. **Act** — Call one of its methods.
1. **Assert** — Make some assertions about the end state.

```js
final class Foo
{
    private int someNumber;

    public function __construct(int startWith)
    {
        this.someNumber = startWith;
    }

    public function increment(): void
    {
        this.someNumber++;
    }

    public function someNumber(): int
    {
        return this.someNumber;
    }
}

final class FooTest
{
    /**
     * @test
     */
    public function you_can_start_with_a_given_number(): void
    {
        // Arrange
        foo = new Foo(10);

        // Act
        // No actual action is performed here. We just verify the expected state of the object.
 
        // Assert
        assertEquals(10, foo.someNumber());
    }

    /**
     * @test
     */
    public function you_can_increment_the_number(): void
    {
        // Arrange
        foo = new Foo(10);

        // Act
        foo.increment(); // Here we call increment(), which is the action. Afterwards, we verify that the object is in the expected state.
 
        // Assert
        assertEquals(11, foo.someNumber());
    }
}
```
`assertEquals()` and related assertions, such as `assertTrue()`, `assertNull()`, and so on, are usually built into the testing framework.

If the object you’re testing has a dependency, you may not want to use the real dependency when testing. For example, maybe that dependency would make changes to a database, or start sending out emails. Every test run would produce these undesired side effects. In a situation like this, we’ll want to replace the actual dependency with a stand-in object that looks like the real dependency but replaces part of its original behavior. 

If you also want to verify that a particular method was called on some object, you can use a special kind of stand-in, called a **mock**. Testing frameworks usually offer special tooling for setting up such a mock and making the required assertions.

### Summary
* A class defines properties, constants, and methods.
* An object is immutable if all of its properties can’t be modified, and if all objects contained in those properties are immutable themselves.
* Dependencies can be created on the fly, fetched from a known location, or injected as constructor arguments (which is called dependency injection).
* Polymorphism means that code can use another object’s methods as defined by its type (usually an interface), but that the runtime behavior can be different depending on the specific instance that is provided by the client.
* When an object assigns other objects to its properties, it’s called composition.
* Unit tests specify and verify the behaviors of an object.
* While testing, you may replace an object’s actual dependencies with stand-ins known as test doubles (such as stubs and mocks)