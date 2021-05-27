lua脚本：

````
 eval "return {redis.call('GET',KEYS[1])}" 1 a
 
 eval "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 username age jack 20
````

