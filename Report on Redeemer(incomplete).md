Machine name : Redeemer
Platform : hack the box 
Difficulty : very easy 
targets IP : 10.129.136.187
OS : Linux 5.X
- then machine was broken and didn't properly respond to redis-client.
Machine info : Redeemer is a very easy Linux machine which explores the enumeration and exploitation of a Redis database server while showcasing the redis-cli command line utility and basic commands to interact with the Redis service.
#### Open Ports : 
- **6379** , State : open , service : redis , version : Redis key- value store.
![[Pasted image 20260712200507.png]]
- Redis is an **in-memory key-value database**. Unlike traditional databases that primarily store data on disk, Redis keeps data in **RAM**, making it extremely fast.

#### Enumeration : 

- making the connection to the machine using redis-client.
```
redis-cli -h <IP> -p 6379
```

rest the machine didn't worked !! did not gave any output(ie. hanged )