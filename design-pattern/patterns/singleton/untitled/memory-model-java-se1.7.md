# Memory Model java se1.7

### [https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html\#jls-17.4](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4)

### 17.4. Memory Model

A _memory model_ describes, given a program and an execution trace of that program, whether the execution trace is a legal execution of the program. The Java programming language memory model works by examining each read in an execution trace and checking that the write observed by that read is valid according to certain rules.

The memory model describes possible behaviors of a program. An implementation is free to produce any code it likes, as long as all resulting executions of a program produce a result that can be predicted by the memory model.

This provides a great deal of freedom for the implementor to perform a myriad of code transformations, including the reordering of actions and removal of unnecessary synchronization.

**Example 17.4-1. Incorrectly Synchronized Programs May Exhibit Surprising Behavior**

The semantics of the Java programming language allow compilers and microprocessors to perform optimizations that can interact with incorrectly synchronized code in ways that can produce behaviors that seem paradoxical. Here are some examples of how incorrectly synchronized programs may exhibit surprising behaviors.

Consider, for example, the example program traces shown in [Table 17.1](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4-table-1). This program uses local variables `r1` and `r2` and shared variables `A` and `B`. Initially, `A == B == 0`.

**Table 17.1. Surprising results caused by statement reordering - original code**

| Thread 1 | Thread 2 |
| :--- | :--- |
| 1: `r2 = A;` | 3: `r1 = B;` |
| 2: `B = 1;` | 4: `A = 2;` |

It may appear that the result `r2 == 2` and `r1 == 1` is impossible. Intuitively, either instruction 1 or instruction 3 should come first in an execution. If instruction 1 comes first, it should not be able to see the write at instruction 4. If instruction 3 comes first, it should not be able to see the write at instruction 2.

If some execution exhibited this behavior, then we would know that instruction 4 came before instruction 1, which came before instruction 2, which came before instruction 3, which came before instruction 4. This is, on the face of it, absurd.

However, compilers are allowed to reorder the instructions in either thread, when this does not affect the execution of that thread in isolation. If instruction 1 is reordered with instruction 2, as shown in the trace in [Table 17.2](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4-table-2), then it is easy to see how the result `r2 == 2` and `r1 == 1` might occur.

**Table 17.2. Surprising results caused by statement reordering - valid compiler transformation**

| Thread 1 | Thread 2 |
| :--- | :--- |
| `B = 1;` | `r1 = B;` |
| `r2 = A;` | `A = 2;` |

To some programmers, this behavior may seem "broken". However, it should be noted that this code is improperly synchronized:

* there is a write in one thread,
* a read of the same variable by another thread,
* and the write and read are not ordered by synchronization.

This situation is an example of a _data race_ \([§17.4.5](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4.5)\). When code contains a data race, counterintuitive results are often possible.

Several mechanisms can produce the reordering in [Table 17.2](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4-table-2). A Just-In-Time compiler in a Java Virtual Machine implementation may rearrange code, or the processor. In addition, the memory hierarchy of the architecture on which a Java Virtual Machine implementation is run may make it appear as if code is being reordered. In this chapter, we shall refer to anything that can reorder code as a _compiler_.

Another example of surprising results can be seen in [Table 17.3](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4-table-3). Initially, `p == q` and `p.x == 0`. This program is also incorrectly synchronized; it writes to shared memory without enforcing any ordering between those writes.

**Table 17.3. Surprising results caused by forward substitution**

| Thread 1 | Thread 2 |
| :--- | :--- |
| `r1 = p;` | `r6 = p;` |
| `r2 = r1.x;` | `r6.x = 3;` |
| `r3 = q;` |  |
| `r4 = r3.x;` |  |
| `r5 = r1.x;` |  |

One common compiler optimization involves having the value read for `r2` reused for `r5`: they are both reads of `r1.x` with no intervening write. This situation is shown in [Table 17.4](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4-table-4).

**Table 17.4. Surprising results caused by forward substitution**

