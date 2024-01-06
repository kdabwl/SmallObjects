Oh, well: a Readme is due and it shall be about design, ideas, desiderata, problems and how that was made to work. I'll make the Readme a log and add new items top down. Enjoy!

### 2. interleaving method activation and dispatch of primitives

One of the goals in SmallObjects is, to do high-level code as much as essential, low-level as little as possible. Therefore, method activation has the same calling convention (`stdcall`) as dispatch of primitives:
```
push aReceiver -- or use the one already on stack (at tos);
push (numArity) argument/s;
dispatch ⁱth primitive; or
invoke ⁱth literal (a method object);
either forget returned value, or replace tos location by it, else push it on the stack;
```
The difference between `dispatch` and `invoke` is just the causing bytecode, same so for the returned value. However, the `primitive` may fail -- and therefore must delegate to a `fallback method`. In that case the operation is like:
```
theMethod := aFallbackMethod;
```
Since the `arity` of the current `activation` is same as that of aFallbackMethod, nothing else is to worry about (given the assignment does set up for `next instruction`). In another posting the role and viability of `theMethod` will be addressed, here we note that it is a `special variable` belonging to, what momentarily the current `activation` is.

### 1. the SmallObjects memory & layout & format spec

From item 0 (below) there are already two classes, `Character` and `SmallInteger` which must describe their respective instances. This and all other, non-`depictors` notwithstanding, objects reside in ObjectMemory. A _Class_ has (at least) a _format_ field which tells the layout (i.e. number of fixed fields, and characteristics of `variable data`) of new instances. For easing the work of allocation and garbage collection, instances allocated in memory are prepended by a `class header` and their variable part by another `varia data` header. The `varia data` header does not occupy space in memory if the `format` spec says so. Thus the instances are arranged consecutively in object memory -- until a `garbage collector` finds they are no longer referenced.
The `ObjectMemory` instance `theHeap` has several fields: the `nil` oop, also the `true` and `false` oop's. Beeing an instance, `ObjectMemory` has fields and `varia data`, but above all it has `absolute` location:
```
_Thread_local oop Thread$isolated$Heap;
```
This `oop` is assigned (trivial from `sbrk`, elaborate from `mmap`) once the `main` (or other launcher) gets control. The `_Thread_local` storage class has became rather bugfree on many platforms. It is used this way:
```
 ObjectMemory* theHeap = (ObjectMemory*)Thread$isolated$Heap;
```
All other objects in memory can only be stored in `theHeap` and the allocated space it describes. The  `varia data` items of `theHeap` instance are indexed by `classId` bits from the `class header` of instances (or defaults for `depictor`'s). After the last `varia data` item of `theHeap` begins the zone of resident objects (up to the `tideLevel` mark), then the zone of jetsam (up to the `shoreline` mark). The fields `tideLevel` and `shoreline` belong to the `fixed fields` of the ObjectMemory instance (many other not mentioned here). Surprise: all this is accessible from ordinary code and therefore from the SmallObjects `Interpreter`.

### 0. how came the term `oop` into the project? the closest I found was:
```
#include <stdint.h>
typedef uintptr_t oop;
```
Things which are `oop` have a tag (bit), as featured in Smalltalk-80. When object-oriented programming was introduced, this was also shown by [Tektronix with 68000](https://retrocomputingforum.com/t/smalltalk-on-the-68000-by-tektronix) microprocessor which had 16 bit bus. So, _one tag bit_ was impressive bang for the bucks. Then 32 bits microprocessors came along, also came the possibility of _two tag bits_:
```
if(anOop &1)
	if(anOop &2) {/* it is *aPointer */} else {/* it is aCharacter */};
else {/* it is aSmallInteger */};
```
The `SmallInteger depictor` is ready for arithmetic (etc) and all else it can do is to cause overflow; the `Character depictor` has plenty of bits room for encoding `UTF8`. For the `*aPointer oop` the compiler can be told to use an `odd` offset instead of an `even`, all CPUs process the offset bits already in the machine instructions … nowhere systemwise untag.
