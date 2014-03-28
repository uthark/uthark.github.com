---
layout: post
title: "Initializing Mockito @Mocks with JUnit @Rule"
date: 2014-03-28 13:24:45 -0400
comments: true
categories: 
    - java
    - mockito
    - junit
    - development
---

In this post I will show how one can implement custom JUnit [@Rule].

As an example let's take [Mockito] and implement custom rule which will initialize Mocks in  test class.

By default, Mockito provides the following methods of mock initialization:

1. Using [Mockito.mock]

``` java Initialize mocks with Mockito.mock
public void FooTest {
    
    private Foo foo;

    @Before
    public void setUp() {
        FooDependency dep = Mockito.mock(FooDependency.class);
        foo = new Foo(dep);
    }

}
```

This is the simplest case.

2. Using [MockitoAnnotations].

``` java Initialize mocks with MockitoAnnotations
public void FooTest {
    
    private Foo foo;

    @Mock
    private FooDependency dep;

    @Before
    public void setUp() {
        MockitoAnnotations.initMocks(this);
        foo = new Foo(dep);
    }
}
```

This method is useful when you have a lot of mocks to inject.

3. Using [MockitoJUnitRunner]
``` java Initialize mocks with MockitoJUnitRunner
@RunWith(MockitoJUnitRunner.class)
public void FooTest {
    
    private Foo foo;

    @Mock
    private FooDependency dep;

    @Before
    public void setUp() {
        foo = new Foo(dep);
    }
}
```

I want to show you another method, by using custom JUnit `@Rule`.

First, we need to implement [TestRule] interface which allows to implement custom behaviour during test execution:

``` java MockitoInitializerRule
public class MockitoInitializerRule implements TestRule {
    private Object test;

    public MockitoInitializerRule(Object test) {
        this.test = test;
    }

    @Override
    public Statement apply(Statement base, Description description) {
        return new MockitInitilizationStatement(base, test);
    }
}
```

We need to pass current test class instance to the rule, so it will be possible inject mocks into class instance. Actual implementation of our rule will go to new class - subclass of [Statement]:

``` java MockitInitilizationStatement
class MockitInitilizationStatement extends Statement {
    private final Statement base;
    private Object test;

    MockitInitilizationStatement(Statement base, Object test) {
        this.base = base;
        this.test = test;
    }

    @Override
    public void evaluate() throws Throwable {
        MockitoAnnotations.initMocks(test);
        base.evaluate();
    }
}

```

What we do is basically initialize mocks in the test class and then proceed with the test method execution.

Now, let's take a look at the usage of newly created Rule:

``` java Example usage of MockitoInitializerRule
public void FooTest {
    
    private Foo foo;

    @Mock
    private FooDependency dep;

    @Rule
    public TestRule mockitoInitializerRule = new MockitoInitializerRule(this);

    @Before
    public void setUp() {
        foo = new Foo(dep);
    }
}
```

Comparing to variant with `MockitoJUnitRunner` this one provides ability to use custom [Runner], i.e. we can continue to use [SpringJUnit4ClassRunner]

[Mockito]: https://code.google.com/p/mockito/
[@Rule]: http://junit-team.github.io/junit/javadoc/4.11/org/junit/Rule.html
[Mockito.mock]: http://docs.mockito.googlecode.com/hg/org/mockito/Mockito.html#mock(java.lang.Class)
[MockitoAnnotations]: http://docs.mockito.googlecode.com/hg/org/mockito/MockitoAnnotations.html#initMocks(java.lang.Object)
[MockitoJUnitRunner]: http://docs.mockito.googlecode.com/hg/org/mockito/runners/MockitoJUnitRunner.html
[TestRule]: http://junit-team.github.io/junit/javadoc/4.11/org/junit/rules/TestRule.html
[Statement]: http://junit-team.github.io/junit/javadoc/4.11/org/junit/runners/model/Statement.html
[Runner]: http://junit-team.github.io/junit/javadoc/4.11/org/junit/runner/Runner.html
[SpringJUnit4ClassRunner]: http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/test/context/junit4/SpringJUnit4ClassRunner.html
