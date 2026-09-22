# Incident Response Playbook — Home SIEM Lab (Wazuh)

> Playbook scris pe baza scenariilor testate personal în lab-ul de mai sus.
> Scop: pași clari de triaj și răspuns pentru un analist SOC L1, pentru fiecare tip de alertă întâlnit.

---

## Playbook #1 — SSH Brute-Force Detectat

**Regulă declanșată:** ID 2502 — "User missed the password more than one time"
**Nivel:** 10 (mediu-ridicat)
**Sursă:** syslog / sshd, pe agentul monitorizat

### 1. Triaj inițial (primele 5 minute)
- [ ] Verific IP-ul sursă al încercărilor — e o adresă internă (cunoscută, poate un coleg care a greșit parola) sau externă/necunoscută?
- [ ] Verific câte încercări au avut loc și în ce interval de timp (concentrate = mai suspect; răspândite pe ore = posibil fals pozitiv)
- [ ] Verific dacă a existat și o autentificare cu SUCCES imediat după eșecurile repetate — asta ar însemna acces obținut, nu doar încercare eșuată

### 2. Decizie
| Situație | Acțiune |
|---|---|
| IP intern cunoscut, câteva eșecuri izolate | Fals pozitiv probabil — documentez și închid |
| IP extern/necunoscut, eșecuri concentrate, fără succes | Suspect — trec la Containment |
| Eșecuri + autentificare reușită ulterior | **Escaladare imediată** — posibil compromis activ |

### 3. Containment (dacă e confirmat suspect)
- [ ] Blochez temporar IP-ul sursă la nivel de firewall (`ufw deny from <IP>` pe agent, sau regulă la nivel de rețea)
- [ ] Verific pe agent dacă există sesiuni SSH active suspecte: `who` / `last`
- [ ] Dacă a existat autentificare reușită: schimb imediat parola contului vizat, verific istoricul de comenzi (`.bash_history`) pentru activitate suspectă

### 4. Documentare
- [ ] Notez: IP sursă, cont vizat, oră start/final, nr. total încercări, acțiune luată
- [ ] Dacă a fost fals pozitiv, notez motivul (util pentru a ajusta pragul de alertă pe viitor)

---

## Playbook #2 — Modificare Fișier Critic Detectată (FIM)

**Regulă declanșată:** grup `syscheck`, eveniment `modified`
**Fișier de exemplu testat:** `/etc/passwd`
**Nivel:** 7 (mediu)

### 1. Triaj inițial
- [ ] Ce fișier exact s-a modificat? (`/etc/passwd`, `/etc/shadow`, un binar din `/usr/bin`, etc. — nivelul de gravitate diferă enorm după fișier)
- [ ] Cine avea acces la momentul modificării — a fost o schimbare planificată (update de sistem, admin cunoscut) sau neașteptată?
- [ ] Verific diff-ul exact (ce linie s-a adăugat/șters/modificat), nu doar faptul că s-a schimbat

### 2. Decizie
| Situație | Acțiune |
|---|---|
| Modificare din update de sistem cunoscut (`apt upgrade`) | Fals pozitiv — documentez |
| Modificare neexplicată, dar fișier cu impact redus | Investighez mai departe, nu urgentă |
| Modificare neexplicată pe fișier critic (`/etc/passwd`, `/etc/shadow`, binare de sistem) | **Escaladare imediată** |

### 3. Containment (dacă e confirmat suspect)
- [ ] Compar fișierul curent cu un backup/versiune cunoscută bună (dacă există)
- [ ] Verific dacă a apărut un utilizator nou neautorizat (relevant mai ales pentru `/etc/passwd`): `cat /etc/passwd | tail -5`
- [ ] Verific procese active suspecte și conexiuni de rețea neobișnuite: `ps aux`, `netstat -tulpn`
- [ ] Izolez sistemul de rețea dacă suspiciunea e mare (oprire temporară a interfeței de rețea)

### 4. Documentare
- [ ] Notez: fișier modificat, ce s-a schimbat exact (diff), ora, cine avea acces, acțiune luată

---

## Lecție generală învățată în lab

La testarea FIM, am descoperit că alerta poate să **nu ajungă la server** dacă serviciul agentului se oprește exact în timpul sincronizării — local, modificarea era detectată, dar niciodată transmisă. Concluzie practică pentru un analist real: **lipsa unei alerte nu înseamnă automat că nu s-a întâmplat nimic** — merită verificat și la nivel de agent local (log-uri, servicii active), nu doar în dashboard-ul central.

---

## Note pentru extindere viitoare

- [ ] Adaugă playbook pentru alertă de malware/fișier suspect (după testare EICAR + VirusTotal)
- [ ] Adaugă playbook pentru scanare de porturi (necesită integrare NIDS separată, ex. Suricata)
