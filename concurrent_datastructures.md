# Concurrent Data Structures

## Summary
- Lock-free 
- Lock-free list
  - Elimenation stack
  
  #10.03.2017 

# Lock free data structure (continued)



## Last time
- lock free stack
- elimenation stack
	- more interesting concurrent stack
	- When we tried to optimize push -> pop opertion by putting them into pairs
		- We used exchanger (to make it efficient we need to have a high frequency of use)
- Lock free linked list

```
try add(Nonde n, Node prev){
	n.next = prev.next; //previous points to nxt
	return CAS(prev.next, n.next, n)
}

//it kind off reminds us of problem in the first assignment
try Remove(Node n, Node prev){
	//prev -> n -> ...
	return CAS(prev.next, n, n.nxt);
}
```

## Consider we have a scenario

- We have two three elements in a list where beginning Node and End node are not changed

### Reminder
- CAS(what we are setting, what it is equal to , thing we try to set to)

```
H -> x -> y -> Z

Problem 1:
T0: Tries remove x
T1: Tries to add w between x and y
T0						T1
tryRemove(x,H)			tryAdd(w,x)
	CAS(H.next, x, y)		w.next = y
							CAS(x.next, y, w) //we think both suceeded but we lost w

Problem 2:
T0: remove x
T1: remove y

T0							T1
tryRemove(x,  H)			tryRemove(y, x)
CAS(H.next, x, y)			CAS(x.next, y, z) //we think that we have deleted Y but Y is still in the list
H -> x -> y -> z -> T
H point to Y
x Point to Z
```

### Solution
- H -> * -> x -> * -> y -> * -> z _> * -> T //pointers
- We are doing wide CAS
	- track <next, bool> //our element has pointer and its bool status
	- Sometimes we remove nodes and sometimes we mark nodes as removed
	- We are going to make a remove into two stages
		- We marks it and then we remove it 
		- We need only effectively mark it

```
tryAdd(Node n, Node prev){
	n.next = prev.next
	return CAS(prev.next, <n.next, false>, <n, false>); // we test if node still exists
}

tryRemove(Node p, Node prev){
	Node succ = n.next;
	if (CAS(n.next, <succ, false>,<succ, true>)): //if points to successor and still exists. we indicate that n does not exists anymore
		CAS(prev.next, <n, false>, <succ, false>)
		return true
	else:
		return false
}
```
```
Problem 1
T0: remove x
T1: add w between x and y

T0									T1
tryRemove(x,H)						tryAdd(w,x)
CAS(x.nextm <y, false>, <y,true>)	CAS(x.next, <y, false>, <y, false>) //mutually exclusive 
CAS(H.next, <x,false>,<y, false>)		

Problem 2
T0: delete x
T1: delete y

T0										T1
//can be done at the same time the line bellow
CAS(x.next, <y, false>, <y, true>)			CAS(y.next, <z, false>, <z, true>) 
CAS(H.next, <x, false>, <y, false>)		CAS(x.next, <y, false>, <z, false>)

H -> x -> y -> T //x is deleted and y is marked as deleted, y is logically deleted
we don't fail to remove y
but we may still have node in the list -> y is marked

- Therefore, at some point, we will have to clean up marked lists
- We can do it in list search
	- Find/search process will also do clean up

restart:
	while (true){
		pred = H
		current = pred.next
		while(current != t){
//we are trying to delete it, if not working then we restart
			if( !CAS(pred.next, <Curr, false>, <succ, false>) ){
				continue restart //we restart
			}
			else{
				curr = succ
				succ = curr.next
			}
		}
		if(curr.data == data){
			return curr
		pred = curr
		curr = sucs
		}
		return null
	}
			
```

## Universal Construction (Any datastructure into lock free form)
- It is atomic some threads are making progress, but not necessarily efficient
- We are making an interface  for our sequential lock free data strucutre SeqObject{} 
- Invocations: it is a data structure operations of some form
	- Push, pop, add, remove etc.
	- You get some response back

```
public interface SeqObject{
	public Response apply(Invoc i); //we call apply with different invocations

}
```
- We are going to build heap of history
- Datastructure changes its state after every operation
	- Threads must agree on the order of operations that are going to modify state of datastructure
		- We need to agree who is going to modify data structure

```
i = new Invoc()
do{
	j = consensus(i) //we try to win a chance to modify the data structure
	//which invocation actually wins
}while(j != i);
s = tail //we start from the initial state of the data structure
state = s.state
//look at the chain of operations until we get to our state
//we apply every invocation until we get to our state
do{
	//loop through states and apply invoke
}
```
What is a consensus object?
- Check in the book

- our consensus operation was one shot
- reusable consensus
	- keep consensus in some sort of a list
		- Winning thread attaches new consensus for the next thread to the list
