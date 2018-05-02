---
title: "Reverse-engineering iOS app"
tags: [code, ios, obj-c, hacking]
redirect_from: "/blog/2013/02/11/reverse-engineering-ios-app/"
---

On one of projects I was involved in, we were implementing `Captive Login` technology, to automatically log in networks that have `Captive Portal` via `WISPr` technology. Everything went quite fine, except that on iOS, you can't do auto-login to WiFi network without user's permission. And the `CaptiveLogin` support is quite limited.

Then we suddenly spotted an app, that could do all that cool things. That was the start of my small research.

## The Beginning Is the End Is the Beginning

For sake of convinience, let's call tha app that we'r interested in `iApp`. We downloaded the app from AppStore, renamed `.ipa` to `.zip`, and started looking inside.

First of all, we opened `Info.plist`. And found a quite interesting string `network-authentication` for `Required background modes`. Google gave us 0 results.

Next thing, I checked `iApp.entitlements` file. And found there a nice key `com.apple.developer.CaptiveNetworkPlugin` saying `YES`. That was something.



## Disassembling

I wanted to check if the app uses some privete libraries or methods related to `CaptiveNetwork`. To find that out, I used [Hopper Disassembler](http://hopperapp.com). For checking the app disassembly, the Demo version is quite good.

After some digging I found a notice of using some `Plugin`

![](/images/2013-02-11/1.png)

That gave me idea of usage of some private `CaptiveNetwork` library. I found one in `/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS6.1.sdk/System/Library/PrivateFrameworks/CaptiveNetwork.framework` (similar path to ARM version)

`nm -g CaptiveNetwork` gave me an interesting list of public methods (one starts with `T`):

``` bash
00001e78 T _CNAccountsAdd
00001e40 T _CNAccountsCopy
00002018 T _CNAccountsResolve
00001cb4 T _CNAccountsUse
000022e4 T _CNCheckControlSettings
00001ad0 T _CNDebugLaunchWebsheet
000022c4 T _CNDumpState
00001bf4 T _CNForgetSSID
00002074 T _CNIAmTheWebsheetApp
00001b60 T _CNLogoff
000049c0 T _CNNetworkCopyPluginNames
00004890 T _CNNetworkGetBSSID
00004858 T _CNNetworkGetSSID
00004820 T _CNNetworkGetSSIDString
00004908 T _CNNetworkGetSignalStrength
00004744 T _CNNetworkGetTypeID
000048c8 T _CNNetworkIsProtected
00004770 T _CNNetworkSetCaptive
000047e0 T _CNNetworkSetConfidence
00004970 T _CNNetworkWasAutoJoined
000050e0 T _CNPluginCommandBindReadStream
00005110 T _CNPluginCommandCopyCurrentNetwork
0000512c T _CNPluginCommandCopyNetworkList
000050d0 T _CNPluginCommandGetInterfaceName
000050cc T _CNPluginCommandGetType
000050a0 T _CNPluginCommandGetTypeID
00004bbc T _CNPluginRegister
00005290 T _CNPluginResponseCreate
00005300 T _CNPluginResponseDeliver
00005264 T _CNPluginResponseGetTypeID
000054b8 T _CNPluginResponseSetNetwork
0000543c T _CNPluginResponseSetNetworkList
00001798 T _CNProberCreate
000019dc T _CNProberCreateRunLoopSource
000016f0 T _CNProberGetTypeID
```


which looked quite interesting. So not thinking for so long, I opened the library in disassembly:

![](/images/2013-02-11/2.png)

Hmm, they are getting the info from some `NSDictionary` ?

## Putting stuff together

Well, I tried making a demo app and using that private framework.

But how do we get access to it's C-functions? Her's the answer:

``` objc
// Get the bundle framework
CFBundleRef bundle = CFBundleGetBundleWithIdentifier(CFSTR("com.apple.CaptiveNetworkSupport"));

// Declare the function

// let's start with some simple one
BOOL (*CNNetworkWasAutoJoined)(CFDictionaryRef) = NULL;
    CNNetworkWasAutoJoined = (void *)CFBundleGetFunctionPointerForName(bundle, CFSTR("CNNetworkWasAutoJoined"));

// register us in networks
CFStringRef ssids[2] = { CFSTR("OurWiFi"), CFSTR("NeighboursWiFi") };
    CFArrayRef arr_ssids = CFArrayCreate(NULL, (const void **)ssids, 2, &kCFTypeArrayCallBacks);

    if( CNSetSupportedSSIDs((CFArrayRef)arr_ssids)) {
        NSLog(@"Successfully registered supported network SSIDs");
    } else {
        NSLog(@"Error: Failed to register supported network SSIDs");
    }
    CFRelease(arr_ssids);

// Find network interfaces by CNCopySupportedInterfaces()
NSString* interface = @"en0";  // or hard code it

// Get network details
NSDictionary* networkDetails = (__bridge NSDictionary*)CNCopyCurrentNetworkInfo((__bridge CFStringRef) interface);

// check it?
NSLog(@"all details: %@", [networkDetails description]);

// Let's use the private method
BOOL x = CNNetworkWasAutoJoined((__bridge CFDictionaryRef)(networkDetails));
NSLog(@"X: %d", x);
```


Now everything seems working till the last one. If we'll check debugger, we're getting the `bundle`, we're getting the private function in `CNNetworkWasAutoJoined`, but we'r getting crash when we try using it. It seems the `networkDetails` doesn't contain the needed information.

## The End Is the Beginning Is the End

Well, idea of putting `com.apple.developer.CaptiveNetworkPlugin` to `Entitlments` did'nt work. There need to be a valid Provisioning profile for this thing. And the valid one can be issued only by Apple.

So it brings us to idea that the creators of this `iApp`
 got in contact with Apple, and got a special priviledges for their app. Well, we also contacted Apple, but still no response from their side.
