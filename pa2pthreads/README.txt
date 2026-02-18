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

We fixed the BLAS2 matrix multiply in the DGEMV loop by setting 
the pointers for the jth column and then calling cblas_dgemv once per column.
Inside the loop over j, I set B_col = B + j*K and C_col = C_dgemv + j*M for 
column-major storage, and then called cblas_dgemv(CblasColMajor, CblasNoTrans, 
M, K, 1.0, A, LDA, B_col, 1, 0.0, C_col, 1). This implements Cj = A · Bj and 
therefore C = A · B.



2. Conduct a latency and GFLOPS comparison of the above 3 when matrix dimension N varies as 50, 200, 800, and 1600. 
Run the code in one thread and 8 threads on an AMD CPU server of Expanse.
List the latency and GFLOPs of  each method in each setting.  
Explain why when N varies from small to large,  Method 1 with GEMM starts to outperform others. 

With 1 thread:
For N=50: DGEMM 0.000064 s, 3.90 GFLOPS; DGEMV loop 0.000061 s, 4.11 GFLOPS; naive 0.000185 s, 1.35 GFLOPS.
For N=200: DGEMM 0.001347 s, 11.88 GFLOPS; DGEMV loop 0.002110 s, 7.58 GFLOPS; naive 0.007878 s, 2.03 GFLOPS.
For N=800: DGEMM 0.029333 s, 34.91 GFLOPS; DGEMV loop 0.108081 s, 9.47 GFLOPS; naive 0.572356 s, 1.79 GFLOPS.
For N=1600: DGEMM 0.215179 s, 38.07 GFLOPS; DGEMV loop 0.762972 s, 10.74 GFLOPS; naive 9.711926 s, 0.84 GFLOPS.

With 8 threads:
For N=50: DGEMM 0.027473 s, 0.01 GFLOPS; DGEMV loop 0.000058 s, 4.32 GFLOPS; naive 0.013572 s, 0.02 GFLOPS.
For N=200: DGEMM 0.021395 s, 0.75 GFLOPS; DGEMV loop 0.002819 s, 5.68 GFLOPS; naive 0.013346 s, 1.20 GFLOPS.
For N=800: DGEMM 0.025561 s, 40.06 GFLOPS; DGEMV loop 0.101767 s, 10.06 GFLOPS; naive 0.079099 s, 12.95 GFLOPS.
For N=1600: DGEMM 0.053793 s, 152.29 GFLOPS; DGEMV loop 0.751822 s, 10.90 GFLOPS; naive 1.272032 s, 6.44 GFLOPS

Method 1 with GEMM starts to outperform others as N grows because it is level 4 BLAS where MKL blocks the computation to reuse cache 
and do lots of work per memory load. The DGEMV-loop is Level 2 BLAS and therefore the
 bandwidth-limited at large N, 
and the naive triple loop has worse cache behavior than MKL’s tuned
 GEMM kernel.