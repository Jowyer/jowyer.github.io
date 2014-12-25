---
layout: post
title:  ReactiveCocoa
date:   2014-12-04 23:13:54
categories: iOS
---
就像[Mattt Thompson][mattt twitter]在他的[博客][nshipster ReactiveCocoa]里讲的一样，**Objective-C**语言经历了四次进化，最新的一次是吸收了其他语言的经典语法，以**Objective-C**的方式来重新实现。

之前在[Mobile Database -- Realm][jowyer blog realm]那篇博客里介绍的，**Active Record Pattern**即是从Ruby语言借鉴过来的，今天要介绍的[ReactiveCocoa][github ReactiveCocoa]也是如此。

## Introduction

> ReactiveCocoa is an open source library that brings Functional Reactive Programming paradigm to Objective-C. 
>
>ReactiveCocoa defines a standard interface for events, so they can be more easily chained, filtered and composed using a basic set of tools.

### Functional Reactive Programming

[Functional Reactive Programming][frp wiki] (FRP) is a way of thinking about software in terms of transforming inputs to produce output continuously over time.

>在命令式编程环境中，a = b + c 表示将表达式的结果赋给a，而之后改变b或c的值不会影响a。但在响应式编程中，a的值会随着b或c的更新而更新。
>
>Excel就是响应式编程的一个例子。单元格可以包含字面值或类似”=B1+C1″的公式，而包含公式的单元格的值会依据其他单元格的值的变化而变化 。

**ReactiveCocoa** combines a couple of programming styles:

* [Functional Programming][fp wiki] which makes use of higher order functions, i.e. functions which take other functions as their arguments
* [Reactive Programming][rp wiki] which focuses of data-flows and change propagation

## When to use ReactiveCocoa

* **Handling Asynchronous Or Event-driven Data Sources:** Much of Cocoa programming is focused on reacting to user events or changes in application state.
* **Chaining Dependent Operations:** Dependencies are most often found in network requests, where a previous request to the server needs to complete before the next one can be constructed.
* **Parallelizing Independent Work:** Working with independent data sets in parallel and then combining them into a final result is non-trivial in Cocoa, and often involves a lot of synchronization.
* **Simplifying Collection Transformations:** Higher-order functions like `map`, `filter`, `fold/reduce` are sorely missing from `Foundation`.

## Prerequisite

