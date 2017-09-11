## Mocks & Stubs

### Test Doubles
Up to this point, we’ve test driven situations where the class we are testing does not depend on any other class and we
only care that we get the right return value from a method. In real life we often have:

* `void` methods which have no return value for us to assert against
* methods that don't take any parameters 
* code that calls methods with behavior that we don't want to happen when we run our tests (e.g. current time, 
`System.out` or database)

These are all situations where we want our tests to behave differently than our production code without having to 
modify our production code in order to test it. 

_How can we change the behavior of our code without changing our code?_

What if `println(String string)` did something different when we call it while testing? In our tests we could have it 
record the string that it was passed to print (without actually printing anything) and in our production code we could 
have it print normally. This makes it safe to use `println()` in our tests and yet still behave properly in real life.

> Exercise 
> Create a HelloWorld program (or open an existing one) in IntelliJ. Click on the `out` in `System.out.println` and hit
> Command-B(Go to implementation). This takes you to the class `System` where you'll see the line:
> ``` java
> public final static PrintStream out = null;
> ```
> 
> This tells us that the variable `out` is of type `PrintStream` which is really nice to know. That means that if we want
> to call `println` all we need is a reference that is a `PrintStream` object.
> 
> Another way to look at this is to select `System.out` and hit Alt-Command-V(Introduce Variable). You'll get something
> that looks like this:
> ``` java
> PrintStream printStream = System.out;
> printStream.println();
> ```

Now we'll use this information to write a some testable code that prints a greeting for us. Here's an untestable
(and untested) version of `GreetingPrinter`:
``` java
public class GreetingPrinter {
    public static void main(String[] args) {
        System.out.println("Greetings!");
    }
}
```

The `main` method isn't testable because we have no mechanism to avoid using a real PrintStream and if we call main the 
program will print to the console, which we don't want to happen when we're running a large test suite.

We'll extract out the PrintStream just like we did in our last example and the code looks like this:
``` java
public static void main(String[] args) {
        PrintStream printStream = System.out;
        printStream.println("Greetings!");
    }
}
```

This still isn't testable, but it showed us that we can create our PrintStream in one place and use it in another. We 
can declare the PrintStream variable in the main method and use it in another method. When we do this we need to 
make the PrintStream variable available in the calling method so we can use it. A great way to do that is by passing 
`printStream` into the constructor of the class that uses it.

``` java
public class com.thoughtworks.tddintro.exercises.library.Main {
    public static void main(String[] args) {
        GreetingPrinter greetingPrinter = new GreetingPrinter(System.out);
        greetingPrinter.printGreeting();
    }
}

public class GreetingPrinter {
    private PrintStream printStream;

    public GreetingPrinter(PrintStream printStream) {
        this.printStream = printStream;
    }

    public void printGreeting(){
        printStream.println("Greetings!");
    }
}
```

This is a key refactoring because it lets the printGreetings method use whatever kind of PrintStream we want. 
For example, we could make a new class called FakePrintStream that extends PrintStream, but doesn't print anything when
we call `println(String string)` and instead records the string that is passed to it.
 
``` java
public class FakePrintStream extends PrintStream {
    private String printedString;

    public FakePrintStream() {
        super(new FakeOutputStream());
    }

    @Override
    public void println(String string) {
        printedString = string;
    }

    public String printedString() {
        return printedString;
    }
}
```

This would let us a write a test that looks like this:
``` java
public class GreetingPrinterTest {

    @Test
    public void shouldPrintGreeting() {
        FakePrintStream printStream = new FakePrintStream();
        GreetingPrinter greetingPrinter = new GreetingPrinter(printStream);

        greetingPrinter.printGreeting();

        assertThat(printStream.printedString(), is("Greetings!"));
    }

}
```

### Stubs
Stubs are modules of code that simulate the behaviors of software components that a module under test depends on. 
In our code example `FakePrintStream` is a stub class and GreetingPrinter is the module under test.

### Dependency Injection
GreetingPrinter is said to be dependent on PrintStream, because it depends on PrintStream to do some work for it. 
One pattern for managing your dependencies is to create them in the constructor of the class that needs them. For 
instance:
``` java
    public Foo() {
        this.bufferedReader = new BufferedReader(new InputStreamReader(System.in));
    }
``` 
The problem with this pattern is that whenever we create a new instance of Foo, it will have to use the BufferedReader 
that reads from `System.in` even if we want it to behave differently (like we do in our tests). This pattern is 
inflexible and you should generally avoid it. 

An alternative is to pass dependencies into the constructor of the class the needs those dependencies. This pattern is 
called _dependency injection_ because we inject a class' dependencies instead of having the class create them itself. 
This pattern increases the flexibility of our code by allowing us to create different instances of the same class that 
behave differently because they use different versions of their dependencies.

### Mocks

> Wikipedia: Mock object
> In object-oriented programming, mock objects are simulated objects that mimic the behavior of real objects in 
> controlled ways. A programmer typically creates a mock object to test the behavior of some other object, in much the 
> same way that a car designer uses a crash test dummy to simulate the dynamic behavior of a human in vehicle impacts.

