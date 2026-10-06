# SSL_ERROR_RX_RECORD_TOO_LONG — Talking HTTP to a TLS Port (or Vice Versa) — NinjaOps

This error means one side encrypted and the other spoke plain text. Ninety percent of cases: you connected https:// to a port serving plain HTTP, or the server block is missing ssl on.

# SSL_ERROR_RX_RECORD_TOO_LONG — Talking HTTP to a TLS Port (or Vice Versa)
  This error means one side encrypted and the other spoke plain text. Ninety percent of cases: you connected https:// to a port serving plain HTTP, or the server block is missing ssl on.

  
    
## What you'll see

    
      - curl: (35) error:0A00010B:SSL routines::SSL3_GET_RECORD:sslv3 alert / or SSL_ERROR_RX_RECORD_TOO_LONG in Firefox
      - openssl s_client shows 'wrong version number' garbage
      - Started after a port change or a new proxy hop
    
  

  
    
## Root causes

    
    
      
### Plaintext service on a TLS port

      https://host:8080 where 8080 serves plain HTTP — the TLS client reads the plaintext HTTP response as a broken TLS record. Test: curl http://host:8080/ — if that answers, it's plaintext.

    
    
      
### Server block missing 'ssl' on the listen directive

      listen 443; without ssl serves plaintext on 443. Must be: listen 443 ssl; (plus http2 if wanted). Same class of bug behind load balancers with wrong backend protocol.

    
  

  
    
## Fix it

    
      
        Confirm which protocol actually answers on that port
        
```
curl -v http://host:PORT/ 2>&1 | head -8   # plaintext answering = protocol mismatch confirmed
```

      
      
        Fix the client URL to match reality (cheapest fix)
        
```
# use http:// for a plain port, or move the service behind TLS properly
```

      
      
        Or fix the server: enable TLS on the listener (nginx)
        
```
# server { listen 443 ssl; server_name x; ssl_certificate ...fullchain.pem; ssl_certificate_key ...key.pem; }
```

      
      
        Behind a load balancer: match backend protocol to the pool setting
        
```
# LB 'SSL/HTTPS' backend → backend must speak TLS on connect; 'HTTP' backend → plaintext backend port
```

      
    
  

  
  
    Partner pick — affiliate link
    [Recommended cloud for this setup →](https://links.ninjaops.win/go/digitalocean?subid=fix-ssl-error-rx-record-too-long)
    Spin up a $4 droplet or managed Kubernetes and test these fixes on a clean box — new accounts get intro credit to work with.

  

  
    
## Field note

    'wrong version number' from openssl s_client is this same bug wearing a different message. Docker port-map confusion is a classic source: -p 443:80 maps 443 to a plaintext container port.

  

  
  
    
## Common questions

    
      Is this a certificate problem?
      No — the handshake never started. A cert error would name the certificate. This is a protocol mismatch: TLS client, plaintext server (or the reverse).

    
    
      Why did it break after adding a reverse proxy?
      The proxy now terminates TLS and forwards plaintext to the backend — but something in the chain still expects TLS. Align each hop: client→proxy TLS, proxy→backend per the pool's protocol setting.

    
  

  
  
    
## Related fixes

    
      - [SSL/TLS Handshake Failures: ssl_error_handshake_failure and Friends](/fixes/ssl-handshake-failed/)
      - [NET::ERR_CERT_AUTHORITY_INVALID — Self-Signed or Missing Chain](/fixes/err-cert-authority-invalid/)
      - [ERR_SSL_VERSION_OR_CIPHER_MISMATCH — No Overlap Between Client and Server](/fixes/err-ssl-version-or-cipher-mismatch/)
      - [curl Error 60: SSL Certificate Problem — Unable to Get Local Issuer Certificate](/fixes/curl-60-unable-to-get-issuer-cert/)
    
  

  
  
    
## Ship it right the first time

    Our most-documented failures, packaged as ready-to-ship starter kits: Docker, Kubernetes, and Terraform.

    [Browse the template store →](https://store.ninjaops.win/templates/)
    One-time. Yours to modify. Instant download from the [NinjaOps template store](https://store.ninjaops.win/templates/).

  

  
    
## Get new fixes by email

    One short email when new fixes and production templates drop. No spam, unsubscribe anytime.


---

**Vetted tooling for this stack** — hardened cloud hosting and monitoring picks, disclosure-first: [links.ninjaops.win/go/digitalocean?subid=gh-ssl-error-rx-record-too-long](https://links.ninjaops.win/go/digitalocean?subid=gh-ssl-error-rx-record-too-long)

*Full guide with all diagnostics: [ninjaops.win/fixes/ssl-error-rx-record-too-long/](https://ninjaops.win/fixes/ssl-error-rx-record-too-long/)*
