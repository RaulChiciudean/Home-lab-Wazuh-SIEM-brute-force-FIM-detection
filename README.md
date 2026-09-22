# Home SIEM Lab — Wazuh

Lab de detecție și răspuns la incidente, construit folosind Wazuh (SIEM open-source), VirtualBox și Kali Linux.

## Arhitectură

- **Server Wazuh** (Ubuntu Server, VM) — manager + indexer + dashboard, instalare all-in-one
- **Agent monitorizat** (Ubuntu Server, VM separat) — endpoint monitorizat de server
- **Kali Linux** (VM) — mașină de atac, folosită pentru simulări
- Toate VM-urile conectate printr-o rețea VirtualBox NAT Network privată

## Ce am testat

| Scenariu | Rezultat | Regulă declanșată |
|---|---|---|
| Brute-force SSH (Kali → agent) | Detectat | Rule 2502, nivel 10 | 
![Alerta brute-force](./wazuh-threat-intelligence.png)
| Scanare porturi Nmap (Kali → agent) | Nedetectat — Wazuh e HIDS, nu monitorizează trafic de rețea din exterior fără o componentă NIDS suplimentară | — |
| Modificare fișier critic (/etc/passwd) | Detectat, după activare monitorizare realtime (inotify) | grup syscheck, nivel 7 |

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
