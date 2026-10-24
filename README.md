## Summary (EN)

A home detection and incident response lab built with Wazuh (open-source SIEM/XDR), VirtualBox and Kali Linux. A Wazuh server monitors an Ubuntu Server agent over a private NAT network, while a Kali Linux VM simulates attacks.

**Scenarios tested**
- **SSH brute force (Kali → agent):** detected (rule 2502, level 10).
- **Nmap port scan (Kali → agent):** not detected. Wazuh is a host-based tool (HIDS) and does not monitor external network traffic without an additional network IDS component.
- **Critical file modification (`/etc/passwd`):** detected by File Integrity Monitoring (syscheck, level 7) after enabling real-time monitoring (inotify).

**Documentation:** a full project journal (steps, errors, fixes, lessons learned) and an incident response playbook with triage and response steps for each tested alert type.

**Key lessons:** the difference between HIDS and NIDS and what each can detect, periodic vs real-time file integrity monitoring, VirtualBox network modes (Bridged vs NAT vs NAT Network), and systematic troubleshooting.

**Tech stack:** Wazuh 4.14.7, VirtualBox, Ubuntu Server, Kali Linux, SSH.

*Romanian documentation follows below.*

# Home SIEM Lab — Wazuh

Lab de detecție și răspuns la incidente, construit folosind Wazuh (SIEM open-source), VirtualBox și Kali Linux.
![VirtualBox](./screenshots/virtual-box.png)

## Arhitectură

- **Server Wazuh** (Ubuntu Server, VM) — manager + indexer + dashboard, instalare all-in-one
- **Agent monitorizat** (Ubuntu Server, VM separat) — endpoint monitorizat de server
- **Kali Linux** (VM) — mașină de atac, folosită pentru simulări
- Toate VM-urile conectate printr-o rețea VirtualBox NAT Network privată
![Wazuh Server](./screenshots/wazuh-endpoints.png)

## Ce am testat

| Scenariu | Rezultat | Regulă declanșată |
|---|---|---|
| Brute-force SSH (Kali → agent) | Detectat | Rule 2502, nivel 10 | 
![Alerta brute-force](./screenshots/wazuh-threat-intelligence.png)
| Scanare porturi Nmap (Kali → agent) | Nedetectat — Wazuh e HIDS, nu monitorizează trafic de rețea din exterior fără o componentă NIDS suplimentară | — |
| Modificare fișier critic (/etc/passwd) | Detectat, după activare monitorizare realtime (inotify) | grup syscheck, nivel 7 |
![FIM](./screenshots/wazuh-fim.png)

## Documentație

- [Jurnal Home SIEM Lab](./jurnal%20Home%20SIEM%20lab.md) — jurnalul complet al proiectului: pași urmați, erori întâlnite, cum le-am rezolvat, ce am învățat
- [incident-response-playbook.md](./incident-response-playbook.md) — playbook de triaj și răspuns pentru fiecare tip de alertă testată

## Ce am învățat

- Configurare rețea VirtualBox (Bridged vs NAT vs NAT Network) și impactul asupra performanței
- Diferența dintre scanare periodică și monitorizare în timp real (inotify) în FIM
- Debugging sistematic: izolarea unei probleme de rețea lentă folosind top, ps aux, teste de viteză
- Diferența dintre un HIDS (Wazuh) și un NIDS — ce poate și ce nu poate detecta fiecare
- Scrierea unui playbook de răspuns la incident, bazat pe scenarii testate personal, nu pe teorie

## Tehnologii folosite

Wazuh 4.14.7, VirtualBox, Ubuntu Server, Kali Linux, SSH
