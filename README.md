- Hardware, atomicity
   - UMA, NUMA, SMP, CMP, CMT, FMT, SMT architecture
   - Amdahal's Law 1/ [ (1 - p) + p/n ]


Independent threads = threads whose write sets are disjoint and write sets don't overla with read set of another

- Mutual exclusion

    - Only one thread at a time can access critical section

- Simple locks 2.4-2.6, 7.1 - 7.4 

    * The Peterson's lock

        - Two monkeys arrive and both have their flags up. However, only one monkey is a victim. Each monkey checks another monkeys flag.

    * The Filter lock

        - We assign each thread to a room. Last thread that comes into a room is a victim. If there is a thread in a next room and I'm a victim in this rooom I wait.

          When I leave, I go to the room 0.

    * Bakery algorithm (Ticket algorithm)

        - Each thread takes a ticket at the door and waits until there is no other thread with later ticket and a flag raised.

    * Atomic instructions

        * Test-and-set \(TS\)

            - It is expensive because has to go to main memory in a loop

        * Test-and-test-and-set (TTS)

            - It checks first in a cashe. If cashe changed we do TS.

        * Fetch and add

        * Compare and swap (CAS)

            - CAS(x,v1, v2) -&gt; (x != v1) ? false : (x = v2) true; 

Questions:

- Why do we always have a flag and a victim?



- Complex locks 7.5, 8.1 - 8.5

    * Queue locks

    * The CLH queue lock



- Termination, barriers, priority, TSD 17.1 - 17.6, A.2.4


    * Barriers 

        * Simple barrier (with semaphores)

        * Sense reversing barrier [Need to review]

        * Combining tree barrier

        * Static tree barrier 

    * [Semaphores](./semaphores.md)

    * [Monitors](./monitors.md)

- Deadlock, race conditions

    * Consensus problem

    * Dining philosophers



- Expressiveness



- Linearizability, scheduling 3.1-3.6

    * Weak fairness

    * Strong fairness

        - Unconditionally fair

    * Burden's Ass

        - Glitches

    * Burden's principle

  

- [Memory Consistency](./memor_models.md) 
   - 3.8
   - Consistency models
   - X86-TSO
   - Java Memory Model

- Concurrent data structures
   - 6.1-6.4, 10.1-10.6, B8 , 11.1 - 11.4, 9.8
- Open MP

- SPMD and [PGAS](/pgas.md)
   - parallel data
   - partitioned global address space
   - asynchronous PAGAS

- [Work stealing](/work_stealing.md)
   -16.1 - 16.5
   - double ended queues
   - cactus stack

- [Transactional memory](/transactional_memory.md)
   - 18.1 - 18.4
   - transactional memory
   - optimistic transaction
      - undo logs
      - isolation
   - hardware transactions
   - hardware lock elision

- [Message passing](/message_passing.md)
   - asynchronous message passing
   - synchronous message passing

- Process algebra

- Dataflow
