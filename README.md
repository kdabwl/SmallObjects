Oh, well: a Readme is due and it shall be about validatory experiments, ideas, desiderata, problems and how that was made to work. I'll make the Readme like a parchment and add new sections top down (click _outline_ button). Enjoy!

### 15. executing `42 benchFib` with recursive somersault

A subset of the bytecode has been completed and executes `42 benchFib` (with recursive somersault) in ***47 seconds***, calling the arithmetic 2,600,966,656 times. Already for `41 benchFib` the counters had to be made `long long`. Whoever wants to measure `43 benchFib` must first implement LargeInteger routines. Printout of the benchFib codons:<br>
```
 1      a·push from ⁱth arg, 1
 2      a·push smi 2
 3      a·dispatch ⁱth intrinsic, 3
 4      b·nop
 5      a·jump onFalse + ⁱth·some byte, 2
 6      a·push smi 1
 7      a·method return, 1+arity, 1
 8      a·push from ⁱth arg, 1
 9      a·push smi 2
10      a·dispatch ⁱth intrinsic, 2
11      b·nop
12      b·divert to ⁱth method, 4
13      a·push from ⁱth arg, 1
14      a·push smi 1
15      a·dispatch ⁱth intrinsic, 2
16      b·nop
17      b·divert to ⁱth method, 4
18      a·dispatch ⁱth intrinsic, 1
19      b·nop
20      a·push smi 1
21      a·dispatch ⁱth intrinsic, 1
22      b·nop
23      a·method return, 1+arity, 1
24      a·push nil
```
<sub><sup>Fineprint·:~$ lsb_release -d `&&` cat /proc/cpuinfo | grep -i 'model name' | sort -u<br>
Output: Ubuntu 20.04.6 LTS `&&` 11th Gen Intel(R) Core(TM) i5-1135G7 @ 2.40GHz.</sup></sub>


### 14. one 2×jump sandwich per ~1×machine code operation, with some looping sauce on top

The interpreter requires coding for {push.store.pop} U {divert.jump.return} U {compute.check}, so 3 bits of the first byte of code are occupied. Three further bits can be used to address up to 7 elements in arguments, local variables, literals and intrinsics (primitive routines); another bit helps to differentiate between these areas. More can then be achieved with a second byte, which contributes a further 7 bits for addressing (then, some 2¹° elements). The two bytes next to each other differ in their sign (the first positive, the second negative), the bytes are concatenated and the negative sign bit goes MSB in the resultant codon. This describes the coding; will omit other details here. It may be interesting to note that ***zero if's (z'if's) are needed in source code***, to follow the path to a bytecode routine.<br>
The leading bits of the codon are then used for a ***calculated jump*** via a table to the corresponding bytecode routine. And there it can be observed that each routine is a ***2×jump sandwich per ~1×machine code operation, with some loop sauce on top***: the decoder jumps to the machine code routine, which does ***1 specific operation***, then the routine jumps to the continuation of the interpreter loop. It can therefore be expected that the effort of bytecode routines is about φ¹ per bytecode.

### 13. the first use case for (someObject = otherObject) occurs to the compiler,

where equivalent literals are the same == object. And in the SmallObjects system, there cannot be a literal that is = another but not == another (saves space and serves as a query feature when users of some specific literal are browsed). Searching for equivalent literals in memory is a matter that can easily be done AoT in O(n).<br>
For literals in methods, the compiler records neither a `class oop` nor a `method oop`; both would force the runtime to access the respective instance `in memory` just to get information about `arity` or bits of some variaEnum. SmallObjects has ***learned this from Big Iron B5000*** where a descriptor informs about its respective instance ***before*** one of the parts in memory is accessed (details about the present associated object table are given elsewhere). This ***also*** means that no measures are needed to avoid ***cyclicality*** via oop accesses. This brings us back to the `equivalence decision` that the compiler needs; we write its code:<br>
```
!Object = anOther!
 (self privateºinstSize) = (anOther privateºinstSize) or: [^false].
 (self privateºbasicSize) = (anOther privateºbasicSize) or: [^false].
 ^false "on ¹st discrepancy, of" eachⁱin: self equivⁱin: anOther! !

!Object ~= anOther!
 (self privateºinstSize) = (anOther privateºinstSize) or: [^true].
 (self privateºbasicSize) = (anOther privateºbasicSize) or: [^true].
 ^true "on ¹st discrepancy, of" eachⁱin: self equivⁱin: anOther! !
 
!Object eachⁱin: someObject equivⁱin: otherObject!
 | approval := self ifTrue: [true] ifFalse: [false] | "MustBeBoolean"
  1 to: someObject privateºinstSize do: [:ⁱth|
   (someObject privateºinstVarAt: ⁱth) = (otherObject privateºinstVarAt: ⁱth)
    or: [^approval].
  ]. 1 to: otherObject privateºbasicSize do: [:ⁱth|
   (someObject privateºbasicAt: ⁱth) = (otherObject privateºbasicAt: ⁱth)
    or: [^approval].
  ]. ^approval not! !
```
It is no coincidence that the same `approval` routine works for true and false.

