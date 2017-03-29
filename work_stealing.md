# Work stealing
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

