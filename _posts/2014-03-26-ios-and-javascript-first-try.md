---
title: iOS and JavaScript First try
redirect_from: "/blog/2014/03/26/ios-and-javascript-first-try/"
tags:
  - iOS
  - "obc-c"
  - javascript
---

As you might know since iOS 7, Apple included `JavaScriptCore` framework, that gives ability to _almost_ write apps in Java Script. So I did a small trial and it seems to work.

We'll use `XCTest` for a playground.

First, let's try logging something

``` obj-c
#import <JavaScriptCore/JavaScriptCore.h>

- (void)testConsoleLog {
    JSContext *context = JSContext.new;

    // Add logging support
    [context evaluateScript:@"var console = {}"];
    context[@"console"][@"log"] = ^(NSString *msg) {
        NSLog(@"JS: %@", msg);
    };
    [context evaluateScript:@"console.log('message!');"];
}
```

Now let's do some simple benchmarking:



We'll use a `pod 'BenchmarkTestCase'` to help:

``` obj-c
#import <AZBenchmarkTestCase.h>
@interface JSBenchmarkTestCase : AZBenchmarkTestCase
@end
```

We'll do some simple insertion of string into array. First, an Objective-C test:

``` obj-c
- (void)benchNSMutableArray
{
    NSMutableArray *array = NSMutableArray.new;
    int n = 500;
    for (int i=0;i<n;i++) {
        NSString *s = [NSString stringWithFormat:@"data #%i", i];
        [array addObject:s];
    }
}
```

Now, a Java Script implementation:

``` obj-c
- (void)benchJSMutableArray
{
    JSContext *context = JSContext.new;

    [context evaluateScript:
     @"var array = [];"
     @"var n = 500;"
     @"for (var i=0;i<n;i++) {"
     @"  var s = 'data #' + i;"
     @"  array.push(s);"
     @"}"
     ];
}
```

Well, the results are good, but still, Objective-C wins:

```
:Name                                              :Total(s)  :Avg.(s)  
-[JSBenchmarkTestCase test_benchNSMutableArray]    0.11345    0.00113    (1/100)
-[JSBenchmarkTestCase test_benchJSMutableArray]    0.61675    0.00617    (1/100)
```

Hey, what if we'll not re-create a `JSContext`, but re-use it?:

``` obj-c
- (void)benchJSMutableArray2
{
    static JSContext *context;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        context = [[JSContext alloc] init];
    });

    [context evaluateScript:
     @"var array = [];"
     @"var n = 500;"
     @"for (var i=0;i<n;i++) {"
     @"  var s = 'data #' + i;"
     @"  array.push(s);"
     @"}"
     ];
}
```

Yep, that's waaayyy better:

```
:Name                                              :Total(s)  :Avg.(s)  
-[JSBenchmarkTestCase test_benchJSMutableArray2]   0.11231    0.00112    (1/100)
-[JSBenchmarkTestCase test_benchJSMutableArray]    0.44110    0.00441    (1/100)
-[JSBenchmarkTestCase test_benchNSMutableArray]    0.13410    0.00134    (1/100)
```

As you may notice, Java Script is even faster!

Conclusion: Using Java Script to write iOS Apps since iOS 7 is not a bad idea overall.
