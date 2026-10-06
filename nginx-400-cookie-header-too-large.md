# Nginx 400 &quot;Request Header or Cookie Too Large&quot; — NinjaOps

Nginx rejects requests whose headers overflow its default buffers — usually a fat cookie. The fix is one directive, but check which cookie is ballooning or it comes back.

# Nginx 400 "Request Header or Cookie Too Large"
  Nginx rejects requests whose headers overflow its default buffers — usually a fat cookie. The fix is one directive, but check which cookie is ballooning or it comes back.

  
    
## What you'll see

    
      - 400 Bad Request with 'Request Header Or Cookie Too Large' in the body/log
      - One user's browser fails while others work fine
      - Clearing cookies in the affected browser fixes it temporarily
    
  

  
    
## Root causes

    
    
      
### A single bloated cookie (often third-party)

      Old analytics scripts or abandoned frameworks park 8KB+ of cookies on your domain. Nginx's client_header_buffer_size default (1k) can't hold it. Devtools → Application → Cookies, sort by size, tells you which one.

    
    
      
### Header buffers undersized for legitimate use

      SSO/JWT setups with long authorization headers legitimately exceed defaults. Here raising the buffer is the correct fix, not cookie hygiene.

    
  

  
    
## Fix it

    
      
        Confirm it's header overflow in the log
        
```
sudo tail -20 /var/log/nginx/error.log | grep -i 'large'   # 'client sent too long header line' or cookie message
```

      
      
        Raise the header buffer (server or http level)
        
```
# client_header_buffer_size 16k; large_client_header_buffers 4 16k;
```

      
      
        Validate and reload
        
```
sudo nginx -t && sudo systemctl reload nginx
```

      
      
        Hunt the fat cookie before it returns
        
```
# devtools → Application → Cookies → sort by Size: delete/expire abandoned mega-cookies; long-term fix is scoping your own cookies tightly
```

      
    
  

  
  
    Partner pick — affiliate link
    [Recommended cloud for this setup →](https://links.ninjaops.win/go/digitalocean?subid=fix-nginx-400-cookie-header-too-large)
    Spin up a $4 droplet or managed Kubernetes and test these fixes on a clean box — new accounts get intro credit to work with.

  

  
    
## Field note

    This 400 arrives before your app sees the request — nothing in application logs. That absence is the diagnostic fingerprint. Browsers cache nothing here, so the fix is instant for affected users once buffers are raised.

  

  
  
    
## Common questions

    
      Why do only some users hit this 400?
      Cookie size varies per user: long session histories, third-party scripts, and stale cookies accumulate. The affected browser carries the oversized cookie; a clean profile works fine.

    
    
      How large can I safely set client_header_buffer_size?
      16k-32k is a sane ceiling for most sites — headers are allocated per connection. If you're tempted to go beyond that, shrink the cookies instead; many clients have their own limits.

    
  

  
  
    
## Related fixes

    
      - [Nginx 400: "Request Header or Cookie Too Large" — Buffer Tuning Done Right](/fixes/nginx-400-header-too-large/)
      - [ERR_TOO_MANY_REDIRECTS: The Redirect Loop You Created](/fixes/too-many-redirects-loop/)
      - [Nginx 413 Request Entity Too Large (client_max_body_size)](/fixes/nginx-413-request-entity-too-large/)
      - [Nginx 502 "Upstream Sent Too Big Header" — Buffers, Not the App](/fixes/nginx-upstream-sent-too-big-header/)
    
  

  
  
    
## Ship it right the first time

    Our most-documented failures, packaged as ready-to-ship starter kits: Docker, Kubernetes, and Terraform.

    [Browse the template store →](https://store.ninjaops.win/templates/)
    One-time. Yours to modify. Instant download from the [NinjaOps template store](https://store.ninjaops.win/templates/).

  

  
    
## Get new fixes by email

    One short email when new fixes and production templates drop. No spam, unsubscribe anytime.


---

**Vetted tooling for this stack** — hardened cloud hosting and monitoring picks, disclosure-first: [links.ninjaops.win/go/digitalocean?subid=gh-nginx-400-cookie-header-too-large](https://links.ninjaops.win/go/digitalocean?subid=gh-nginx-400-cookie-header-too-large)

*Full guide with all diagnostics: [ninjaops.win/fixes/nginx-400-cookie-header-too-large/](https://ninjaops.win/fixes/nginx-400-cookie-header-too-large/)*
