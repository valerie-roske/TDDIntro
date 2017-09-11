## Writing a test - an example

If you have a good idea of the behavior you want to test then you should just do this:

1. Create the test method with a name that describe what you are testing

2. Write the test in whatever way makes sense to you

If you don’t have a good sense of what your test to look like when you start writing it, try using this process:

1. Write an empty test with an meaningless name

2. Create an instance of the class you want to test

3. Call the method you want to test

4. Setup the case that will cause the result

5. Assert some result you expect 

6. Name your test

Here’s an example of Test Driving a **`StringJoiner`** class whose job is to *Join* strings.

> **Join** -
>Joining a list of strings means creating a single new string by concatenating the string in list together with a
> delimiter between them. For instance, joining the strings {"a", "b”, "c”} on the delimiter ",” would result in the
> string "a,b,c”. Note that there is not a leading or trailing comma.

1) Create all of the test scaffolding and get it to compile (name your test something ugly).

``` java
public class StringJoinerTests {
    @Test
    public void shouldFooWhenBar() {
    }
}
```

2 & 3) Next, create an instance of the class you are testing and call the method. You should type out the name of the
class and method even if they don’t exist yet. In the example below, assume that the class **`StringJoiner`** doesn’t
exist yet.

``` java
public class StringJoinerTests {
    @Test
    public void shouldFooWhenBar() {
        String result = new StringJoiner().join();
    }
}
```

Right now the class **`StringJoiner`** doesn't exist. This means we need to create it. You can create it manually or
use your IDE to create it for you.

> IntelliJ will highlight **`StringJoiner`** in red. We can click on the class name, press Alt-Enter and choose the
> option **`Create Class ‘StringJoiner’`**. This will automatically create the class for you.

After the class is created our test will look like this...

``` java
public class StringJoinerTests {
    @Test
    public void shouldFooWhenBar() {
        String result = new StringJoiner().join();
    }
}
```

Now the **`join`** method doesn't exist. Create it yourself of use your IDE to create it for you.

> In IntelliJ **`join`** will be red because it’s not implemented. Click on the method name, hit Alt-Enter, and choose
> **`Create Method ’join’`**.

Now we’re calling the method we want to test. Here are some useful questions we can ask:

* What is the smallest piece of new behavior that we can add? 

* What change would it cause? 

These questions should lead us to add an assert that verifies that we got the correct result from joining some strings. 
Right now the **`join`** method returns **`null`**. Let’s add the behavior that causes **`join`** to return the empty string when 
the list that is passed in is empty.

Wait a minute, we aren’t passing a list of strings into the **`join`** method yet. We also suddenly have enough 
information to name our test. And we want to add an assert too. When you have more than one thing that could be your 
next change, write down the options and do them one at at time. Here’s our current ToDo list:

1. Pass list of strings to join

2. Add assert to test

3. Rename this test to ???

4) Let’s pass an empty list into **`join`** first.

``` java
public class StringJoinerTests {
    @Test
    public void shouldFooWhenBar() {
        List<String> strings = new ArrayList<String>();
        String result = new StringJoiner().join(strings);
    }
}
```


5) Now we can add our assert that verifies that we got an empty string back.

``` java
public class StringJoinerTests {
    @Test
    public void shouldFooWhenBar() {
        List<String> strings = new ArrayList<String>();
        String result = new StringJoiner().join(strings);
        assertThat(result, is(""));
    }
}
```


6) Now that we really understand what we are testing we can rename our test. Normally you will name your test **before** 
you start writing it. You should only name it later when you have trouble understanding what you want to test. Let’s 
rename the test based on what we know about the purpose of our test.

``` java
public class StringJoinerTests {
    @Test
    public void shouldJoinIntoAnEmptyStringWhenListIsEmpty() {
        List<String> strings = new ArrayList<String>();
        String result = new StringJoiner().join(strings);
        assertThat(result, is(""));
    }
}
```

This is a complete unit test. Let’s split it up to clarify what the three sections of the test are.

``` java
public class StringJoinerTests {
    @Test
    public void shouldJoinIntoAnEmptyStringWhenListIsEmpty() {
        // Arrange
        StringJoiner joiner = new StringJoiner();
        List<String> strings = new ArrayList<String>();

        // Action
        String result = joiner.join(strings);

        // Assert
        assertThat(result, is(""));
    }
}
```

We can run the test and watch it fail by clicking anywhere in the test file and hitting Ctrl-Shift-F10.

Now we want to make it pass by writing the simplest code possible. This is how we can make the test pass:

``` java
public class StringJoiner {
    public String join(List<String> strings) {
        return "";
    }
}
```

This is our first passing test! If there was anything for us to refactor we would do so now. Now let’s write more tests.
We’ll move a little faster now.

``` java
    @Test
    public void shouldJoinIntoTheStringWhenListIsOneString(){
        List<String> strings = new ArrayList<String>();
        String aString = "A String";
        strings.add(aString);
        StringJoiner joiner = new StringJoiner();

        String result = joiner.join(strings);

        assertThat(result, is(aString));
    }
```

This is a much more interesting test than the first one. There’s a lot going on in the Arrange section, although most
of it is just adding a string to the list. Other than the change to how we arrange the objects this test is mostly the 
same as the first one. The test fails as expect and this is a simple way to make it pass.

``` java
public class StringJoiner {
    public String join(List<String> strings) {
        if (strings.size() > 0){
            return strings.get(0);
        }
        return "";
    }
}
```


