---
tags:
  - Cisco missing
  - Cisco IOS XR missing
  - FRR missing
  - Huawei VRP missing
  - Junos missing
  - Nokia SR OS missing
  - OpenBGPD missing
  - RtBrick RBFS missing
---

# Tier 1 ASNs

As a Tier 2/3 Network, you may want to filter out routes that have traversed Tier 1 ASNs at IXPs or from customers, as you probably don't want to provide transit for Tier 1 Networks or accept transit for Tier 1 Networks from your customers.

!!! danger "Caution"

    You should only apply this filter on your customers and IXP peers, not on your upstream transit providers.

Be aware that you need to manually check the prefix list as you could peer with for instance Microsoft of other parties on the list. So you need to do a quick sanity check on the AS numbers to fit your need.

=== "BIRD 2/3"
    ```
    define TRANSIT_ASNS = [
      174,                  # Cogent
      701,                  # UUNET
      1299,                 # Telia
      2914,                 # NTT Ltd.
      3257,                 # GTT Backbone
      3320,                 # Deutsche Telekom AG (DTAG)
      3356,                 # Level3
      3491,                 # PCCW
      4134,                 # Chinanet
      5511,                 # Orange opentransit
      6453,                 # Tata Communications
      6461,                 # Zayo Bandwidth
      6762,                 # Seabone / Telecom Italia
      6830,                 # Liberty Global
      7018                  # AT&T
    ];

    function reject_transit_paths()
    {
      if (bgp_path ~ TRANSIT_ASNS) then {
        # optional logging
        # print "Reject: Transit ASNs found on IXP Peering: ", net, " ", bgp_path;
        reject;
      }
    }

    protocol bgp example_ix_rs {
      ipv4 {
        import filter {
          reject_transit_paths();
          # other filters
          accept;
        };
      }
      ipv6 {
        import filter {
          reject_transit_paths();
          # other filters
          accept;
        };
      }
    }
    ```

