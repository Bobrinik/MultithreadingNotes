# Monitors

It is an abstract data type (ADT) that permits one process to access data that it contains at the time. 

- Process can access data only through defined methods. 
- Monitor defines condition construct
	- each condition has a q associated with it
	- condition.wait() //process that invokes it goes to sleep
	- condition.signal() //process wakes up only one process

# Exercises:

Exercise 1:
 
A modulo semaphore is similar to a counting semaphore. Rather than a normal down() operation, however, it provides a down(int m) operation. This genaralizes down(), causing a caller to block as long as the count is 0 mod m instead of just 0.

GIve an implementation in pseudo-code of modulo semaphores using monitors and condition variables.

```
	public class MS {
		private int count;
	
		public MS(int init){
			count = init;
		}

		public synchronized void down(int m){
			while(count % m == 0){
				try{wait ();	} catch (InterruptedException ie){}
			count--;
			notifyAll (); //decr may change waiting condition
		}
	
		public synchronized void up(){
			count++;
			notifyAll(); //incr may change waiting condition
		}
	}

```

Exercise 2:

A bounded semaphore, Sb, is a general (counting) semaphore which both cannot be decremented below 0, and cannot be incremented above a given upper-bound smax > 0. A thread which tries to violate one of those properties blocks until it can complete its operation. Bounded semaphores may also support a setBound(int n) method, which allows threads to atomically change smax to any value > 0.

Give pseudo-code based on monitors for an implementation of bounded semaphores that supports the P(), V(), and setBound(int n) methods.

```
public calss MB{
	volatile private int count;
	public MB(int init_count){
		count = init_count;
	}
	
	public void synchronized setBound(int n){
		count = n;
	}
	
	public void synchronized V(){ //we increase

	}
}

```

