---
title: "How I RestKit"
tags: [code, ios, obj-c]
redirect_from: "/blog/2012/05/17/how-i-restkit/"
---

*Note:* The post is quite outdated. Please use the manual on the official RestKit project website.

In this short post I'll show how I use [RestKit](http://restkit.org) library in my iOS projects.

First of all, I want to say that for managing all my libraries in iOS projects I use [CocoaPods](http://cocoapods.org) (kind of Gemfile bundler, but for iOS), which I strongly recommend using.

So, RestKit... open up your `Podfile` and add

``` ruby
dependency 'RestKit/Network'
dependency 'RestKit/UI'
dependency 'RestKit/ObjectMapping'
dependency 'RestKit/ObjectMapping/CoreData'
dependency 'RestKit/ObjectMapping/JSON'
```


I also use [Injective](https://github.com/farcaller/Injective) dependency a lot, so I add `dependency 'Injective'` to my Podfile



Next, run `pod install` which will download latest RestKit (In my case - version `0.10.0`) and integrate it to your project.

## The code
Examples I'll provide will use `RM` class prefix, and API I'll refer is from my start-up `rabat.me`. First I'll write examples without data persistancy, than I'll provide examples how to use CoreData. Also, I add `#import <RestKit/RestKit.h>` and `#import "IJContext.h"` to my `App-Prefix.pch`

You can find full source code of the project on GitHub [https://github.com/xslim/RabatMe](https://github.com/xslim/RabatMe)

## Initialization
For initializing Injective library, edit your `AppDelegate.m`

``` objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    [IJContext setDefaultContext:[[IJContext alloc] init]];
```


What I also like doing, is initializing RestKit-s logging, so I can use it in my app and changing verbocity level from enviroment variables.

``` objc
// This goes to application:didFinishLaunchingWithOptions:
RKLogInitialize();
RKLogConfigureFromEnvironment();
```


Create class `RMApiConnector` which we will use for all our RestKit bootstrapping and network connections.

``` objc
NSString * const RMErrorDomain = @"org.API.ErrorDomain";

@interface RMApiConnector ()
- (void)fireErrorBlock:(RKRequestDidFailLoadWithErrorBlock)failBlock onErrorInResponse:(RKResponse *)response;
@end

@implementation RMApiConnector

injective_register_singleton(RMApiConnector)

- (id)init {
    self = [super init];
    if (self) {
        [self setupConnector];
        [self setupMapping];
    }
    return self;
}
```


`injective_register_singleton` will add RMApiConnector to singleton collection on DI, so we will be able to access it like

``` objc
RMApiConnector *connector = [RMApiConnector injectiveInstantiate]
```


## Setting up Object Loader

``` objc
- (void)setupConnector {
    NSString *baseUrl = @"http://example.org/api/"
    RKObjectManager *manager = [RKObjectManager sharedManager];

    if (!manager) {
        manager = [RKObjectManager objectManagerWithBaseURL:[RKURL URLWithString:baseUrl]];
        manager.client.serviceUnavailableAlertEnabled = YES;
        manager.requestQueue.showsNetworkActivityIndicatorWhenBusy = YES;
    } else {
        manager.client.baseURL = [RKURL URLWithString:baseUrl];
    }
}
```


## Setting up Object Mapping

``` objc
- (void)setupMapping {
    RKObjectMappingProvider *omp = [RKObjectManager sharedManager].mappingProvider;

    RKObjectMapping *productMapping = [RMProduct mapping];
    [omp addObjectMapping:productMapping];
    [omp setObjectMapping:productMapping forResourcePathPattern:@"/products"];

    RKObjectMapping *shopMapping = [RMShop mapping];
    [omp addObjectMapping:shopMapping];
    [omp setObjectMapping:shopMapping forResourcePathPattern:@"/shops/:id"];
    [omp setObjectMapping:shopMapping forResourcePathPattern:@"/shops/near/:lat/:lng/:radius"];
}
```


The mapping itself I store with Models like

``` objc
@class RKObjectMapping;

@interface RMShop : NSObject

@property (nonatomic, strong) NSString *itemId;
@property (nonatomic, strong) NSNumber *points;
@property (nonatomic, strong) NSString *name;
// Some other properties

@property (nonatomic, strong) NSArray *products;
@property (nonatomic, strong) NSArray *rewards;

+ (RKObjectMapping *)mapping;

@end
```


``` objc
@implementation RMShop

@synthesize itemId, points; // and others

+ (RKObjectMapping *)mapping {
    RKObjectMapping *objectMapping = [RKObjectMapping mappingForClass:[self class] usingBlock:^(RKObjectMapping *mapping) {
        [mapping mapAttributes:@"name", @"address", @"distance", @"location", @"points", nil];
        [mapping mapKeyPathsToAttributes:
         @"id", @"itemId",
         @"description", @"descriptionText",
         @"distance_unit", @"distanceUnit",
         nil];
    }];
    [objectMapping hasMany:@"products" withMapping:[RMProduct mapping]];
    [objectMapping hasMany:@"rewards" withMapping:[RMReward mapping]];
    return objectMapping;
}

@end
```


## Creating API requests

Let's create few requests

``` objc
- (void)loadNearbyShopsForLocation:(CLLocation *)location onLoad:(RKObjectLoaderDidLoadObjectsBlock)loadBlock onError:(RKRequestDidFailLoadWithErrorBlock)failBlock
{
    NSString *params = [NSDictionary dictionaryWithKeysAndObjects:
                        @"lat", [NSNumber numberWithDouble:location.coordinate.latitude],
                        @"lng", [NSNumber numberWithDouble:location.coordinate.longitude],
                        @"radius", [NSNumber numberWithInt:20], // later change this to accuracy
                        nil];

    NSString *url = [@"/shops/near/:lat/:lng/:radius" interpolateWithObject:params];

    RKObjectManager *manager = [RKObjectManager sharedManager];
    [manager loadObjectsAtResourcePath:url usingBlock:^(RKObjectLoader *loader) {
        loader.onDidLoadObjects = loadBlock;
        loader.onDidFailWithError = failBlock;
        loader.onDidFailLoadWithError = failBlock;
        loader.onDidLoadResponse = ^(RKResponse *response) {
            [self fireErrorBlock:failBlock onErrorInResponse:response];
        };
    }];
}
```


What we can see here, is that I'm using block-based API of RestKit. `loadBlock` will get array of loaded objects. `failBlock` will get `NSError` object on error. Also I'm using `fireErrorBlock:onErrorInResponse:` helper to get error message from my API

 ``` objc
- (void)fireErrorBlock:(RKRequestDidFailLoadWithErrorBlock)failBlock onErrorInResponse:(RKResponse *)response {

    if (![response isOK]) {
        id parsedResponse = [response parsedBody:NULL];
        NSString *errorText = nil;
        if ([parsedResponse isKindOfClass:[NSDictionary class]]) {
            errorText = [parsedResponse objectForKey:@"error"];
        }
        if (errorText) failBlock([self errorWithMessage:errorText code:[response statusCode]]);
    } else {
        //id parsedResponse = [response parsedBody:NULL];
        //RKLogTrace(@"response: [%@] %@\n%@", [parsedResponse class], parsedResponse, [response bodyAsString]);
    }
}
```


What if we want to make a *POST* call? Easy. Just tell the loader `loader.method = RKRequestMethodPOST;` and pass `NSDictionary` *POST* params to `loader.params`

``` objc
RKObjectManager *manager = [RKObjectManager sharedManager];
[manager loadObjectsAtResourcePath:@"/stamps" usingBlock:^(RKObjectLoader *loader) {
    loader.method = RKRequestMethodPOST;
    loader.params = [NSDictionary dictionaryWithKeysAndObjects:
                         @"code", clientCode,
                         @"products", productsToSend,
                         nil];
```


What if we don't need to load any objects? Let's say we want to make client Authentication.

``` objc
- (void)authenticateWithLogin:(NSString *)login password:(NSString *)password onLoad:(RKRequestDidLoadResponseBlock)loadBlock onFail:(RKRequestDidFailLoadWithErrorBlock)failBlock
{
    [[RKClient sharedClient] post:@"/login" usingBlock:^(RKRequest *request) {
        request.params = [NSDictionary dictionaryWithKeysAndObjects:
                          @"employee[email]", login,
                          @"employee[password]", password,
                          nil];
        request.onDidLoadResponse = ^(RKResponse *response) {

            id parsedResponse = [response parsedBody:NULL];
            NSString *token = [parsedResponse valueForKey:@"authentication_token"];
            //NSLog(@"response: [%@] %@", [parsedResponse class], parsedResponse);

            if (token.length > 0) {
                NSLog(@"response status: %d, token: %@", response.statusCode, token);
                [[RKClient sharedClient] setValue:token forHTTPHeaderField:@"X-Rabatme-Auth-Token"];

                if (loadBlock) loadBlock(response);
            }

            [self fireErrorBlock:failBlock onErrorInResponse:response];
        };
        request.onDidFailLoadWithError = failBlock;
    }];
}
```


## Integrating CoreData
Great. What about CoreData? Change *RMApiConnector*'s *init* method

``` objc
    if (self) {
        [self setupConnector];
        [self setupDB];
        [self setupMapping];
    }
```


``` objc
- (void)setupDB {
    RKObjectManager *manager = [RKObjectManager sharedManager];
    manager.objectStore = [RKManagedObjectStore objectStoreWithStoreFilename:@"HotSpots.sqlite"];
}
```


For mapping Managed Objects you do

``` objc
- (void)setupMapping {
    RKObjectManager *manager = [RKObjectManager sharedManager];
    RKObjectMappingProvider *omp = [RKObjectManager sharedManager].mappingProvider;

    RKManagedObjectMapping *hotSpotMapping = [RKManagedObjectMapping mappingForClass:[HotSpot class] inManagedObjectStore:manager.objectStore];
    [hotSpotMapping mapAttributes:@"name", @"city", @"country", @"latitude", @"longitude", @"type", @"zipcode", nil];
    [hotSpotMapping mapKeyPathsToAttributes:
     @"address", @"street",
     @"description", @"details",
     @"id", @"hotSpotID",
     @"openinghours", @"openingHours",
     nil];
    hotSpotMapping.primaryKeyAttribute = @"hotSpotID";

    [omp addObjectMapping:hotSpotMapping];
    [omp setObjectMapping:hotSpotMapping forResourcePathPattern:@"app/location"];
```


Now if Object Mapper will see that the object he's mapping is a `NSManagedObject` he will check the DB if the'r already object with same primaryId. If yes - he'll update it with new values, if no - he'll create one for you.
