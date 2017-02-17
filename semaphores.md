# Semaphores

# What is a SEMAPHORE?

1. When you create one, you can initialize to any value
2. You can only change its state by using increment and decrement
3. When a thread increments a semaphore one of the waiting threads gets unblocked

Thread is blocked: thread tells a scheduler that it cannot proceed. Scheduler does not let thread to run untill there is an event that occurs and unblocks (wakes) thread.

# How do we implement semaphore?

- Semaphore should be incremented and decremented in an atomic way. It should be done in one indivisible step.
- To avoid starvation semaphore should have queues associated on its ```p()``` operation. When process decrements value bellow 0, semaphore adds process to waiting queue.
```

```