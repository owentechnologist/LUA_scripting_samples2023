This script captures the length of a Randomly selected SortedSet:

``` 
SCRIPT LOAD "local response = '' local pcaltype = '' local response = {'fail'} local zlength = 0 local pcl = '' local keyname = redis.call('RANDOMKEY') pcl = redis.pcall('ZCARD',keyname) pcaltype = type(pcl) if pcaltype=='table' then response = ('keyname: '..keyname..' is not a zset') else zlength = pcl response = ('keyname: '..keyname..' ZCARD result: '..zlength) redis.call('XADD','XLOGCAPTURE'..KEYS[1],'*','response',response) end return response"
```

When you call it, you must provide a routing value as the key argument
- example {1a}
- it will then pick a random key from the shard matching the routing value.

Calling it several times writes the entries to a Stream that has the name XLOGCAPTURE{1a} which can then be queried:

``` 
> SCRIPT LOAD "local response = '' local pcaltype = '' local response = {'fail'} local zlength = 0 local pcl = '' local keyname = redis.call('RANDOMKEY') pcl = redis.pcall('ZCARD',keyname) pcaltype = type(pcl) if pcaltype=='table' then response = ('keyname: '..keyname..' is not a zset') else zlength = pcl response = ('keyname: '..keyname..' ZCARD result: '..zlength) redis.call('XADD','XLOGCAPTURE'..KEYS[1],'*','response',response) end return response"
"ad1931135604ad3387d2c26630652ea1f312affc" 

> EVALSHA ad1931135604ad3387d2c26630652ea1f312affc 1 {1a} 
"keyname: a{3} is not a zset" 

> EVALSHA ad1931135604ad3387d2c26630652ea1f312affc 1 {1a} 
"keyname: bb{1a} ZCARD result: 2" 

> EVALSHA ad1931135604ad3387d2c26630652ea1f312affc 1 {1a} 
"keyname: a{1a} ZCARD result: 2099"

> XREAD COUNT 100 STREAMS XLOGCAPTURE{1a} 0 1) 1) "XLOGCAPTURE{1a}"    2) 1) 1) "1688065326670-0"          2) 1) "response"             2) "keyname: bb{1a} ZCARD result: 2"       2) 1) "1688065332315-0"          2) 1) "response"             2) "keyname: plun:rally200 ZCARD result: 2009"
```
