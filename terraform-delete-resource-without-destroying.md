# Terraform: Remove a Resource Without Destroying It — NinjaOps

You want Terraform to stop managing something — a database another team adopted, a resource you now manage by hand — without terraform destroy touching it.

# Terraform: Remove a Resource Without Destroying It
  You want Terraform to stop managing something — a database another team adopted, a resource you now manage by hand — without terraform destroy touching it.

  
    
## What you'll see

    
      - terraform plan wants to destroy a resource you still need
      - Reorganizing modules shows destroy+recreate on resources you cannot lose
    
  

  
    
## Root causes

    
    
      
### Resource moved out of Terraform management scope

      Removing the block from config tells Terraform it should delete the real object. That is the default — and the trap.

    
  

  
    
## Fix it

    
      
        Remove the resource from state, keeping the real object
        
```
terraform state rm aws_instance.legacy_app
```

      
      
        Then delete the block from your config and run plan
        
```
terraform plan   # should no longer mention the resource
```

      
      
        For moved/renamed blocks, use mv instead of remove+import
        
```
terraform state mv 'aws_instance.app' 'module.app.aws_instance.app'
```

      
      
        To adopt an existing resource later, import it
        
```
terraform import aws_instance.legacy_app i-0abc123
```

      
    
  

  
  
    Partner pick — affiliate link
    [Recommended cloud for this setup →](https://links.ninjaops.win/go/digitalocean?subid=fix-terraform-delete-resource-without-destroying)
    Spin up a $4 droplet or managed Kubernetes and test these fixes on a clean box — new accounts get intro credit to work with.

  

  
    
## Field note

    state rm is a state operation only — the provider never gets called, so nothing in the cloud changes. That is exactly why it is the right tool and exactly why you should back up state first.

  

  
  
    
## Common questions

    
      How do I remove one resource from state without deleting it?
      terraform state rm <resource.address> removes it from management only — the real infrastructure keeps running, now unmanaged. Next plans won't touch it.

    
    
      How do I stop Terraform from recreating a resource I edited manually?
      Import the drift: terraform import, or in newer versions update the state to match reality. Alternatively `lifecycle { ignore_changes = [...] }` for fields that legitimately change outside Terraform's control.

    
  

  
  
    
## Related fixes

    
      - [Terraform: 'Error Acquiring the State Lock'](/fixes/terraform-state-lock-stuck/)
      - [Terraform Plan Suddenly Wants to Destroy Everything](/fixes/terraform-plan-destroys-everything/)
      - [Terraform Init: 'No Available Releases Match the Given Constraints'](/fixes/terraform-init-no-available-releases/)
      - [Terraform Provider Version Mismatch Errors After Upgrade](/fixes/terraform-provider-version-mismatch/)
    
  

  
  
    
## Ship it right the first time

    An opinionated VPC module: per-AZ NAT, explicit dependencies, EKS-ready outputs.

    [Terraform AWS Foundation — $37 →](https://store.ninjaops.win/templates/terraform-aws-foundation/)
    One-time. Yours to modify. Instant download from the [NinjaOps template store](https://store.ninjaops.win/templates/).

  

  
    
## Get new fixes by email

    One short email when new fixes and production templates drop. No spam, unsubscribe anytime.


---

**Vetted tooling for this stack** — hardened cloud hosting and monitoring picks, disclosure-first: [links.ninjaops.win/go/digitalocean?subid=gh-terraform-delete-resource-without-destroying](https://links.ninjaops.win/go/digitalocean?subid=gh-terraform-delete-resource-without-destroying)

*Full guide with all diagnostics: [ninjaops.win/fixes/terraform-delete-resource-without-destroying/](https://ninjaops.win/fixes/terraform-delete-resource-without-destroying/)*
