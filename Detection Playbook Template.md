# Detection Playbook Template

Oct 21, 2026 · @DaddyB

## How to use this template

One playbook per detection, completed by the engineer before handoff to 24/7 operations. Clone this doc, replace every `[bracketed]` placeholder, including the examples inside them, and keep section intros as guidance for the next reader. No *section is optional*.

- Write for an analyst who has never seen the detection and has a notable open in front of them.
- A section that truly doesn't apply gets one line saying why, not deletion.
- Store the playbook in the same repo as the detection and bump the version together on every change.
- Handoff is blocked until Metadata, Triage Steps, and Validation are complete.

## 1. Metadata

| Field | Value |
| --- | --- |
| Detection ID | \[DET-0000\] |
| Detection name | \[Specific behavior, e.g. "NetScaler path traversal followed by web-shell write"\] |
| ATT&CK tactic(s) | \[Initial Access (TA0001)\] |
| ATT&CK technique(s) | \[Exploit Public-Facing Application (T1190)\] — list every ID, sub-techniques included |
| Severity | \[Critical / High / Medium / Low\] |
| Response SLA | \[Time to triage, e.g. 15 min\] |
| Detection owner | \[Engineer name\] |
| Ops point of contact | \[Team or on-call rotation\] |
| Status | \[Engineering / Parallel run / Production / Retired\] |
| Version | \[1.0\] |
| Linked correlation search | \[Exact saved search name in ES\] |
| Schedule / time window | \[e.g. every 15 min, earliest=-20m\] |
| Last validated | \[YYYY-MM-DD\] |
| Related CVE / KEV | \[CVE-YYYY-NNNNN, KEV date added\] |

## 2. Summary

**What fires it:** \[One or two sentences in plain English. Example: "A request to a known exploit path on an internet-facing NetScaler returns HTTP 200, and within 10 minutes the appliance writes a new file under the web root."\]

**Why it matters:** \[What an attacker gains if this is real, e.g. unauthenticated code execution on an edge device.\]

**What a true positive usually means:** \[Attempted only / successful exploitation / post-exploitation already underway.\]

## 3. Data dependencies

If any source below goes quiet, this detection fails silently. Analysts should run the health check before closing a shift with zero alerts on a Critical detection.

| Source | Index / sourcetype | Required fields | Confirmed populated? |
| --- | --- | --- | --- |
| \[Web / proxy / WAF logs\] | \[index=... sourcetype=...\] | \[src\_ip, uri\_path, status, http\_method\] | \[Yes / No — date checked\] |
| \[EDR or Sysmon\] | \[index=... sourcetype=...\] | \[ParentImage, Image, CommandLine\] | \[Yes / No — date checked\] |

**Coverage scope:** \[Which hosts or assets are in scope, and which are known to lack telemetry.\]

**Data health check:**

- Title: \[Short summary title\]
- Search: \[Paste a search that confirms each source is reporting and the required fields are non-null over the last 24 h\]
- Expected result: \[e.g. every in-scope asset shows events within the last 60 min\]

## 4. Detection logic

**In plain English:** \[Step by step, what the search looks for. Example: 1) requests to listed exploit paths, 2) that returned 2xx, 3) correlated with a file write or child process on the same host within 10 min.\]

**Thresholds and time windows:** \[e.g. one or more matching events in 10 min; why that number was chosen.\]

**Built-in exclusions:** \[Macros or filters inside the search, and what each removes. Full rationale goes in section 9.\]

**Detection search:**

- Title: \[Short summary title\]
- Search: \[Paste the production search exactly as saved in the correlation search\]

**Notable fields passed to the analyst:** \[The fields the notable carries, e.g. src\_ip, dest, uri\_path, user. These become the tokens used in section 7.\]

## 5. Normal vs. malicious behavior

Use real, sanitized events from your environment. Bold or call out the fields that decide which side an event falls on.

### Normal (benign)

**Description:** \[e.g. Internet scanner probes the exploit path and gets 404 or 403; no follow-on activity on the host.\]

**Sample event:**

```
[Paste sanitized raw event]
```

**Deciding fields:** \[e.g. status=404, no child process from the web service, src\_ip is a known scanner ASN.\]

### Malicious

**Description:** \[e.g. Exploit path returns 200, then the web server process spawns a shell or writes a file under the web root.\]

**Sample event:**

```
[Paste sanitized raw event, from emulation or a confirmed incident]
```

**Deciding fields:** \[e.g. status=200, ParentImage=httpd, Image=sh, new .php under /var/www.\]

### Gray area

&#91;Activity that looks malicious but is often authorized, e.g. internal vulnerability scanner or a sanctioned pentest. Say how to tell them apart.\]

## 6. Triage steps

Work top to bottom. Each step ends in a decision: close as false positive, keep going, or escalate now.

1. **Confirm the alert is real data.** \[Check the raw event exists and fields parsed correctly.\]
   - Close as FP if: \[parsing error, duplicate notable, test host\]
