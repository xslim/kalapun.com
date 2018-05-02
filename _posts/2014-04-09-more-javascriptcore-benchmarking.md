---
title: More JavaScriptCore benchmarking on iOS7
redirect_from: "/blog/2014/04/09/more-javascriptcore-benchmarking/"
tags:
  - javascript
  - objc
  - benchmarking
---

So I've made a few more JavaScriptCore VS Native benchmarking on iOS7. All the code is available at [xslim/iJSPlayground](http://github.com/xslim/iJSPlayground).

_Smaller numbers are better._

## JSON parsing
As simple as using `JSON.parse(string)`:

``` obj-c
- (void)benchOCParser {
    NSError *error = nil;
    id parsedData = [NSJSONSerialization JSONObjectWithData:jsonData options:kNilOptions error:&error];
    XCTAssertNotNil(parsedData, @"Should not be nil");
}

- (void)benchJSParser {
    JSValue *func = context[@"parser"];
    JSValue *res = [func callWithArguments:@[jsonString]];
    NSDictionary *parsedData = [res toDictionary];
    XCTAssertNotNil(parsedData[@"name"], @"Should not be nil");
}
```

Result:

```
:Name                   :Total(s)  :Avg.(s)  
-[x test_benchOCParser] 0.02302    0.00023    (1/100)
-[x test_benchJSParser] 0.08478    0.00085    (1/100)
```

Yep, Objective-C is faster. Let's do more interesting:

## Object mapping: Mantle VS JS
I've wrote my simple mapper that maps json to objects and has ability to specify key-to-property mapping. No relationship.



For single object the difference is not that big:

```
:Name                    :Total(s)  :Avg.(s)
-[x test_benchM1_Mantle] 0.03335    0.00033    (1/100)
-[x test_benchM1_JS]     0.02366    0.00024    (1/100)
```

For Array of 150 objects the difference is more vizible:

```
:Name                    :Total(s)  :Avg.(s)
-[x test_benchMA_Mantle] 1.11922    0.01119    (1/100)
-[x test_benchMA_JS]     0.93283    0.00933    (1/100)
```
Yes, JS is faster.

## Hasher

So I have a hasher (token generation) class in Objective-C, and I've re-implemented the same in Java Script. It uses custom `Base64` function and `HMAC SHA1`. On Objective-c it uses `CommonCrypto` framework, and on JS side I've found `crypto.js` and bundled it with `browserify.js`.

So of course, the Objective-C implementation works faster, but still, the JS is not that bad:

```
:Name                                              :Total(s)  :Avg.(s)  
-[HmacBenchmarkTestCase test_benchOCHmac]          0.00039    0.00000    (1/100)
-[HmacBenchmarkTestCase test_benchJSHmac]          0.01479    0.00015    (1/100)
-[HmacBenchmarkTestCase test_benchOCHasher]        0.00261    0.00003    (1/100)
-[HmacBenchmarkTestCase test_benchJSHasher]        0.04126    0.00041    (1/100)
```