### @weakify @strongify
Blocks capture and retain values from the enclosing scope, therefore referencing ***self*** in a block actually creates a **strong reference cycle to self** ([Apple](https://developer.apple.com/library/ios/documentation/cocoa/conceptual/ProgrammingWithObjectiveC/WorkingwithBlocks/WorkingwithBlocks.html#//apple_ref/doc/uid/TP40011210-CH8-SW16)).

The [obvious solution](http://aceontech.com/objc/ios/2014/01/10/weakify-a-more-elegant-solution-to-weakself.html) to this problem is to define a weak reference to *self* before the block (let's call it `weakSelf`), and use that instead while calling into *self* from a block. For example:

{% highlight objc %}
// Create a weak reference to self
__weak typeof(self)weakSelf = self;
[self.context performBlock:^{
    // Create a strong reference to self, based on the previous weak reference.
    // This prevents a direct strong reference so we don't get into a retain cycle to self.
    // Also, it prevents self from becoming nil half-way, but still properly decrements the retain count
    // at the end of the block.
    __strong typeof(weakSelf)strongSelf = weakSelf;

    // Do something else

    NSError *error;
    [strongSelf.context save:&error];

    // Do something else
}];
{% endhighlight %}

Yet we can adopt a more elegant solution to this by using the **@weakify()** and **@strongify()** macros provided by the [libextobjc](https://github.com/jspahrsummers/libextobjc) library.

{% highlight objc %}
#import <EXTScope.h>
// #import "RACEXTScope.h"   Use this with ReactiveCocoa..

// Do stuff
@weakify(self)
[self.context performBlock:^{
    // Analog to strongSelf in previous code snippet.
    @strongify(self)

    // You can just reference self as you normally would. Hurray.
    NSError *error;
    [self.context save:&error];

    // Do something
}];
{% endhighlight %}

 The **@weakify** macro allows you to create *shadow* variables which are weak references (*you can pass multiple variables if you require multiple weak references*), the **@strongify** macro allows you to create strong references to variables that were previously passed to **@weakify**.

## Overview

ReactiveCocoa is comprised of two major components: **signals** (`RACSignal`) and **sequences** (`RACSequence`).

Both signals and sequences are kinds of **streams** (any series of object values), sharing many of the same operators. ReactiveCocoa has done well to abstract a wide scope of functionality into a semantically dense, consistent design: signals are a **push-driven** stream, and sequences are a **pull-driven** stream.

## RACSignal

`RACSignal` send a **stream** of **events** to their subscribers. There are three types of **events** to know: `next`, `error` and `completed`. A signal may send any number of *next events* before it *terminates after an error*, or it *completes*.

The lifetime of a signal consists of any number of `next` events, followed by one `error` or `completed` event (but not both).

`RACSignal` has a number of methods you can use to subscribe to these different event types.

Each operation on an `RACSignal` also returns an `RACSignal`.

## Basic Operators

### Performing side effects with signals
Most signals start out "cold", which means that they will NOT do any work until **subscription**.

Upon subscription, a signal or its subscribers can perform ***side effects***, like logging to the console, making a network request, updating the user interface, etc.

Side effects can also be **injected** into a signal, where they won't be performed immediately, but will instead take effect with each subscription later.

#### Subscription
The [-subscribe…](https://github.com/ReactiveCocoa/ReactiveCocoa/blob/master/ReactiveCocoa/RACSignal.h) methods give you access to the current and future values in a signal:
{% highlight objc %}
RACSignal *letters = [@"A B C D E F G H I" componentsSeparatedByString:@" "].rac_sequence.signal;

// Outputs: A B C D E F G H I
[letters subscribeNext:^(NSString *x) {
    NSLog(@"%@", x);
}];
{% endhighlight %}

#### Injecting effects

The [-do…](https://github.com/ReactiveCocoa/ReactiveCocoa/blob/master/ReactiveCocoa/RACSignal+Operations.h) methods add side effects to a signal without actually subscribing to it:
{% highlight objc %}
__block unsigned subscriptions = 0;

RACSignal *loggingSignal = [RACSignal createSignal:^ RACDisposable * (id<RACSubscriber> subscriber) {
    subscriptions++;
    [subscriber sendCompleted];
    return nil;
}];

// Does not output anything yet
loggingSignal = [loggingSignal doCompleted:^{
    NSLog(@"about to complete subscription %u", subscriptions);
}];

// Outputs:
// about to complete subscription 1
// subscription 1
[loggingSignal subscribeCompleted:^{
    NSLog(@"subscription %u", subscriptions);
}];
{% endhighlight %}

### Transforming streams
These operators transform a single stream into a new stream.

#### Filtering
The `-filter:` method uses a block to test each value, including it into the resulting stream only if the test passes.

{% highlight objc %}
[self.usernameTextField.rac_textSignal filter:^BOOL(NSString *text) {
	return text.length > 3;
}];
{% endhighlight %}

#### Mapping
The `-map:` method is used to transform the values in a stream, and create a new stream with the results.

{% highlight objc %}
[self.usernameTextField.rac_textSignal map:^id(NSString *text) {
	return @([self isValidUsername:text]);
}];

- (BOOL)isValidUsername:(NSString *)username {
  return username.length > 3;
}
{% endhighlight %}

### Combining signals
These operators combine multiple signals into a single new `RACSignal`.

#### Combining latest values
The `+combineLatest:reduce:` methods will watch multiple signals for changes, and then send the latest values from *all* of them when a change occurs. Each time either of the multiple source signals emits a new value, the reduce block executes, and the value it returns is sent as the next value of the combined signal.

{% highlight objc %}
RACSignal *signUpActiveSignal =
[RACSignal combineLatest:@[validUsernameSignal, validPasswordSignal] reduce:^id(NSNumber *usernameValid, NSNumber *passwordValid) {
	return @([usernameValid boolValue] && [passwordValid boolValue]);
}];
{% endhighlight %}

#### Sequencing
The `-then:` method waits until a **completed** event is emitted, then subscribes to the signal returned by its block parameter. This effectively *passes control from one signal to the next*.

{% highlight objc %}
- (RACSignal *)requestAccessToTwitterSignal {  
    // 1 - define an error
    NSError *accessError = [NSError errorWithDomain:RWTwitterInstantDomain
                                               code:RWTwitterInstantErrorAccessDenied
                                           userInfo:nil];
    
    // 2 - create the signal
    @weakify(self)
    return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        // 3 - request access to twitter
        @strongify(self)
        [self.accountStore
         requestAccessToAccountsWithType:self.twitterAccountType
         options:nil
         completion:^(BOOL granted, NSError *error) {
             // 4 - handle the response
             if (!granted) {
                 [subscriber sendError:accessError];
             } else {
                 [subscriber sendNext:nil];
                 [subscriber sendCompleted];
             }
         }];
        return nil;
    }];
}

@weakify(self)
[[[self requestAccessToTwitterSignal]
  then:^RACSignal *{
    @strongify(self)
    return self.searchText.rac_textSignal;
  }]
  subscribeNext:^(id x) {
    NSLog(@"%@", x);
  } error:^(NSError *error) {
    NSLog(@"An error occurred: %@", error);
  }];
{% endhighlight %}

The `-then:` method passes **error** events through. Therefore the final `subscribeNext:error:` block still receives errors emitted by the initial access-requesting step.

This is most useful for executing all the *side effects* of one signal, then starting another, and only returning the second signal's values.

## Combining streams
These operators combine multiple streams into a single new stream.

### Mapping and flattening
`-flattenMap:` is used to transform each of a stream's values into a *new stream*. Then, all of the streams returned will be flattened down into a single stream. In other words, it's `-map:` followed by `-flatten`.

An outer signal that contains an inner signal is called the ***signal of signals***. Use `-flattenMap:` to unwrap it.

{% highlight objc %}
-(RACSignal *)signInSignal {
  return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    [self.signInService
     signInWithUsername:self.usernameTextField.text
     password:self.passwordTextField.text
     complete:^(BOOL success) {
       [subscriber sendNext:@(success)];
       [subscriber sendCompleted];
     }];
    return nil;
  }];
}

[[[self.signInButton
   rac_signalForControlEvents:UIControlEventTouchUpInside]
   flattenMap:^id(id x) {
     return [self signInSignal];
   }]
   subscribeNext:^(id x) {
     NSLog(@"Sign in result: %@", x);
   }];
{% endhighlight %}

### Threading
Although events are guaranteed to be serial, sometimes stronger guarantees are needed, like when performing UI updates (which must occur on the main thread).

Whenever such a guarantee is important, the `-deliverOn:` operator should be used to force a signal's events to arrive on a specific `RACScheduler`.

{% highlight objc %}
[[[[[[self requestAccessToTwitterSignal]
  then:^RACSignal *{
    @strongify(self)
    return self.searchText.rac_textSignal;
  }]
  filter:^BOOL(NSString *text) {
    @strongify(self)
    return [self isValidSearchText:text];
  }]
  flattenMap:^RACStream *(NSString *text) {
    @strongify(self)
    return [self signalForSearchWithText:text];
  }]
  deliverOn:[RACScheduler mainThreadScheduler]]
  subscribeNext:^(id x) {
    NSLog(@"%@", x);
  } error:^(NSError *error) {
    NSLog(@"An error occurred: %@", error);
  }];
{% endhighlight %}

Generally, the use of `-deliverOn:` should be restricted to the end of a signal chain – e.g., before subscription, or before the values are bound to a property.

## Other Operations

### Throttling
The `-throttle:` operation will only send a next event if another next event isn’t received within the given time period. 

If a `next` is received, and then another `next` is received before `interval` seconds have passed, the first value is discarded.

{% highlight objc %}
[[[[[[[self requestAccessToTwitterSignal]
  then:^RACSignal *{
    @strongify(self)
    return self.searchText.rac_textSignal;
  }]
  filter:^BOOL(NSString *text) {
    @strongify(self)
    return [self isValidSearchText:text];
  }]
  throttle:0.5]
  flattenMap:^RACStream *(NSString *text) {
    @strongify(self)
    return [self signalForSearchWithText:text];
  }]
  deliverOn:[RACScheduler mainThreadScheduler]]
  subscribeNext:^(NSDictionary *jsonSearchResult) {
    NSArray *statuses = jsonSearchResult[@"statuses"];
    NSArray *tweets = [statuses linq_select:^id(id tweet) {
      return [RWTweet tweetWithStatus:tweet];
    }];
    [self.resultsViewController displayTweets:tweets];
  } error:^(NSError *error) {
    NSLog(@"An error occurred: %@", error);
  }];
{% endhighlight %}

## Macros

### RAC 

The `RAC` macro allows you to assign the **output** of a signal to the **property** of an **object**. It takes two arguments, the first is the object that contains the property to set and the second is the property name. Each time the signal emits a next event, the value that passes is assigned to the given property.

{% highlight objc %}
RACSignal *validUsernameSignal =
[self.usernameTextField.rac_textSignal map:^id(NSString *text) {
	return @([self isValidUsername:text]);
}];
     
RAC(self.usernameTextField, backgroundColor) =
[validUsernameSignal map:^id(NSNumber *passwordValid) {
	return [passwordValid boolValue] ? [UIColor clearColor] : [UIColor yellowColor];
}];
{% endhighlight %}

### RACObserve

Returns a signal which sends the current value of the key path on subscription, then sends the new value every time it changes, and sends completed if self or observer is deallocated.

It's a kind of KVO using blocks.

{% highlight objc %}
[RACObserve(self.textField, text) subscribeNext:^(NSString *newName) {
        NSLog(@"%@", newName);
    }];
{% endhighlight %}

## [Reference](#Reference)
[Raywenderlich][raywenderlich ReactiveCocoa]

[NSHipster][nshipster ReactiveCocoa]

[Limboy](http://limboy.me/ios/2013/06/19/frp-reactivecocoa.html)

[iTiger](http://www.itiger.me/?p=38)

[ReactiveCocoa Documentation][RAC doc]

[fuckingblocksyntax.com][block web]


[mattt twitter]: https://twitter.com/mattt
[jowyer blog realm]: http://jowyer.github.io/personal/2014/11/06/Realm.html#active-record-pattern
[frp wiki]: http://en.wikipedia.org/wiki/Functional_reactive_programming
[fp wiki]: http://en.wikipedia.org/wiki/Functional_programming
[rp wiki]: http://en.wikipedia.org/wiki/Reactive_programming

[raywenderlich ReactiveCocoa]: http://www.raywenderlich.com/62699/reactivecocoa-tutorial-pt1
[nshipster ReactiveCocoa]: http://nshipster.com/reactivecocoa/
[github ReactiveCocoa]: https://github.com/ReactiveCocoa/ReactiveCocoa
[RAC doc]: https://github.com/ReactiveCocoa/ReactiveCocoa/tree/master/Documentation
[block web]: http://fuckingblocksyntax.com/

