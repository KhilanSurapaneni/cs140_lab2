Last name of Student 1: Surapaneni
First name of Student 1: Khilan
Email of Student 1: ksurapaneni@ucsb.edu
Last name of Student 2: Eirini
First name of Student 2: Schoinas
Email of Student 2: eirini@ucsb.edu


If CSIL is used for performance assessment instead of Expanse, make sure you evaluate when such a machine is lightly 
loaded using “uptime”. Please  indicate your evaluation is done on CSIL and list the uptime index of that CSIL machine.  

Report 
----------------------------------------------------------------------------
1. How is the code parallelized? Show your solution by listing the key computation parallelized with
  OpenMP and related code. 

In parallel_itmv_mult, the program first tells OpenMP how many threads to use (omp_set_num_threads(threadcnt)). 
Then it chooses a scheduling policy based on mappingtype: “block mapping” uses static scheduling with a big contiguous chunk 
per thread, “block-cyclic” uses static scheduling with a fixed chunksize, and “dynamic” uses dynamic scheduling with chunksize. 
After that, omp_set_schedule(sched, chunk) saves that choice, and the two loops inside each iteration use #pragma omp parallel 
for schedule(runtime) so OpenMP actually applies the chosen schedule. In each iteration k, the first parallel loop splits the rows 
i=0..n-1 across threads and calls mv_compute(i) to compute one output row y[i] = d[i] + A[i]*x. The second parallel loop copies y 
back into x (each thread copies different indices), so the next iteration uses the new x. The “upper triangular” optimization 
(skipping the zero part) happens inside mv_compute by starting j at i for the triangular case, so you don’t waste time 
multiplying by zeros.

----------------------------------------------------------------------------
2.  Report the parallel time, speedup, and efficiency with blocking mapping, block cyclic mapping with block size 1 
and block size 16 using  2 cores (2 threads), and 4 cores (4 threads) for parallelizing the code 
in handling a full dense matrix with n=4096 and t=1024. 

For the dense case (your Test 9–11), with 2 threads the latencies were 10.680871s (block), 12.638258s (cyclic r=1), and 
11.389939s (cyclic r=16), with GFLOPS 3.2169, 2.7187, and 3.0167 respectively. 
With 4 threads, the latencies dropped to 5.615381s (block), 6.884282s (cyclic r=1), and 5.808514s (cyclic r=16), 
with GFLOPS 6.1189, 4.9910, and 5.9154.  ￼ Comparing 4 threads to 2 threads, the speedups were about 1.90× (block), 
1.84× (cyclic r=1), and 1.96× (cyclic r=16), which is close to the ideal 2× when doubling threads. The “cyclic r=1” option 
was actually slower than block mapping here, which can happen on dense work because chunk size 1 adds extra scheduling overhead 
and can hurt cache locality (threads bounce around rows instead of working on a big contiguous block).

----------------------------------------------------------------------------
3.  Report the parallel time, speedup, and efficiency with blocking mapping, block cyclic mapping with block size 1 
and block size 16 using  2 cores (2 threads), and 4 cores (4 threads) for parallelizing the code 
in handling an upper triangular matrix (n=4096 and t=1024).

Write a short explanation on why one mapping method is significantly faster than or similar to another.

For the upper triangular case (Test 12–14), with 2 threads block mapping took 8.902025s (1.9304 GFLOPS), 
cyclic r=1 took 6.922056s (2.4825 GFLOPS), and cyclic r=16 took 6.423528s (2.6752 GFLOPS). 
With 4 threads, block mapping took 5.035183s (3.4128 GFLOPS), cyclic r=1 took 3.780632s (4.5453 GFLOPS), 
and cyclic r=16 took 3.056218s (5.6227 GFLOPS). 
The big reason cyclic wins here is load balancing: in an upper triangular matrix, 
early rows do a lot of work (many non-zeros) and later rows do very little work, so if you do block mapping, 
one thread can get stuck with “heavy” rows while another finishes early and sits idle. 
Block-cyclic spreads heavy and light rows across all threads, 
so everyone stays busy and total time drops a lot; that’s why cyclic r=16 is your best result. 
Dynamic scheduling can also balance load, but it has extra runtime overhead from handing out chunks on the fly, 
so it doesn’t beat the best static cyclic setting in your data.