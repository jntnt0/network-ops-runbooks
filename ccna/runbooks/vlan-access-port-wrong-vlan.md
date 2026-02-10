# Access port in wrong VLAN

### A/B/C Tag
- Tag: A

### Symptoms
- Endpoint has link, may get DHCP, but cannot reach the expected gateway/resources.
- Endpoint lands in the wrong subnet (wrong DHCP scope) or cannot authenticate to the right services.
- You see the endpoint MAC learned, but in the wrong VLAN.

### Scope
- Devices involved: SW1 (access switch), optional SW-CORE, optional R1 or L3 switch.
- Protocols involved: VLAN access assignment, trunk carry (sanity check), DHCP (symptom amplifier).
- Blast radius: single access port (or multiple ports if a range was configured wrong).

### Topology
- Endpoint on SW1 access port, upstream trunk to SW-CORE and routing.
- Link to diagram: /diagrams/vlan-access-port-wrong-vlan.png

### Preconditions
- Access needed: console/SSH to SW1 (and SW-CORE/R1 if you verify upstream).
- Privilege level: enable + config terminal.
- Known starting assumptions:
  - You know the correct VLAN for that endpoint (example: VLAN 20 should be Users, but it’s currently in VLAN 10).
  - You know the exact interface (example: Gi0/7).

### Triage checklist
1) Confirm the port is in access mode and see which VLAN it is actually using
   - `show interfaces gi0/7 switchport`
   - Healthy: Access Mode VLAN matches expected
   - Broken: Access Mode VLAN shows a different VLAN than expected

2) Confirm VLAN membership from the VLAN view
   - `show vlan brief`
   - Healthy: port appears under the correct VLAN ID
   - Broken: port is listed under the wrong VLAN

3) Confirm where the endpoint MAC is learned
   - `show mac address-table interface gi0/7`
   - Healthy: MAC is learned in the expected VLAN
   - Broken: MAC learned under wrong VLAN

4) Confirm interface operational status
   - `show interfaces status | include Gi0/7`
   - Healthy: connected
   - Broken: err-disabled/down (different issue, fix that first)

5) If DHCP is involved, sanity check which VLAN/subnet the client is pulling from (if snooping enabled)
   - `show ip dhcp snooping binding | include Gi0/7`
   - Healthy: binding shows correct VLAN
   - Broken: binding indicates wrong VLAN

6) Quick upstream sanity (optional): trunk carries both VLANs so the symptom isn’t masked by missing VLAN
   - `show interfaces trunk`
   - Healthy: both VLANs present in allowed/active lists
   - Broken: missing VLAN on trunk can create confusing partial failures

### Fix steps
Minimal change set: put the access port in the correct VLAN and keep it explicitly in access mode.

1) Configure the access port
   - `conf t`
   - `interface gi0/7`
   - `description ACCESS-CLIENT (MOVE TO VLAN20)`
   - `switchport mode access`
   - `switchport access vlan 20`
   - `spanning-tree portfast`
   - `end`

2) Save
   - `write memory`

If you suspect the port was configured as a trunk or dynamic trunking, explicitly wipe trunking-related lines first:
- `conf t`
- `interface gi0/7`
- `no switchport trunk native vlan`
- `no switchport trunk allowed vlan`
- `no switchport mode trunk`
- `switchport mode access`
- `switchport access vlan 20`
- `end`
- `write memory`

### Verification
- Confirm switchport VLAN
  - `show interfaces gi0/7 switchport`
  - Success: Access Mode VLAN: 20, Operational Mode: static access

- Confirm VLAN membership
  - `show vlan brief | include 20`
  - Success: Gi0/7 listed under VLAN 20

- Confirm MAC learning
  - `show mac address-table interface gi0/7`
  - Success: endpoint MAC shows under VLAN 20

- Optional: confirm trunk carry (if relevant)
  - `show interfaces trunk`
  - Success: VLAN 20 active on uplink trunk

### Prevention
- Use consistent interface templates for access ports (mode access + access vlan + portfast).
- Avoid “range” commands without double-checking the target interfaces.
- Use descriptions with VLAN intent.
- Periodically audit:
  - `show vlan brief`
  - `show interfaces status`
  - `show interfaces switchport` (spot-check)

### Rollback
Revert the port back to the prior VLAN (example: VLAN 10):

- `conf t`
- `interface gi0/7`
- `switchport mode access`
- `switchport access vlan 10`
- `end`
- `write memory`

Verify after rollback:
- `show interfaces gi0/7 switchport`
- `show vlan brief | include 10`

### Evidence to collect
Save under: `/evidence/vlan-access-port-wrong-vlan/` using the repo naming convention.  [oai_citation:1‡legend-and-scope.txt](sediment://file_00000000a2ec71fd934eb7c28243c546)

- `02-before-show-interfaces-switchport-gi0-7.txt`
  - Command: `show interfaces gi0/7 switchport`
- `02-before-show-vlan-brief.txt`
  - Command: `show vlan brief`
- `02-before-show-mac-gi0-7.txt`
  - Command: `show mac address-table interface gi0/7`
- `03-before-running-config.cfg`
  - Command: `show running-config | section interface Gi0/7`
- `04-fix-commands.txt`
  - Paste the exact command block you applied
- `05-after-show-interfaces-switchport-gi0-7.txt`
  - Command: `show interfaces gi0/7 switchport`
- `05-after-show-vlan-brief.txt`
  - Command: `show vlan brief`
- `05-after-show-mac-gi0-7.txt`
  - Command: `show mac address-table interface gi0/7`
- Optional: `01-topology.png`

### Next 3 actions
1) Create a clean “wrong VLAN” failure in the lab (put the port in VLAN 10 when it should be VLAN 20) and capture the “before” artifacts.
2) Apply the minimal fix and capture the “after” artifacts plus the exact `04-fix-commands.txt`.
3) Add a short note in `06-notes.md` describing how you confirmed the wrong VLAN (MAC table + switchport view).