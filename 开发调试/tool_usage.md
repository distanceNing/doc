 perf stat ./a.out

````
 Performance counter stats for './a.out':

        226.371929      task-clock (msec)         #    0.999 CPUs utilized          
                 0      context-switches          #    0.000 K/sec                  
                 0      cpu-migrations            #    0.000 K/sec                  
               331      page-faults               #    0.001 M/sec                  
                 0      cycles                    #    0.000 GHz                    
                 0      stalled-cycles-frontend   #    0.00% frontend cycles idle   
                 0      stalled-cycles-backend    #    0.00% backend  cycles idle   
                 0      instructions              #    0.00  insns per cycle        
                 0      branches                  #    0.000 K/sec                  
                 0      branch-misses             #    0.000 K/sec                  

       0.226567038 seconds time elapsed
````

