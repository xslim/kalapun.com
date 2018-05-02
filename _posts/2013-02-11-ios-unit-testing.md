---
title: "iOS Unit testing"
redirect_from: "/blog/2013/02/11/ios-unit-testing/"
tags: [code, ios, obj-c, testing, tdd]
---

Let me tell you, dear reader, how I simplify my life by writing small and effective unit tests for iOS projects.

I'm quite lazy person, so if I can simplify my life and work - I do so. Of course you agree, that writing tests is cool. Having tests means you have less bugs in code, and if someone changes something, he can check if he's patch breaks anything in existing implementation. And of course, writing unit tests makes you thinking about better architecture, and making smaller, better methods. You know beforehand, that testing a method that is few hundreds of lines of code is a big pain in the ass...

But why people often avoid writing tests? Well, as I see often - because they think it's a lot of additional code to write.

So I'll try to tell from my experience, how to write less test code :)

## Asserts
Let's start with simple thing, thing that you can spot in many test samples:

``` objc
STAssertNotNil(objects, @"Could not load objects");
```


Well, if it's one line in code - it's kind of OK. But if you need to write these `STAssers`-s over 9000 times... you know what I mean. It's getting even worse if you want to compare simple types like `int` or `float`.

So my number one pick is a small library called [Expecta](https://github.com/petejkim/expecta). Just add to your `Podfile` (I think you'r using [cocoapods](http://cocoapods.org), don't you?) `pod 'Expecta'` and you'll have a simple and nice assert tests like

``` objc
expect(objects).toNot.beNil();
expect(objects.count).to.equal(3);
expect(evaluated).to.equal(YES);

// or more fancy stuff
NSDictionary *dict2 = @{@"id" : @"articleId", @"name" : @"name", @"title" : @"title"};
expect(dict).to.equal(dict2);
```





## HTTP requests
A lot of developers find it hard to test their `API`-classes and network-related code. Well, I'll give you some tips on it.

I'm using [Nocilla](https://github.com/luisobo/Nocilla) library. (`pod 'Nocilla'`). It's a small library that can stub some network requests, and return back stuff like Headers, response code, etc. It's in active development, so if you want to refer to latest code, you should put in your `Podfile` `pod 'Nocilla', podspec: 'https://github.com/luisobo/Nocilla/raw/master/Nocilla.podspec'`

Some code samples:

``` objc
NSString *data = @"Some data here";
stubRequest(@"GET", @"http://localhost/articles").
andReturn(200).withHeaders(@{@"Content-Type": @"application/json"}).withBody(data);
```


Now if you'll make a normal `NSURLRequest` to `http://localhost/articles` you'll get a response code `200`, Header `Content-Type: application/json` and, of course, your piece of data :). Oh, and don't forget to initialize and clear the stubs:

``` objc
- (void)setUp {
    [[LSNocilla sharedInstance] start];
}
- (void)tearDown {
    // Clear Network stubs
    [[LSNocilla sharedInstance] clearStubs];
    [[LSNocilla sharedInstance] stop];
}
```


## Asynchronous testing
Network tests have delays. yep. and you need to deal with them. And your tests need to wait for it. Same for blocks. Methods, where you need to check return values in blocks need asynchronous testing.

In internet, you can find a lot of examples how to do that. The one I've used is

``` objc
dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
[Thing doSomethingOnSuccess:^(NSArray *objects) {
        expect(objects).toNot.beNil();
        dispatch_semaphore_signal(semaphore);
    } failure:^(NSError *error) {
        dispatch_semaphore_signal(semaphore);
    }];
while (dispatch_semaphore_wait(semaphore, DISPATCH_TIME_NOW)) {
        [[NSRunLoop currentRunLoop] runMode:NSDefaultRunLoopMode
                                 beforeDate:[NSDate dateWithTimeIntervalSinceNow:2]];
}
```


Writing this once - kind of OK. But again, we can make it simpler. Personally, I've created a small library for that. [TKSenTestAsync](https://github.com/xslim/TKSenTestAsync). Just add `pod 'TKSenTestAsync'` and the code above can be re-written as

``` objc
[self runTestWithBlock:^{
        [Thing doSomethingOnSuccess:^(NSArray *objects) {
        expect(objects).toNot.beNil();
         [self blockTestCompleted]; // required
    } failure:^(NSError *error) {
        [self blockTestCompleted]; // required
    }];
}];
```


looks much more clear?

## CoreData and.. yeah, RestKit stuff
Well, currently I'm using [RestKit](https://github.com/RestKit/RestKit) a lot, and a nice thing about it, that it has few nice helpers for Unit testing.

First of all, you need to add it's testing sub-library. Currently, due to some bug in main cocoapods `spec`, you need to write in your `Podfile`:

``` ruby
pod 'RestKit', podspec: 'https://github.com/RestKit/RestKit/raw/development/RestKit.podspec'
pod 'RestKit/Testing', podspec: 'https://github.com/RestKit/RestKit/raw/development/RestKit.podspec'
```


Now, before starting, check that you have included `Your.xcdatamodeld` in Unit tests target.

Setting up stuff:

``` objc
        RKManagedObjectStore *store = [RKTestFactory managedObjectStore];
        [RKManagedObjectStore setDefaultStore:store];
```


Now I found a small bug in this step... in my case, a directory was missing in iPhone Simulator, so the above code was failing. To fix this, just run following before

``` objc
+ (void)checkPathForCoreDataFile {
    NSString *path = RKApplicationDataDirectory();
    NSError *error = nil;
    BOOL isDir = YES;
    NSFileManager *fm= [NSFileManager defaultManager];
    if (![fm fileExistsAtPath:path isDirectory:&isDir]) {
        if (![fm createDirectoryAtPath:path withIntermediateDirectories:YES attributes:nil error:&error]) {
            NSLog(@"Error: Create folder failed");
        }
    }
}
```


Now, let's test it:

``` objc
- (void)testCoreDataIntegration {
    RKManagedObjectStore *managedObjectStore = [RKManagedObjectStore defaultStore];

    expect(managedObjectStore).toNot.beNil();

    NSManagedObjectContext *moc = managedObjectStore.persistentStoreManagedObjectContext;
    expect(moc).toNot.beNil();


    Article *mo = [RKTestFactory insertManagedObjectForEntityForName:@"Article" inManagedObjectContext:moc withProperties:nil];
    mo.title = @"SomeObject";

    [moc saveToPersistentStore:NULL];

    expect(mo).toNot.beNil();


    NSFetchRequest *fetchRequest = [[NSFetchRequest alloc] initWithEntityName:@"Article"];

    NSError *error = nil;
    NSArray *objects = [moc executeFetchRequest:fetchRequest error:&error];

    expect(objects).toNot.beNil();
}
```


## Putting it all together

I can write a lot of code here, but the easiest thing is to point you to the library I'm currently developing, which includes all this tests: [RKInjective](https://github.com/AppFellas/RKInjective)

You should check the `RKInjectiveTests` folder contents.

My next article will be about testing a small game engine using [Kiwi](https://github.com/allending/Kiwi) test framework.

## The end.

Well, I hope you liked the stuff I covered here. I hope it can help you in your work. Please leave your comments, tweets, etc. I'll be happy.
