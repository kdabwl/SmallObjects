Oh, well: a Readme is due and it shall be about design, ideas, desiderata, problems and how that was made to work. I'll make the Readme a log and add new items top down. Enjoy!

0. how came the term `oop` into the project? the closest I found was:
```
#include <stdint.h>
typedef uintptr_t oop;
```
Things which are `oop` have a tag (bit), as featured in Smalltalk-80. When object-oriented programming was introduced, is was also shown by [Tektronix with 68000](https://retrocomputingforum.com/t/smalltalk-on-the-68000-by-tektronix) microprocessor which had 16 bit bus. So, _one tag bit_ was impressive bang for the bucks. Then 32 bits microprocessors came along, also came the possibility of _two tag bits_:
```
if(anOop &1)
	if(anOop &2) {/*it is *aPointer */} else {/* it is aCharacter */};
else {/* it is aSmallInteger */};
```
The `SmallInteger depictor` is ready for arithmetic (etc) and all else it can do is to cause overflow; the `Character depictor` has plenty of bits room for encoding `UTF8`. For the `*aPointer oop` the compiler can be told to use an `odd` offset instead of an `even`, all CPUs process the offset bits already in the machine instructions.
