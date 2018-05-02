---
published: true
title: How to prove that integrator is using a wrong lib version
---

So let's say an integrator tells you that he's using the iOS library version 1.10.10, and he found the crash:

``` 
Fatal Exception: NSInvalidArgumentException 
*** -[__NSPlaceholderDictionary initWithObjects:forKeys:count:]: attempt to insert nil object from objects[0]
```

And the Crash log looks like this:

```
9  libobjc.A.dylib                0x39a6ada3 objc_exception_throw + 250
10 CoreFoundation                 0x2eddd413 -[__NSPlaceholderDictionary initWithObjects:forKeys:count:] + 530
11 CoreFoundation                 0x2eddd1db +[NSDictionary dictionaryWithObjects:forKeys:count:] + 50
12 payleven-iPad                  0x00422699 -[ADYTransactionProcessor setFailureInfoFromResponse:] (ADYTransactionProcessor.m:732)
```

So we see the problem is in file `ADYTransactionProcessor` on line `732`. So you go to your source code, you look the tag `1.10.10` and you look on that line, and guess what do you see - the code is not there, and you already included the check for that error. 

Well, maybe when you made a build something *BAD* happened, and the old file got in? How can we tell this?

There is a way! Reverse engineering!

So we download the compiled library, same thet the integrator is using, and we extract the symbol from it:

```
lipo -info AToolkit
lipo AToolkit -thin armv7 -output a7
ar -t a7
ar -xv a7 ADYTransactionProcessor.o
lldb ADYTransactionProcessor.o
```

Now we check the crashed address `0x00422699`:

```
(lldb)image lookup --address 0x00422699
```

Nothing. To find out where the function is:

```
(lldb) image lookup -n setFailureInfoFromResponse:
1 match found in ADYTransactionProcessor.o:
Address: ADYTransactionProcessor.o[0x00004068] (ADYTransactionProcessor.o.__TEXT.__text + 16488)
```
BTW, we could also lookup the line number, knowing what function should be there. For example, in 1.10.10, the line number 1320 should be in function `numericState`, while in the other version, it could be `statusIsFinal`:

```
(lldb) image lookup -l 1320 -f ADYTransactionProcessor.m
1 match found in ADYTransactionProcessor.m:1320
Summary: ADYTransactionProcessor.o`-[ADYStatusTenderResponse numericState]
```

*UPDATE:* As noted by a friend, another way to get the info about the address is:

```
atos -arch armv7 -o ADYTransactionProcessor.o 0x00004068
-[ADYTransactionProcessor setFailureInfoFromResponse:] (in ADYTransactionProcessor.o) (ADYTransactionProcessor.m:659)
```
