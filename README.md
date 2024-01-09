Oh, well: a Readme is due and it shall be about validatory experiments, ideas, desiderata, problems and how that was made to work. I'll make the Readme like a parchment and add new sections top down (click _outline_ button). Enjoy!

### 11. how many machine instruction bytes contributed to performance

The more `code pathes` are ***made*** available, the more ***shortcuts*** can be realized. This I want to investigate with the `SmallObjects` features, mainly for, but not restricted to, `perform profiling without a simulator`. Due to the nature of `primitive succeded` I seperate pathes taken into parts belonging to `primitive failed` and all the rest. After `primitive routines` have done their thing (just before they return the result), the size (in bytes) of their `machine instruction` s is accumulated for `investigating change of instruction path length` in `relation to timing` of the test.<br>
Thus in `SmallObjects`, benchmark tests can report difference in [performance](https://cseweb.ucsd.edu/classes/sp14/cse141-a/Slides/02_performance_annotated-0417.pdf) if ***not*** the ***code in both parts*** (succeeded | failed) is changed for the same run.<br>
On modern microprocessor chips, the `CPU` runs parallel to a companion `FPU`; as with stacks, the `FPU` is `nett at tos` after it was used for arithmetic. Thus in `SmallObjects`, the accumulation of `machine instruction`s size in bytes, is running on the `actual tos` of the `FPU` (notably, in parallel to the rest of the world).

### 10. inner (nested) loop before computed jump to branch targets, exit condition for outer loop
```
 ok_to_break_outer = 0;
 do { /* interpreter loop, local declarations common to all the following routines */
  do { /* nested loop for concatenating bytecode parms */ } while(more bytecode parms);
  prepare & perform computed jump goto ⁱth branch target (announce "memory" clobber);
  /* in the bytecode routines, handle call-clobbering directly at the call site */
  ¹st_target: … and goto check_for_break;
  ²nd_target: … and goto check_for_break;
  …
  ⁿth_target: … and goto check_for_break;
  check_for_break: … conditionally set ok_to_break_outer;
 } while(!ok_to_break_outer);
```
***Problem***/s (tested only C·lang): the _inner_ loop computes bytes and bits which are also used at the _branch_ targets, so C·lang [spills](https://discourse.llvm.org/t/the-current-state-of-spilling-function-calls-and-related-problems/2863) them for later re-load; ***remedy***: declare `struct*` reference (line before inner loop) which does not exist in the stack, then access fields in the` struct*` ;<br>
***new problem***/s: spilling changed a bit but it still occurs; strange: the `stack*` is _mem_ , the `struct*` is _mem_ , why shuffle things around which can***not change by read-access*** in the _branch_ target routines … ***remedy***: make the fields `volatile` .<br>
***new problem***/s: spilling changed a bit but it still occurs … ***remedy***: declare the `struct*` reference `volatile`.<br>
***Now***! ***spilling no more***, and the `Interpreter` _function prolog_ refraines from handling _call-preserved_ registers :-D<br>
(part of) mission ***accomplished***: the stack is left untouched by C·lang's inventions, thus the _bytecode routines_ can asm "push" & "store" & "pop" (etc) ***from/to*** the `native` machine `stack` , without disturbing the `Interpreter` prolog/epilog (by `calling convention` agreed upon) of C·lang compiler.<br>
P.S. `primitive1Add_` was used as test subject, it performed on _smi_ 3 with _smi_ 4, using [builtin arithmetic with overflow checking](https://gcc.gnu.org/onlinedocs/gcc/Integer-Overflow-Builtins.html), validating by experiment the (`stdcall` with oop receiver) calling convention.<br>
Of note: in the `primitive` routines there is ***no restriction*** for C·lang on the use of the stack.

### 9. recursive multiprocessing in parallel performing cores

In `SmallObjects` I want that each individual `ObjectMemory` instance _theHeap_ is firmly associated k:k with one of the _core_ s in my _CPU_ chip. Having worked with _multiprocessor machines_ since the earliest steps of my work-life, I can't wait that my notebook performs an `oop`-rogram on all its cylinders (_core_ s) and that the (_programmable_ ) task at hand is divided into smaller _and parallel_ pieces of work, to then combine the interim results for the big whole.<br>
Here is my first draft, derived from the classic `benchFib` performance benchmark:
```
!Integer benchFib "handy message-heavy benchmark"
 | subtask := [(self -1) benchFib]. interim |
 self < 2 ifTrue: [^1].
 " at >= 43 (on -m32), LargeInteger arithmetic takes over from SmallInteger "
 self < 43 ifFalse: [interim := subtask promiseUnless: theHeap·idlerCount < 1]
 ifTrue: [interim := subtask "do all myself"].
 ^(self -2) benchFib +interim value +1!!
```
### 8. inviting some C·compiler to jump for nirvana
```
 static oop jumpTargetRoutines[] = {(oop)&&¹st,(oop)&&²nd,…,(oop)&&ⁿth};
 asm goto("jump *%0"::"r"(jumpTargetRoutines[ⁱth])::¹st,²nd,…,ⁿth /*labels*/);
```
This statement I want for jumping according to `computed label jumpTargetRoutines[ⁱth]`; there are _not_ many [asm goto](https://gcc.gnu.org/onlinedocs/gcc/Extended-Asm.html#:~:text=asm%20goto%20allows) examples which demonstrate the previous. At first, the statement does not encode some `jump` to any of the label references %l1, %l2, etc -- yet the label names are passed proforma so that C·compiler knows expected branch targets. At second, the computed target is passed to the statement -- which indeed emits code for the effective jump. There are many things to consider: how the routines at the branch targets have to be set up, how to act (re: control flow) and refer to variables, eventually go further in their enclosing `Interpreter loop`; these tests are work in progress at the time of this writing.

### 7. the adaptive specialist and its carryall

In the previous section (6. below) need arised for `private field`s which belong to _subcontractors_. In the `SmallObjects` system this has rather little to do with _inheritance_, the specialists are trained in fields whose intersection (material type & form, tools, etc) is mostly empty (that makes any pair of _subcontractors_ specialists). And the individual specialists posess sort of `carryall` which they do not share with other _subcontractor_ s.<br>
The solution in `SmallObjects` is to maintain `private field`s (the `carryall`) after the last data item of the `variaPart` (of the _patron_ instance, e.g. a `ByteString` ;-). And the `class header` bits have  (plenty of room ;-) the count 0…15 of `private field`s.<br>
Each `private field` has a pair of `accessor`s which the _compiler_ knows are implemented for `self`. Now, to make the definition and use of skilled specialists and subcontractors not overly complicated, the respective implementations belong to a skill set (of `methods`) which is determined by the presence and absence of `selector`s. Example: … `withSkill: #next thoughNot: #nextPut:` defines the skill set of `ReadStream`s, and `withSkill: #nextPut:` defines the skill set for `ReadWriteStream`s (users of _Browser_ et alii, i.e. realworld developers, appreciate patent navigation through the jungle of _selector_ s).<br>
In `SmallObjects` the `methods` who access the same `private field` belong to the same skill set, yet the developer can append a _suffix_ to `privateField*` `accessor`s for deconfusing the `source method` text. Example:<br>
```
 self privateField1position: 1+ self privateField1position "advance the stream's position".
```
### 6. subcontracting to specialists trained in nonconcurrent fields

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

From section 0 (below) there are already two classes, `Character` and `SmallInteger` which must describe their respective instances. This and all other, their `depictor` ing notwithstanding, objects reside in ObjectMemory. And every _Class object_ has (at least) a _formatInfo_ field which tells the layout (i.e. number of fixed fields, and characteristics of `variable data`) of new (and old, existing) instances. For easing the work of allocation and garbage collection, instances allocated in memory are prepended by a `class header` and their variable part by another, `varia data` header. The `varia data` and its header does not occupy space in memory if the `formatInfo` spec says so. Thus the instances are arranged consecutively in object memory -- until a `garbage collector` finds they are no longer referenced.
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
Things which are `oop` have a tag (bit), as featured in Smalltalk-80. When [object-oriented programming](https://www.oscar.nierstrasz.org/posts/2023-06-12-MindTheGap#:~:text=Object%2Doriented%20programming) was introduced, this was also shown by [Tektronix with 68000](https://retrocomputingforum.com/t/smalltalk-on-the-68000-by-tektronix) microprocessor which had 16 bit bus. So, _one tag bit_ was impressive bang for the bucks. Then 32 bits microprocessors came along, also came the possibility of _two tag bits_:
```
 if(anOop &1)
  if(anOop &2) {/* it is *aPointer */} else {/* it is aCharacter */};
 else {/* it is aSmallInteger */};
```
The `SmallInteger depictor` is ready for _smi_ arithmetic (etc) and all else it can do is to cause overflow; the `Character depictor` has plenty of bits room for encoding `UTF8`. For the `*aPointer oop` the compiler can be told to use an `odd` offset instead of an `even`, all CPUs process the offset bits already in the machine instructions … nowhere systemwise untag.
