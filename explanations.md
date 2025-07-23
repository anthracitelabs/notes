https://stackoverflow.com/questions/71006170/does-memcpy-copy-bytes-in-reverse-order

https://www.baeldung.com/linux/profiling-processes

https://stackoverflow.com/questions/10204471/convert-char-array-to-a-int-number-in-c

split 16 bit into two bytes
unsigned int x = 0b1010001000110011;
uint8_t xlow = x & 0xff;
uint8_t xhigh = (x >> 8);
