# Semaphores

# What is a SEMAPHORE?

1. When you create one, you can initialize to any value
2. You can only change its state by using increment and decrement
3. When a thread increments a semaphore one of the waiting threads gets unblocked

Thread is blocked: thread tells a scheduler that it cannot proceed. Scheduler does not let thread to run untill there is an event that occurs and unblocks (wakes) thread.

# How do we implement semaphore?

- Semaphore should be incremented and decremented in an atomic way. It should be done in one indivisible step.
- To avoid starvation semaphore should have queues associated on its ```p()``` operation. When process decrements value bellow 0, semaphore adds process to waiting queue.
- When process increments value, it should remove process from waiting queue and 
notify (wake up) it.

```
Note all functions bellow are ATOMIC
Down(p):
    if(value is 0 or less):
        wait until it is not
    decrement once it is above 0

Up(V)
    increment the calue

```
### Note:

* We can arrange waiting queue based on different rules. For example, we can make a priority queue.


# How can we use semaphores?

1. Binary semaphore
    * Its value can be 0 or 1 -> simillar to mutex locks (mutex has ownership not semaphore)
2. Signaling semaphore
    * Its value starts at 0. When another process calls down, it goes into wait stage. 
3. We can provide a solution for producer consumer problem by using them.
4. We can build barriers

### Barriers 
#### Simple barrier

```
s1.init(0)
s2.init(0)
T1 --------- T2

s2.v() ------- s1.v()
s1.p() ------- s2.p()       
```

