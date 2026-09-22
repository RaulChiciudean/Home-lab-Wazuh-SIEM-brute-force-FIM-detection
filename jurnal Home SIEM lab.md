Implementare Home SIEM lab - Wazuh:

Voi documenta fiecare pas si dificultate.

1.Am instalat Ubuntu pe VirtualBox, apoi am incercat sa ma conectez cu SSH.
Problema: Am primit Connection refused din cauza ca serverul nu este pornit
Cauza: Nu am instalat pachetele pentru OpenSSH server - lipsa de pachete
Solutie: Instalare OpenSSH server -- 1.sudo apt install-server -y
									 2.sudo systemctl status ssh 
									 

2.Am rulat comanda de instalare a serverului Wazuh, instalarea s-a blocat fara progres.
Problema: Instalarea serverului Wazuh nu mai progresa.
Cauza: Conexiunea la internet era slaba
Diagnosticare: Am rulat top si ps aux ca sa vad daca instalarea mai era in proces, apoi am rulat un test pentru conexiunea la internet si am observat
		ca se descarca cu 11Kb/s.
Solutie: Schimbare Network ului din Brigde Adapter Wifi in NAT Network + Port Forwarding


3.Simulare brute-force SSH din Kali catre agent.
M-am conectat cu ssh la client, acesta m-a limitat doar la 3 incercari de conectare. Am rulat comanda repetat introducand parola gresita.

Apoi in verificat in Dashboard-ul din Wazuh: 32 auth failures, unde inital am crezut ca am 0 alerte de nivel mare.
Exista o alerta de nivel 10 cu descrierea: "syslog: User missed the password more than one time" si id-ul 2502.


4.Am testat detectarea unei scanari Nmap si am constat ca Wazuh, ca solutie HIDS (Host-based Intrusion Detection System) 
nu detecteaza activitate de scanare de retea din exterior.
Pentru asta ai nevoie de o componenta de NIDS (Network Intrusion Detection System) cum ar fi Suricate.


5.Testare File Integrity Monitoring.
Am testat daca Wazuh detecteaza modificari la /etc/passwd: am modificat fisierul, am dat restart la agent, m-am uitat in dahsboard si nu a aparut nimic
Comenzi: echo "# test FIM" | sudo tee -a /etc/passwd
			   sudo systemctl restart wazuh-agent
			   sudo sed -n '114p' /var/ossec/etc/ossec.conf   (verificare config)
			   sudo nano /var/ossec/etc/ossec.conf   (adăugat realtime="yes" pe linia <directories>)
Erori intalnite: Alerta FIM lipsea complet, Serviciul se oprea exact in timpul evenimentului trimiterii catre server (scan local reusea, dar sincronizarea era intrerupta),
				am rezolvat-o renuntand la scan-pe-restart si am activat monitorizarea realtime, care detecteaza instant, fara restart
				
**Ceva nou ce am invata**:
-FIM functioneaza pe baza ce checksum: Wazuh calculeaza o amprenta a fiecarui fisier monitorizat si o compara periodic cu una salvata local.
									   Daca difera e semn ca s-a schimbat
-exista 2 moduri de verificare: una periodica si una realtime