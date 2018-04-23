## various commit SLAX scripts for Junos

### GTSM.slax

A commit script that will look for ```apply-macro GTSM``` in a BGP group and take the following action:
 * add ```multi-hop ttl``` 255 to the BGP group
 * add the neighbor address to a family-specific prefix list (v4 or v6)

This script assumes that there is a filter applied to the loopback that uses these prefixes lists to perform the filtering.
For example:
```
family inet6 {
    filter protect-RE_v6 {
        term GTSM {
            from {
                prefix-list {
                    pf_GTSM_neighbors_v6;
                }
                next-header tcp;
                port bgp;
                hop-limit-except 254;
            }
            then discard;
        }
    }
}


family inet {
    filter protect-RE_v4 {
        term GTSM {
            from {
                prefix-list {
                    pf_GTSM_neighbors_v4;
                }
                protocol tcp;
                ttl-except 254;
                port bgp;
            }
            then {
                discard;
            }
        }
    }
}
```

### requireIFdescriptions.slax

A commit script that will cause commit to fail unless eac PHY inteface that exists in the configuration has a description

If the interface has multiple units, each unit requires a description (the logic here is that a with a single unit, the description of the parent PHY will suffice)


### requireBGPpolicy.slax

A commit script to check all external BGP peers- if no policy, apply default DENY policy.  This implements the behavior
described in [RFC 8212](https://tools.ietf.org/html/rfc8212 "Default External BGP (EBGP) Route Propagation Behavior without Policies")

requires `po_DEFAULT-BGP-POLICY-DENY-ALL` to be configured
```
policy-statement po_DEFAULT-BGP-POLICY-DENY-ALL {
    term reject-everything {
        then reject;
    }
}
```


script files go in `/var/db/scripts/commit` and are configured under [edit system scripts commit]
```
system{
    scripts {
        commit {
            file GTSM.slax
            file requireIFdescriptions.slax;
            file requireBGPpolicy.slax;
		}
	}
}
```
