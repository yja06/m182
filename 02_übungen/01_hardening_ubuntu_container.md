# Modul 182: Systemhärtung eines Ubuntu Docker Containers

**Auftrag:** Option A

## 1. Ausgangslage

Zuerst habe ich Docker Desktop eingerichtet, einen Ubuntu-Container erstellt und die nötigen Basis-Tools sowie den SSH-Dienst für die Übung eingerichtet.

<img width="891" height="71" alt="image" src="https://github.com/user-attachments/assets/3e32f88f-ee09-4cf1-afee-55aa1ff3c3ab" />


## 2. IST-Zustand vor der Härtung

Bevor ich irgendwelche Einstellungen angepasst habe, habe ich mir angeschaut, welche Ports offen sind, welche Prozesse im Hintergrund laufen und welche Benutzerkonten existieren:

* **Offene Ports:** Nur der TCP-Port `22` für SSH ist offen (über IPv4 `0.0.0.0:22` und IPv6 `[::]:22`). Andere Ports lauschen nicht.
* **Laufende Prozesse:** Es laufen nur drei Prozesse unter `root`: meine aktive Shell (`/bin/bash`, PID 1), der SSH-Server (`sshd`, PID 3620) und der Befehl `ps aux` selbst (PID 3622).
* **Benutzerkonten:** Neben `root` gibt es bloss vorinstallierte Systemkonten wie `daemon`, `www-data` oder `sshd`, die sich alle nicht einloggen können (`/usr/sbin/nologin`). Das Verzeichnis `/home` ist noch komplett leer, da noch kein normaler Benutzer erstellt wurde.

<img width="1003" height="93" alt="image" src="https://github.com/user-attachments/assets/31dace43-43c2-4b68-ba35-fb94255f7bf9" />

<img width="1008" height="105" alt="image" src="https://github.com/user-attachments/assets/6ec8750c-b23d-40c9-a909-d71d1c3748fb" />

<img width="1006" height="449" alt="image" src="https://github.com/user-attachments/assets/13c44613-23d2-44b8-8c80-3f23abd888f1" />


## 3. Durchgeführte Härtungsmassnahmen

Danach bin ich die acht Massnahmen aus der Aufgabe der Reihe nach durchgegangen und habe jeweils direkt kontrolliert, ob die Änderung gegriffen hat:

| Massnahme | Was habe ich gemacht? | Warum ist das sicher? | Wie habe ich es geprüft? |
| :--- | :--- | :--- | :--- |
| **1. SSH-Root-Login deaktivieren** | In der Datei `/etc/ssh/sshd_config` die Option `PermitRootLogin no` gesetzt und den SSH-Dienst neu gestartet. | Wenn man sich nicht mehr direkt als `root` über SSH anmelden kann, laufen Brute-Force-Angriffe auf das Hauptkonto ins Leere. | `grep PermitRootLogin /etc/ssh/sshd_config` gibt `PermitRootLogin no` zurück. |
| **2. Starkes Root-Passwort setzen** | Mit `passwd root` ein langes Passwort aus Gross- und Kleinbuchstaben, Zahlen und Sonderzeichen vergeben. | Ein starkes Passwort schützt den Root-Account vor lokalem Zugriff und vor unbefugtem Wechsel über `su`. | `passwd -S root` zeigt `root P ...` an (das `P` steht für ein aktives Passwort). |
| **3. Neuen Benutzer anlegen** | Mit `adduser student` einen normalen User erstellt und ihn per `usermod -aG sudo student` zur Gruppe `sudo` hinzugefügt. | So arbeitet man nicht ständig als `root`, sondern holt sich Admin-Rechte nur dann, wenn man sie wirklich braucht (Least Privilege). | `id student` bestätigt die Gruppe `27(sudo)` und `ls /home` zeigt das neue Verzeichnis `student`. |
| **4. Nur SSH laufen lassen** | Mit `ps aux` überprüft, ob neben SSH noch andere Dienste im Hintergrund aktiv sind. | Jeder zusätzliche Dienst vergrössert die Angriffsfläche. Wenn nur SSH läuft, gibt es weniger mögliche Schwachstellen. | `ps aux` listet weiterhin bloss `/bin/bash` und `sshd` auf. |
| **5. UFW installieren und aktivieren** | `ufw` installiert, mit `ufw allow ssh` den Port 22 freigegeben und versucht, die Firewall mit `ufw enable` zu aktivieren. | Eine Firewall blockiert grundsätzlich jeden eingehenden Datenverkehr und lässt nur explizit erlaubte Dienste durch. | `ufw status` meldet im Container `Permission denied` bei `iptables`, weil Docker den Zugriff auf den Kernel blockiert. |
| **6. TCP- und UDP-Ports prüfen** | Erneut `ss -tuln` ausgeführt und die Ausgabe mit dem Zustand vom Anfang verglichen. | Damit stellt man sicher, dass durch die Installationen nicht unbemerkt neue Ports geöffnet wurden. | `ss -tuln` zeigt wie zu Beginn nur Port 22 auf IPv4 und IPv6. |
| **7. IPv6 deaktivieren** | `net.ipv6.conf.all.disable_ipv6 = 1` an `/etc/sysctl.conf` angehängt und `sysctl -p` ausgeführt. | Wenn IPv6 im Setup nicht benötigt wird, schaltet man es ab, damit darüber keine unkontrollierten Verbindungen möglich sind. | `tail -n 2 /etc/sysctl.conf` zeigt den Eintrag; `sysctl -p` meldet aber `Read-only file system`, da der Kernel vom Host kommt. |
| **8. Unnötige Software entfernen** | `nano` und `net-tools` über `apt remove -y nano net-tools && apt autoremove -y` wieder entfernt. | Tools, die nicht mehr auf dem System liegen, können weder Sicherheitslücken haben noch nach einem Einbruch missbraucht werden. | `which nano net-tools` liefert keine Ausgabe mehr, beide Pakete sind weg. |

