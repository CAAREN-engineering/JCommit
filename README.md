various SLAX scripts for Junos

### GTSM.slax

A commit script that will look for "apply-macro GTSM" in a BGP group and take the following action:
 * add multi-hop ttl 255 to the BGP group
 * add the neighbor address to a family-specific prefix list (v4 or v6)

### requireIFdescriptions.slax

A commit script that will cause commit to fail unless PHY inteface has a description

if the interface has multiple units, each unit requires a description (the logic here is that a with a single unit, the description of the parent PHY will suffice)


### requireBGPpolicy.slax

A commit script to check all external BGP peers- if no policy, apply default DENY

requires `po_DEFAULT-BGP-POLICY-DENY-ALL` to be configured
```
policy-statement po_DEFAULT-BGP-POLICY-DENY-ALL {
    term reject-everything {
        then reject;
    }
}

set policy-options policy-statement po_DEFAULT-BGP-POLICY-DENY-ALL term reject-everything then reject
```


script files go in `/var/db/scripts/commit` and are configured under [edit system scripts commit]
```
system{
    scripts {
        commit {
            file requireIFdescriptions.slax;
            file requireBGPpolicy.slax;
		}
	}
}
```
