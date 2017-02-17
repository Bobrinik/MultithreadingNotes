# Monitors

## What is a MONITOR?

- Monitor is a data type that protects its private data from concurrent access. 
- Monitor <b>enforces mutual exclusion</b> in its public methods. Only one process at a time can modify data by using public method of a monitor. 

### Analogy
Think about claw machine that you often see in cinemas or theaters. There is only one person operating it at the time. The person is not reaching to take toys directly but instead is using joystick. The way joystick is built ensures that there is only one person operating it at time. Other people wait in queue. 

## How is it implemented?
- It is an abstract data type (ADT) that programmer can ```extend``` to include her private data and to ```@Override``` some methods. 

- In Java methods can be marked ```synchronized``` to make sure that there is only one process using them at the time. 
- Monitor defines condition construct
	- each condition has a q associated with it
	- condition.wait() //process that invokes it goes to sleep
	- condition.signal() //process wakes up only one process

- In java each object has one lock and one condition variable associated with it

```
wait(): // puts thread into wait q and then unlocks and goes to sleep
	unlock(this)
	thread.current.sleep()
	
notify(): // gets one thread from wait q and puts it into enter q
	this.notify() //notifies thread that does wait on this
```

## Monitor semantics
1. Signal and continue (SC): notifier wakes up someone and continues inside a monitor
2. Signal and wait (SW): notifier is kicked out of the monitor and signalee enters into monitor without competition
3. Signal and urgent wait (SUW): it is simillar to signal and wait, but notifier is guranteed to go next in
4. Signal and return (SR): notifier thread exits the monitor at the same time it signals
5. Automatic signalling: wake up everyone every time there is a monitor exit



# Exercises:

#### Exercise 1:
 
A modulo semaphore is similar to a counting semaphore. Rather than a normal down() operation, however, it provides a down(int m) operation. This genaralizes down(), causing a caller to block as long as the count is 0 mod m instead of just 0.

GIve an implementation in pseudo-code of modulo semaphores using monitors and condition variables.

```
	public class MS {
		private int count;
	
		public MS(int init){
			count = init;
		}

		public synchronized void down(int m){
			while(count % m == 0)
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

#### Exercise 2:

A bounded semaphore, Sb, is a general (counting) semaphore which both cannot be decremented below 0, and cannot be incremented above a given upper-bound smax > 0. A thread which tries to violate one of those properties blocks until it can complete its operation. Bounded semaphores may also support a setBound(int n) method, which allows threads to atomically change smax to any value > 0.

Give pseudo-code based on monitors for an implementation of bounded semaphores that supports the P(), V(), and setBound(int n) methods.

```
public calss MB{
	volatile private int count;
	volatile private int upperbound;
	
	public MB(int init_count){
		count = init_count;
	}
	
	public void synchronized setBound(int n){
		count = n;
		notifyAll(); //some threads may be unblocked after change
	}
	
	public void synchronized V(){ //we increase
		while(count+1 !< upperbound)
			try{wait ();	} catch (InterruptedException ie){}
		count++;
		notifyAll();
	}
	
	public void synchronized V(){ //we increase
		while(count-1 !> 0)
			try{wait ();	} catch (InterruptedException ie){}
		count--;
		notifyAll();
	}	
}

```
#### Exercise 3:
Implement a semaphore using a monitor.

```
public class Semaphore{
	private int count;
	
	public Semaphore(int init){
		count = init;
	}
	
	public synchronized void V(){
		count++;
		notify();
	}
	
	public synchronized void P(){
		while(count == 0)
			wait()
		count--;
		notify();
	}
}
```


## Common problems
1. Producer/Consumer
2. Readers and Writers

## Side notes
- Spontaneous wake ups: It is expensive to ensure NO SPONTANEOUS WAKEUPS

```
while(b){
	wait();
}

```
- How do we wake up only producer?
	- We do notifyAll() consumers will wakeup and go to sleep (while loops); however, producer will start producing



# Producer/Consumer with monitor

```
private boolean filled;
private int buffer;

synchronized void produce(){
	if(filled){
		wait();
	}
	buffer = ...;//put somethin in
	filled = true;
	notify(); //wake up consumer
}

synchronized void consume(){
	if(!filled){
		wait();
	}
	consume(buffer);
	filled = false;
	notify(); //wake up producer
}
Note:
wait() is a subject to spontaneous wake up. 
Therefore we need to double check condition.
while(b){
	wait();
}
```
## Readers/Writers
## How to implement condition variable in Java?
```
public class CV{
	public void cv_wait(mutex m){
		try{
			synchronized(this){
				m.unlock();//1
				wait(); //2 
				//1 and 2 effectively atomic since there is no thread that can cause a problem
			}
		} catch(InterruptedException ie){}
		finally{
			m.lock(); //executes no matter what
		}
	}
	
	public synchronized CV_notify(){
		notify();
	}
}
```