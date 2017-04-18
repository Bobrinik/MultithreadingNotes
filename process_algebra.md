# Process algebra
- It is a system  for representing concurrent processes
	- It focuses on communication
- In general, we discuss (communication function as essence of concurrent computing)
	- message passing
	- synchronous communication
- interleavings

### CSP ( Communicating Sequential Processors )

- define named processes

```
P :: ____  //Process name:: body of process
Q :: ____
```
- CSP is built on regular language: vars, conditions, loops ...
- CSP is a basis for some actual languages //synchronous
	- Occam has some features and ideas
	- GO takes ideas from CSP

- process definitions
	- regular language features 
		- vars, types, control flow constructs ..
		- Interesting constructs in CSP (not in another languages)
			- Processe can communicate
				- processes are isolated (don't have shared memory)
				- they cooperate by sending and receiving messages
					- synchronous send and receive

```
P:: Q ! e //P is a process that sends e to Q
//we are explicitly saying to whom we are sending messages

Q:: P ? x //Q receives a message from P and binds the result to X
//it is the same as x = q.receive(p)
```
We can get more sophisticated in telling if things match or not.
- In order for this communication to happen, p and q must be executing both in paralell
Sequential composition (;)
x ; y; // do x and then do y

parallel composition
P || Q 

example:

P :: Q`e || Q :: P ? x //both processes communicate and then independently proceed

- Guarded commands
G --> C

if (G) do C

G includes communication
P ::: Q ` e --> ... //if I can send to q then I can do that
// if it is going to work somehow, then we are doing something else

```
exp;
single-cell button
A -> C -> B
- processes communicate through intermediarry state (process)

A :: C ! m
B :: C ? x
C :: A ? x -> B ! x //we can int x to specify type
```
```
//what if A and B want to communicate more than once?
//syntax for making iteration
C :: *[int x, A ? x - > B ! x] //it is a while loop out of which you cannot block

//communication blocking
//a choice operator
C1 []  C2 //it is a square
- you are either doing C1 or C2 once. You cannot do them at once or redo them later
```
```
//vending machine

V :: *[(customer ? $1.00 -> make tea()) [] (customer ? $1.25 - > make coffee)] //we need to receive exactly one dollar

V || Customer :: V ! $1.05

[] - it is called external choice
- it is external because it makes choice based on what arrives from outside

v :: *[(customer? $1.00 -> makeTea()) [] (customer ? $1.00 -> makeCofee())] //we are getting tea or coffe at random in this case
v :: Custom ! $1.00

//internal choice [i]
//we are commiting to one side or another arbitrarily

x :: (A ? x -> STOP) [] ( B ? x -> STOP )

A :: x ! 3
B :: x ! 7
A || x
//communication will happen
B || x -> communication happens

x :: (A ? x -> STOP) [i]  (B ? x -> STOP)
A || X
B || X
//x decides which one to pick 
//communication may or may not happen
//it either decides to commit or to listen for another connection 
//if connection is not there then we are stuck
```

## Producer / consumer

P -> B -> C
```
P :: *[ x == produce(); B!x]
C :: *[ int x; B ?  x -> consume(x)]
B :: *[ int x; P ? x -> C ! x ]
```
```
//easy to extend
if we want to have multiple consumer and producers
producer buffer and two consumers
C1 :: *[int x; B ?  x -> consume(x)]
C2 :: *[int x; B ?  x -> consume(x)]
//it should have an option to send to C1 or C2
B :: *[ int x, P ? x -> ( (C1 ! x ) [] (C2 ! x )] //whoever is ready we are going to send to them
//if both are ready with send to one arbitrary
//you can view buffer as a channel between processes
```
- How do we turn B into a buffer FIFO?
- P -> B1 -> B2 -> B3 -> C

```
P  :: *[int x = produce() ; P ? x -> B1 ! x]
B1 :: *[int x; P ? x -> B2 ! x]
B2 :: *[int x; B1 ? x -> B3 ! x]
B3 :: *[int x ; B2 ? x -> C ! x]
C  :: *[int x ; B3 ? x -> consume (x)]

	   B1
	/		\
P - B1 - B3 - B5 - C
	\		/
		B4

B1 :: *[int x; P ? x -> (B2 ! x [] B3 ! x [] B4 ! x)] //we are sending to anyone who is ready to receive
b5 :: *[int x; (B2 ? x -> C ! x) [] (B3 ? x -> C ! x) [] (B4 ? x -> C ! x)]
```

- Thinking about computation as a flow through some processes is merging naturally with some ideas coming from the world of functional programming.
- In pure functional programming
	- Functions do not have side effects. if I compute a function , it does not have an impact on anyone else. h(  f( x, y), g( x, z),  y)
		- What matters is that h can be computed without carrying about in which order f or g are going to be computed
			- could eveluate f(x, y) and g(x, y)


External sends to the working one
Internal sends to someone at random, it can be the working one or it can be the idle one.
