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
