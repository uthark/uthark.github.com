---
layout: post
title: "Using Unitils ReflectionAssert"
date: 2014-04-16 00:10:10 -0400
comments: true
categories: 
    - development
    - article
    - java
    - unit testing
    - unitils
    - reflectionassert
---

Often it is needed to compare two different instances of the class inside test. I.e. we save object into database, then fetch it back from db and we want to be sure if nothing was lost during saving/reading. 

In order to make such assertions more easier and maintainable one can use great Unitils library which has useful class [ReflectionAssert]

First, update parent's `pom.xml`:
``` xml pom.xml changes in parent module
<properties>
    <unitils.version>3.4</unitils.version>
</properties>
<dependency>
    <groupId>org.unitils</groupId>
    <artifactId>unitils-core</artifactId>
    <version>${unitils.version}</version>
    <scope>test</scope>
</dependency>
```

Then, add dependency to the child module:   
``` xml pom.xml changes in child module
<dependency>
    <groupId>org.unitils</groupId>
    <artifactId>unitils-core</artifactId>
</dependency>
```

Usage is very simple:
Add static import to the class.
``` 
import static org.unitils.reflectionassert.ReflectionAssert.assertReflectionEquals;
```

and use it instead of regular JUnit assert:

``` java
    assertReflectionEquals(expected, actual);
```

It is possible to pass reflection comparator modes as optional parameters to the above method.

Be ware that this method is not very useful if you want to compare objects which has floating point or double parameters - ReflectionAssert compares `double` and `float` values using `==`, so it is not very useful. It is not possible to pass epsilon for comparison.

``` java excerpt from org.unitils.reflectionassert.comparator.impl.LenientNumberComparator#compare
// ...
Double leftDouble = getDoubleValue(left);
Double rightDouble = getDoubleValue(right);
if (!leftDouble.equals(rightDouble)) {
    return new Difference(differenceMessage, left, right);
}
// ...
```

If you need to compare double values in JUnit assertions then it is better to use [hamcrest library]:

``` java
import static org.hamcrest.number.IsCloseTo.closeTo;
import static org.junit.Assert.assertThat;

// in @Test method:
assertThat(actual.getFederalIncomeTax(), closeTo(expected.getFederalIncomeTax(), EPSILON))
```

Happy hacking!

[Unitils]: http://www.unitils.org/summary.html
[ReflectionAssert]: http://www.unitils.org/tutorial-reflectionassert.html
[hamcrest library]: https://code.google.com/p/hamcrest/
