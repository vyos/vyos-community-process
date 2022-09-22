# VyOS 1.4 Firewall Redesign

## Introduction

### Interface-based model is flawed

The current firewall syntax was designed in the early days of Vyatta, seemingly with a goal to mimic Cisco IOS:
every network interface in the config has subcommands for assigning firewall rulesets for different directions (in, out, local).

There are multiple problems with that approach:

First, It isn’t how netfilter works, so configuration scripts need to generate intentionally unidiomatic rules (as in, hardly any Linux user would write firewall rules that way).

Second, that model breaks down when it comes to interfaces created dynamically or on demand. Cisco solves that problem by allowing the user to create "virtual templates"
for dynamically created interfaces, but we certainly don’t want to do that — it’s both ugly and hard to implement.

Third, that model also doesn’t fit netfilter’s “two-axis” model where directions (in, out, forward) and tables/hooks (filter, mangle, bridge…) are independent.

### Interface-based model is already violated in the current CLI

Even in Vyatta times, people created commands outside of the interface-based model to work around its limitations:

```
set firewall ping/all-ping/broadcast-ping
set firewall state-policy
set interfaces … adjust-mss
```

Does anyone need any more proof that the interface-based model is flawed and needs to be replaced completely? ;)

But it gets worse.

### `set policy route` is a misnomer

Until Vyatta 6.4, there were “firewall name” and “firewall modify” subtrees. The architects at Vyatta inc. were likely aware how weird it was starting to sound and decided to rename “set firewall modify” to “set policy route”.
That made things much worse:

* The subtree that is allegedly about routing policy includes commands for TCP MSS clamping and setting netfilter marks — features that have nothing to do with routing whatsoever.
* The fact that it’s named “policy route” discourages adding support for more packet mangling features.
* Users need to learn that part of the functionality is in a completely different subtree.

## Goals

* Get rid of the leaky abstraction of the interface-based firewall.
* Allow users to express a full range of filtering criteria (now you can’t say "any traffic to destination port 80", for example — you have to assign a ruleset with that rule to every network interface where you want it to apply).
* Open up a path to supporting all netfilter features in our CLI.
* Merge different subsystems that duplicate each other (“firewall” and “policy route”).
* Preserve the existing zone-based firewall syntax as much as possible.

## Configuration commands

```
firewall
  ruleset
      rule $number
        <everything found under "firewall name" and "policy route" rules now>
        inbound-interface <intf | interface group>
        outbound-interface <intf | interface group>
        action
          <existing actions: drop, reject, accept>
          return
          call $ruleset
  group
    # Existing group types
    address-group
    port-group
    # New group types
    interface-group
  policy
    ipv4
      filter
        in $ruleset
        forward $ruleset
        out $ruleset
      modify
        forward $ruleset
        out $ruleset
    ipv6
      ...
    bridge
      filter
        forward $ruleset
```

One interesting question is whether we should use separate subtrees for different kinds of rulesets.
Many actions are only available in specific hooks, such as settings marks and table that only works in mangle rules.

Another question is how to reconcile the use of rulesets in the zone-based firewall with new inbound/outbound interface options.
The simplest approach is to disallow referencing such rulesets in zones.

Another question is whether we want to use direction names from netfilter (in/out/forward) instead of Vyatta inventions.

## Migration

* Rename `firewall name` to `firewall ruleset`.
* Create empty rulesets `IPv4-Filter-In`, `IPv4-Filter-Out`
* For every interface, collect firewall statements from them and convert to rules with `action call $origRuleset` and `inbound/outbound-interface` options.
* Assign those rulesets to appropriate hooks and directions (`firewall policy ipv4 filter in` etc.).

## Operational commands

`* firewall name/ipv6 name` command must be renamed to "ruleset" instead.

## Implementation considerations

Zone-based firewall scripts will need to be adjusted for the new config paths.
