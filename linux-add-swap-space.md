# Adding Swap on Linux Without a Reinstall (the 4-Command Version) — NinjaOps

A 2–4GB swap file takes four commands and gives your server a memory cushion. Here&#39;s when swap genuinely helps, when it hides a problem, and the exact settings to use.

# Adding Swap on Linux Without a Reinstall (the 4-Command Version)
  A 2–4GB swap file takes four commands and gives your server a memory cushion. Here's when swap genuinely helps, when it hides a problem, and the exact settings to use.

  
    
## What you'll see

    
      - free -h shows Swap: 0B on a VM that occasionally gets memory spikes
      - OOM kills during backups, deploys, or nightly cron bursts
      - Monitoring shows memory usage pinned near 100% with nowhere to go
    
  

  
    
## Root causes

    
    
      
### No swap configured at all

      Cloud images ship without swap. Linux can then only shed pressure by killing — there's no slow path. swapon --show confirms.

    
    
      
### Swap is expected to substitute for RAM

      Swap on spinning disks (or heavily throttled cloud disks) can make things worse — it's a cushion, not capacity. Keep swappiness low so swap absorbs spikes, not steady load.

    
  

  
    
## Fix it

    
      
        Create and enable the swap file
        
```
sudo fallocate -l 2G /swapfile && sudo chmod 600 /swapfile && sudo mkswap /swapfile && sudo swapon /swapfile
```

      
      
        Persist across reboots
        
```
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

      
      
        Tune swappiness: swap for spikes, not steady state
        
```
sudo sysctl vm.swappiness=10 && echo 'vm.swappiness=10' | sudo tee /etc/sysctl.d/99-swap.conf
```

      
      
        Verify
        
```
free -h && swapon --show && cat /proc/sys/vm/swappiness
```

      
    
  

  
  
    Partner pick — affiliate link
    [Recommended cloud for this setup →](https://links.ninjaops.win/go/digitalocean?subid=fix-linux-add-swap-space)
    Spin up a $4 droplet or managed Kubernetes and test these fixes on a clean box — new accounts get intro credit to work with.

  

  
    
## Field note

    Size guidance: 2GB is usually plenty as a cushion on 4–8GB servers; match RAM only on desktops/hibernation setups. Containers don't hold swap config — set it on the host; some container runtimes report swap as 0 inside the namespace even when the host has it.

  

  
  
    
## Common questions

    
      Does swap hurt performance?
      Active steady-state swapping does (disk is ~1000x slower than RAM). A small, low-swappiness swap used only during spikes costs nothing at rest and prevents OOM kills. Watch si/so columns in vmstat to see if you're swapping constantly.

    
    
      Can I use a file instead of a partition?
      Yes — swap files on modern ext4/xfs work fine and are easier to resize. Prefer fallocate (instant); on filesystems where it's unsupported, use dd if=/dev/zero.

    
  

  
  
    
## Related fixes

    
      - [The Linux OOM Killer Is Taking Out Your Services](/fixes/linux-oom-killer-hunting-processes/)
      - [Server Load Is High — Find What's Actually Causing It](/fixes/linux-diagnose-high-load/)
      - ["Cannot Allocate Memory" at fork — With Free RAM: pids.max and overcommit](/fixes/linux-fork-cannot-allocate-memory/)
      - [Cron Job 'Installed' but Never Runs](/fixes/cron-job-not-running/)
    
  

  
  
    
## Ship it right the first time

    Our most-documented failures, packaged as ready-to-ship starter kits: Docker, Kubernetes, and Terraform.

    [Browse the template store →](https://store.ninjaops.win/templates/)
    One-time. Yours to modify. Instant download from the [NinjaOps template store](https://store.ninjaops.win/templates/).

  

  
    
## Get new fixes by email

    One short email when new fixes and production templates drop. No spam, unsubscribe anytime.


---

**Vetted tooling for this stack** — hardened cloud hosting and monitoring picks, disclosure-first: [links.ninjaops.win/go/digitalocean?subid=gh-linux-add-swap-space](https://links.ninjaops.win/go/digitalocean?subid=gh-linux-add-swap-space)

*Full guide with all diagnostics: [ninjaops.win/fixes/linux-add-swap-space/](https://ninjaops.win/fixes/linux-add-swap-space/)*
