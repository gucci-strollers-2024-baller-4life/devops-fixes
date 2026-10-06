# npm ERR! code EACCES — Cache and Prefix Permission Wars — NinjaOps

npm tried to write ~/.npm or a global prefix it doesn&#39;t own — usually after running npm once with sudo. The fix is ownership, not more sudo.

# npm ERR! code EACCES — Cache and Prefix Permission Wars
  npm tried to write ~/.npm or a global prefix it doesn't own — usually after running npm once with sudo. The fix is ownership, not more sudo.

  
    
## What you'll see

    
      - npm ERR! code EACCES ... permission denied, access '/home/user/.npm/_cacache' or /usr/lib/node_modules
      - Install fails for global packages but works locally (or vice versa)
      - Started after one sudo npm install
    
  

  
    
## Root causes

    
    
      
### Root-owned cache from past sudo usage

      sudo npm install once and parts of ~/.npm (or the global prefix) become root's. Every later non-sudo npm run trips over them. ls -la ~/.npm shows the ownership takeover.

    
    
      
### Global prefix points at a system directory

      npm prefix -g defaults to /usr/lib on some distros — a root-owned tree. Install globals to your own directory instead of fighting it with sudo.

    
  

  
    
## Fix it

    
      
        Reclaim your cache and config dirs
        
```
sudo chown -R $(whoami):$(id -gn) ~/.npm ~/.config 2>/dev/null
```

      
      
        Point the global prefix somewhere you own
        
```
mkdir -p ~/.npm-global && npm config set prefix '~/.npm-global' && echo 'export PATH="$HOME/.npm-global/bin:$PATH"' >> ~/.bashrc
```

      
      
        Clean any corrupted state left behind
        
```
npm cache clean --force 2>/dev/null || sudo npm cache clean --force
```

      
      
        Never sudo again — verify with a global install
        
```
npm install -g typescript && which tsc
```

      
    
  

  
  
    Partner pick — affiliate link
    [Recommended cloud for this setup →](https://links.ninjaops.win/go/digitalocean?subid=fix-npm-eacces-cache-permission)
    Spin up a $4 droplet or managed Kubernetes and test these fixes on a clean box — new accounts get intro credit to work with.

  

  
    
## Field note

    Never sudo npm install (any flavor): it solves today's EACCES by creating tomorrow's. Fix ownership once, configure prefix once. nvm sidesteps the entire class of problem — everything lives under your own home dir.

  

  
  
    
## Common questions

    
      Why did sudo npm break my normal npm?
      Root ran as you but wrote files as root: cache entries, lock files, and global packages became root-owned. Your next unprivileged run hits EACCES on exactly those paths.

    
    
      What if I need a global package available to all users on the server?
      Set the global prefix to a shared root-owned location deliberately, install via sudo npm install -g with your eyes open — or better, run the tool via its docker image. But for single-user servers, the home-dir prefix is the clean path.

    
  

  
  
    
## Related fixes

    
      - [npm ERR! code ELIFECYCLE — The Script Failed, npm Is Just the Messenger](/fixes/npm-err-elifecycle/)
      - [Node.js "Cannot find module" — Path, Deps, or Case Sensitivity](/fixes/node-cannot-find-module/)
      - [Node.js "ERR_REQUIRE_ESM" — require() Against an ESM-Only Package](/fixes/err-require-esm/)
      - [npm ERR! ERESOLVE: Peer Dependency Conflict](/fixes/npm-eresolve-peer-dependency-conflict/)
    
  

  
  
    
## Ship it right the first time

    Our most-documented failures, packaged as ready-to-ship starter kits: Docker, Kubernetes, and Terraform.

    [Browse the template store →](https://store.ninjaops.win/templates/)
    One-time. Yours to modify. Instant download from the [NinjaOps template store](https://store.ninjaops.win/templates/).

  

  
    
## Get new fixes by email

    One short email when new fixes and production templates drop. No spam, unsubscribe anytime.


---

**Vetted tooling for this stack** — hardened cloud hosting and monitoring picks, disclosure-first: [links.ninjaops.win/go/digitalocean?subid=gh-npm-eacces-cache-permission](https://links.ninjaops.win/go/digitalocean?subid=gh-npm-eacces-cache-permission)

*Full guide with all diagnostics: [ninjaops.win/fixes/npm-eacces-cache-permission/](https://ninjaops.win/fixes/npm-eacces-cache-permission/)*
