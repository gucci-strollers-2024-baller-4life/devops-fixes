# SSH Connection Times Out (Hanging Before Auth) — NinjaOps

A hang then &#39;Connection timed out&#39; means packets are being silently dropped — firewall, wrong IP, or nothing listening. It never got to your keys.

# SSH Connection Times Out (Hanging Before Auth)
  A hang then 'Connection timed out' means packets are being silently dropped — firewall, wrong IP, or nothing listening. It never got to your keys.

  
    
## What you'll see

    
      - 'ssh: connect to host ... port 22: Connection timed out'
      - No password/key prompt ever appears
    
  

  
    
## Root causes

    
    
      
### Firewall DROP between you and the host

      DROP gives timeouts (REJECT would give 'connection refused' fast). Cloud security groups, host firewalls, or corporate egress filters.

    
    
      
### Wrong address or NAT/port-forward missing

      Private IP from outside the VPC, stale DNS, or the router's forward rule was removed.

    
    
      
### ssh not listening / host down

      sshd stopped or server powered off — looks identical from outside when a firewall drops packets.

    
  

  
    
## Fix it

    
      
        Is the port even open? (timeout = filtered, refused = open-but-dead)
        
```
nc -zv -w 5 host 22
```

      
      
        See where packets die
        
```
traceroute -T -p 22 host
```

      
      
        From another path (console, VPN, bastion), check the listener and firewall
        
```
sudo ss -tlnp | grep :22 && sudo ufw status   # or nft list ruleset
```

      
      
        Check cloud security group allows your current IP
        
```
# provider console: inbound 22 from your IP — not 0.0.0.0/0
```

      
    
  

  
  
    Partner pick — affiliate link
    [Recommended cloud for this setup →](https://links.ninjaops.win/go/digitalocean?subid=fix-ssh-connection-times-out)
    Spin up a $4 droplet or managed Kubernetes and test these fixes on a clean box — new accounts get intro credit to work with.

  

  
    
## Field note

    Timeout vs refused is your first diagnostic: refused means the network is fine and sshd is the problem; timeout means the network is the problem and sshd is unknown. Always keep one out-of-band console path to every server you care about.

  

  
  
    
## Common questions

    
      Timeout vs connection refused — what's the difference?
      Refused = something answered with 'no' (nothing listening, or actively rejected). Timeout = no answer at all: firewall DROP, wrong IP, or the host down. The distinction eliminates half the search space before you start.

    
    
      SSH worked yesterday and now times out — what changed?
      Most commonly: the host IP changed (DHCP/cloud), a firewall rule or security group tightened, or fail2ban banned your address after failed attempts. Check from another network first — if it works there, the block is on the path from YOUR address.

    
  

  
  
    
## Related fixes

    
      - ["No Route to Host" — Networking's Different Beast from Connection Refused](/fixes/linux-no-route-to-host/)
      - [SSH: 'Permission Denied (publickey)'](/fixes/ssh-permission-denied-publickey/)
      - [Find Which Process Is Using a Port (Linux)](/fixes/linux-find-process-listening-on-port/)
      - [SSH "Connection Timed Out" on Port 22 — and the Firewall Chain Test](/fixes/ssh-port-22-connection-timeout/)
    
  

  
  
    
## Ship it right the first time

    Our most-documented failures, packaged as ready-to-ship starter kits: Docker, Kubernetes, and Terraform.

    [Browse the template store →](https://store.ninjaops.win/templates/)
    One-time. Yours to modify. Instant download from the [NinjaOps template store](https://store.ninjaops.win/templates/).

  

  
    
## Get new fixes by email

    One short email when new fixes and production templates drop. No spam, unsubscribe anytime.


---

**Vetted tooling for this stack** — hardened cloud hosting and monitoring picks, disclosure-first: [links.ninjaops.win/go/digitalocean?subid=gh-ssh-connection-times-out](https://links.ninjaops.win/go/digitalocean?subid=gh-ssh-connection-times-out)

*Full guide with all diagnostics: [ninjaops.win/fixes/ssh-connection-times-out/](https://ninjaops.win/fixes/ssh-connection-times-out/)*