2. **Check the source.** \[Who or what triggered it: IP reputation, asset owner, user.\]
   - Close as FP if: \[authorized scanner, sanctioned pentest with ticket #\]
   - Escalate now if: \[source on threat intel watchlist\]
3. **Check whether it succeeded.** \[Response code, bytes returned, process outcome.\]
   - Close as attempted-only if: \[all failures, no follow-on activity — document and monitor\]
4. **Look for follow-on activity.** \[Run pivots in section 7 against the target host and source.\]
   - Escalate now if: \[child process, new file, outbound connection, new account\]
5. **Scope it.** \[Same source against other assets; same behavior from other sources.\]
6. **Decide and document.** \[Disposition, evidence links, and pivots run, in the notable comments.\]

**Disposition codes:** \[True positive – successful / True positive – attempted / Benign true positive / False positive / Inconclusive\]

## 7. Pivot and investigation searches

Write each search with tokens (`$src_ip$`, `$dest$`, `$user$`, `$_time$`) that match the notable fields in section 4, so analysts paste values straight in. Mirror these in the correlation search's drill-down so the two never drift.

| # | Title | Question it answers | When to run (triage step) | Search |
| --- | --- | --- | --- | --- |
| P1 | \[Short summary title\] | \[e.g. What else did this source hit?\] | \[Step 2\] | \[Paste search\] |
| P2 | \[Short summary title\] | \[e.g. Did the target spawn processes after the request?\] | \[Step 4\] | \[Paste search\] |
| P3 | \[Short summary title\] | \[e.g. Any outbound connections from the target after the event?\] | \[Step 4\] | \[Paste search\] |
| P4 | \[Short summary title\] | \[e.g. Are other assets showing the same behavior?\] | \[Step 5\] | \[Paste search\] |

**Out-of-SIEM pivots:** \[EDR console views, RTR commands, threat intel lookups, packet capture, and where to find each.\]

## 8. Response and escalation

**Escalate immediately when:** \[e.g. confirmed successful exploitation, follow-on activity, or a crown-jewel asset involved.\]

| Condition | Escalate to | How | Within |
| --- | --- | --- | --- |
| \[Confirmed successful exploitation\] | \[IR lead / on-call\] | \[Phone, ticket queue, channel\] | \[15 min\] |
| \[Attempted only, high-value asset\] | \[Tier 2\] | \[Ticket\] | \[1 h\] |
| \[Asset owner action needed\] | \[App / infra owner\] | \[Ticket\] | \[Next business day\] |

**First containment actions (with approval authority):**

- &#91;Block source IP at edge — who can approve\]
- &#91;Isolate host in EDR — who can approve\]
- &#91;Disable affected account\]
- &#91;Preserve evidence: logs, memory, disk — before any reboot or reimage\]

**Do not:** \[Actions that destroy evidence or tip off the attacker, e.g. rebooting the appliance before capture.\]

## 9. Known false positives and tuning

Every suppression lists its reason, so ops doesn't remove it and re-create noise, or add a broader one that hides real activity.

| Date | Exclusion / change | Reason | Scope (source or dest side) | Approved by | Review by |
| --- | --- | --- | --- | --- | --- |
| \[YYYY-MM-DD\] | \[Macro or filter name\] | \[e.g. internal Nessus scanner subnet\] | \[Source\] | \[Name\] | \[YYYY-MM-DD\] |

**Recurring benign patterns not suppressed:** \[Noise analysts will see and should close quickly, and why it's left in.\]

**How to request a tuning change:** \[Ticket queue or channel, and what evidence to attach.\]

## 10. Validation

**Positive test (it fires):** \[Emulation used: Atomic test ID, PoC reference, or manual steps, run in an isolated lab.\]

**Ground-truth event:** \[Saved event or test that must always trigger the detection. Re-run after every structural change.\]

**Negative test (it stays quiet):** \[Benign activity confirmed not to fire, e.g. scanner traffic from section 5.\]

**Test host check:** \[Confirm the test host is not inside a suppression macro before testing.\]

**Parallel run:** \[Dates run alongside any existing detection and how the outputs compared.\]

| Date | Test | Result | Tester |
| --- | --- | --- | --- |
| \[YYYY-MM-DD\] | \[Positive / Negative / Regression\] | \[Fired / Quiet / Failed\] | \[Name\] |

## 11. Blind spots and known gaps

A quiet detection does not mean a clean environment. List what this detection will not see.

- **Telemetry gaps:** \[Hosts or segments without the required logs or sensor coverage.\]
- **Evasion paths:** \[Variants that bypass the logic, e.g. encoded payloads, alternate exploit paths, TLS-terminated traffic not logged.\]
- **Timing gaps:** \[Activity that falls outside the search window or arrives with ingest lag.\]
- **Complementary detections:** \[Other detection IDs that cover these gaps.\]

## 12. References and change log

**References:**

- &#91;MITRE ATT&CK technique page\]
- &#91;Vendor advisory / CVE / KEV entry\]
- &#91;Threat intel report or PoC source\]
- &#91;Related detections and playbooks\]

**Change log (newest first):**

| Date | Version | Change | Author |
| --- | --- | --- | --- |
| \[YYYY-MM-DD\] | \[1.0\] | \[Initial handoff to operations\] | \[Name\] |
