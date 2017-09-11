# Test Driven Development in Java

## Table of Contents
* [Introduction](#introduction)
* [What is TDD?](#what-is-tdd)
* [Anatomy of a Unit Test](#anatomy-of-unit-tests)
* [Writing a Test](WritingTestsExample.md)
* [Try It For Yourself](#try-it-for-yourself)
* [Mocks & Stubs](MocksAndStubs.md)
* [TDD Patterns](#tdd-patterns)
* [TDD Anti-Patterns](#tdd-anti-patterns)
* [Advice](#advice)
* [Further Reading](#further-reading)

<a id="introduction"></a>
## Introduction 

These lessons will teach you the basics of Test Driven Development (TDD) in Java, using JUnit, Mockito, and IntelliJ.

We’re assuming that we don’t need to convince you why you want to do TDD and we’ll only touch lightly on the principles
of TDD. Instead we’ll be focusing on the what and how.

<a id="what-is-tdd"></a>
## What is TDD?

TDD is the practice of writing a small amount of code (a unit test) that describes the new behavior you wish to add to
your program before you implement the behavior itself. You can think of unit tests as tiny programs we write to verify that the methods in our classes do what we expect them to do.

### The TDD Cycle

The following sequence is based on the book *[Test-Driven Development by Example](http://en.wikipedia.org/wiki/Test-Driven_Development_by_Example)*.

The basics steps for this process look like this:

1. Write a small failing unit test

2. Make this new test pass in the simplest way possible

3. Clean up any messes we created

![image](https://github.com/BillSchofield/TDDIntro/blob/master/src/common/images/TDDCycle.png?raw=true)

#### Add a test

In test-driven development, each new feature begins with writing a test. This test must inevitably fail because it is
written before the feature has been implemented. (If it does not fail, then either the proposed "new" feature already
exists or the test is defective.) To write a test, the developer must clearly understand the feature's specification
and requirements. The developer can accomplish this through [use cases](http://en.wikipedia.org/wiki/Use_case) and
[user stories](http://en.wikipedia.org/wiki/User_story) to cover the requirements and exception conditions, and can
write the test in whatever testing framework is appropriate to the software environment. This could also be a
modification of an existing test. This is a differentiating feature of test-driven development versus writing unit tests
*after* the code is written: it makes the developer focus on the requirements *before* writing the code, a subtle
but important difference.

#### Run all tests and see if the new one fails

This validates that the [test harness](http://en.wikipedia.org/wiki/Test_harness) is working correctly and that the new
test does not mistakenly pass without requiring any new code. This step also tests the test itself; it rules out the
possibility that the new test always passes, and therefore is worthless. The new test should also fail for the expected
reason. This increases confidence (though does not guarantee) that it is testing the right thing, and passes only in
intended cases.

#### Write some code

The next step is to write code that causes the test to pass. The new code written at this stage is not perfect, and may,
for example, pass the test in an inelegant way. That is acceptable because later steps improve and hone it.

At this point, the only purpose of the written code is to pass the test; no further (and therefore untested)
functionality should be predicted and 'allowed for' at any stage.  This prevents unnecessary and unspecified code from
being written, helping avoid [YAGNI](http://en.wikipedia.org/wiki/You_aren't_gonna_need_it) functionality.

#### Run tests

If all test cases now pass, the programmer can be confident that the code meets all the tested requirements. This is a
good point from which to begin the final step of the cycle.

#### Refactor code

Now the code should be cleaned up. Move code to where it logically belongs. Remove duplication. Make sure variable and
method names represent their current use. Clarify constructs that might be misinterpreted. Use the [Four Rules of
Simple Design](https://theholyjava.wordpress.com/2011/02/14/clean-code-four-simple-design-rules/) to guide you, as well
as anything else you know about writing clean code. By re-running test cases, you
can be confident that [refactoring](http://en.wikipedia.org/wiki/Code_refactoring) is not damaging any existing
functionality.

The concept of removing duplication is an important aspect of any software design. In this case it also applies to
removing duplication between test code and production code—for example
[magic numbers or strings](http://en.wikipedia.org/wiki/Magic_number_(programming)) repeated in both to make the test
pass in the "Write some code" step.

> Four Rules of Simple Design
> * Passes all the tests.
> * Expresses every idea that we need to express.
> * Says everything OnceAndOnlyOnce.
> * Has no superfluous parts.

#### Repeat

Starting with another new test, repeat the cycle to push forward the functionality. The size of the steps should always
be small, with as few as 1 to 10 edits between each test run. If new code does not rapidly satisfy a new test, or other
tests fail unexpectedly, the programmer should [undo](http://en.wikipedia.org/wiki/Undo) or revert in preference to
excessive [debugging](http://en.wikipedia.org/wiki/Debugging).
[Continuous integration](http://en.wikipedia.org/wiki/Continuous_integration) helps by providing revertible checkpoints.
When using external libraries do not make increments so small that they merely testing the library itself, unless there
is some reason to believe that the library is buggy or not sufficiently feature-complete to serve all the needs of the
main program being written.

<a id="anatomy-of-unit-tests"></a>
## Anatomy of a Unit Test

As we mentioned above, unit tests are small programs that we use to verify the correctness of our "production" code.
The word *unit* refers to a subdivision of the overall program. While others might consider *unit* to mean a class or
package, we will only be unit testing at the method level.

Before you start writing a unit test you should know what behavior you want to verify. For instance, you might have a
method that returns the plural version of a word. In English, the way we pluralize most words is by adding the letter
‘s’ to the end of the word. That sounds like a great first test case.

Once you know what behavior you want to verify you can name your test. A great format for test names is,
`should<expected behavior>When<situation that behavior depends on>`. In our pluralizing example, the expected
behavior is ‘add an s’ and the situation is ‘normal word’. That means that we could name our test
`shouldAddSWhenWordIsNormal`. Since it’s not necessarily clear what it means for a word to be ‘normal’, we could also
name the test `shouldAddS` or `shouldAddSToWord`.

Once you know the behavior you want to verify and the method where you expect add that behavior, you can start writing
your test. We’ll show you how to do this in JUnit.

### JUnit Example

JUnit is a popular Java unit testing framework. We’re going to use JUnit to create our TDD unit tests. 

There are three sections to every unit test. One set of names for these sections is: Arrange, Action, Assert.
Another is: Given, When, Then.

Here's a brief example:

``` java
public class PluralizerTests {
    ...

    @Test
    public void shouldAddSWhenWordIsNormal() {
    // Arrange our objects
    Pluralizer pluralizer = new Pluralizer();

    // Action we are testing
    String result = pluralizer.pluralize("Cat");

    // Assert that the action caused the expected result
    assertThat(result, is("Cats"));
    }
    ...
}
```

#### Test Classes

We call the class that we are testing the *class under test*. In the example above, the class under test is
**`Pluralizer`**. All of the unit tests for the *class under test* will live inside a test class named:
**`<class under test>Tests.java`**.

##### Arrange/Given 

This is where we set the stage for our scenario. That means that we create all of the objects we need for the test in
this section. While arranging happens at the top of our test, we often make changes here after working on the other two
sections.

##### Action/When

The Action/When section is where we call the method that we are testing (the Action). This should usually be a single
method call.

##### Assert/Then

We verify that the method under test caused the right thing to happen in Assert/Then section of our tests. If you feel
like you need more than one assert you should probably split your test.



<a id="try-it-for-yourself"></a>
## Try It For Yourself

As a result of our disciplined practice of TDD, we have evidence that our code is correct and we were able to safely
refactor it into code that is easier to read, extend, and test. Now **you** can try your hand at TDD!

### Factorial Exercise

Open the class **`com.thoughtworks.tddintro.exercises.factorial.FactorialTests`**. You'll find five unit tests there. Your goal is to make
changes to the class **`Factorial`** so that one more test passes than the last time you made a change. Essentially,
you're doing the *Make the failing test pass* step of TDD. This should help you get used to the rhythm of TDD before
you have to write your own tests. Here's the cycle you should go through once for each test.

1. Run all of the tests by clicking anywhere inside the test class between the test methods and then hit
Control-Shift-F10.
2. Look at the assert line of the test you are trying to make pass (do them in order) and change the **`compute`**
method so that the assert will pass.
3. Run all of the tests. The only new test that should pass is the one you are currently trying to make pass. If more
than one new test passes, you are adding too much functionality. Revert back to the last time you made a new test pass
and try again. You should also try again if one of the previously passing tests now fails.
4. Now that you have one more test passing, you should commit you change so you can revert back to a good state later if
you need to.

### Write your own tests

Now you're going to write your own test.

Look in the class **`com.thoughtworks.tddintro.exercises.accountbalance.AccountTests`**. You'll see three commented out empty unit tests
(one for each of the test cases listed below).

For each of the test cases:

1. Implement the test for that test case. Uncomment it and add a test code inside it.
2. Fix compile errors.
3. Watch the test fail.
4. Write now code that you expect to make the test pass.
5. Watch the test pass. If any of your tests fail, you should repeat step #4.
6. Commit your changes and go back to Step #1 for the next test case.

| Given                     | When            | Then                                |
| ------------------------- | --------------- | ----------------------------------- |
| I have $100 in my account | I deposit $50   | I see that my account contains $150 |
| I have $100 in my account | I withdraw $50  | I see that my account contains $50  |
| I have $50 in my account  | I withdraw $100 | I see that my account contains $50  |

<a id="tdd-patterns"></a>
## TDD Patterns

These concepts/strategies lead us to write tests that lead to testable and flexible code. Think of them as recipes for 
cooking successful tests and code. They are guidelines to help you succeed when you first start writing tests. Over time
you will learn when and where to improvise on these recipes.

### One Constructor Per Class
If we have more than one constructor in each class then it's possible that a test could pass if we used one constructor 
and fail if we used a different one. That would lead us to want to create a duplicate of each test case for each different 
 constructor. It's simpler to limit ourselves to one constructor.
 
``` java
public class Product {
    private int numberOfItems;
    private double totalPrice;
    public Product() {
        numberOfItems = 0;
        totalPrice = 0.0;
    }
    public Product(int numberOfItems, double totalPrice) {
        this.numberOfItems = numberOfItems;
        this.totalPrice = totalPrice;
    }
    public double pricePerItem(){
        return totalPrice/numberOfItems;
    }
    public void setTotalPrice(double totalPrice) {
        this.totalPrice = totalPrice;
    }
}

public class ProductTest {
    @Test
    public void pricePerItemShouldBeSameAsTotalPriceWhenThereIsOneItem() {
        Product product = new Product(1, 14.0);

        assertThat(product.pricePerItem(), is(14.0));
    }
    @Test
    @Ignore // This test fails due to price per item being Infinity
    public void pricePerItemShouldBeSameAsTotalPriceWhenThereIsOneItemWithDefaultConstructor() {
        Product product = new Product();
        product.setTotalPrice(14.0);
        assertThat(product.pricePerItem(), is(14.0));
    }
}
``` 

### Avoid Behavior in the Constructor by Keeping Complex Object Creation Behavior in Factories
Since we are going to be creating a new instance of our class in each test, we will end up executing any behavior that lives 
in the constructor in every one of our tests. That means that every test for our class could fail if the behavior in the 
constructor stops working. It's simpler to let this behavior live in other methods. Limit yourself to assignments in your
constructors. Do something like this:

``` java
public class Shape {
    private String name;
    private final double area;
    private final double perimeter;

    public Shape(String name, double area, double perimeter) {
        this.name = name;
        this.area = area;
        this.perimeter = perimeter;
    }
}
```

We can also run into the problem where we want our constructor to create different types of objects that are instances
of the same class. For instance, we might have constructors in our Shape class that create instances of squares and 
circles. Both squares and circles only need one value (probably a double) in order to describe them (length for square 
and radius for circle). How do we distinguish between those constructors since they both want a signature that looks like 
`public Shape(double size)`? Sometimes people will use a flag to indicate which type of object they want (an enum or 
boolean). This results in a complex constructor that we will have to write a lot of tests for. In the example below we
use double to indicate that we want a square and float to indicate a circle. Imagine how easy it would be to call the 
wrong constructor. Avoid code that looks like this:

``` java
public class Shape {
    private String name;
    private final double area;
    private final double perimeter;

    // Square Constructor
    public Shape(double length){
        name = "Square";
        area = length * length;
        perimeter = length * 4;
    }

    // Circle Constructor
    public Shape(float radius){
        name = "Circle";
        area = PI * radius * radius;
        perimeter = 2 * PI * radius;
    }
}
```

Sometimes we need to do something complex in order to create an instance of our object. If we can't do that work in the 
constructor, we need to put it somewhere. Factories are a design pattern where we use a different class to encapsulate 
the complexity of creating a new instance of another class. This is an example of a static factory method:

``` java
public class Shape {
    private String name;
    private final double area;
    private final double perimeter;

    public Shape(String name, double area, double perimeter) {
        this.name = name;
        this.area = area;
        this.perimeter = perimeter;
    }

    public static Shape createSquareWithSidesOfLength(int length) {
        String name = "Square";
        double area = length * length;
        double perimeter = length * 4;
        return new Shape(name, area, perimeter);
    }
}
```

When we use this static factory we do something like:
``` java
Shape square = Shape.createSquareWithSidesOfLength(5);
```

An abstract factory looks like this (note that the abstract factory either implements a factory interface or extends an
abstract factory class):
``` java
public class CircleFactory implements ShapeFactory {
    @Override
    public Shape create(int diameter) {
        double radius = 1.0 * diameter/2.0;
        return new Shape("Circle", PI * radius * radius, 2 * PI * radius);
    }
}

// Usage looks like:
Shape circle = new CircleFactory().create(5);
```
 
### Use Only Required Arguments Constructors
It's generally good practice to not have partially initialized objects. That means that we need to initialize all of our
class' required fields during construction. The simplest way to do this is to pass in values for all of these fields as
constructor arguments.

### Use Dependency Injection

### Mock Everything Except the Class under Test

### No Static Variables or Methods

<a id="tdd-anti-patterns"></a>
## TDD Anti-patterns

### Chained mocks and the Law of Demeter

### Excessive Setup
A test that requires a lot of work setting up in order to even begin testing. Sometimes several hundred lines of code is 
used to setup the environment for one test, with several objects involved, which can make it difficult to really 
ascertain what is tested due to the "noise" of all of the setup going on.

### The Giant
A unit test that, although it is validly testing the object under test, can span thousands of lines and contain many 
many test cases. This can be an indicator that the system under tests is a 
[God Object](http://en.wikipedia.org/wiki/God_object)

### Generous Leftovers
An instance where one unit test creates data that is persisted somewhere, and another test reuses the data for its own 
devious purposes. If the "generator" is run afterwards, or not at all, the test using that data will fail.

### The Dodger
A unit test which has lots of tests for minor (and presumably easy to test) side effects, but never tests the core 
desired behavior. Sometimes you may find this in database access related tests, where a method is called, then the test 
selects from the database and runs assertions against the result.

<a id="advice"></a>
## Advice
* Initially, go slow. Do it right.
* Minimize untested code
* Don’t unit test main()
* Minimize the complexity of your main method by only calling one method in main. You could write something like 
`new Foo().run();` for example
* Work on one thing at a time. Seriously.
* Only work on one feature at a time
* Change one behavior at a time
* Red/Green/Refactor. Know where you are.
* Always know where you are in the process. You should be able to say, "We are working on the first acceptance criteria of 
the 'List Books' story, we have two passing tests, and are about to refactor our tests to use a setup method".
* Focus on what the code should do, not how you want to implement it. Call a method before you implement it.
* Code from the outside in (or top to bottom). All of your new code should be executed when you run `main()`.
* Only add test to methods that already exist. The test will drive you to implement the new behavior that you want 
the method to have.
* Move/duplicate tests before you move code to a new class
* Don’t refactor code without new tests already in place

<a id="further-reading"></a>
## Further reading:
 * [Martin Fowler](http://martinfowler.com/articles/mocksArentStubs.html)’s essay exploring differences between mocks and stubs.
 * [Test Double](http://en.wikipedia.org/wiki/Test_double) provides a concise overview of different types of test doubles:
 * [Test stub](http://en.wikipedia.org/wiki/Test_stubs) (used for providing the tested code with "indirect input")
 * [Mock object](http://en.wikipedia.org/wiki/Mock_object) (used for verifying "indirect output" of the tested code, by first defining the expectations before the tested code is executed)
 * [Test spy](http://en.wikipedia.org/w/index.php?title=Test_spy&action=edit&redlink=1) (used for verifying "indirect output" of the tested code, by asserting the expectations afterwards, without having defined the expectations before the tested code is executed)
 * [Fake object](http://en.wikipedia.org/wiki/Fake_object) (used as a simpler implementation, e.g. using an in-memory database in the tests instead of doing real database access)
 * [Dummy object](http://en.wikipedia.org/w/index.php?title=Dummy_object&action=edit&redlink=1) (used when a parameter is needed for the tested method but without actually needing to use the parameter)
 * [TDD Anti-Patterns by James Carr](http://blog.james-carr.org/2006/11/03/tdd-anti-patterns/)