### 12. adaptive frame boundary shifting for enabling more tailcalls

The improvement through tailcalls has two points, for they omit two things: execution of the current epilog and execution of the next prolog, only to "find" that the next context `frame` is practically identically to the current one (linkage at same position and to same locations on the stack).<br>
A comparison between `*Stream*·#nextPut:` and the subsequently executed `*Array*·#at:put:` shows why a tailcall is desired here but prevented: the next free `position` determined by `#nextPut:` is missing.<br>
To enable this, `position` is defined as the first `local` variable and the frame boundary (swapping fields) is trivially moved so that `position` now belongs to the arguments.<br>
If this also swaps the arguments, there is nothing to prevent a tailcall from `#nextPut:` to `#at:put:`.<br>
This practically saves more time (compared to pushing arguments anew + epilog + prolog handling) than the trivial shifting of the frame boundary can cost -- and this for by far the most frequent usecase of Streams.

### 11. how many machine instruction bytes contributed to performance

The more `code pathes` are ***made*** available, the more ***shortcuts*** can be realized. This I want to investigate with the `SmallObjects` features, mainly for, but not restricted to, `performing profiling without a simulator`. Due to the nature of `primitive succeded` I separate pathes taken, into parts belonging to `primitive failed` and all the rest. After `primitive routines` have done their thing (just before they return the result), the size (in bytes) of their `machine instruction` s is accumulated for `investigating change of instruction path length` in `relation to timing` of ***benchmark*** tests.<br>
Thus in `SmallObjects`, benchmark tests can report difference in [performance](https://cseweb.ucsd.edu/classes/sp14/cse141-a/Slides/02_performance_annotated-0417.pdf) if ***not*** the ***code in both parts*** (succeeded | failed) is changed for the same run. And if neither the `benchmark subject` (bytecodes) nor the primitives are changed, then the report reflects changes to `method lookup` (inheritance check) and/or other parts of Interpreter.<br>
On modern microprocessor chips, the `CPU` runs parallel to a companion `FPU`; and as with stacks, the `FPU` is `nett at tos` after it was used for arithmetic. Thus in `SmallObjects`, the accumulation of `machine instruction`s size in bytes, is running on the `actual tos` of the `FPU` (notably, in parallel to the rest of the world, measured, imperceptible).

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
***Now***! ***spilling no more***, and the `Interpreter` _function prolog_ occasionally refrains from handling _call-preserved_ registers :-D<br>
(part of) mission ***accomplished***: the stack is left untouched by C·lang's inventions, thus the _bytecode routines_ can asm "push" & "store" & "pop" (etc) ***from/to their frame*** in the `native` machine `stack` , without disturbing the `Interpreter` prolog/epilog (by `calling convention` agreed upon) of C·lang compiler.<br>
P.S. `primitive1Add_` was used as test subject, it performed on _smi_ 3 with _smi_ 4, using [builtin arithmetic with overflow checking](https://gcc.gnu.org/onlinedocs/gcc/Integer-Overflow-Builtins.html), validating by experiment the (`stdcall` with oop receiver) calling convention.<br>
Of note: in the `primitive` routines there is ***no restriction*** for C·lang on the use of the stack.

### 9. recursive multiprocessing in parallel performing cores

In `SmallObjects` I want that each individual `ObjectMemory` instance _tlsMemory_ is firmly associated k:k with one of the _core_ s in my _CPU_ chip. Having worked with _multiprocessor machines_ since the earliest steps of my work-life, I can't wait that my notebook performs an `oop`-rogram on all its cylinders (_core_ s) and that the (_programmable_ ) task at hand is divided into smaller _and parallel_ pieces of work, to then combine the interim results for the big whole.<br>
Here is my first draft, derived from the classic `benchFib` performance benchmark:
```
!Integer benchFib "handy message-heavy benchmark"!
 | subtask := [(self -1) benchFib] blockCopy. interim |
 self < 2 ifTrue: [^1].
 " at >= 43 (on -m32), LargeInteger arithmetic takes over from SmallInteger "
 self < 43 ifFalse: [interim := subtask promiseUnless: tlsMemory idlerCount < 1]
  ifTrue: [interim := subtask "do all myself"].
 ^(self -2) benchFib +(interim "future" value +1)! !
```
### 8. inviting some C·compiler to jump for nirvana
```
 static oop jumpTargetRoutines[] = {(oop)&&¹st,(oop)&&²nd,…,(oop)&&ⁿth};
 asm goto("jmp *%0"::"r"(jumpTargetRoutines[ⁱth])::¹st,²nd,…,ⁿth /*labels*/);
```
This statement I want for jumping according to `computed label jumpTargetRoutines[ⁱth]`; there are _not_ many [asm goto](https://gcc.gnu.org/onlinedocs/gcc/Extended-Asm.html#:~:text=asm%20goto%20allows) examples which demonstrate the previous. At first, the statement does not encode some `jump` to any of the label references %l1, %l2, etc -- yet the label names are passed proforma so that C·compiler knows expected branch targets. At second, the computed target is passed to the statement -- which indeed emits code for the effective jump. There are many things to consider: how the routines at the branch targets have to be set up, how to act (re: control flow) and refer to variables, eventually go further in their enclosing `Interpreter loop`; these tests are work in progress at the time of this writing.

### 7. the adaptive specialist and its carryall

In the previous section (6. below) need arised for `private field`s which belong to _subcontractors_. In the `SmallObjects` system this has rather little to do with _inheritance_, the specialists are trained in fields whose intersection (material type & form, tools, etc) is mostly empty (that makes any pair of _subcontractors_ specialists). And the individual specialists posess sort of `carryall` which they do not share with other _subcontractor_ s.<br>
The solution in `SmallObjects` is to maintain `private field`s (the `carryall`) ***before the first*** data item of the `variaPart` (of the _patron_ instance, e.g. a `ByteString` ;-). And the `class header` bits have  (plenty of room ;-) for the count 0…15 of `private field`s.<br>
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
 | tideLevel := tlsMemory garbageCollectMost; tideLevel. requestResponse := self … |
 requestResponse := requestResponse taskDesired perform: #realization with: requestResponse.
 tlsMemory garbageCollectMost: "the previous" tideLevel.
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
This is sufficient for the `compiler` to emit proper offset in machine instructions for the memory reference `anOop->inMemory`. Also, the `class header` bits are not seen (but can be addressed using `&anOop[0]` as base for offset). Thus, the first few `fixed field`s of `tlsMemory` (see 1. memory layout, below) are declared:
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
The `ObjectMemory` instance `tlsMemory` has several _constant_ fields, for: the `nil` oop, also the `true` and `false` oop's. Beeing an instance, `ObjectMemory` has fields and `varia data`, but above all it has `absolute` location (warranted by the `OS`):
```
 _Thread_local oop Thread$isolated$Heap;
```
This `oop` is assigned (trivial from `sbrk`, elaborate from `mmap`) once the `main` (or other launcher) gets control. The `_Thread_local` storage class has became rather bugfree on many platforms. In `SmallObjects` it is used this way:
```
 ObjectMemory* tlsMemory = (ObjectMemory*)Thread$isolated$Heap;
```
All other objects in memory can only be stored in `tlsMemory` and the allocated space it describes. The  `varia data` items of `tlsMemory` instance are indexed by `classId` bits from the `class header` of instances (or defaults for `depictor`'s). After the last `varia data` item of `tlsMemory` begins the zone of resident objects (up to the `tideLevel` mark), then the zone of jetsam (up to the `shoreline` mark). The fields `tideLevel` and `shoreline` belong to the `fixed fields` of the ObjectMemory instance (many other not mentioned here). Surprise: all this is accessible from ordinary code and therefore from the SmallObjects `Interpreter`.

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
The `SmallInteger depictor` is ready for _smi_ arithmetic (etc) and all else it can do is to cause overflow; the `Character depictor` has plenty of bits room for encoding `UTF8`. For the `*aPointer oop` the compiler can be told to use an `odd` offset instead of an `even` , for `addressing fields or values`, all CPUs process the offset bits already in the machine instructions … thus in `SmallObjects` , nowhere is systemwise untag.
