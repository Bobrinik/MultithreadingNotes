# OpenMP

- Languge on top of an existing language
	- Originally Fortran/C using "#pragma omp"
	- Extension of java
	- Shared memory paradigm

```
|
|
#pragma omp parallel
//there is no ordering between paralle threads
{
	region to be executed in parallel
	we do fork here 
}
join()
|
|
```
- parallel region 
- when executed a thread master + additional  threads
	- execute the code
- after parallel code is executed all threads are joined
- nested parallelism is possible
- ordering within a parallel region is not defined
- can't branch into or out of the parallel region
- The parallel construct can have different clauses
		- num threads
			- spawn a thread
			- #pragma omp parallel num_threads(4)

# OpenMP Worksharing

- How can threads communicate between each other?
- parallel do or for loop
```
|
--------
| | | | | //fork
--------
| //join
```

## Sections 
- break work into discrete sections

```
#pragma omp section
{
	#pragma omp section
		<code 1>
	#pragma omp section
		<code 2>
}

| | | |
--  | |
| | | |
------
| //join
```


# REDUCTION
- performs a reduction on variables in the list
```
sum = 0
#pragma ...
for i = 0; i < n; i++:
	sum = sum + a[i]

sum?
- 
```



## References
- [Open MP tutorials](https://www.youtube.com/watch?v=nE-xN4Bf8XI&list=PLLX-Q6B8xqZ8n8bwjGdzBJ25X2utwnoEG)


