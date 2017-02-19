# Process

- When program runs, it's loaded into address space. 
- In a way process is a file in a format defined by OS.

# Thread

Thread can be viewed as a lighter process.


# What threads can be used for?

* Speed things up
* Hide latency
* Responsiveness

# Hardware basics

1. Basic uniprocessor
2. Multiprocessor

# Processor architecture

CPU is core in the following notes.

1. NUMA (Non uniform memory access)
2. SMP (Symmetric multiprocessor)
3. CMP (On chip multiprocessor)
4. CMT (Coarse-Grain multithreading)
5. FMT (Fine-Grain multithreading)
6. SMT (Simultaneous multithreading)


* NUMA
    * CPU's share second level cache
  
* SMP
   * All CPU's share the same cache 
   
* CMP
   * Each CPU has its onw cache

* CMT
   * Provides hardware support for context switches
   
* FMT
   * Like CMT but switches every cycle
   * Has threads in a circular structure and switches them by turning it

* SMT (Hyperthreading)
   * Resources of each CPU are available to both CPUs 
   * Both CPUs may share
      * ALU: Arithmetic Logic Unit
      * FPU: Floating point unit
      * LSU: Load/Store unit
      * BPU: Branch prediction unit 
   