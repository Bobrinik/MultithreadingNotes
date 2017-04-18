# Consensus number
- We want to have some sort of a procedure or a rule that can  let us compare atomic operations. We want to measure a power of atomic operation to solve different multithreading problems.
	- One way to evaluate is to measure how easy it is to implement shared data structure like queue, stack ...

- Consensus number is the maximum number of threads for which objects of the class can solve an elementary synchronization problem called consensus.

## Consensus
- A consensus-object provides  a single method called decide() 
- Each thread calls the decide() method with its input v at most once 
	- The method decide() is going to return a vaue meeting the following conditions:
		- consistent: all threads decide the same value
		- valid: the common decision value is some thread's input

```
public interface Consensus<T>{
T decide (T value);
}
```

## Vocabulary
- wait-free: concurrent object is wait free if each method  call finishes in a finite number of steps
- lock-free:  method is lock-free if it gurantees that infinitely often some method call finishes in a finite number of steps
