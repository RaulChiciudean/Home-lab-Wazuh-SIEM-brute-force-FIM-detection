# Incident Response Playbook — Home SIEM Lab (Wazuh)

> Playbook written from the scenarios tested in this lab.

---

## Playbook #1: SSH Brute Force Detected

**Rule triggered:** ID 2502, "User missed the password more than one time"
**Level:** 10 (medium-high)
**Source:** syslog / sshd, on the monitored agent

### 1. Initial triage (first 5 minutes)

- Check the source IP of the attempts: is it an internal, known address (possibly a colleague who mistyped a password) or an external/unknown one?
- Check how many attempts occurred and over what time frame (concentrated attempts are more suspicious; attempts spread over hours may be a false positive).
- Check whether a **successful** login happened right after the repeated failures. That would mean access was obtained, not just attempted.

### 2. Decision

| Situation | Action |
|---|---|
| Known internal IP, a few isolated failures | Probable false positive: document and close |
| External/unknown IP, concentrated failures, no success | Suspicious: move to containment |
| Failures followed by a successful login | **Escalate immediately**: possible active compromise |

### 3. Containment (if confirmed suspicious)

- Temporarily block the source IP at firewall level (`ufw deny from <IP>` on the agent, or a network-level rule).
- Check the agent for suspicious active SSH sessions: `who` / `last`.
- If a login succeeded: change the targeted account's password immediately and review the command history (`.bash_history`) for suspicious activity.

### 4. Documentation

- Record: source IP, targeted account, start/end time, total number of attempts, action taken.
- If it was a false positive, record the reason (useful for tuning the alert threshold in the future).

---

## Playbook #2: Critical File Modification Detected (FIM)

**Rule triggered:** `syscheck` group, `modified` event
**Example file tested:** `/etc/passwd`
**Level:** 7 (medium)

### 1. Initial triage

- Which exact file was modified? (`/etc/passwd`, `/etc/shadow`, a binary in `/usr/bin`, etc.; severity differs enormously by file.)
- Who had access at the time of the modification? Was it a planned change (system update, known admin) or an unexpected one?
- Check the exact diff (which line was added, removed or changed), not just the fact that the file changed.

### 2. Decision

| Situation | Action |
|---|---|
| Modification from a known system update (`apt upgrade`) | False positive: document |
| Unexplained modification, but low-impact file | Investigate further, not urgent |
| Unexplained modification on a critical file (`/etc/passwd`, `/etc/shadow`, system binaries) | **Escalate immediately** |

### 3. Containment (if confirmed suspicious)

- Compare the current file with a backup or known-good version (if one exists).
- Check whether an unauthorized new user appeared (especially relevant for `/etc/passwd`): `tail -5 /etc/passwd`.
- Check for suspicious active processes and unusual network connections: `ps aux`, `netstat -tulpn`.
- Isolate the system from the network if suspicion is high (temporarily bring the network interface down).

### 4. Documentation

- Record: modified file, exactly what changed (diff), time, who had access, action taken.

---

## Lesson learned in the lab

When testing FIM, I discovered that an alert may **never reach the server** if the agent service stops exactly while syncing. The modification was detected locally but never transmitted. Practical takeaway for a real analyst: **the absence of an alert does not automatically mean nothing happened**. It is worth checking at the local agent level (logs, running services), not only in the central dashboard.