We're primarily going to use mock objects to:
* verify object interactions
* provide return values from dependencies

<a id="mockito"></a>
## Mockito

Mockito is a Java library that lets you mock and stub objects with impunity.  It provides two extraordinarily useful
methods:
* `when/thenReturn` (for stubbing)
* `verify` (for mocking)

The [Mockito homepage](http://site.mockito.org/) is a great reference resource.

### Verify example
In our previous example where we used a FakePrintStream, we had to create two new classes just to verify a single 
interaction. This adds a lot of overhead even for simple tests. Mocking frameworks, like Mockito, allow for much simpler
mocking. If we used Mockito in our previous example it would look something like this:

``` java
@Test
public void shouldPrintGreeting() {
    PrintStream printStream = mock(PrintStream.class);
    GreetingPrinter greetingPrinter = new GreetingPrinter(printStream);

    greetingPrinter.printGreeting();

    verify(printStream).println("Greetings!");
}
```

The important parts of this version of the test are `mock(PrintStream.class)` and `verify`. The mock statement tells 
Mockito to create a new mock object that honors the PrintStream interface (but has none of the behavior of PrintStream).
Verify asks Mockito to assert that the println method was call on the printStream object with the parameter "Greetings!".

### When/thenReturn

Sometimes our tests need specific return values from the objects they depend upon. Mockito provides the when/thenReturn
functionality to support this.

In the example below, we want to make sure that our `TimePrinter` object prints out the time that is provided by the
`DateTime` object that we inject into it. It's much easier for us to use a mock `DateTime` and instruct that mock to
return a specific value (in this case "2013-04-08 16:33:17") that to create a real `DateTime` object and force it to
have a specific value. Mocking this object is especially valuable in this case because it insulates our tests from
changes in the behavior of `DateTime` formatting.

``` java
@Test
public void shouldPrintTime() { 
    PrintStream printStream = mock(PrintStream.class);
    DateTime dateTime = mock(DateTime.class);
    when(dateTime.toString()).thenReturn("2013-04-08 16:33:17");
    TimePrinter timePrinter = new TimePrinter(printStream, dateTime);
    
    timePrinter.print();
    
    verify(printStream).println("2013-04-08 16:33:17");
}
```

### Write some tests using Mockito

In this exercise we're going to implement some test for an existing class, `Library`, that prints a list of books to a
`PrintStream`. Since we're implementing the tests after the code under test is already written we are NOT doing TDD. Most
programmers call this development approach Test Last (instead of Test First). You should generally avoid Test
Last development, but it's a smart thing to do if you inherit untested code.

#### Using Verify

Find the class `com.thoughtworks.tddintro.exercises.library.Main` and run it. This shows you the existing behavior of the program; which is
to print out the three books that are added in the `Main` class. Note that we are passing the list of books and the
`PrintStream` into the constructor of `Library`. This lets us use a real `PrintStream` in our main method and a mock
`PrintStream` in our tests.

When we run main books print to the console, but when we finish writing our `Library` tests
nothing will print to the console except the test results. This is important because in real projects we might have tens
of thousands of tests and if many of them printed to the console we wouldn't be able to find the test results in all of
spam from our program printing so much.

Now go to the class `com.thoughtworks.tddintro.exercises.library.LibraryTest` (it's located in the `test/java` directory). This class has three
unit tests in it. The first one is mostly implemented. You should add a `verify` statement to make sure that the correct
string is being printed to the mock `PrintStream`.

Once you get the first test completed, you should implement the next two tests one at a time. They should mostly be similar to the
first one. This is a good time to introduce a setup method using the `@Before` annotation. A good way to ensure that your
tests are testing the right thing is to change the code that you are testing and make sure the test fail the way you
expect them to. For instance, you could change the listBooks method to always print "Book Title" exactly one time. This
should make some tests fail and still allow others to pass. Try it and see if your tests do what you think they do. Then
put the listBooks method back to it's original state.

#### Using when/thenReturn

Work through the remaining tests in `LibraryTest` the same way you did the first three. This time we're testing the
`listBooks` method and using `when/thenReturn` to make our mock `DateTimeFormatter` return some specific values when it is asked
to print the time.

## Why TDD & Dependency Injection

### Breaking Dependencies
TDD helps expose our dependencies and dependency injection is a tool for breaking dependencies.

#### Sensing & Separation
We break dependencies:
 * so we can *sense* when we can't access values our code computes
 * to *separate* when we can't even get a piece of code into a test harness to run.

##### Sensing
Mocks let us sense interactions that are important to the tests we're writing but are not easy to verify without mocks.
`Mockito.verify` allows us to easily sense these interactions. 

##### Separation
We often have code that is called in production that we would never want to call in our tests. Some reasons we might not 
want to call this code is that it:
 * is slow and we always want our tests to run fast
 * interacts with a real resources that we don't want to interact with in tests
 * allows us to avoid using real resources
 * helps us write maintainable tests
