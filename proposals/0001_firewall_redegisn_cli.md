# VyOS Firewall cli Redesign
Main purpose
* Get rid of the necessity of particular ruleset/chains attached to interfaces.
* Generate a new cli, which contains all features that are currently supported.
* Move to a more 'netfilter' structure, since it's what is used under the hood.
* Define correct place and structure for upcoming new features, so cli can get reacher and mor flexible, without another big re-writting.
* Be able to write more optimal configuration, with less jumps (for example apply all filtering in forward hook).

```
firewall
    group [address|domain|ipv6-address|ipv6-network|mac|network|port|interface]
    state-policy
    filter
        Ipv4-filter
            name <name>
                Default-action
                Default-jump-target
                Description
                Enable-default-log
                Rule <number> …
            forward
                Default-action
                Default-jump-target
                Description
                Enable-default-log
                Rule <number> …
            input
                Default-action
                Default-jump-target
                Description
                Enable-default-log
                Rule <number> …
            output
                Default-action
                Default-jump-target
                Description
                Enable-default-log
                Rule <number> …
            Flowtables|offloads|acceleration…
        ipv6-filter
            name <name>
                Default-action
                Default-jump-target
                Description
                Enable-default-log
                Rule <number> …
            forward
                Default-action
                Default-jump-target
                Description
                Enable-default-log
                Rule <number> …
            input
                Default-action
                Default-jump-target
                Description
                Enable-default-log
                Rule <number> …
            output
                Default-action
                Default-jump-target
                Description
                Enable-default-log
                Rule <number> …
        Inet-filter|ipv4_and_ipv6_filter
            name <name>
                Default-action
                Default-jump-target
                Description
                Enable-default-log
                Rule <number> …
            forward
                Default-action
                Default-jump-target
                Description
                Enable-default-log
                Rule <number> …
            input
                Default-action
                Default-jump-target
                Description
                Enable-default-log
                Rule <number> …
            output
                Default-action
                Default-jump-target
                Description
                Enable-default-log
                Rule <number> …
        bridge-filter
            name <name>
                Default-action
                Default-jump-target
                Description
                Enable-default-log
                Rule <number> …
            forward
                Default-action
                Default-jump-target
                Description
                Enable-default-log
                Rule <number> …
            input
                Default-action
                Default-jump-target
                Description
                Enable-default-log
                Rule <number> …
            output
                Default-action
                Default-jump-target
                Description
                Enable-default-log
                Rule <number> …
            Flowtables|offloads|acceleration…
        arp-filter
            name <name>
                Default-action
                Default-jump-target
                Description
                Enable-default-log
                Rule <number> …
            input
                Default-action
                Default-jump-target
                Description
                Enable-default-log
                Rule <number> …
            output
                Default-action
                Default-jump-target
                Description
                Enable-default-log
                Rule <number> …
        Netdev|ingress
        .
        .
        .
    Policy|modify|mangle
    .
    .
    .
```

Obs:
* Continue supporting global state polciies.
* Add interface groups.
* Why words filter in 'set firewall filter [ipv4-filter|ipv6-filter|bridge-filter|...]'?
    * It gives users clear information on what to do in that section.
    * Later, we can move 'policy route' (mangling) to a new section 'set firewall policy-route' or 'set firewall mangle' or any other name.
    * Same applies for adding filtering at an earlier stage, like raw -> 'set firewall early-filtering' or 'set firewall raw-filter' or any other name.
* I forgot about zones. I'll start thinking/working on it, but I guess it would be suitable to keep it as it is under "set firewall zone"