### Nachweise der Massnahmen

<img width="1007" height="158" alt="image" src="https://github.com/user-attachments/assets/be296d54-b86d-4df1-bb42-1a012da8f7b1" />

<img width="1006" height="277" alt="image" src="https://github.com/user-attachments/assets/5cc24187-d843-4aab-86a9-a277c110bfa8" />

<img width="1005" height="89" alt="image" src="https://github.com/user-attachments/assets/9f09ed9f-045d-4cf7-b3e9-d66c7d48648e" />

<img width="1008" height="137" alt="image" src="https://github.com/user-attachments/assets/fb15c5f7-a66a-471a-81bd-fe2a79f6ca3f" />

<img width="1004" height="131" alt="image" src="https://github.com/user-attachments/assets/3808867f-b933-49c6-a868-aa38b86f6b35" />

<img width="1014" height="90" alt="image" src="https://github.com/user-attachments/assets/4322c8ef-2acc-416b-9219-1dbd29ed7341" />

<img width="1008" height="166" alt="image" src="https://github.com/user-attachments/assets/1285f8bb-cb6e-45f1-93de-32b01787a77a" />

<img width="1010" height="370" alt="image" src="https://github.com/user-attachments/assets/c7281153-2c12-405f-8258-aeee4ba855a0" />


## 4. Container speichern und wieder starten

Nachdem ich mit allen Anpassungen fertig war, habe ich den Container mit `exit` verlassen und den aktuellen Zustand als neues Docker-Image gespeichert:

```bash
exit
docker commit hardeninglab hardened-ubuntu
```

Wenn ich später wieder einen Container auf Basis dieses gehärteten Images starten möchte, geht das mit folgendem Befehl:

```bash
docker run -it --name mein-container hardened-ubuntu /bin/bash
```
<img width="1006" height="176" alt="image" src="https://github.com/user-attachments/assets/b66b5321-6fea-4489-9a59-4429e0bb264b" />


## 5. Reflexion

* **Welche Dienste waren unnötig offen?** Eigentlich braucht ein Docker-Container überhaupt keinen offenen SSH-Dienst auf Port 22, weil man in der Praxis direkt über `docker exec` vom Host aus in den Container wechselt.
* **Welche Konfiguration war besonders sicherheitsrelevant?** Am wichtigsten war es, `PermitRootLogin no` zu setzen und einen normalen Benutzer (`student`) anzulegen. Dadurch kann niemand mehr direkt über das Netzwerk versuchen, das Root-Passwort zu erraten.
* **Wie unterscheidet sich ein gehärteter Container von einem «Vanilla»-Image?** Das unveränderte Basis-Image `ubuntu:22.04` ist komplett minimal gehalten und hat weder SSH noch eine Firewall oder normale Benutzer. Mein fertiges Image bringt zwar SSH mit, ist aber durch das gesperrte Root-Login, den eigenen Sudo-User, starke Passwörter und das Löschen unnötiger Pakete (`nano`, `net-tools`) deutlich besser abgesichert.
