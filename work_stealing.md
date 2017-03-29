## Work stealing
- Cilk++ is a language that has a greedy scheduler
	- It has an idea of spawning tasks
		- We can create a thread to execute a function asynchroniously
	- When thread executes it starts creating tasks
	- When thread finishes executing its task, it starts consuming tasks that it previously produced
		- When thread is idle we need to find a task to execute
			- How do we choose a task which to execute from  pool of tasks?
	
```
Tasks queue
a <-- Thread grabs task from here
|
b
|
c <-- Thread grabs task from here
```

- Double ended queue
	- we push tasks as we create them and we pop them when we execute them
	- What if we have a thread wants to do something?
		- Idle threads are allowed to steal tasks (work stealing approach)
		- There is a little bit of contention if we are going to fight over one end of queue
		- To solve this, we are going to grab tasks from the top of the queue
- Execution stack management is more complex
	- Who is executing what and in what context

```
A(){
spawn B()
spawn C()
synhronize //wait for tasks B() and C(0 to finish
}

B(){
	spawn D()
	spawn E()
	synchronize
	foo() //sequential function
}

c(){
	spawn F()
	spawn G()
	synchronize
}

main(){
	A()
}
```
- spawning tasks means that we are going to execute some code
	- very time we are spawning a task we want a free stack allocated to every task

```
P0:  	A => A - B => A - B - D


P1:		C


P2:					E

- As time goes on, we still execute E, C spawns F() and G()
- One processor may be doing tasks assynchroniously and then we have sequential function for which we need to allocate space on stack for this
	- It creates extra complexity to the construction of stack

call chain
   a
/	\
b	  \
	   c
	/	\
	d	e

a => A - B => A-C => A - C - D => A-C-E => A-C => A
```

- In parallel systems
	- we don't create a stack that is like a chain. We create a stack which is called cactus stack.
		- Cactus-stack is a stack with branching

## PGAS
- parallel programming models
- We have a processes running independently and on top of them we have a layer of hared memory
```
[		]
 |	|	|
p1	p2	p3
```
- data parallel
	- We have one istruction stream
	- All threads execute the same instructions at the same time but on different pieces of data

- Partitioned global address space
	- We want to have an idea of shared and private memory
		- We want to  ave efficient access to shared memory
		- we want to take advantage of locality, we are using shared memory but it is owned by one process at the time. Thread that owns this shared memory gets faster and more efficient access to this segment of data

- Asynchrononous PAGAS
	- We can have read only data and everyone has access to it
	- Threads can be executed in different places
		- Processes decide where they need to be in order to do some computation
		- This is an interesting idea invented by IBM dfor X10 language

### Basic mechanism

```
async s //statement
- it creates a process to execute s
	- we are not oblige to do this, if thread is there then we execute it else we don't do it
	- it is like a suggestionto make something parallel to the system by the programmer

async {x = x + 1}
y = y + 1
- we can have two threads or one thread

nestings of async are allowed
async{
	x = 1
	async {
	y = 2
	}
}
```
- When do async finish?
	- In java we can have a handle on a thread, but in async we don't have a handle and we cannot really ask it any questions
	- Async process does not have a handle, so we cannot joint on it. We cannot ask it questions if it is finished.
- finish
	- acts as a baseline
```
fn (i in 0 - 100){
	async {a[i] = a[i]+1}

}
- what do we know about contents of a?
	- a[i] could be 0 or 1

finish S
- all asyncs inside of S must be done
- we  cannot progress untill all asyncs in S are finished
```
- X10
	- has a construct for conditional synchronization

```
when(C) S
	wait for C to be true, and then execute S atomically

at(P) S
- at place p go execute s //migrate computation to where it is needed
- we are moving computation to data
- data does deep copy of local data then goes to the place does computation and discard out copy
- we take a temporary transient copy
- there are different ways of specifying location like labeling or refering to host of where the process is running
```
- X10 has different modes
	- it can compile to java virtual machine
	- there is a vversion that compiles to C++

