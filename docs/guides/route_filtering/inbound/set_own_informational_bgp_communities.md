---
tags:
  - Cisco missing
  - Cisco IOS XR missing
  - FRR missing
  - Huawei VRP missing
  - Junos missing
  - Mikrotik missing
  - Nokia SR OS missing
  - OpenBGPD missing
  - RtBrick RBFS missing
---
# Set own informational communities
Informational communities propagate information about the prefixes and enable you (and others) to get insights or filter on the basis of this information. Common attributes include origin type, country/region and IXP.

=== "BIRD 2/3"
Add a community to routes imported from one session
    ```
    protocol bgp customer {
      ipv4 {
        import filter {
          bgp_community.add((<your ASN>,220));
        }
      }
      ipv6 {
        import filter {
          bgp_community.add((<your ASN>,220));
        }
      }
    }
    ```

=== "Arista EOS"
    Add a community to routes imported from one session, existing communities are kept (`additive`):
    ```
    route-map customer-in permit 10
       set community 64500:220 additive
    !
    router bgp 64500
       address-family ipv4
          neighbor 198.51.100.1 route-map customer-in in
       address-family ipv6
          neighbor 2001:db8::1 route-map customer-in in
    ```

    See the EOS manual: [ACLs and Route Maps](https://www.arista.com/en/um-eos/eos-acls-and-route-maps).

=== "VyOS"
    VyOS has two modes (operational and configuration mode). Enter configuration mode with
    `configure` to make changes. Use `commit` to apply them and `save` to keep them after reboot.

    Add a community to routes imported from one session, existing communities are kept (`add`):
    ```
    set policy route-map customer-in rule 10 action permit
    set policy route-map customer-in rule 10 set community add 64500:220
    set protocols bgp neighbor 198.51.100.1 address-family ipv4-unicast route-map import customer-in
    set protocols bgp neighbor 2001:db8::1 address-family ipv6-unicast route-map import customer-in
    ```

    See the VyOS documentation: [Route Map](https://docs.vyos.io/en/latest/configuration/policy/route-map.html) and [BGP](https://docs.vyos.io/en/latest/configuration/protocols/bgp.html).
