Oh, well: a Readme is due and it shall be about design, ideas, desiderata, problems and how that was made to work. I'll make the Readme like a parchment and add new items top down (click _outline_ button). Enjoy!

### 8. inviting some C·compiler to jump for nirvana
```
 static oop jumpTargetRoutines[] = {(oop)&&¹st,(oop)&&²nd,…,(oop)&&ⁿth};
 asm goto("jump *%0"::"r"(jumpTargetRoutines[ⁱth])::¹st,²nd,…,ⁿth /*labels*/);
```
This statement we want for jumping according to `computed label jumpTargetRoutines[ⁱth]`; there are _not_ many [asm goto](https://gcc.gnu.org/onlinedocs/gcc/Extended-Asm.html#:~:text=asm%20goto%20allows) examples which demonstrate the previous. At first, the statement does not jump to any of the label references %1, %2, etc -- yet the label names are given proforma so that C·compiler knows expected branch targets. At second, the computed target is passed to the statement -- which indeed emits code for the effective jump. There are many things to consider: how the routines at the branch targets have to be set up, how to act and refer to variables, and then go further in their enclosing `Interpreter loop`; these tests are work in progress at the time of this writing.

### 7. the adaptive specialist and its carryall

In the previous item (6. below) need arised for `private field`s which belong to _subcontractors_. In the `SmallObjects` system this has rather little to do with _inheritance_, the specialists are trained in fields whose intersection (material type & form, tools, etc) is mostly empty (that makes any pair of _subcontractors_ specialists). And the individual specialists posess sort of `carryall` which they do not share with other _subcontractor_ s.<br>
The solution in `SmallObjects` is to maintain `private field`s (the `carryall`) after the last data item of the `variaPart` (of the _patron_ instance, e.g. a `ByteString` ;-). And the `class header` bits (have plenty of room ;-) to count for 0…15 `private field`s.<br>
Each `private field` has a pair of `accessor`s which the _compiler_ knows are implemented for `self`. Now, to make the definition and use of skilled specialists and subcontractors not overly complicated, the respective implementations belong to a skill set (of `methods`) which is determined by the presence and absence of `selector`s. Example: … `withSkill: #next thoughNot: #nextPut:` defines the skill set of `ReadStream`s, and `withSkill: #nextPut:` defines the skill set for `ReadWriteStream`s (users of _Browser_ et alii, i.e. realworld developers, appreciate patent navigation through the jungle of _selector_ s).<br>
In `SmallObjects` the `methods` who access the same `private field` belong to the same skill set, the developer can append a _suffix_ to `accessor`s for deconfusing the `source method` text. Example:<br>
```
 self privateField1position: 1+ self privateField1position "advance the stream's position".
```
### 6. subcontracting to specialists trained in other fields

Suppose that `ArrayedCollection`s have a _private field_ named `position`; then every of its subclasses (e.g. `ByteString`,`Array`, there are about 47 in _Squeak6r0_) is then usable as `ReadStream`; the `readLimit` field of `PositionableStream`s is then the present size of the `ArrayedCollection` (default, anyways). The previous is by far the most frequent usecase of `Stream`s, and that alone deserves extra attention -- since creating a `ReadStream` on an`ArrayedCollection` \~doubles the cost of `garbage collection` for the respective `ArrayedCollection`. We have seen not the least in times not long ago that billionaire software shops invented `slice`d arrays for getting performance max from `positionable` access.<br>
But where can the _private field_ be stored, without disturbing the existing integrity (etc) of `ArrayedCollection`s (and subclasses) -- that cannot be supplemented by a named field? This is a question for specialists in [`mixin` and `traits`](https://boris.unibe.ch/104671/1/Nier05eFlatteningTraitsTR.pdf).

### 5. parting-up decisions towards a responsive free-programmable Polymorphic Inline Cache

There are situations where a `method activation` can see that the decisions made, in a call site, for branching, [could have been more to the case](https://rmod-files.lille.inria.fr/Team/Texts/Papers/Milo18a-SICP-InlineCache.pdf#:~:text=lack%20of%20static%20type%20information%20in%20dynamically). If the decisions were a piece of bits, we want to move that piece either closer to the callee or (in parts) back to the call site. This results in a basically free-programmable Polymorphic Inline Cache. When the compiler has no choice, as in e.g. compiling `self zork`, it defers to guard the sender side with `self doesNotUnderstand: #zork`.<br>
If, on the other side, there can only be one implementor of e.g. `#evaluate:in:to:notifying:ifFail:`, it is sufficient to guard the implementor side with `(self class inheritsFrom: theMethod methodClass) or: [self doesNotUnderstand: theMethod selector]`. Contrast this with `method lookup` where `selectors` (in perhaps huge dictionaries on perhaps plenty of levels of hierarchy, unprogrammable at runtime …) are searched for -- here we 'only' check the `inheritance relation` path.<br>
The `SmallObjects compiler` can automate this and add a &lt;pragma …&gt; to the `compiled method`. This is made available for editing the `method source`. At the present time, the mechanism/s can be added manually but there is as yet no tool support. Yet the `primitiveMethodclassGuardOr_` is already available and used.<br>
Based on the previous, the `fallback method` for `primitiveAdd_` is, implemented in `Number`:
```
 !Number «arithmetic» + aNumber « … »!
  " supra tail-tails for #+ primitive º1 ← primitiveAdd_ "
  self ¹class "act as free-programmable Polymorphic Inline Cache"
    ²== SmallInteger and: [^theMethod ← "fallback" Integer·#+];
    ²== LargeInteger and: [^theMethod ← LargeInteger·#+];
    ²== Fraction and: [^theMethod ← Fraction·#+];
    ²== Float and: [^theMethod ← Float·#+];
    ²== Point and: [^theMethod ← Point·#+].
 ^self doesNotUnderstand: #+! !
```
### 4. the best developed talent for the most complex task (~chiropractic to ~oneself)

When talented craftsmen come to your construction site (of ideas, what else), they bring tools and materials with them; and they clean up before they leave -- but they handover the `result`s to the patron. This is on offer for garbage collection (on `boot level, root level` and beyond) in the `SmallObjects runtime` -- and things have already been provided below. We add the concept of `temporary variables`, they are pushed on the stack during the `prolog` of `method activation`, in `Smalltalk syntax` (plus initialization during declaration):
```
 | requestResponse := self provide input for craftsmen. |
 requestResponse := requestResponse taskDesired perform: #realization with: requestResponse.
 ^requestResponse
```
This shall be the general scheme for contract work; we now add `garbage collection`:
```
 | tideLevel := theHeap garbageCollectMost; tideLevel. requestResponse := self … |
 requestResponse := requestResponse taskDesired perform: #realization with: requestResponse.
 theHeap garbageCollectMost: "the previous" tideLevel.
 ^requestResponse
```
In the `booting from roots` system, the `garbageCollectMost*` selectors can be omitted (or return default/s) without change of function, if it is known what the craftsmen need -- but this is the case during  `booting from roots`.<br>
At another time and during other situations the `SmallObjects runtime` will need support in its empire to clean up `non-referenceable objects` in ObjectMemory, but not already at first.

### 3. telling the `compiler` about the `fixed field`s needed by the `Interpreter`

All instances `inherit` the `fixed field`s `formatInfo` which have already been declared in their respective `superclass`. This must have had a beginning and here we meet the `odd` tagged `oop` again:
```
typedef struct Small$Object {
#pragma pack(1)
  unsigned char $i$n$M$e$m$o$r$y$[sizeof(char)];
  oop inMemory[0];
#pragma pack()
} Small$Object;
```
This is sufficient for the `compiler` to emit proper offset in machine instructions for the memory reference `anOop->inMemory`. Also, the `class header` bits are not seen (but can be addressed using `&anOop[0]` as base for offset). Thus, the first few `fixed field`s of `theHeap` (see 1. memory layout, below) are declared:
```
typedef struct Object$Memory {
#pragma pack(1)
 unsigned char $i$n$M$e$m$o$r$y$[sizeof(Small$Object)];
 oop nilObj /* boot's ²nd object */;
 oop formatInfo /* 36r686m32, or 36rarm32f */;
 oop systemKeys /* anArray associated ↔ k:k ↔ with items in my variaPart */;
 oop trueObj, falseObj;
 oop heapCircle /* collection of forked threads */;
 oop newMethod …
	…
 oop variaData[0] /* oop(s) of ⁱth aClass, ⁱth > 0, in this variaPart */;
#pragma pack()
} Object$Memory;
```
On the side, there are CPU chips which enable developers to [trapping misaligned memory access](https://scholar.google.com/scholar?hl=en&q=+An+Evaluation+of+Misaligned+Data+Access+Handling+Mechanisms+in+Dynamic); when the native CPU supports this, the SmallObjects `Interpreter runtime` is wrapped by this hardware feature (and unwrapped around invocation of `OS` system function -- who could no care less about alignment):
```
…asm("pushfl;xorl $0x40000,(%esp);popfl")…
```
Such code snippets are made as `define macro` which can be translated at central place for support of other platforms (etc).

### 2. interleaving method activation and dispatch of primitives

One of the goals in SmallObjects is, to do high-level code as much as essential, low-level as little as possible. Therefore, `sending a message` has the same calling convention (`stdcall` with oop receiver) as dispatch of primitives (tradition since Smalltalk-80 et alii):
```
 push aReceiver -- or use the one already on stack (at tos);
 push (numArity) argument/s;
 dispatch ⁱth primitive; or
 invoke ⁱth literal (a method object);
 either forget returned value, or replace tos location by it, else push it on the stack;
```
The difference between `dispatch` and `invoke` is just the causing bytecode, same so for the returned value. However, the `primitive` may fail -- and therefore must delegate to a `fallback method`. In that case the operation is like:
```
 theMethod := aFallbackMethod /* acts like tailcall */;
```
Since the `arity` of the current `activation` is same as that of aFallbackMethod, nothing else is to worry about (given the assignment does set up for `next instruction`). In another posting the role and viability of `theMethod` will be addressed, here we note that it is a `special variable` belonging to, what momentarily the current `activation` is.</br>
N.B. [tailcall](https://clang.llvm.org/docs/AttributeReference.html#musttail) is in developer's hand for C·lang.

### 1. the SmallObjects memory & layout & format info

From item 0 (below) there are already two classes, `Character` and `SmallInteger` which must describe their respective instances. This and all other, non-`depictors` notwithstanding, objects reside in ObjectMemory. A _Class_ has (at least) a _formatInfo_ field which tells the layout (i.e. number of fixed fields, and characteristics of `variable data`) of new (and old, existing) instances. For easing the work of allocation and garbage collection, instances allocated in memory are prepended by a `class header` and their variable part by another `varia data` header. The `varia data` and its header does not occupy space in memory if the `formatInfo` spec says so. Thus the instances are arranged consecutively in object memory -- until a `garbage collector` finds they are no longer referenced.
The `ObjectMemory` instance `theHeap` has several _constant_ fields, for: the `nil` oop, also the `true` and `false` oop's. Beeing an instance, `ObjectMemory` has fields and `varia data`, but above all it has `absolute` location (warranted by the `OS`):
```
 _Thread_local oop Thread$isolated$Heap;
```
This `oop` is assigned (trivial from `sbrk`, elaborate from `mmap`) once the `main` (or other launcher) gets control. The `_Thread_local` storage class has became rather bugfree on many platforms. In `SmallObjects` it is used this way:
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
