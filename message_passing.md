# Message Passing
- We are dealing with distributed memory. Memory that is not shared
- We are going to have processors and CPU attached to their own memory. In order to communicate between each other, we are going to pass messages between processes.
	- It makes certain problems less significant and maybe easier to understand
- There are two forms of message passing:
	- Asynchronous message passing
		- We are not blocking on sending.Drop a message and go on with you business. 
		- We are blocking on receiving
	- Synchronous message passing 
		- We are waiting for response after we have sent. Bring a message and wait for me
		- We are blocking on receiving

- Note that there are a lot of variations in what features message passing channels can implement:
	- ordering of messages
	- capacity of channel
	- reliability of a channel (can we loose packets)

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

```
P
send(Q, 23)

Q
x = 0
x = receive(p)
//asynchronous
//what does P know about x?
//we don't actually know if Q has received message or not
//in asynchronous design, we don't know if the value was updated with 23 or no

//synchronous
//after the send P knows that Q must have received
//therefore X must be 23, and X cannot be 0 after this
```

## Common knowledge
- What do the processes know bout the state of the system?
```
p -----> q

async
- p knows that it sent message to q
- p does not know if q received it until q sends ACK back
- but then q does not know that p received it
```


### 2 Army problem
- There are two mountains. On two sommets there are red army one and two. In the middle of a valley there is blue army. Both red armies have to attack blue army at the same time.
- They need to send a messsager to another red army. 