Now we want to run both of our tests. A simple way to do this is to click anywhere in the test file that is not inside 
of a method and hit Ctrl-Shift-F10. Now both tests pass and it’s time to think about refactoring. There’s nothing 
obvious to refactor in **`StringJoiner`**, but there’s a lot of duplication in our test class. After removing comments 
and blank lines, it looks like this:

``` java
public class StringJoinerTests {
    @Test
    public void shouldJoinIntoAnEmptyStringWhenListIsEmpty(){
        List<String> strings = new ArrayList<String>();
        StringJoiner joiner = new StringJoiner();
        String result = joiner.join(strings);
        assertThat(result, is(""));
    }
    @Test
    public void shouldJoinIntoTheStringWhenListIsOneString(){
        List<String> strings = new ArrayList<String>();
        String aString = "A String";
        strings.add(aString);
        StringJoiner joiner = new StringJoiner();
        String result = joiner.join(strings);
        assertThat(result, is(aString));
    }
}
```

The **`new ArrayList<String>()`** and **`new StringJoiner()`** lines are exactly the same in both tests. Let’s fix this while our tests are passing so we can have
confidence that we didn’t break anything. We can move change these local variables into instance variables which we 
initialize in setup method like this:

``` java
public class StringJoinerTests {
    private List<String> strings;
    private StringJoiner joiner;
    @Before
    public void setUp() throws Exception {
        strings = new ArrayList<String>();
        joiner = new StringJoiner();
    }
    @Test
    public void shouldJoinIntoAnEmptyStringWhenListIsEmpty(){
        assertThat(joiner.join(strings), is(""));
    }
    @Test
    public void shouldJoinIntoTheStringWhenListIsOneString(){
        String aString = "A String";
        strings.add(aString);
        assertThat(joiner.join(strings), is(aString));
    }
}
```


Note that we removed the **`result`** variable to improve readability.

There’s also a new method call **`setUp`** which has the **`@Before`** annotation. Any method that is marked with 
**`@Before`** will be executed before each test in that same class. This allows us to reset the strings and joiner 
instance variables so that they don’t allow the actions of one test to affect another.

So far we have taken such small steps that our **`StringJoiner`** class doesn’t do much. This is normal for TDD, we’re
implementing the behavior we want in very small slices but they will quickly add up to everything we need. Here’s our 
next test.

``` java
    @Test
    public void shouldContainBothStringsWhenListIsTwoStrings(){
        strings.add("A");
        strings.add("B");
        assertThat(joiner.join(strings),                     
            both(containsString("A")).
            and(containsString("B")));
    }
```    

The assert in this test is more complex than in previous tests. **`ContainsString`** verifies that the result of join 
contains a certain string (in this case "A" or "B”). The **`both/and`** construct means that both **`containsString`** 
verifications must be true for the assert to pass. 

When we run all of our tests, we’re happy to see that this new test fails and all of our old tests pass. Now we need to 
make the new test pass.

``` java
public class StringJoiner {
    public String join(List<String> strings) {
        String result = "";
        for (String string : strings) {
            result += string;
        }
        return result;
    }
}
```

This makes all of our tests pass. Great! So far our **`StringJoiner`** joins all of the strings together, but it doesn’t
even know what a delimiter is, much less how to put it between the strings in the list. Our new test should fix that…

``` java
    @Test
    public void shouldPutDelimiterBetweenStrings(){
        StringJoiner joinerWithDelimiter = new StringJoiner(",");
        strings.add("A");
        strings.add("B");
        assertThat(joinerWithDelimiter.join(strings), is("A,B"));
    }
```


Our **`StringJoiner`** now knows about delimiters and it’s constructor takes one as a parameter. Because our constructor
changed we have to update all of the places that we create a new **`StringJoiner`**. Fortunately, there is only one
other **`new StringJoiner`** in our tests because we removed duplication early on and moved creation of our
**`StringJoiner`** into the **`setUp`** method. Now we need to figure out the simplest way to make all of our tests pass.

``` java
public class StringJoiner {
    private String delimiter;

    public StringJoiner(String delimiter) {
        this.delimiter = delimiter;
    }

    public String join(List<String> strings) {
        String result = "";
        if (strings.size() > 0){
            List<String> allExceptFirstString = 
                new ArrayList<String>(strings);
            result += allExceptFirstString.remove(0);
            for (String string : allExceptFirstString) {
                result += delimiter + string;
            }
        }
        return result;
    }
}
```


This sure doesn’t look simple, but it *was* easy to implement and is a small incremental change to our previous code.
We know that our code does the right thing because all of our tests pass now and our most recent test didn’t pass before
we wrote this code.

This is the first time that we’ve had code that we might want to refactor. It’s safe to refactor because all of our code
is covered by tests and all of those tests are passing. Here’s a slightly cleaner version of the code.

``` java
public class StringJoiner {
    private String delimiter;

    public StringJoiner(String delimiter) {
        this.delimiter = delimiter;
    }
    
    public String join(List<String> strings) {
        if (!strings.isEmpty()){
            String firstString = strings.get(0);
            List<String> remainingStrings = 
                strings.subList(1, strings.size());
            return firstString +           
                   concatenateWithDelimiter(otherStrings);
        }
        return "";
    }

    private String concatenateWithDelimiter(List<String> strings) {
        String result = "";
        for (String string : strings) {
            result += delimiter + string;
        }
        return result;
    }
}
```