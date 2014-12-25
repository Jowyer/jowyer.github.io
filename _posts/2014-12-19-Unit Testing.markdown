---
layout: post
title:  Unit Testing
date:   2014-12-19 23:15:39
categories: iOS
---
从Xcode 5开始，Apple终于发布了自己的单元测试框架（之前接入的是OCUnit）-- **XCTest**，而且在Xcode 6中正变得愈发强大，是时候把单元测试捡起来了 :smile:

## Introduction

从上至下，单元测试的结构可以分为四级：

### Test suite

This is the entire collection of tests for your project. In Xcode, the test suite is set up as a separate build **target**.

### Test case classes

As you might expect in an object-oriented system, tests are grouped into classes. Each test class usually corresponds to a single class in your app. For example, the `DeveloperTests` class could contain tests for the `Developer` class.

### Test case methods

Each test case class contains multiple methods to test various features of the class. Just as methods and functions should be as short as possible and do one thing well, each test case should test one specific outcome — and test it fully.

Test case methods must *start with the word **test*** so the test runner can find them.

### Assertions

Assertions check specific conditions against an expected result. If the condition does not match the expected result, Xcode throws an error to indicate an assertion failure. For example, you could assert that your `Developer` class responds to the `writeKillerApp:` message; if it doesn’t, the assertion will fail and an exception will be thrown.

## setUp & tearDown

`-setUp` is called before each test in an XCTestCase is run, and when that test finishes running, `-tearDown` is called.
If you have multiple test methods, then `-setUp` and `-tearDown` are called *multiple times* in a single test session — once per test case method!

## Naming Standard

A common and useful [naming standard](http://osherove.com/blog/2005/4/3/naming-standards-for-unit-tests.html) for test cases is
*unitOfWork_stateUnderTest_expectedBehavior*. For example:

{% highlight objc %}
- (void)test_addition_twoPlusTwo_isFour
{% endhighlight %}

In this example, the unit of work being tested is addition, the test state is 2 + 2, and the expected behavior is that the result is 4.

## XCTest Assertions

All XCTest assertions begin with the prefix **XCT**.
XCTest comes with a number of [built-in assertions](https://developer.apple.com/library/prerelease/ios/documentation/DeveloperTools/Conceptual/testing_with_xcode/testing_3_writing_test_classes/testing_3_writing_test_classes.html#//apple_ref/doc/uid/TP40014132-CH4-SW35), below are some essentials:

### Fundamental Test
All of the XCTest assertions come down to a single, base assertion:

{% highlight objc %}
XCTAssert(expression, format...)
{% endhighlight %}

The first parameter is an expression that is expected to evaluate to true, while the NSLog-style parameters following the expression define the message displayed if the assertion fails.

### Boolean Tests
To test whether the expressions are true/false:

{% highlight objc %}
XCTAssertTrue(expression, format...)
XCTAssertFalse(expression, format...)
{% endhighlight %}

`XCTAssert` is equivalent to `XCTAssertTrue`.

### Equality Tests
To test whether two values are equal or not, greater or less:

{% highlight objc %}
XCTAssertEqual(expression1, expression2, format...) // test for scalars.
XCTAssertNotEqual(expression1, expression2, format...)
XCTAssertEqualWithAccuracy(expression1, expression2, accuracy, format...) // test for scalars such as floats and doubles
XCTAssertEqualObjects(expression1, expression2, format...)
XCTAssertGreaterThan(expression1, expression2, format...)
XCTAssertLessThanOrEqual(expression1, expression2, format...)
{% endhighlight %}

### Nil Tests
To test the existence (or non-existence) of a given value:

{% highlight objc %}
XCTAssertNil(expression, format...)
XCTAssertNotNil(expression, format...)
{% endhighlight %}

### Unconditional Fail
The `XCTFail` assertion will always fail:

{% highlight objc %}
XCTFail(format...)
{% endhighlight %}

`XCTFail` is most commonly used to denote a placeholder for a test that should be made to pass. It is also useful for handling error cases already accounted by other flow control structures, such as the **else** clause of an if statement testing for success.

## XCTestExpectation
Perhaps the most exciting feature added in Xcode 6 is built-in support for **asynchronous testing**, with the `XCTestExpectation` class. Now, tests can wait for a specified length of time for certain conditions to be satisfied, without resorting to complicated GCD incantations.

{% highlight swift %}
// 1. Create an expectation.
let expectation: XCTestExpectation = 
self.expectationWithDescription("Async something")

// 2. Do some async operations. 
myObject.doSomethingAsyncWithCompletion({
    ...
    // 4. Fulfill the expectation. 
    expectation.fulfill();
})

// 3. The test will pause here, running the run loop, until 
// the timeout is hit or all expectations are fulfilled. 
self.waitForExpectationsWithTimeout(1.0, handler: { (error) in
    // 5. Expectation failed.
    XCTAssertNil(error, "Async test didn't finish before timeout."); 
})
{% endhighlight %}

## Performance Testing
Performance tests which is introduced in Xcode 6, help establish a baseline of performance for hot code paths. Sprinkle them into your test cases to ensure that significant algorithms and procedures remain performant as time goes on.

* `measureBlock`: which takes a block of code and measures the execution time of the entire block. It runs whatever code you put inside it 10 times and measures a default set of metrics.
* `measureMetrics`: is a more intimate version of `measureBlock` that offers fine grained control of what to measure, and when to measure it.

{% highlight swift %}
func testDateFormatterPerformance() {
    let dateFormatter = NSDateFormatter()
    dateFormatter.dateStyle = .LongStyle
    dateFormatter.timeStyle = .ShortStyle

    let date = NSDate()

    measureBlock() {
        let string = dateFormatter.stringFromDate(date)
    }
}
{% endhighlight %}

## TDD
最后来说说[Test-Driven Development](http://en.wikipedia.org/wiki/Test-driven_development)。

测试驱动开发是极限编程*eXtreme Programming*中倡导的程序开发方法：先写测试程序，然后编码实现其功能。
测试驱动开发是戴两顶帽子思考的开发方式：先戴上实现功能的帽子，在测试的辅助下，快速实现其功能；再戴上重构的帽子，在测试的保护下，通过去除冗余的代码，提高代码质量。测试驱动着整个开发过程：首先，驱动代码的设计和功能的实现；其后，驱动代码的再设计和重构。

这个理念个人感觉还是不错的，有其优秀的地方，但也不是传说中的那么神。

推荐看两篇文章：

[浅谈测试驱动开发](https://www.ibm.com/developerworks/cn/linux/l-tdd/)

[TDD并不是看上去的那么美](http://coolshell.cn/articles/3649.html)

## Reference
[RayWenderlich](http://www.raywenderlich.com/store/ios-7-by-tutorials)

[NSHipster](http://nshipster.com/xctestcase/)

[Apple Docs](https://developer.apple.com/library/prerelease/ios/documentation/DeveloperTools/Conceptual/testing_with_xcode/Introduction/Introduction.html)


