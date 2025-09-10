https://stackoverflow.com/questions/71006170/does-memcpy-copy-bytes-in-reverse-order

https://www.baeldung.com/linux/profiling-processes

https://stackoverflow.com/questions/10204471/convert-char-array-to-a-int-number-in-c

split 16 bit into two bytes
unsigned int x = 0b1010001000110011;
uint8_t xlow = x & 0xff;
uint8_t xhigh = (x >> 8);

-------------------------------------------------------------------------------------------------------------------
https://stackoverflow.com/questions/41132753/how-can-i-build-an-android-apk-without-gradle-on-the-command-line

Use the following steps to build your apk manually, if you don't want use ant/gralde to build. But you must have Android SDK installed at least.

create R.java from aapt
use javac to compile all java source to *.class
use d8 (or dx in older SDKs) to convert all *.class to dex file, e.g output is classes.dex
create initial version of APK from assets, resources and AndroidManfiest.mk, e.g output is MyApplication.apk.unaligned
use aapt to add classes.dex generated in step 3 to MyApplication.apk.unaligned
use jarsigner to sign MyApplication.apk.unaligned with debug or release key
use zipalign to align the final APK, e.g output is MyApplication-debug.apk or MyApplication-release.apk if signing with release key
Done

I have created a sample script to do all the stuffs above, see here:
https://github.com/WanghongLin/miscellaneous/blob/master/tools/build-apk-manually.sh

Actually, Some articles have discussed this topic, see the following links.

https://www.apriorit.com/dev-blog/233-how-to-build-apk-file-from-command-line
https://spin.atomicobject.com/2011/08/22/building-android-application-bundles-apks-by-hand/
-------------------------------------------------------------------------------------------------------------------

Android Build Toolchain

https://android-developers.googleblog.com/2024/09/become-better-android-developer-compiler-explorer.html

typedef
=======================================
Typedef is used to create aliases to existing types. It's a bit of a misnomer: typedef does not define new types as the new types are interchangeable with the underlying type. Typedefs are often used for clarity and portability in interface definitions when the underlying type is subject to change or is not of importance.

For example:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ C
// Possibly useful in POSIX:
typedef int filedescriptor_t;

// Define a struct foo and then give it a typedef...
struct foo { int i; };
typedef struct foo foo_t;

// ...or just define everything in one go.
typedef struct bar { int i; } bar_t;

// Typedef is very, very useful with function pointers:
typedef int (*CompareFunction)(char const *, char const *);
CompareFunction c = strcmp;
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Typedef can also be used to give names to unnamed types. In such cases, the typedef will be the only name for said type:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ C
typedef struct { int i; } data_t;
typedef enum { YES, NO, FILE_NOT_FOUND } return_code_t;
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
