# Understanding intra-thread semantics

[https://stackoverflow.com/questions/25711048/understanding-intra-thread-semantics](https://stackoverflow.com/questions/25711048/understanding-intra-thread-semantics)

### Q:

Could you explain in simple words what "the program satisfies intra-thread semantic" means? Is it possible to provide simple examples of programs which satisfy and which don't satisfy such semantics?

### Ans:

The notion of _intra-thread semantics_ is discussed in the [JLS section 17.4](http://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4), which covers the Java Memory Model. The JMM is a set of requirements and constraints on the execution of Java programs by JVMs. Here's the relevant section of text from 17.4:

> The memory model determines what values can be read at every point in the program. The actions of each thread in isolation must behave as governed by the semantics of that thread, with the exception that the values seen by each read are determined by the memory model. When we refer to this, we say that the program obeys _intra-thread semantics_. Intra-thread semantics are the semantics for single-threaded programs, and allow the complete prediction of the behavior of a thread based on the values seen by read actions within the thread. To determine if the actions of thread t in an execution are legal, we simply evaluate the implementation of thread t as it would be performed in a single-threaded context, as defined in the rest of this specification.

This means that, as far as a single thread is concerned, the values visible in objects' fields are either the fields' initial values \(zero, false, or null\) or are values that this thread has previously written.

This is so obvious as to be elementary; why bother stating it?

Consider a single-threaded Java program with a few `int` fields:

```text
field1 = 1;                // 1
field2 = 2;                // 2
field3 = field1 + field2;  // 3
```

then clearly the value of `field3` must be 3. This is because the values visible in `field1` and `field2` at line 3 _must_ reflect the earlier values written at lines 1 and 2. It would be incorrect if the initial value of zero for `field1` or `field2` were used in the computation at line 3, since the assignment of those fields occurs earlier in the program than the computation.

What is less obvious are the constraints that are _not_ present. For example, there is no constraint here over the ordering of the _writes_ of `field1` and `field2`. The JVM could execute line 2 before line 1 and the result of the program would be the same. Or, it could delay the writes to `field1` and `field2` and keep these values in registers, and do register-based addition at line 3. The actual writes of all the fields could be delayed until much later. Or they could even be omitted entirely if the values are subsequently overwritten by this thread. Again, the outcome of the program would be the same.

And that's the point: JVMs are free to rearrange the execution of a program \(mainly so that it can run faster\), but only as long it doesn't change the results of that program if it were run single-threaded. These constraints are referred to as _intra-thread semantics_. Any rearrangements that don't violate intra-thread semantics are permitted.

\(Note that the paragraph quoted above talks about a "program that obeys intra-thread semantics" but what it really means is the _execution_ of the program obeys intra-thread semantics. Text in later sections, such as 17.4.7, is more precise, referring to whether an _execution_ of a program obeys intra-thread consistency or whether a _set of actions performed_ is in accord with intra-thread semantics.\)