=== "MikroTik"
    Source: [NLNOG](https://bgpfilterguide.nlnog.net/guides/no_transit_leaks/#routeros-v7)
    ```
    /routing/filter/num-list
    add list=TRANSIT_ASNS range=174 comment="Cogent" 
    add list=TRANSIT_ASNS range=701 comment="UUNET" 
    add list=TRANSIT_ASNS range=1299 comment="Telia" 
    add list=TRANSIT_ASNS range=2914 comment="NTT Ltd." 
    add list=TRANSIT_ASNS range=3257 comment="GTT Backbone" 
    add list=TRANSIT_ASNS range=3320 comment="Deutsche Telekom AG (DTAG)" 
    add list=TRANSIT_ASNS range=3356 comment="Level3" 
    add list=TRANSIT_ASNS range=3491 comment="PCCW" 
    add list=TRANSIT_ASNS range=4134 comment="Chinanet" 
    add list=TRANSIT_ASNS range=5511 comment="Orange opentransit" 
    add list=TRANSIT_ASNS range=6453 comment="Tata Communications" 
    add list=TRANSIT_ASNS range=6461 comment="Zayo Bandwidth" 
    add list=TRANSIT_ASNS range=6762 comment="Seabone / Telecom Italia" 
    add list=TRANSIT_ASNS range=6830 comment="Liberty Global"
    add list=TRANSIT_ASNS range=7018 comment="AT&T"  
    
    /routing/filter/rule
    add chain=reject_transit rule="if (bgp-as-path [[:TRANSIT_ASNS:]]){ reject }"

    add chain=CUSTOMER-IN rule="jump reject_transit"
    add chain=PEERING-OUT rule="jump reject_transit"
    ```

=== "Arista EOS"
    The same list of ASNs as in the BIRD example. The route-map needs further `permit` entries, otherwise all other routes are denied as well.
    ```
    ip as-path access-list tier1-asns permit _174_ any
    ip as-path access-list tier1-asns permit _701_ any
    ip as-path access-list tier1-asns permit _1299_ any
    ip as-path access-list tier1-asns permit _2914_ any
    ip as-path access-list tier1-asns permit _3257_ any
    ip as-path access-list tier1-asns permit _3320_ any
    ip as-path access-list tier1-asns permit _3356_ any
    ip as-path access-list tier1-asns permit _3491_ any
    ip as-path access-list tier1-asns permit _4134_ any
    ip as-path access-list tier1-asns permit _5511_ any
    ip as-path access-list tier1-asns permit _6453_ any
    ip as-path access-list tier1-asns permit _6461_ any
    ip as-path access-list tier1-asns permit _6762_ any
    ip as-path access-list tier1-asns permit _6830_ any
    ip as-path access-list tier1-asns permit _7018_ any
    !
    route-map peer-in deny 10
       match as-path tier1-asns
    !
    router bgp 64500
       address-family ipv4
          neighbor 198.51.100.1 route-map peer-in in
       address-family ipv6
          neighbor 2001:db8::1 route-map peer-in in
    ```

    See the EOS manual: [Border Gateway Protocol (BGP)](https://www.arista.com/en/um-eos/eos-border-gateway-protocol-bgp) (AS path access lists) and [ACLs and Route Maps](https://www.arista.com/en/um-eos/eos-acls-and-route-maps).

=== "VyOS"
    VyOS has two modes (operational and configuration mode). Enter configuration mode with
    `configure` to make changes. Use `commit` to apply them and `save` to keep them after reboot.

    The same list of ASNs as in the BIRD example. The route-map needs further `permit` rules, otherwise all other routes are denied as well.
    ```
    set policy as-path-list tier1-asns rule 10 action permit
    set policy as-path-list tier1-asns rule 10 regex _174_
    set policy as-path-list tier1-asns rule 20 action permit
    set policy as-path-list tier1-asns rule 20 regex _701_
    set policy as-path-list tier1-asns rule 30 action permit
    set policy as-path-list tier1-asns rule 30 regex _1299_
    set policy as-path-list tier1-asns rule 40 action permit
    set policy as-path-list tier1-asns rule 40 regex _2914_
    set policy as-path-list tier1-asns rule 50 action permit
    set policy as-path-list tier1-asns rule 50 regex _3257_
    set policy as-path-list tier1-asns rule 60 action permit
    set policy as-path-list tier1-asns rule 60 regex _3320_
    set policy as-path-list tier1-asns rule 70 action permit
    set policy as-path-list tier1-asns rule 70 regex _3356_
    set policy as-path-list tier1-asns rule 80 action permit
    set policy as-path-list tier1-asns rule 80 regex _3491_
    set policy as-path-list tier1-asns rule 90 action permit
    set policy as-path-list tier1-asns rule 90 regex _4134_
    set policy as-path-list tier1-asns rule 100 action permit
    set policy as-path-list tier1-asns rule 100 regex _5511_
    set policy as-path-list tier1-asns rule 110 action permit
    set policy as-path-list tier1-asns rule 110 regex _6453_
    set policy as-path-list tier1-asns rule 120 action permit
    set policy as-path-list tier1-asns rule 120 regex _6461_
    set policy as-path-list tier1-asns rule 130 action permit
    set policy as-path-list tier1-asns rule 130 regex _6762_
    set policy as-path-list tier1-asns rule 140 action permit
    set policy as-path-list tier1-asns rule 140 regex _6830_
    set policy as-path-list tier1-asns rule 150 action permit
    set policy as-path-list tier1-asns rule 150 regex _7018_

    set policy route-map peer-in rule 10 action deny
    set policy route-map peer-in rule 10 match as-path tier1-asns

    set protocols bgp neighbor 198.51.100.1 address-family ipv4-unicast route-map import peer-in
    set protocols bgp neighbor 2001:db8::1 address-family ipv6-unicast route-map import peer-in
    ```

    See the VyOS documentation: [AS Path List](https://docs.vyos.io/en/latest/configuration/policy/as-path-list.html), [Route Map](https://docs.vyos.io/en/latest/configuration/policy/route-map.html) and [BGP](https://docs.vyos.io/en/latest/configuration/protocols/bgp.html).
