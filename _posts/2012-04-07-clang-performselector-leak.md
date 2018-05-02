---
title: "CLANG performSelector leak"
tags: [code, obj-c]
---

If you ever get a compiler warning *performSelector may cause a leak because its selector is unknown* you can suppress it in LLVM 3.0:

``` objc
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Warc-performSelector-leaks"
    [self.target performSelector:@selector(someAction:) withObject:self];
#pragma clang diagnostic pop
```
