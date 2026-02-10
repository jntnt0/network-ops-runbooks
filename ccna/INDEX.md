# Index (CCNA Ops Labs)

Legend:
- A = Fully demonstrable in lab tools with configs + show outputs saved as text
- B = Higher-fidelity multi-device labs, more realistic behavior
- C = Documentation/story pack only unless you have enterprise gear

Start here:
- /templates/legend-and-scope.md
- /runbooks/README.md
- /templates/runbook-template.md
- /templates/story-pack-template.md

## Runbooks

### Switching
- VLANs and trunks
  - /runbooks/vlan-access-port-wrong-vlan.md
  - /runbooks/vlan-trunk-not-forming.md
  - /runbooks/vlan-allowed-list-missing-vlan.md
  - /runbooks/native-vlan-mismatch.md
- STP
  - /runbooks/stp-root-misplacement.md
  - /runbooks/stp-port-roles-and-selection.md
  - /runbooks/stp-bpdu-guard-errdisable-recovery.md
- EtherChannel
  - /runbooks/etherchannel-lacp-not-forming.md
- Switch security
  - /runbooks/port-security-violation-recovery.md
  - /runbooks/dhcp-snooping-trust-boundary-breakage.md
  - /runbooks/dai-breakage-and-fix.md

### Routing
- Static and default routing
  - /runbooks/static-route-next-hop-wrong.md
  - /runbooks/default-route-missing-blackhole.md
- OSPF
  - /runbooks/ospf-neighbor-area-mismatch.md
  - /runbooks/ospf-hello-dead-mismatch.md
  - /runbooks/ospf-mtu-mismatch-exstart.md
- Inter-VLAN routing
  - /runbooks/router-on-a-stick-missing-encapsulation.md
  - /runbooks/svi-down-down-troubleshooting.md

### Services
- DHCP
  - /runbooks/dhcp-scope-wrong-network.md
  - /runbooks/dhcp-relay-missing.md
  - /runbooks/dhcp-scope-exhaustion.md
- NAT/PAT
  - /runbooks/nat-overload-acl-mismatch.md
  - /runbooks/nat-inside-outside-misassigned.md
- NTP/DNS basics
  - /runbooks/ntp-not-synchronized.md
  - /runbooks/dns-resolution-troubleshooting-basics.md

### Security
- ACLs
  - /runbooks/standard-acl-wrong-placement.md
  - /runbooks/extended-acl-implicit-deny-breakage.md
  - /runbooks/acl-direction-in-out-mistake.md
- Device hardening
  - /runbooks/ssh-not-working-missing-keys-vty-config.md

### Wireless (concept)
- /runbooks/wireless-troubleshooting-flow.md

## Evidence

Evidence folders are per scenario:
- /evidence/<scenario-slug>/
  - before outputs (.txt)
  - configs (.cfg/.txt)
  - fix commands (fix-commands.txt)
  - after outputs (.txt)
  - topology image (.png)
  - optional notes.md

## Diagrams
- /diagrams/

## Configs
- /configs/

## Story packs
- /story-packs/

## Templates
- /templates/legend-and-scope.md
- /templates/runbook-template.md
- /templates/story-pack-template.md