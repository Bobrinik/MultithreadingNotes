# Memory Models

- It is essentially about what variables thread can access or cannot
- Write buffers are invisible to programmer; however, they can produce unintended results

### Vocabulary
- Strict consistency
- Sequence consistency
- Coherence

## Concistency
1. Strict concistency: every operation of every processor is executed in order, and operations are not interleaved
2. Sequence concistency: Multithreaded program is S.C. if any execution is the same as if the operations of all processes were executed in some sequential order, and operations of each processor respect program order.
 	- All actions within a program appear to happen in a fixed global order

## Weak memory models

- Sequence concistency: tells us that all statements are ordered in one timeline
- Coherence: we have a separate timeline on each variable. It means that every thread agrees on the order of operations on one variable. 
	- Every thread has the same table that tells in which order variable is being accessed


| P0 |P1   | 
|---|---|
|x=1 |  a = y |   
|x=2  |  b =x|   
|y=3  |  c = x|   

- Our reads and writes can happen in different order
	- b -> c  -> (b,c)= 0,0 //P1 runs before P0
	- x -> b -> c -> (b,c) = 1,1 //P0 executes first assignment then we have P1 running
	- x -> a -> b -> x -> c (b,c) = 1,2 
- Note that we can interleave two threads but order within thread (intrathread order) is preseved

### Consider the following:
|P1   | 
|---|
|  a = y (is 3) |   
|  b =0 |   
|  c = 0|   

* Notice that there is no interleaving between P0 and P1 that can produce the result above. 
* The example above is coherent because intrathread order is preserved, but it is not sequentially consistent because it is imposibble to make sequential program out of P0 and P1 to produce the same result.


## Processor consistency
- Processor consistency: we have a separate timeline for every processor. It means that everyone agrees on the actions of each individual processor.


## Write Buffering
- It is a technique where written data is actually stored in a faster write buffer proper to the processor before being flushed to memory (only when necessary and in batch). The intent is to save time. 
- The flushing is entirely hidden from programmer and in multi processor context leads to weird things.

```
P0		P1 
x = 1	y = 1 
a = y	b = x

P0 WB	P1 WB	
x = 1	y = 1	
a = 0	b = 0	
```

- Before reading a variable process checks if the variable is stored in the write buffer. If it is not in the write buffer the process reads it from main memory.
- However, it leads to the weird results because buffers can be flushed at memory at different times and processes can do context switch in the middle of their execution.

1. P0 puts x = 1 in its write buffer
2. P0 reads y from main memory since it does not have a copy of it in a buffer
3. P0 stops and P1 starts working (the same thing happens)
4. Both P1 and P2 flush their buffers and we get

```
Memory:
x = 1
y = 1
a = 0
b = 0 
```
## Questions
- What is a processor consistency?
- What is a coherence?
	- Is it intrathread order?
- What is strict consistency?
- What is the problem with X86-TSO?
	- I know that it is somehow related to temporary flushes been done at different times
- What is the problem with n6 and n5?
