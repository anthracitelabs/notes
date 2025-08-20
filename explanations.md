https://stackoverflow.com/questions/71006170/does-memcpy-copy-bytes-in-reverse-order

https://www.baeldung.com/linux/profiling-processes

https://stackoverflow.com/questions/10204471/convert-char-array-to-a-int-number-in-c

split 16 bit into two bytes
unsigned int x = 0b1010001000110011;
uint8_t xlow = x & 0xff;
uint8_t xhigh = (x >> 8);

-------------------------------------------------------------------------------------------------------------------
Use the following steps to build your apk manually, if you don't want use ant/gralde to build. But you must have Android SDK installed at least.

create R.java from aapt
use javac to compile all java source to *.class
use d8 (or dx in older SDKs) to convert all *.class to dex file, e.g output is classes.dex
create initial version of APK from assets, resources and AndroidManfiest.mk, e.g output is MyApplication.apk.unaligned
use aapt to add classes.dex generated in step 3 to MyApplication.apk.unaligned
use jarsigner to sign MyApplication.apk.unaligned with debug or release key
use zipalign to align the final APK, e.g output is MyApplication-debug.apk or MyApplication-release.apk if signing with release key
Done

I have created a sample script to do all the stuffs above, see here
Actually, Some articles have discussed this topic, see the following links.

https://www.apriorit.com/dev-blog/233-how-to-build-apk-file-from-command-line
https://spin.atomicobject.com/2011/08/22/building-android-application-bundles-apks-by-hand/
-------------------------------------------------------------------------------------------------------------------
