# Redis Connection Refused (localhost:6379) — NinjaOps

Refused means the Redis process isn&#39;t accepting connections on the port you&#39;re hitting. Walk the standard chain — process, bind, protected-mode, firewall — and it&#39;s usually a five-minute fix.

# Redis Connection Refused (localhost:6379)
  Refused means the Redis process isn't accepting connections on the port you're hitting. Walk the standard chain — process, bind, protected-mode, firewall — and it's usually a five-minute fix.

  
    
## What you'll see

    
      - redis-cli ping → Could not connect to Redis at 127.0.0.1:6379: Connection refused
      - App errors: Error connecting to Redis on localhost:6379 (ECONNREFUSED)
      - Sometimes intermittent: works until restart, then fails
    
  

  
    
## Root causes

    
    
      
### Redis isn't running (or crashed)

      systemctl status redis-server and redis-cli ping tell you in ten seconds. Crashes often follow OOM kills or a corrupted AOF on restart — check journalctl -u redis-server -n 50.

    
    
      
### Bind address or port changed

      Default bind 127.0.0.1 (-::1). If you connect remotely, redis.conf must bind your interface — protected-mode also blocks external connections by default.

    
    
      
### Docker networking mismatch

      localhost:6379 on the host isn't the container's localhost. The port must be published (-p 6379:6379), or the app should join the same docker network and use the service name.

    
  

  
    
## Fix it

    
      
        Check process and service state
        
```
systemctl status redis-server --no-pager; redis-cli -h 127.0.0.1 -p 6379 ping
```

      
      
        Read the startup logs for the crash reason
        
```
journalctl -u redis-server -n 50 --no-pager | tail -20   # look for 'Killed', AOF/RDB load errors, or overcommit warnings
```

      
      
        Fix remote access deliberately (never 0.0.0.0 + no auth)
        
```
# /etc/redis/redis.conf: bind  127.0.0.1 ; protected-mode yes ; requirepass  ; port 6379
```

      
      
        Docker: publish the port or use the network
        
```
docker run -d --name redis -p 6379:6379 redis:7   # app containers: use 'redis' as hostname on a shared network
```

      
    
  

  
  
    Partner pick — affiliate link
    [Recommended cloud for this setup →](https://links.ninjaops.win/go/digitalocean?subid=fix-redis-connection-refused)
    Spin up a $4 droplet or managed Kubernetes and test these fixes on a clean box — new accounts get intro credit to work with.

  

  
    
## Field note

    Redis refuses to start with a 'Memory overcommit must be enabled' warning on some kernels: sysctl vm.overcommit_memory=1 fixes fork() failures during saves. If Redis only dies under memory pressure, it's the host OOM killer — check kernel logs before blaming Redis.

  

  
  
    
## Common questions

    
      Redis runs but my app can't connect from another host. Why?
      Default bind is loopback-only and protected-mode blocks external clients. Set bind to include the right interface, keep protected-mode on with a requirepass, then reload: systemctl restart redis-server.

    
    
      How is Redis connection refused different from a timeout?
      Refused = host reachable, nothing listening (process down or bind/port mismatch). Timeout = packets dropped (firewall, wrong host). That distinction cuts your debug time in half.

    
  

  
  
    
## Related fixes

    
      - [Redis: "Connection Refused" on 6379 — Find the Refusing Layer](/fixes/redis-connection-refused-6379/)
      - [Redis "MISCONF Redis is configured to persist RDB snapshots"](/fixes/redis-misconf-persistence/)
      - [Redis "LOADING Redis Is Loading the Dataset in Memory"](/fixes/redis-loading-dataset-in-memory/)
      - [Redis "WRONGTYPE Operation Against a Key Holding the Wrong Kind of Value"](/fixes/redis-wrongtype-error/)
    
  

  
  
    
## Ship it right the first time

    Our most-documented failures, packaged as ready-to-ship starter kits: Docker, Kubernetes, and Terraform.

    [Browse the template store →](https://store.ninjaops.win/templates/)
    One-time. Yours to modify. Instant download from the [NinjaOps template store](https://store.ninjaops.win/templates/).

  

  
    
## Get new fixes by email

    One short email when new fixes and production templates drop. No spam, unsubscribe anytime.


---

**Vetted tooling for this stack** — hardened cloud hosting and monitoring picks, disclosure-first: [links.ninjaops.win/go/digitalocean?subid=gh-redis-connection-refused](https://links.ninjaops.win/go/digitalocean?subid=gh-redis-connection-refused)

*Full guide with all diagnostics: [ninjaops.win/fixes/redis-connection-refused/](https://ninjaops.win/fixes/redis-connection-refused/)*
