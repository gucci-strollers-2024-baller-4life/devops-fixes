# HTTP 429 Too Many Requests: Read the Headers, Then Back Off — NinjaOps

A 429 is an application-level &#39;slow down&#39;, not a network failure. The response headers usually tell you exactly when to retry and how — clients that honor them fix themselves.

# HTTP 429 Too Many Requests: Read the Headers, Then Back Off
  A 429 is an application-level 'slow down', not a network failure. The response headers usually tell you exactly when to retry and how — clients that honor them fix themselves.

  
    
## What you'll see

    
      - API responses come back 429 Too Many Requests
      - Bursty jobs (fan-out scripts, CI pipelines) fail partway through
      - Retrying immediately makes it worse
    
  

  
    
## Root causes

    
    
      
### Client exceeds the provider's quota

      Rate limits are per token, per IP, or per endpoint. Check the vendor's docs against your request pattern — fan-out loops and retry storms are the usual offenders.

    
    
      
### Missing retry-after handling

      The 429 usually carries Retry-After or X-RateLimit-Reset headers. Clients that ignore them and hammer anyway get deprioritized further or temporarily blocked.

    
  

  
    
## Fix it

    
      
        Read the rate-limit headers
        
```
curl -sI https://api.example.com/v1/things | grep -iE 'retry-after|rate-limit|429'
```

      
      
        Implement exponential backoff with jitter
        
```
# retry on 429/503: wait = min(base * 2^n, cap) + random_jitter; honor Retry-After when present
```

      
      
        Batch or throttle the workload
        
```
# e.g. p-limit in Node, ratelimit in Python, or a token bucket in front of fan-out calls
```

      
      
        Verify the limit isn't a misconfiguration
        
```
# your own nginx? limit_req settings; your own app? middleware defaults — 429s from your stack are yours to tune
```

      
    
  

  
  
    Partner pick — affiliate link
    [Recommended cloud for this setup →](https://links.ninjaops.win/go/digitalocean?subid=fix-http-429-rate-limit-diagnosis)
    Spin up a $4 droplet or managed Kubernetes and test these fixes on a clean box — new accounts get intro credit to work with.

  

  
    
## Field note

    Shared pools: some providers count by IP for anonymous traffic — a NAT office can trip limits for everyone. Log 429s with the retry window; alerting on them without the window tells you nothing actionable.

  

  
  
    
## Common questions

    
      Should I just retry 429s immediately in a loop?
      Never — immediate retries add load and often extend throttling. Honor Retry-After if present, otherwise exponential backoff with jitter, and cap total retries.

    
    
      How do I raise the limit?
      If it's a third-party API: higher-tier plans, API keys (per-token limits usually beat per-IP), or asking for a quota bump. If it's your own service: tune the limiter middleware deliberately.

    
  

  
  
    
## Related fixes

    
      - [CORS Error: "Blocked by CORS Policy" — The Server's Job, Not the Browser's](/fixes/browser-cors-error-blocked/)
      - [nginx 502 Bad Gateway: Finding the Real Culprit](/fixes/nginx-502-bad-gateway/)
      - [curl: (6) Could Not Resolve Host — The Three DNS Layers](/fixes/curl-could-not-resolve-host/)
      - ["ERR_CONNECTION_REFUSED" — Nothing Is Listening on That Port](/fixes/err-connection-refused-diagnosis/)
    
  

  
  
    
## Ship it right the first time

    Our most-documented failures, packaged as ready-to-ship starter kits: Docker, Kubernetes, and Terraform.

    [Browse the template store →](https://store.ninjaops.win/templates/)
    One-time. Yours to modify. Instant download from the [NinjaOps template store](https://store.ninjaops.win/templates/).

  

  
    
## Get new fixes by email

    One short email when new fixes and production templates drop. No spam, unsubscribe anytime.


---

**Vetted tooling for this stack** — hardened cloud hosting and monitoring picks, disclosure-first: [links.ninjaops.win/go/digitalocean?subid=gh-http-429-rate-limit-diagnosis](https://links.ninjaops.win/go/digitalocean?subid=gh-http-429-rate-limit-diagnosis)

*Full guide with all diagnostics: [ninjaops.win/fixes/http-429-rate-limit-diagnosis/](https://ninjaops.win/fixes/http-429-rate-limit-diagnosis/)*
