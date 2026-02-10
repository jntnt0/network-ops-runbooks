# VLAN missing or misconfigured on access port

### A/B/C Tag
- Tag: A

### Symptoms
- A single endpoint on an access port cannot reach anything outside its own NIC (often no DHCP lease, or gets an APIPA address).
- Other devices on the same switch may work fine.
- Ping to default gateway fails from the endpoint, or the switch shows no MAC learned where you expect.

### Scope
- Devices involved: SW1 (access switch), optional upstream SW-CORE, optional R1 (router-on-a-stick) or L3 SVI.
- Protocols involved: VLAN, access switching, STP (sanity check only).
- Blast radius: single access port (or multiple ports if VLAN not present on the switch).

### Topology
- One access switch (SW1) uplinked to distribution/core (SW-CORE) or to a router/L3 switch doing inter-VLAN routing.
- Link to diagram: /diagrams/vlan-access-port-vlan-missing.png

### Preconditions
- Access needed: console or SSH to SW1 (and SW-CORE/R1 if needed).
- Privilege level: enable + config terminal.
- Known starting assumptions:
  - You know the correct VLAN ID and purpose (example: VLAN 20 = Users).
  - You know the access interface (example: Gi0/3).
  - Uplink trunk is expected to carry that VLAN end-to-end.

### Triage checklist
1) Identify the port and current switchport mode/VLAN
   - `show interfaces status | include Gi0/3`
   - `show interfaces gi0/3 switchport`
   - Healthy: Administrative Mode: static access (or access), Access Mode VLAN: <expected>
   - Broken: VLAN shows 1 or unexpected, or mode is dynamic/trunk, or VLAN name is "inactive"

2) Confirm the VLAN exists locally and is active
   - `show vlan brief | include 20`
   - Healthy: VLAN 20 listed and active
   - Broken: VLAN 20 missing from output (not created) or shows as unsupported/inactive state

3) Confirm MAC learning on the port
   - `show mac address-table interface gi0/3`
   - Healthy: endpoint MAC learned in the expected VLAN
   - Broken: no MAC learned, or MAC learned in VLAN 1/other VLAN

4) Check if the interface is err-disabled or down
   - `show interfaces gi0/3`
   - `show interfaces status | include Gi0/3`
   - Healthy: connected, line protocol up
   - Broken: err-disabled, notconnect, or down

5) If the VLAN exists, ensure the uplink trunk carries it (quick sanity)
   - `show interfaces trunk`
   - Healthy: VLAN 20 appears in allowed and active lists on the uplink
   - Broken: VLAN 20 not allowed, or not active on trunk

6) If DHCP is expected, confirm client behavior indirectly (optional but useful)
   - `show ip dhcp snooping binding` (only if snooping is enabled)
   - Healthy: binding shows the client on the expected port/VLAN
   - Broken: no binding for the client, or binding shows a different VLAN/port

### Fix steps
Minimal change set: create the VLAN (if missing) and set the access port correctly.

1) Create the VLAN (if missing) and name it
   - `conf t`
   - `vlan 20`
   - `name USERS`
   - `end`

2) Configure the access port explicitly
   - `conf t`
   - `interface gi0/3`
   - `description ACCESS-CLIENT (VLAN20)`
   - `switchport mode access`
   - `switchport access vlan 20`
   - `spanning-tree portfast`
   - `end`

3) Save
   - `write memory`

### Verification
Run these on SW1:

- Confirm VLAN exists and is active
  - `show vlan brief | include 20`
  - Success: VLAN 20 listed as active, ports include Gi0/3

- Confirm switchport assignment
  - `show interfaces gi0/3 switchport`
  - Success: Operational Mode: static access, Access Mode VLAN: 20

- Confirm MAC learning
  - `show mac address-table interface gi0/3`
  - Success: client MAC shows up under VLAN 20

- If uplink involved, confirm trunk carries VLAN
  - `show interfaces trunk`
  - Success: VLAN 20 allowed and active on the trunk

### Prevention
- Create and standardize VLANs on all switches before assigning access ports (don’t rely on VTP in CCNA labs unless you’re explicitly practicing it).
- Avoid dynamic trunking on user-facing ports. Use `switchport mode access` everywhere it belongs.
- Add simple guardrails:
  - `switchport nonegotiate` on trunks where appropriate (paired with explicit trunk config).
  - Descriptions on access ports to reduce fat-finger mistakes.
- Capture a baseline: keep a known-good `show vlan brief` and `show interfaces trunk` snapshot per lab topology.

### Rollback
If you need to revert only the access port change:

- `conf t`
- `interface gi0/3`
- `no switchport access vlan 20`
- `no spanning-tree portfast`
- `end`
- `write memory`

If the VLAN was created only for this test and must be removed (be careful: removing a VLAN affects any ports using it):
- `conf t`
- `no vlan 20`
- `end`
- `write memory`

Verify after rollback:
- `show interfaces gi0/3 switchport`
- `show vlan brief | include 20`

### Evidence to collect
Save under: `/evidence/vlan-access-port-vlan-missing/` using the repo naming convention.  [oai_citation:0‡legend-and-scope.txt](sediment://file_00000000a2ec71fd934eb7c28243c546)

- `02-before-show-interfaces-switchport-gi0-3.txt`
  - Command: `show interfaces gi0/3 switchport`
- `02-before-show-vlan-brief.txt`
  - Command: `show vlan brief`
- `02-before-show-mac-gi0-3.txt`
  - Command: `show mac address-table interface gi0/3`
- `03-before-running-config.cfg`
  - Command: `show running-config | section interface Gi0/3`
- `04-fix-commands.txt`
  - Paste the exact command block you applied
- `05-after-show-interfaces-switchport-gi0-3.txt`
  - Command: `show interfaces gi0/3 switchport`
- `05-after-show-vlan-brief.txt`
  - Command: `show vlan brief`
- `05-after-show-mac-gi0-3.txt`
  - Command: `show mac address-table interface gi0/3`
- Optional: `01-topology.png`
  - Update or export topology diagram

### Next 3 actions
1) Build the failure on purpose in your lab (remove VLAN 20, keep port set to VLAN 20) and capture the “before” artifacts.
2) Apply the minimal fix and capture the “after” artifacts plus `04-fix-commands.txt`.
3) Add the diagram at `/diagrams/vlan-access-port-vlan-missing.png` and cross-check trunk carry end-to-end with `show interfaces trunk`.