| Thread 1 | Thread 2 |
| :--- | :--- |
| `r1 = p;` | `r6 = p;` |
| `r2 = r1.x;` | `r6.x = 3;` |
| `r3 = q;` |  |
| `r4 = r3.x;` |  |
| `r5 = r2;` |  |

Now consider the case where the assignment to `r6.x` in Thread 2 happens between the first read of `r1.x` and the read of `r3.x` in Thread 1. If the compiler decides to reuse the value of `r2` for the `r5`, then `r2` and `r5` will have the value `0`, and `r4` will have the value `3`. From the perspective of the programmer, the value stored at `p.x` has changed from `0` to `3` and then changed back.  


The memory model determines what values can be read at every point in the program. The actions of each thread in isolation must behave as governed by the semantics of that thread, with the exception that the values seen by each read are determined by the memory model. When we refer to this, we say that the program obeys _intra-thread semantics_. Intra-thread semantics are the semantics for single-threaded programs, and allow the complete prediction of the behavior of a thread based on the values seen by read actions within the thread. To determine if the actions of thread _t_ in an execution are legal, we simply evaluate the implementation of thread _t_ as it would be performed in a single-threaded context, as defined in the rest of this specification.

Each time the evaluation of thread _t_ generates an inter-thread action, it must match the inter-thread action _a_ of _t_ that comes next in program order. If _a_ is a read, then further evaluation of _t_ uses the value seen by _a_ as determined by the memory model.

This section provides the specification of the Java programming language memory model except for issues dealing with `final` fields, which are described in [§17.5](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.5).

The memory model specified herein is not fundamentally based in the object-oriented nature of the Java programming language. For conciseness and simplicity in our examples, we often exhibit code fragments without class or method definitions, or explicit dereferencing. Most examples consist of two or more threads containing statements with access to local variables, shared global variables, or instance fields of an object. We typically use variables names such as `r1` or `r2` to indicate variables local to a method or thread. Such variables are not accessible by other threads.

#### 17.4.1. Shared Variables

Memory that can be shared between threads is called _shared memory_ or _heap memory_.

All instance fields, `static` fields, and array elements are stored in heap memory. In this chapter, we use the term _variable_ to refer to both fields and array elements.

