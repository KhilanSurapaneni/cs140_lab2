Last name of Student 1: Surapaneni
First name of Student 1: Khilan
Email of Student 1: ksurapaneni@ucsb.edu
Last name of Student 2: Eirini
First name of Student 2: Schoinas
Email of Student 2: eirini@ucsb.edu

See the description of this assignment  for detailed reporting requirements 


Part B

Q2.a List parallel code that uses at most two barrier calls inside the while loop
In work_block() and work_blockcyclic(), each iteration uses two pthread_barrier_wait(&mybarrier) calls: 
one after computing y, and one after updating x = y and the stop decision, so all threads stay in sync.

Q2.b Report parallel time, speedup, and efficiency for  the upper triangular test matrix case when n=4096 and t=1024. 
Use 2 threads and 4  threads (1 thread per core) under blocking mapping, and block cyclic mapping with block size 1 and block size 16.    
Write a short explanation on why one mapping method is significantly faster than or similar to another.

Upper triangular case (n=4096, t=1024). 
Upper block mapping took 0.100034 s with 1 thread, 0.069936 s with 2 threads, and 0.038424 s with 4 threads. 
Upper block-cyclic with r=1 took 0.086955 s with 1 thread, 0.051896 s with 2 threads, and 0.027023 s with 4 threads. 
Upper block-cyclic with r=16 took 0.086376 s with 1 thread, 0.052840 s with 2 threads, and 0.025816 s with 4 threads.

For upper block mapping, the speedup and efficiency were 1.43 and 0.72 with 2 threads, and 2.60 and 0.65 with 4 threads. 
For upper block-cyclic r=1, they were 1.68 and 0.84 with 2 threads, and 3.22 and 0.80 with 4 threads. 
For upper block-cyclic r=16, they were 1.63 and 0.82 with 2 threads, and 3.35 and 0.84 with 4 threads.

Block-cyclic mapping is faster here because the upper triangular matrix makes early rows much more expensive than later rows. 
With block mapping, one thread can get more of the expensive early rows which causes extra waiting at the barrier each iteration. 
Block-cyclic instaed allows you to spreads expensive and cheap rows across threads, improving balance and reducing waiting. 



Please indicate if your evaluation is done on CSIL and if yes, list the uptime index of that CSIL machine.  
Machine: csilvm-02.cs.ucsb.edu. Uptime: 13:16:11 up 8:15, 10 users, load average: 0.02, 0.44, 0.51.



-----------------------------------------------------------------
Part C

1. Report what code changes you made for blasmm.c. 




2. Conduct a latency and GFLOPS comparison of the above 3 when matrix dimension N varies as 50, 200, 800, and 1600. 
Run the code in one thread and 8 threads on an AMD CPU server of Expanse.
List the latency and GFLOPs of  each method in each setting.  
Explain why when N varies from small to large,  Method 1 with GEMM starts to outperform others. 
