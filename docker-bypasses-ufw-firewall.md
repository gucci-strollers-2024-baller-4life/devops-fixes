# Docker Is Bypassing UFW — Published Ports Are Open to the World — NinjaOps

docker run -p 5432:5432 punches through UFW by writing its own iptables rules. Your database may be publicly reachable even though UFW &#39;denies&#39; it.

# Docker Is Bypassing UFW — Published Ports Are Open to the World
  docker run -p 5432:5432 punches through UFW by writing its own iptables rules. Your database may be publicly reachable even though UFW 'denies' it.

  
    
## What you'll see

    
      - Port you thought was firewalled is reachable from outside
      - ufw status verbose says deny, but the port still answers
    
  

  
    
## Root causes

    
    
      
### Docker writes NAT rules ahead of UFW

      The DOCKER chain processes published ports before UFW's INPUT rules ever see the packet.

    
  

  
    
## Fix it

    
      
        Never publish to all interfaces; bind to loopback or the internal net
        
```
docker run -p 127.0.0.1:5432:5432 postgres  # or omit -p and use a network
```

      
      
        If you must expose, control it in the DOCKER-USER chain
        
```
iptables -I DOCKER-USER -p tcp --dport 5432 ! -s 10.0.0.0/8 -j DROP
```

      
      
        Audit what is actually exposed right now
        
```
ss -tlnp | awk '$4 !~ /127.0.0.1|\[::1\]/'
```

      
    
  

  
  
    Partner pick — affiliate link
    [Recommended: lock down access too →](https://links.ninjaops.win/go/nordlayer?subid=fix-docker-bypasses-ufw-firewall)
    Fixing exposure mistakes is the right call — a zero-trust business VPN layer in front of your servers keeps the next mistake from being public.

  

  
    
## Field note

    Assume every -p is public until proven otherwise. Cloud firewalls (security groups, Cloudflare Access) are the reliable boundary — host firewalls do not reliably constrain Docker.

  

  
  
    
## Common questions

    
      Why do my published Docker ports bypass UFW rules?
      Docker writes its own iptables NAT rules (DOCKER chain) ahead of UFW's chains: published ports are exposed before UFW INPUT rules evaluate. It's a rule-ordering reality, not a UFW bug — UFW's deny does not apply to Docker's forwarded traffic.

    
    
      How do I protect published ports then?
      Bind them to the intended interface: -p 127.0.0.1:5432:5432 for local-only, or the LAN IP for internal services. Optionally use DOCKER-USER chain rules for explicit allowlists — the bind approach is simpler and self-documenting.

    
  

  
  
    
## Related fixes

    
      - [UFW Rules Not Taking Effect — Debugging Order That Works](/fixes/ufw-rules-not-working/)
      - [SSH "Host Key Verification Failed" — Changed Keys and known_hosts](/fixes/ssh-host-key-verification-failed/)
      - [Docker: 'Permission Denied While Trying to Connect to the Docker Daemon Socket'](/fixes/docker-permission-denied-docker-sock/)
      - [fail2ban Locked You Out of Your Own Server](/fixes/fail2ban-banned-own-ip/)
    
  

  
  
    
## Ship it right the first time

    Our most-documented failures, packaged as ready-to-ship starter kits: Docker, Kubernetes, and Terraform.

    [Browse the template store →](https://store.ninjaops.win/templates/)
    One-time. Yours to modify. Instant download from the [NinjaOps template store](https://store.ninjaops.win/templates/).

  

  
    
## Get new fixes by email

    One short email when new fixes and production templates drop. No spam, unsubscribe anytime.


---

**Vetted tooling for this stack** — hardened cloud hosting and monitoring picks, disclosure-first: [links.ninjaops.win/go/digitalocean?subid=gh-docker-bypasses-ufw-firewall](https://links.ninjaops.win/go/digitalocean?subid=gh-docker-bypasses-ufw-firewall)

*Full guide with all diagnostics: [ninjaops.win/fixes/docker-bypasses-ufw-firewall/](https://ninjaops.win/fixes/docker-bypasses-ufw-firewall/)*