Local variables \([§14.4](https://docs.oracle.com/javase/specs/jls/se7/html/jls-14.html#jls-14.4)\), formal method parameters \([§8.4.1](https://docs.oracle.com/javase/specs/jls/se7/html/jls-8.html#jls-8.4.1)\), and exception handler parameters \([§14.20](https://docs.oracle.com/javase/specs/jls/se7/html/jls-14.html#jls-14.20)\) are never shared between threads and are unaffected by the memory model.

Two accesses to \(reads of or writes to\) the same variable are said to be _conflicting_ if at least one of the accesses is a write.

#### 17.4.2. Actions

An _inter-thread action_ is an action performed by one thread that can be detected or directly influenced by another thread. There are several kinds of inter-thread action that a program may perform:

* _Read_ \(normal, or non-volatile\). Reading a variable.
* _Write_ \(normal, or non-volatile\). Writing a variable.
* _Synchronization actions_, which are:
  * _Volatile read_. A volatile read of a variable.
  * _Volatile write_. A volatile write of a variable.
  * _Lock_. Locking a monitor
  * _Unlock_. Unlocking a monitor.
  * The \(synthetic\) first and last action of a thread.
  * Actions that start a thread or detect that a thread has terminated \([§17.4.4](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4.4)\).
* _External Actions_. An external action is an action that may be observable outside of an execution, and has a result based on an environment external to the execution.
* _Thread divergence actions_ \([§17.4.9](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4.9)\). A thread divergence action is only performed by a thread that is in an infinite loop in which no memory, synchronization, or external actions are performed. If a thread performs a thread divergence action, it will be followed by an infinite number of thread divergence actions.

  Thread divergence actions are introduced to model how a thread may cause all other threads to stall and fail to make progress.

This specification is only concerned with inter-thread actions. We do not need to concern ourselves with intra-thread actions \(e.g., adding two local variables and storing the result in a third local variable\). As previously mentioned, all threads need to obey the correct intra-thread semantics for Java programs. We will usually refere to inter-thread actions more succinctly as simply _actions_.

An action _a_ is described by a tuple &lt; _t_, _k_, _v_, _u_ &gt;, comprising:

* _t_ - the thread performing the action
* _k_ - the kind of action
* _v_ - the variable or monitor involved in the action.

  For lock actions, _v_ is the monitor being locked; for unlock actions, _v_ is the monitor being unlocked.

  If the action is a \(volatile or non-volatile\) read, _v_ is the variable being read.

  If the action is a \(volatile or non-volatile\) write, _v_ is the variable being written.

* _u_ - an arbitrary unique identifier for the action

An external action tuple contains an additional component, which contains the results of the external action as perceived by the thread performing the action. This may be information as to the success or failure of the action, and any values read by the action.

Parameters to the external action \(e.g., which bytes are written to which socket\) are not part of the external action tuple. These parameters are set up by other actions within the thread and can be determined by examining the intra-thread semantics. They are not explicitly discussed in the memory model.

In non-terminating executions, not all external actions are observable. Non-terminating executions and observable actions are discussed in [§17.4.9](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4.9).

#### 17.4.3. Programs and Program Order

Among all the inter-thread actions performed by each thread _t_, the _program order_ of _t_ is a total order that reflects the order in which these actions would be performed according to the intra-thread semantics of _t_.

A set of actions is _sequentially consistent_ if all actions occur in a total order \(the execution order\) that is consistent with program order, and furthermore, each read _r_ of a variable _v_ sees the value written by the write _w_ to _v_ such that:

* _w_ comes before _r_ in the execution order, and
* there is no other write _w_' such that _w_ comes before _w_' and _w_' comes before _r_ in the execution order.

Sequential consistency is a very strong guarantee that is made about visibility and ordering in an execution of a program. Within a sequentially consistent execution, there is a total order over all individual actions \(such as reads and writes\) which is consistent with the order of the program, and each individual action is atomic and is immediately visible to every thread.

If a program has no data races, then all executions of the program will appear to be sequentially consistent.

Sequential consistency and/or freedom from data races still allows errors arising from groups of operations that need to be perceived atomically and are not.

If we were to use sequential consistency as our memory model, many of the compiler and processor optimizations that we have discussed would be illegal. For example, in the trace in [Table 17.3](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4-table-3), as soon as the write of `3` to `p.x` occurred, subsequent reads of that location would be required to see that value.

#### 17.4.4. Synchronization Order

Every execution has a _synchronization order_. A synchronization order is a total order over all of the synchronization actions of an execution. For each thread _t_, the synchronization order of the synchronization actions \([§17.4.2](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4.2)\) in _t_ is consistent with the program order \([§17.4.3](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4.3)\) of _t_.

Synchronization actions induce the _synchronized-with_ relation on actions, defined as follows:

* An unlock action on monitor _m_ _synchronizes-with_ all subsequent lock actions on _m_ \(where "subsequent" is defined according to the synchronization order\).
* A write to a volatile variable _v_ \([§8.3.1.4](https://docs.oracle.com/javase/specs/jls/se7/html/jls-8.html#jls-8.3.1.4)\) _synchronizes-with_ all subsequent reads of _v_ by any thread \(where "subsequent" is defined according to the synchronization order\).
* An action that starts a thread _synchronizes-with_ the first action in the thread it starts.
* The write of the default value \(zero, `false`, or `null`\) to each variable _synchronizes-with_ the first action in every thread.

  Although it may seem a little strange to write a default value to a variable before the object containing the variable is allocated, conceptually every object is created at the start of the program with its default initialized values.

* The final action in a thread `T1` _synchronizes-with_ any action in another thread `T2` that detects that `T1` has terminated.

  `T2` may accomplish this by calling `T1.isAlive()` or `T1.join()`.

* If thread `T1` interrupts thread `T2`, the interrupt by `T1` _synchronizes-with_ any point where any other thread \(including `T2`\) determines that `T2` has been interrupted \(by having an `InterruptedException` thrown or by invoking `Thread.interrupted` or `Thread.isInterrupted`\).

The source of a _synchronizes-with_ edge is called a _release_, and the destination is called an _acquire_.

#### 17.4.5. Happens-before Order

Two actions can be ordered by a _happens-before_ relationship. If one action _happens-before_ another, then the first is visible to and ordered before the second.

If we have two actions _x_ and _y_, we write _hb\(x, y\)_ to indicate that _x happens-before y_.

* If _x_ and _y_ are actions of the same thread and _x_ comes before _y_ in program order, then _hb\(x, y\)_.
* There is a _happens-before_ edge from the end of a constructor of an object to the start of a finalizer \([§12.6](https://docs.oracle.com/javase/specs/jls/se7/html/jls-12.html#jls-12.6)\) for that object.
* If an action _x_ _synchronizes-with_ a following action _y_, then we also have _hb\(x, y\)_.
* If _hb\(x, y\)_ and _hb\(y, z\)_, then _hb\(x, z\)_.

The `wait` methods of class `Object` \([§17.2.1](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.2.1)\) have lock and unlock actions associated with them; their _happens-before_ relationships are defined by these associated actions.

It should be noted that the presence of a _happens-before_ relationship between two actions does not necessarily imply that they have to take place in that order in an implementation. If the reordering produces results consistent with a legal execution, it is not illegal.

For example, the write of a default value to every field of an object constructed by a thread need not happen before the beginning of that thread, as long as no read ever observes that fact.

More specifically, if two actions share a _happens-before_ relationship, they do not necessarily have to appear to have happened in that order to any code with which they do not share a _happens-before_ relationship. Writes in one thread that are in a data race with reads in another thread may, for example, appear to occur out of order to those reads.

The _happens-before_ relation defines when data races take place.

A set of synchronization edges, _S_, is _sufficient_ if it is the minimal set such that the transitive closure of _S_ with the program order determines all of the _happens-before_ edges in the execution. This set is unique.

It follows from the above definitions that:

* An unlock on a monitor _happens-before_ every subsequent lock on that monitor.
* A write to a `volatile` field \([§8.3.1.4](https://docs.oracle.com/javase/specs/jls/se7/html/jls-8.html#jls-8.3.1.4)\) _happens-before_ every subsequent read of that field.
* A call to `start()` on a thread _happens-before_ any actions in the started thread.
* All actions in a thread _happen-before_ any other thread successfully returns from a `join()` on that thread.
* The default initialization of any object _happens-before_ any other actions \(other than default-writes\) of a program.

When a program contains two conflicting accesses \([§17.4.1](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4.1)\) that are not ordered by a happens-before relationship, it is said to contain a _data race_.

The semantics of operations other than inter-thread actions, such as reads of array lengths \([§10.7](https://docs.oracle.com/javase/specs/jls/se7/html/jls-10.html#jls-10.7)\), executions of checked casts \([§5.5](https://docs.oracle.com/javase/specs/jls/se7/html/jls-5.html#jls-5.5), [§15.16](https://docs.oracle.com/javase/specs/jls/se7/html/jls-15.html#jls-15.16)\), and invocations of virtual methods \([§15.12](https://docs.oracle.com/javase/specs/jls/se7/html/jls-15.html#jls-15.12)\), are not directly affected by data races.

Therefore, a data race cannot cause incorrect behavior such as returning the wrong length for an array.

A program is _correctly synchronized_ if and only if all sequentially consistent executions are free of data races.

If a program is correctly synchronized, then all executions of the program will appear to be sequentially consistent \([§17.4.3](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4.3)\).

This is an extremely strong guarantee for programmers. Programmers do not need to reason about reorderings to determine that their code contains data races. Therefore they do not need to reason about reorderings when determining whether their code is correctly synchronized. Once the determination that the code is correctly synchronized is made, the programmer does not need to worry that reorderings will affect his or her code.

A program must be correctly synchronized to avoid the kinds of counterintuitive behaviors that can be observed when code is reordered. The use of correct synchronization does not ensure that the overall behavior of a program is correct. However, its use does allow a programmer to reason about the possible behaviors of a program in a simple way; the behavior of a correctly synchronized program is much less dependent on possible reorderings. Without correct synchronization, very strange, confusing and counterintuitive behaviors are possible.

We say that a read _r_ of a variable _v_ is allowed to observe a write _w_ to _v_ if, in the _happens-before_ partial order of the execution trace:

* _r_ is not ordered before _w_ \(i.e., it is not the case that _hb\(r, w\)_\), and
* there is no intervening write _w_' to _v_ \(i.e. no write _w_' to _v_ such that _hb\(w, w'\)_ and _hb\(w', r\)_\).

Informally, a read _r_ is allowed to see the result of a write _w_ if there is no _happens-before_ ordering to prevent that read.

A set of actions _A_ is _happens-before consistent_ if for all reads _r_ in _A_, where _W\(r\)_ is the write action seen by _r_, it is not the case that either _hb\(r, W\(r\)\)_ or that there exists a write _w_ in _A_ such that _w.v_ = _r.v_ and _hb\(W\(r\), w\)_ and _hb\(w, r\)_.

In a _happens-before consistent_ set of actions, each read sees a write that it is allowed to see by the _happens-before_ ordering.

**Example 17.4.5-1. Happens-before Consistency**

For the trace in [Table 17.5](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4.5-table-1), initially `A == B == 0`. The trace can observe `r2 == 0` and `r1 == 0` and still be _happens-before consistent_, since there are execution orders that allow each read to see the appropriate write.

**Table 17.5. Behavior allowed by happens-before consistency, but not sequential consistency.**

| Thread 1 | Thread 2 |
| :--- | :--- |
| `B = 1;` | `A = 2;` |
| `r2 = A;` | `r1 = B;` |

Since there is no synchronization, each read can see either the write of the initial value or the write by the other thread. An execution order that displays this behavior is:

```text
1: B = 1;
3: A = 2;
2: r2 = A;  // sees initial write of 0
4: r1 = B;  // sees initial write of 0
```

Another execution order that is happens-before consistent is:

```text
1: r2 = A;  // sees write of A = 2
3: r1 = B;  // sees write of B = 1
2: B = 1;
4: A = 2;
```

In this execution, the reads see writes that occur later in the execution order. This may seem counterintuitive, but is allowed by _happens-before_ consistency. Allowing reads to see later writes can sometimes produce unacceptable behaviors.  


#### 17.4.6. Executions

An execution _E_ is described by a tuple &lt; _P, A, po, so, W, V, sw, hb_ &gt;, comprising:

* _P_ - a program
* _A_ - a set of actions
* _po_ - program order, which for each thread _t_, is a total order over all actions performed by _t_ in _A_
* _so_ - synchronization order, which is a total order over all synchronization actions in _A_
* _W_ - a write-seen function, which for each read _r_ in _A_, gives _W\(r\)_, the write action seen by _r_ in _E_.
* _V_ - a value-written function, which for each write _w_ in _A_, gives _V\(w\)_, the value written by _w_ in _E_.
* _sw_ - synchronizes-with, a partial order over synchronization actions
* _hb_ - happens-before, a partial order over actions

Note that the synchronizes-with and happens-before elements are uniquely determined by the other components of an execution and the rules for well-formed executions \([§17.4.7](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4.7)\).

An execution is _happens-before consistent_ if its set of actions is _happens-before consistent_ \([§17.4.5](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4.5)\).

#### 17.4.7. Well-Formed Executions

We only consider well-formed executions. An execution _E_ = &lt; _P, A, po, so, W, V, sw, hb_ &gt; is well formed if the following conditions are true:

1. Each read sees a write to the same variable in the execution.

   All reads and writes of volatile variables are volatile actions. For all reads _r_ in _A_, we have _W\(r\)_ in _A_ and _W\(r\).v_ = _r.v_. The variable _r.v_ is volatile if and only if _r_ is a volatile read, and the variable _w.v_ is volatile if and only if _w_ is a volatile write.

2. The happens-before order is a partial order.

   The happens-before order is given by the transitive closure of synchronizes-with edges and program order. It must be a valid partial order: reflexive, transitive and antisymmetric.

3. The execution obeys intra-thread consistency.

   For each thread _t_, the actions performed by _t_ in _A_ are the same as would be generated by that thread in program-order in isolation, with each write _w_ writing the value _V\(w\)_, given that each read _r_ sees the value _V\(W\(r\)\)_. Values seen by each read are determined by the memory model. The program order given must reflect the program order in which the actions would be performed according to the intra-thread semantics of _P_.

4. The execution is _happens-before consistent_ \([§17.4.6](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4.6)\).
5. The execution obeys synchronization-order consistency.

   For all volatile reads _r_ in _A_, it is not the case that either _so\(r, W\(r\)\)_ or that there exists a write _w_ in _A_ such that _w.v_ = _r.v_ and _so\(W\(r\), w\)_ and _so\(w, r\)_.

#### 17.4.8. Executions and Causality Requirements

We use _f_\|_d_ to denote the function given by restricting the domain of _f_ to _d_. For all _x_ in _d_, _f_\|_d_\(_x_\) = _f_\(_x_\), and for all _x_ not in _d_, _f_\|_d_\(_x_\) is undefined.

We use _p_\|_d_ to represent the restriction of the partial order _p_ to the elements in _d_. For all _x_,_y_ in _d_, _p_\(_x_,_y_\) if and only if _p_\|_d_\(_x_,_y_\). If either _x_ or _y_ are not in _d_, then it is not the case that _p_\|_d_\(_x_,_y_\).

A well-formed execution _E_ = &lt; _P, A, po, so, W, V, sw, hb_ &gt; is validated by _committing_ actions from _A_. If all of the actions in _A_ can be committed, then the execution satisfies the causality requirements of the Java programming language memory model.

Starting with the empty set as _C0_, we perform a sequence of steps where we take actions from the set of actions _A_ and add them to a set of committed actions _Ci_ to get a new set of committed actions _Ci+1_. To demonstrate that this is reasonable, for each _Ci_ we need to demonstrate an execution _E_ containing _Ci_ that meets certain conditions.

Formally, an execution _E satisfies the causality requirements of the Java programming language memory model_ if and only if there exist:

* Sets of actions _C0_, _C1_, ... such that:

  * _C0_ is the empty set
  * _Ci_ is a proper subset of _Ci+1_
  * _A_ = ∪ \(_C0_, _C1_, ...\)

  If _A_ is finite, then the sequence _C0_, _C1_, ... will be finite, ending in a set _Cn_ = _A_.

  If _A_ is infinite, then the sequence _C0_, _C1_, ... may be infinite, and it must be the case that the union of all elements of this infinite sequence is equal to _A_.

* Well-formed executions _E1_, ..., where _Ei_ = &lt; _P, Ai, poi, soi, Wi, Vi, swi, hbi_ &gt;.

Given these sets of actions _C0_, ... and executions _E1_, ... , every action in _Ci_ must be one of the actions in _Ei_. All actions in _Ci_ must share the same relative happens-before order and synchronization order in both _Ei_ and _E_. Formally:

1. _Ci_ is a subset of _Ai_
2. _hbi_\|_Ci_ = _hb_\|_Ci_
3. _soi_\|_Ci_ = _so_\|_Ci_

The values written by the writes in _Ci_ must be the same in both _Ei_ and _E_. Only the reads in _Ci-1_ need to see the same writes in _Ei_ as in _E_. Formally:

1. _Vi_\|_Ci_ = _V_\|_Ci_
2. _Wi_\|_Ci-1_ = _W_\|_Ci-1_

All reads in _Ei_ that are not in _Ci-1_ must see writes that happen-before them. Each read _r_ in _Ci_ - _Ci-1_ must see writes in _Ci-1_ in both _Ei_ and _E_, but may see a different write in _Ei_ from the one it sees in _E_. Formally:

1. For any read _r_ in _Ai_ - _Ci-1_, we have _hbi\(Wi\(r\), r\)_
2. For any read _r_ in \(_Ci_ - _Ci-1_\), we have _Wi\(r\)_ in _Ci-1_ and _W\(r\)_ in _Ci-1_

Given a set of sufficient synchronizes-with edges for _Ei_, if there is a release-acquire pair that happens-before \([§17.4.5](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4.5)\) an action you are committing, then that pair must be present in all _Ej_, where _j_ ≥ _i_. Formally:

1. Let _sswi_ be the _swi_ edges that are also in the transitive reduction of _hbi_ but not in _po_. We call _sswi_ the _sufficient synchronizes-with edges for Ei_. If _sswi\(x, y\)_ and _hbi\(y, z\)_ and _z_ in _Ci_, then _swj\(x, y\)_ for all _j_ ≥ _i_.

   If an action _y_ is committed, all external actions that happen-before _y_ are also committed.

2. If _y_ is in _Ci_, _x_ is an external action and _hbi\(x, y\)_, then _x_ in _Ci_.

**Example 17.4.8-1. Happens-before Consistency Is Not Sufficient**

Happens-before consistency is a necessary, but not sufficient, set of constraints. Merely enforcing happens-before consistency would allow for unacceptable behaviors - those that violate the requirements we have established for programs. For example, happens-before consistency allows values to appear "out of thin air". This can be seen by a detailed examination of the trace in [Table 17.6](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4.8-table-1).

**Table 17.6. Happens-before consistency is not sufficient**

| Thread 1 | Thread 2 |
| :--- | :--- |
| `r1 = x;` | `r2 = y;` |
| `if (r1 != 0) y = 1;` | `if (r2 != 0) x = 1;` |

The code shown in [Table 17.6](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4.8-table-1) is correctly synchronized. This may seem surprising, since it does not perform any synchronization actions. Remember, however, that a program is correctly synchronized if, when it is executed in a sequentially consistent manner, there are no data races. If this code is executed in a sequentially consistent way, each action will occur in program order, and neither of the writes will occur. Since no writes occur, there can be no data races: the program is correctly synchronized.

Since this program is correctly synchronized, the only behaviors we can allow are sequentially consistent behaviors. However, there is an execution of this program that is happens-before consistent, but not sequentially consistent:

```text
r1 = x;  // sees write of x = 1
y = 1;
r2 = y;  // sees write of y = 1
x = 1; 
```

This result is happens-before consistent: there is no happens-before relationship that prevents it from occurring. However, it is clearly not acceptable: there is no sequentially consistent execution that would result in this behavior. The fact that we allow a read to see a write that comes later in the execution order can sometimes thus result in unacceptable behaviors.

Although allowing reads to see writes that come later in the execution order is sometimes undesirable, it is also sometimes necessary. As we saw above, the trace in [Table 17.5](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4.5-table-1) requires some reads to see writes that occur later in the execution order. Since the reads come first in each thread, the very first action in the execution order must be a read. If that read cannot see a write that occurs later, then it cannot see any value other than the initial value for the variable it reads. This is clearly not reflective of all behaviors.

We refer to the issue of when reads can see future writes as _causality_, because of issues that arise in cases like the one found in [Table 17.6](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4.8-table-1). In that case, the reads cause the writes to occur, and the writes cause the reads to occur. There is no "first cause" for the actions. Our memory model therefore needs a consistent way of determining which reads can see writes early.

Examples such as the one found in [Table 17.6](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4.8-table-1) demonstrate that the specification must be careful when stating whether a read can see a write that occurs later in the execution \(bearing in mind that if a read sees a write that occurs later in the execution, it represents the fact that the write is actually performed early\).

The memory model takes as input a given execution, and a program, and determines whether that execution is a legal execution of the program. It does this by gradually building a set of "committed" actions that reflect which actions were executed by the program. Usually, the next action to be committed will reflect the next action that can be performed by a sequentially consistent execution. However, to reflect reads that need to see later writes, we allow some actions to be committed earlier than other actions that happen-before them.

Obviously, some actions may be committed early and some may not. If, for example, one of the writes in [Table 17.6](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4.8-table-1) were committed before the read of that variable, the read could see the write, and the "out-of-thin-air" result could occur. Informally, we allow an action to be committed early if we know that the action can occur without assuming some data race occurs. In [Table 17.6](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4.8-table-1), we cannot perform either write early, because the writes cannot occur unless the reads see the result of a data race.  


#### 17.4.9. Observable Behavior and Nonterminating Executions

For programs that always terminate in some bounded finite period of time, their behavior can be understood \(informally\) simply in terms of their allowable executions. For programs that can fail to terminate in a bounded amount of time, more subtle issues arise.

The observable behavior of a program is defined by the finite sets of external actions that the program may perform. A program that, for example, simply prints "Hello" forever is described by a set of behaviors that for any non-negative integer _i_, includes the behavior of printing "Hello" _i_ times.

Termination is not explicitly modeled as a behavior, but a program can easily be extended to generate an additional external action _executionTermination_ that occurs when all threads have terminated.

We also define a special _hang_ action. If behavior is described by a set of external actions including a _hang_ action, it indicates a behavior where after the external actions are observed, the program can run for an unbounded amount of time without performing any additional external actions or terminating. Programs can hang if all threads are blocked or if the program can perform an unbounded number of actions without performing any external actions.

A thread can be blocked in a variety of circumstances, such as when it is attempting to acquire a lock or perform an external action \(such as a read\) that depends on external data.

An execution may result in a thread being blocked indefinitely and the execution's not terminating. In such cases, the actions generated by the blocked thread must consist of all actions generated by that thread up to and including the action that caused the thread to be blocked, and no actions that would be generated by the thread after that action.

To reason about observable behaviors, we need to talk about sets of observable actions.

If _O_ is a set of observable actions for an execution _E_, then set _O_ must be a subset of _E_'s actions, _A_, and must contain only a finite number of actions, even if _A_ contains an infinite number of actions. Furthermore, if an action _y_ is in _O_, and either _hb\(x, y\)_ or _so\(x, y\)_, then _x_ is in _O_.

Note that a set of observable actions are not restricted to external actions. Rather, only external actions that are in a set of observable actions are deemed to be observable external actions.

A behavior _B_ is an allowable behavior of a program _P_ if and only if _B_ is a finite set of external actions and either:

* There exists an execution _E_ of _P_, and a set _O_ of observable actions for _E_, and _B_ is the set of external actions in _O_ \(If any threads in _E_ end in a blocked state and _O_ contains all actions in _E_, then _B_ may also contain a _hang_ action\); or
* There exists a set _O_ of actions such that _B_ consists of a _hang_ action plus all the external actions in _O_ and for all _k_ ≥ \| _O_ \|, there exists an execution _E_ of _P_ with actions _A_, and there exists a set of actions _O_' such that:
  * Both _O_ and _O_' are subsets of _A_ that fulfill the requirements for sets of observable actions.
  * _O_ ⊆ _O_' ⊆ _A_
  * \| _O_' \| ≥ _k_
  * _O_' - _O_ contains no external actions

Note that a behavior _B_ does not describe the order in which the external actions in _B_ are observed, but other \(internal\) constraints on how the external actions are generated and performed may impose such constraints.

