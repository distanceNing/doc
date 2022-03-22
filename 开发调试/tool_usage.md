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



查看dns解析服务器配置

````
cat /etc/resolv.conf
````



````
traceroute --tcp -p 80 -n baidu.com
````

查找：

````
find ./ -mtime +30 -type f -name '*iap*'
````

查看core文件路径

````
cat /proc/sys/kernel/core_pattern
````

存储问题排查：

````
iotop
iostat
blktrace
block_dump
````

du与df查看磁盘空间不一致



取出某个key对应的value：

````
 head -1 | awk -F"key=" '{print $2}' | awk -F"&" '{print $1}'
````

去重：

````
| sort | uniq -c
````
