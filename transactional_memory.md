# Transactional memory
- Transaction memory model (another way to think about concurrent programming)
- Locking is hard
	- main goal of locking is to sequentialize our code 

```
- conceptually mutual exclusive
atomic ( ) {
	x = y
	y = 3
	q =  x + y
}

- if everything is atomic then we just need a global lock
- how should it interact with parts that are not atomic
```

- Implementing atomic
	- We have a global lock that needs to be acquired. This way we ensure mutual exclusion among all atomic statements
		- All atomic statements access it
- Global lock is pessimistic because we assume that there is some contention happening


```
atomic {
	a = y
	x = 2
	x = 3 
}
```

- We can have a single bottle neck

```
atomic1 {
	x++
}

atomic2 {
	y++
}

- Global lock does not permit to execute two atomic statements concurrently

atomic {
	A
}

let Arw allows access in A

- We have a lock on every varibale
- All a in Arw we lock a
- All a in Arw we unlock a
```
- Finding whta to lock can become compex if we are dealing with huge data structures
	- Arw allows access in A
	- We need to find Arw  (How to find what needs to be locked in atomic statements?)

```
- in simple 

atomic {
	x++
	y++
}
- it is clear that we need to have lock on x and y
- but it is more complex if we deal with linked list or some other more complex data structure
	- in this case we have to do some estimates
```

- How to lock and ensure that no deadlock occures (When we lock stuff how to make sure that we are not creating deadlock?)
	- We can use tryLocks
	- We can try to sort data and acquire locks for data in a certain order
		- We need to find some property based on which we can order
- We are acquiring and releasing locks
	- it may lead to a lot of locks
	- More locks create more locking overhead

## Optimistic transaction

- We are not doing any locking. The optimism is that we are assume that there won't be any contention. We assume that contention if happens, it happens rarely. We can have a more complex solution (lock based) for cases where we have a contention.
- We can execute code as usual. However, after we have executed code, we are going to check if there are any conflicts. If we detect a conflict, we do something about it. We are giving up and restarting.
- Detect conflicts -> rollback if necessary  -> redo the operation with locks
- We can have two approaches:
	- undo logs
	- isolation
	
### UNDO LOGS

- We start executing and keep track of changes that we did. It will gives as a way to go back. On conflict we just undo our changes.


```
atomic {
	x = 1
	y = 2
	z = x
}
```
- assign initial values if we detect a conflict

- disadvantage
	- When we undo things, we need to make sure that no one sees partial state.


### Isolation

- We have two buffers:
	- write buffer: it keeps track of all writes. Essentialy, it prevents 
	- read buffer: keep track of everything we read

```
atomic {
	a = x
	x++
	b = x
}
//we go through the read buffer and check if everything inside the memory looks the same
//verify all vars we read have the same value (no value has been changed)
//if nothing has been changed we commit (flush) atomically our write buffer to main memory
//if read buffer does not validate, we discard and we redo it
//in essence state before atomic should be equal to the state after atomic. If it is true we commit to main memory
```
- How do we design our language to support this?
	- Should we have nested atomic statements?
		- It seems to be redundant (Weird =S)
			- Why we might need to allow it?

```
- we might want entire function atomic
- However, our function body can be calling another function
- therefore, nested function should be atomic as well
- if we don't have nested atomic statements, then we can't handle it
```
- What abut wait/ notify?
	- We all saw monitors. They provided mutual exclusion. We also wanted to have a way to notify threads inside to tell threads to move on.
	- often, as well as atomic, you get some flavor of retry. Retry says give up. It means when you go through the transaction you notice that something is not working then we can just give up.

```
produce (){
	//we produce, if no space to produce give up
	atomic {
		if(q.full) retry
	}
}


consume (){
	//we produce, if no space to produce give up
	atomic {
		if(q.emptyl) retry
	}
}

- we can communicate between consumer and producer
- when we give up we give another one a chance to perform its action
```
- it would be nicer if retry would specify conditions 
- retry (q.empty == false)

### Other constructs
- none of these ideas are mainstream yet
```
- we can have anbility to detect if transaction failed

atomic{
	A
} or else {
	B
}
```
## Hardware transactions
- Intel (2014) Haswell chips
	- has a bug in design
- IBM Blue Geen/Q
- Both have transactional extensions in hardware
- TSX: Transactional Extensions gives two flavors or HTM (hardware transactional machine)
	- RTM: (restricted transaction memory) actual transaction mode
		- You get 4 instructions where 2 are essential ones for doing transactions

```
XBEGIN //start of transaction. It has error handler as well
	|
	XABORT //it is a retry (give up and restart)
	|
XEND //end of transaction

XTEST //TRUE if you are in a transaction
- it uses isolation
	- L1 cache has a section where it has R/W buffer
	- It can see who touches values in buffers. If someone else touches buffer, it invalidates the whole buffer
	- We don't want to give all of L1 cache to transactions. Therefore, seuqence of transactions is small (limited capacity of L1)
	- It expects small transactions
	- It can detect conflicts on cache line granularity
	- If two processes write to the same cache line then it is considered as a faillure
```
### HLE: (hardware lock elision)
		- It gives transactional support for locks trying to avoid locks
		- Seemlessly switch to transactions
		- use locks to convert to transactional design

```
//instead of locking and unlocking you acquire you do acquire and release
//we can just change locking sections into acquire and release sections
XACQUIRE 
	//if transaction succeeds we are all done
	//if it fails, then we do normal locking code
XRELEASE
```
- Note it is not quite the same as lock and unlock

```
T0
lock
	a = y
	x = 3
	b =c
unlock
T1
x = 2

- lock and unlock will prevent race conditions
- acquire and release is going to execute it and fail and then move to execution
```

# Message Passing
- We are dealing with distributed memory. Memory that is not shared
- We are going to have processors and CPU attached to their own memory. In order to communicate between each other, we are going to pass messages between processes.
	- It makes certain problems less significant and maybe easier to understand
- There are two forms of message passing:
	- Asynchronous message passing
		- you drop a message and go on with you business. We are not blocking on sending
		- 
	- Synchronous message passing 
		- we are waiting for response after we have sent

- Synchronous and asynchronous block when receiving

- Note there are a lot of variations in message passing channels:
	- ordering
	- capacity
	- reliability

```
public class SynchChannel {
	//channel has capaciy 1
	private Object message;
	//producer consumer
	public synchronized void Send(Object message){
		this.message = message;
		notify(); //tell consumer 
		while(this.message != null) //for synchronized sender
			wait( );
	}

	public synchronized Object receive(){
		while(this.message != null)
			wait();
		object received = message
		m = null;
		notify(); //for synchronized sender 
		return received;
	}
```
- Which one is better?
	- Synchronous has higher expressiveness. There are thing we can do with synchronous channel that we cannot do with asynchronous channel.
