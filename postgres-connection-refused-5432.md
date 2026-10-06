# PostgreSQL &quot;Connection Refused&quot; on 5432: Cluster Down or Not Listening — NinjaOps

Refused on 5432 means nothing is accepting there: cluster not running, listening on localhost only, or wrong port. Postgres makes each of these easy to check directly.

# PostgreSQL "Connection Refused" on 5432: Cluster Down or Not Listening
  Refused on 5432 means nothing is accepting there: cluster not running, listening on localhost only, or wrong port. Postgres makes each of these easy to check directly.

  
    
## What you'll see

    
      - psql: error: could not connect to server: Connection refused — Is the server running on that host and accepting TCP/IP connections?
      - Local psql works, remote/app connections refused
      - Fails after a restart or config change
    
  

  
    
## Root causes

    
    
      
### Cluster not running (or crashed on bad config)

      systemctl status postgresql and journalctl -u postgresql -n 30 show a stop or a config rejection (e.g., a syntax error prevents startup after an edit).

    
    
      
### listen_addresses is 'localhost' by default

      Remote connections are refused until listen_addresses includes the network interface (or '*'). Check: psql -c 'SHOW listen_addresses;' while local.

    
    
      
### Port moved or multiple clusters

      Debian supports versioned clusters on 5433+; pg_lsclusters lists them and their actual ports.

    
  

  
    
## Fix it

    
      
        Check cluster state and startup logs
        
```
systemctl status postgresql --no-pager; journalctl -u postgresql -n 30 --no-pager
```

      
      
        Confirm where it's actually listening
        
```
ss -tlnp | grep 543; sudo -u postgres psql -c 'SHOW listen_addresses;'
```

      
      
        Open it for remote access deliberately
        
```
# postgresql.conf: listen_addresses = '*'  +  pg_hba.conf: hostssl all all 10.0.0.0/8 scram-sha-256   then: systemctl reload postgresql
```

      
      
        Docker: publish the port or use the internal network
        
```
docker run -d -p 5432:5432 -e POSTGRES_PASSWORD=... postgres:16   # app containers on the same network: use the service hostname
```

      
    
  

  
  
    Partner pick — affiliate link
    [Recommended cloud for this setup →](https://links.ninjaops.win/go/digitalocean?subid=fix-postgres-connection-refused-5432)
    Spin up a $4 droplet or managed Kubernetes and test these fixes on a clean box — new accounts get intro credit to work with.

  

  
    
## Field note

    A postgres that refuses config edits startup with FATAL hints — read the log line right after your change; it names the file and directive. Expose only with hostssl + scram in pg_hba; 'trust' for remote subnets is how databases get dropped.

  

  
  
    
## Common questions

    
      Local psql connects but my app can't. What's different?
      Local connections use the unix socket; apps usually use TCP. If listen_addresses is localhost and the app is on another host, TCP is refused. Show it with SHOW listen_addresses and widen it plus a matching pg_hba rule.

    
    
      How do I know if it's a firewall instead?
      Refused = an active rejection (nothing listening or REJECT rule). Timeout = dropped packets (firewall drop). That distinction tells you which layer to debug — ss -tlnp for the former, security groups for the latter.

    
  

  
  
    
## Related fixes

    
      - [PostgreSQL: "Deadlock Detected" (40P01) — Ordering and the Retry](/fixes/postgres-deadlock-detected/)
      - [PostgreSQL "FATAL: role \"postgres\" does not exist"](/fixes/postgres-role-does-not-exist/)
      - [Postgres Down or Slow: Disk Filled by WAL (pg_wal) — Space Recovery Order](/fixes/postgres-disk-full-wal/)
      - [Postgres "the database system is starting up" — Crash Recovery or Stuck?](/fixes/postgres-database-in-recovery/)
    
  

  
  
    
## Ship it right the first time

    Our most-documented failures, packaged as ready-to-ship starter kits: Docker, Kubernetes, and Terraform.

    [Browse the template store →](https://store.ninjaops.win/templates/)
    One-time. Yours to modify. Instant download from the [NinjaOps template store](https://store.ninjaops.win/templates/).

  

  
    
## Get new fixes by email

    One short email when new fixes and production templates drop. No spam, unsubscribe anytime.


---

**Vetted tooling for this stack** — hardened cloud hosting and monitoring picks, disclosure-first: [links.ninjaops.win/go/digitalocean?subid=gh-postgres-connection-refused-5432](https://links.ninjaops.win/go/digitalocean?subid=gh-postgres-connection-refused-5432)

*Full guide with all diagnostics: [ninjaops.win/fixes/postgres-connection-refused-5432/](https://ninjaops.win/fixes/postgres-connection-refused-5432/)*
