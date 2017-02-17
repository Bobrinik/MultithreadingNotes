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
In database, there are different processes that read and write. We need to make sure that no 2 threads right at the same time. We don't want something being read while writer is writing. However, we can have multiple readers reading at the same time.

### Readers preference

#### Solution with using semaphores

```
int readers; // we count readers
BinarySemaphore r = 1; rw =1;

//Reader
while(true):
	r.down()//we need to make sure that readers increment sequentially
	reader++
	if(readers == 1):
		rw.down() //we lock writers out
	r.up()
	read()
	r.down() //we need to exit sequentially
	reader--
	if(readers == 0):
		rw.up()
	r.up()
	
//Writer	
while(true):
	rw.down()
	write()
	rw.up()
```

- When there is a stream of readers then it is not known when writer can write "Writer may starve"
- If reader and writer come at the same time, we don't know who comes first "Weak reader preference"

#### Solution with using monitors

```
int nr = 0; //number of readers
int nw = 0; //number of writers
int ww = 0; //number of waiting writers

//Readers

read(){
	synchronized(this){ //make readers wait when there is a writer waiting or working
		while(nw != 0 || ww > 0)
			try{wait()}catch(ie){}
		nr++
	}
	read_something()
	synchronized(this){
		nr--;
		notifyAll(); //other readers will go to sleep if there is a writer
	}
}

//Writer

write():
	synchronized(this){ //writers should enter sequentially
		while( nr != 0): //writer that is the next to come in is waiting for reader to finish
			ww++
			try{wait()}catch(InterruptedException ie){}
		ww--
		nw++
	}
	write_something()
	synchronized(this){
		nw--
		notifyAll()
	}
```
- If there is a stream of writers, readers may starve -> Weak writer preference


#### Fair solution

```
int nr, nw, ww, wr;
mutex l;
CV okread, okwrite;

//Reader
Lock l;
lock(l);
									//we check if there is a writer or waiting writer
if(nw > 0 || ww > 0){
	wr++;
	wait(l, okread); 				//note that we need to protect it against spontaneous wakeup
}
else{
	nr++;
}
unlock(l);

read_something();

									//when we unlock we want to let in others
									//we need to notify waiting readers or writers
lock(l);
nr--;//reader leaves
if(nr == 0){//last reader to leave
	if(ww > 0){ // we need to check if we have writers waiting
		ww--;
		nw ++;
		notify(okwrite)
	}
}
unlock(l);

//Writer
Lock l;

lock(l)
									   //if we have readers or writers we cannot get in
if(nw > 0 || nr > 0 || wr > 0){
	ww++;
	wait(l, okwrite);				  //adds to waiting queue
}else{ 								//no writers or readers
	nw++;
}
unlock(l);

write_something();

lock(l);
nw--;
if( wr > 0){
	nr = wr;
	wr = 0;
	signalAll(okread);
} else if(wr){
	rw--;
	nw++;
	signal(okwrite);
}
unlock(l)
```

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