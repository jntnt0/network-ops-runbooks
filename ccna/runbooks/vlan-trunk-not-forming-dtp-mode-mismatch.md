# Trunk not forming due to DTP expectations or mode mismatch

### A/B/C Tag
- Tag: A

### Symptoms
- Link is up/up but VLANs do not pass between switches.
- `show interfaces trunk` shows no trunks.
- MAC learning is isolated per-switch; devices in the same VLAN on different switches cannot communicate.
- You may see the port operating as access when you expected a trunk.

### Scope
- Devices involved: SW1 and SW2 (or SW-CORE), the inter-switch uplink interfaces.
- Protocols involved: 802.1Q trunking, DTP negotiation (where supported), VLAN propagation end-to-end, STP sanity check.
- Blast radius: can be entire site segment if the uplink is the only path.

### Topology
- Two switches connected back-to-back (example: SW1 Gi0/1 <-> SW2 Gi0/1).
- Link to diagram: /diagrams/vlan-trunk-not-forming-dtp-mode-mismatch.png

### Preconditions
- Access needed: console/SSH to both ends of the link.
- Privilege level: enable + config terminal.
- Known starting assumptions:
  - You know the uplink interfaces on both switches.
  - You know which VLANs must traverse the trunk (example: 10,20,40,50,99 native).

### Triage checklist
1) Confirm physical and line status on both ends
   - `show interfaces status | include Gi0/1`
   - Healthy: connected on both sides
   - Broken: notconnect/down (fix cabling, speed/duplex, shutdown)

2) Check trunk state (most direct signal)
   - `show interfaces trunk`
   - Healthy: interface listed as trunk with encapsulation dot1q
   - Broken: interface not listed (no trunk)

3) Check switchport mode and negotiation on both ends
   - `show interfaces gi0/1 switchport`
   - Pay attention to:
     - Administrative Mode (access, trunk, dynamic desirable, dynamic auto)
     - Operational Mode (trunk vs access)
     - Negotiation of Trunking: On/Off
   - Healthy: trunk/trunk (or desirable on one side with auto on the other, where DTP exists)
   - Broken: auto/auto, access/access, trunk negotiation disabled unexpectedly, or one side forced access

4) Look for DTP status (if supported on the platform)
   - `show dtp interface gi0/1`
   - Healthy: DTP indicates trunk negotiation success (when used)
   - Broken: DTP disabled or both ends passive (auto/auto)

5) Confirm VLANs that should pass are allowed and active (once trunk exists, this matters)
   - `show interfaces trunk`
   - Healthy: required VLANs in “allowed” and “active” lists
   - Broken: trunk up but VLAN list missing needed VLANs

6) Sanity check STP isn’t blocking the uplink unexpectedly (less common, but fast to verify)
   - `show spanning-tree interface gi0/1`
   - Healthy: forwarding
   - Broken: blocking (usually design-related, not a trunk formation issue)

### Fix steps
The traditional, reliable fix: hard-set both sides as trunks and stop relying on negotiation.

Assume:
- Native VLAN: 99
- Allowed VLANs: 10,20,40,50,99
- Uplink interfaces: SW1 Gi0/1, SW2 Gi0/1

On SW1:
- `conf t`
- `interface gi0/1`
- `description TRUNK_TO_SW2`
- `switchport trunk encapsulation dot1q` (if the command exists on your image; if it errors, skip it)
- `switchport mode trunk`
- `switchport trunk native vlan 99`
- `switchport trunk allowed vlan 10,20,40,50,99`
- `switchport nonegotiate`
- `end`
- `write memory`

On SW2:
- `conf t`
- `interface gi0/1`
- `description TRUNK_TO_SW1`
- `switchport trunk encapsulation dot1q` (if available)
- `switchport mode trunk`
- `switchport trunk native vlan 99`
- `switchport trunk allowed vlan 10,20,40,50,99`
- `switchport nonegotiate`
- `end`
- `write memory`

If you must use DTP for a lab objective, the minimal negotiated pairing is usually:
- One side: `switchport mode dynamic desirable`
- Other side: `switchport mode dynamic auto`
But for real ops discipline, forced trunk on both ends is the standard.

### Verification
Run on both switches:

- Confirm trunk is up
  - `show interfaces trunk`
  - Success: Gi0/1 listed as trunk, native VLAN correct, VLANs allowed/active

- Confirm switchport operational mode is trunk
  - `show interfaces gi0/1 switchport`
  - Success: Operational Mode: trunk

- Confirm STP is forwarding on the uplink
  - `show spanning-tree interface gi0/1`
  - Success: forwarding

Optional functional proof:
- Verify MAC learning across the trunk by generating traffic and checking:
  - `show mac address-table dynamic | include Gi0/1`

### Prevention
- Don’t rely on DTP in production-style configs. Hard-set trunks.
- Standardize trunk parameters on both ends (native VLAN, allowed VLAN list).
- Add port descriptions on uplinks.
- Keep a baseline snapshot of:
  - `show interfaces trunk`
  - `show interfaces <uplink> switchport`
  - `show spanning-tree interface <uplink>`

### Rollback
If you need to revert to a negotiated setup (lab-only), remove the hard trunking and nonegotiate.

On each switch (interface gi0/1 example):
- `conf t`
- `interface gi0/1`
- `no switchport nonegotiate`
- `no switchport trunk native vlan`
- `no switchport trunk allowed vlan`
- `switchport mode dynamic auto`
- `end`
- `write memory`

Verify rollback:
- `show interfaces gi0/1 switchport`
- `show interfaces trunk` (expect trunk may drop depending on the peer mode)

### Evidence to collect
Save under: `/evidence/vlan-trunk-not-forming-dtp-mode-mismatch/` using the repo naming convention.  [oai_citation:2‡legend-and-scope.txt](sediment://file_00000000a2ec71fd934eb7c28243c546)

- `02-before-show-interfaces-trunk.txt`
  - Command: `show interfaces trunk`
- `02-before-show-interfaces-switchport-gi0-1.txt`
  - Command: `show interfaces gi0/1 switchport`
- `02-before-show-dtp-gi0-1.txt` (if supported)
  - Command: `show dtp interface gi0/1`
- `03-before-running-config.cfg`
  - Commands:
    - `show running-config interface gi0/1`
    - (and on the other switch) `show running-config interface gi0/1`
- `04-fix-commands.txt`
  - Paste both-side command blocks, clearly labeled SW1 and SW2
- `05-after-show-interfaces-trunk.txt`
  - Command: `show interfaces trunk`
- `05-after-show-interfaces-switchport-gi0-1.txt`
  - Command: `show interfaces gi0/1 switchport`
- `05-after-show-spanning-tree-gi0-1.txt`
  - Command: `show spanning-tree interface gi0/1`
- Optional: `01-topology.png`

### Next 3 actions
1) Force a failure with auto/auto (or access on one side) and capture “before” artifacts from both switches.
2) Apply the forced-trunk fix on both ends and capture “after” artifacts plus `04-fix-commands.txt`.
3) Validate VLAN carriage by testing a single VLAN end-to-end and noting the proof commands used in `06-notes.md`.