---
published: true
title: Checking C code with Valgrind on Yosemite
---

Coding in C can be hard. Specially finding a memory leaks after. The help can be found in a tool called [Valgrind](http://valgrind.org). Her's a quick start how to install it on Mac OS X Yosemite and youse it with a simple C app.

## Installing
On Yosemite, we need to rebuild the Valgrind from scratch. To do so, get the source code from the SVN trunk and re-build it:

``` bash
svn co svn://svn.valgrind.org/valgrind/trunk valgrind
cd valgrind
./autogen.sh
./configure
make
make install
```

## Using
The project has a nice [Quick Start Guide](http://valgrind.org/docs/manual/quick-start.html#quick-start.intro), and I suggest reading it later on.

For our case, let's create a simple app in Xcode. Go to _File -> New -> Project_ and select "Command line Tool". 

``` c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, const char * argv[]) {
    char *str = malloc(sizeof(char)*5);
    sprintf(str, "test");
    printf("Output: %s\n", str);
    return 0;
}
```

You can see that we have a memory leak here - we allocated the memory with `malloc`, but we never called `free` to release it. But if you will run the Xcode's _Analyze_, it will show you nothing. It works perfectly for _Objective-C_ projects, but does nothing on the _C_ ones. Let's check th Valgrind.

For the Valgrind to show you where exactly in your code the problem is, it needs to have a `dSYM` file. The easy way is to tell Xcode to always generate the `dSYM` file: go to target build options and set _Debug Information Format_ to `DWARF with dSYM File` (`DEBUG_INFORMATION_FORMAT = dwarf-with-dsym`).

Now build the project, and open the build directory (Somewhere in `~/Library/Developer/Xcode/DerivedData/test-some-hash/Build/Products/Debug`) in the terminal. Try running it to see that it works: `./test` should output `Output: test`. Now run it with Valgrind:

``` bash
valgrind --leak-check=yes ./test
```

You will see a lot of output, but the most interesting parts are one showing if the leak is detected:

```
LEAK SUMMARY:
	definitely lost: 21 bytes in 2 blocks
	indirectly lost: 0 bytes in 0 blocks
		possibly lost: 13,130 bytes in 120 blocks
	still reachable: 25,574 bytes in 304 blocks
		suppressed: 0 bytes in 0 blocks
```

and a place in our code that is leaking:

```
5 bytes in 1 blocks are definitely lost in loss record 1 of 80
	at 0x1000066F1: malloc (vg_replace_malloc.c:303)
	by 0x100000EF7: main (main.c:13)
```

It shows you exactly where it leaked.

But what's with the other output that is about leaks in system libraries that is not relevant to your code? We can suppress it!

## Suppressing the output
To suppress the Valgrind output, you will need to have the `supp` file, and you will need to specify it: `--suppressions=objc.supp`. But if you need to create one, you can use the `--gen-suppressions=yes` to guide you, copy paste the needed suppression blocks in the `supp` file and edit them. For example:

Run the app line this:

``` bash
valgrind --leak-check=yes --gen-suppressions=yes ./test
```

Now you will see, Valgrand detects a possible leak in the system library:

```
16 bytes in 1 blocks are definitely lost in loss record 8 of 80
	at 0x100006A6C: malloc_zone_malloc (vg_replace_malloc.c:305)
	by 0x1004A39B1: recursive_mutex_init (in /usr/lib/libobjc.A.dylib)
    by 0x1004A36FC: _objc_init (in /usr/lib/libobjc.A.dylib)
    by 0x10010C87C: _os_object_init (in /usr/lib/system/libdispatch.dylib)
```

Pressing `Y` will show the suppression block, that we can trim a bit, give it a name and put in our `objc.supp` file:

```
{
   objc_init
   Memcheck:Leak
   match-leak-kinds: definite
   fun:malloc_zone_malloc
   fun:recursive_mutex_init
   fun:_objc_init
   fun:_os_object_init
   fun:libdispatch_init
   fun:libSystem_initializer
}
```

Next time, you can run the app specifying the `supp` file, and Valgrind will skip that blocks.

My suppression for Objective-C can be found in a [gist](https://gist.github.com/xslim/ab46a930a956fac27f65). Feel free to propose changes.

## Useful links
- [Quick Start Guide](http://valgrind.org/docs/manual/quick-start.html#quick-start.intro)
- [Using Valgrind to Find Memory Leaks and Invalid Memory Use](http://www.cprogramming.com/debugging/valgrind.html)