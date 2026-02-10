# Evidence README (CCNA Ops Labs)

### A/B/C Tag
- Tag: A

### Symptoms
- Not applicable. This file defines how evidence must be captured and stored.

### Scope
- Repo areas involved: `/evidence`, `/runbooks`, optional `/diagrams`.
- Evidence types: show command outputs, config snippets, fix command blocks, optional topology images.
- Blast radius: repo-wide consistency and auditability.

### Topology
- Not applicable.
- Link to diagram: /diagrams/repo-evidence-layout.png (optional)

### Preconditions
- You can run Cisco IOS CLI show commands in your lab.
- You can copy terminal output into text files without screenshots.
- You will redact any real identifiers before commit.

### Triage checklist
1) Every scenario has its own folder: `/evidence/<scenario-slug>/`
2) “Before” evidence exists (at least 2 show outputs + interface config snippet)
3) “Fix” evidence exists (`04-fix-commands.txt`)
4) “After” evidence exists (at least 2 show outputs proving resolution)
5) Optional: topology image exists and matches the scenario slug
6) Filenames follow the numeric ordering and stay stable over time
7) Outputs are plain text (no screenshots unless truly needed)
8) Any redactions are stated in `06-notes.md` (what was redacted, not the secret value)

### Fix steps
Create folders and capture artifacts using this required naming convention.  [oai_citation:3‡legend-and-scope.txt](sediment://file_00000000a2ec71fd934eb7c28243c546)

Required folder structure:
- `/evidence/<scenario-slug>/`
  - `01-topology.png` (optional)
  - `02-before-show-<command>.txt`
  - `03-before-running-config.cfg` (or `.txt`)
  - `04-fix-commands.txt`
  - `05-after-show-<command>.txt`
  - `06-notes.md` (optional)

Notes on `<command>` in filenames:
- Use a readable, filesystem-safe version of the command.
- Example:
  - Command: `show interfaces trunk`
  - File: `02-before-show-interfaces-trunk.txt`

Minimum required artifacts for every A-level runbook:
- At least two “before” show outputs that prove the failure
- One config snippet that shows the relevant interface(s) or VLAN section
- One fix command block with exactly what you typed
- At least two “after” show outputs that prove resolution

### Verification
For any scenario folder, you should be able to open the folder and answer, in order:
1) What was the topology? (optional image)
2) What did the device show before the fix? (02-before-*.txt)
3) What was the relevant config before? (03-before-*)
4) What exact commands were applied? (04-fix-commands.txt)
5) What did the device show after? (05-after-*.txt)
6) Any special notes, caveats, or redactions? (06-notes.md)

### Prevention
- Prefer text evidence over screenshots, always.
- Keep runbooks and evidence aligned by using the same scenario slug:
  - Runbook: `/runbooks/<scenario-slug>.md`
  - Evidence: `/evidence/<scenario-slug>/`
- Standardize device names in notes (SW1, SW2, R1) so evidence is readable later.
- If a platform lacks a command, document the substitute in `06-notes.md` and keep moving.

### Rollback
If a commit includes sensitive info:
1) Remove it from the files in the evidence folder.
2) Add a short line in `06-notes.md` stating what was redacted.
3) If it was already pushed, rewrite history using your preferred git method and rotate the secret if applicable.

### Evidence to collect
This file describes evidence collection itself. For each scenario, follow the required folder structure and minimum artifacts above.  [oai_citation:4‡legend-and-scope.txt](sediment://file_00000000a2ec71fd934eb7c28243c546)

### Next 3 actions
1) Create `/evidence/<scenario-slug>/` folders for the three VLAN scenarios and add placeholder empty files with correct names.
2) Run each scenario once in the lab and populate the placeholders with real CLI output and config snippets.
3) Add a simple repo-wide diagram (optional) showing how `/runbooks`, `/evidence`, and `/diagrams` connect.