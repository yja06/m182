# Systemhärtung eines Ubuntu Docker Containers

**Modul 182 · System- und Netzwerksicherheit**  
**Auftrag:** Option A

---

## 1. Ausgangslage

Zuerst habe ich Docker Desktop eingerichtet, einen Ubuntu-Container erstellt und die nötigen Basis-Tools sowie den SSH-Dienst für die Übung eingerichtet.

**Erstellter Ubuntu-Container in Docker Desktop:**

```text
Name           Container ID   Image          Port(s)   CPU (%)   Last started
hardeninglab   0bd702b6fd14   ubuntu:22.04   -         N/A       34 minutes ago
```

---

## 2. IST-Zustand vor der Härtung

Bevor ich irgendwelche Einstellungen angepasst habe, habe ich mir angeschaut, welche Ports offen sind, welche Prozesse im Hintergrund laufen und welche Benutzerkonten existieren:

* **Offene Ports:** Nur der TCP-Port `22` für SSH ist offen (über IPv4 `0.0.0.0:22` und IPv6 `[::]:22`). Andere Ports lauschen nicht.
* **Laufende Prozesse:** Es laufen nur drei Prozesse unter `root`: meine aktive Shell (`/bin/bash`, PID 1), der SSH-Server (`sshd`, PID 3620) und der Befehl `ps aux` selbst (PID 3622).
* **Benutzerkonten:** Neben `root` gibt es bloss vorinstallierte Systemkonten wie `daemon`, `www-data` oder `sshd`, die sich alle nicht einloggen können (`/usr/sbin/nologin`). Das Verzeichnis `/home` ist noch komplett leer, da noch kein normaler Benutzer erstellt wurde.

### Offene Ports: TCP-Port 22 aktiv

```text
root@0bd702b6fd14:/# ss -tuln
Netid      State       Recv-Q      Send-Q           Local Address:Port            Peer Address:Port     Process
tcp        LISTEN      0           128                    0.0.0.0:22                   0.0.0.0:*
tcp        LISTEN      0           128                       [::]:22                      [::]:*
root@0bd702b6fd14:/#
```

### Laufende Prozesse: nur Bash und SSH

```text
root@0bd702b6fd14:/# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   4636  3836 pts/0    Ss   13:47   0:00 /bin/bash
root        3620  0.0  0.0  15456  9236 pts/0    S    13:48   0:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startu
root        3622  0.0  0.0   7080  3128 pts/0    R+   13:49   0:00 ps aux
root@0bd702b6fd14:/#
```

### Benutzerkonten in `/etc/passwd`: nur `root` und Systembenutzer vorhanden

```text
root@0bd702b6fd14:/# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
systemd-network:x:101:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:102:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:104::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:104:105:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
sshd:x:105:65534::/run/sshd:/usr/sbin/nologin
root@0bd702b6fd14:/#
```

---

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

---

### Nachweise der Massnahmen (Konsolenausgaben)

#### Massnahme 1: `PermitRootLogin no` in der SSH-Konfiguration

```text
root@0bd702b6fd14:/# grep PermitRootLogin /etc/ssh/sshd_config
[2] 3626
PermitRootLogin no
# the setting of "PermitRootLogin without-password".
root@0bd702b6fd14:/#
```

#### Massnahme 2: Root-Passwort gesetzt und mit `passwd -S` geprüft

```text
root@0bd702b6fd14:/# passwd root
New password:
Retype new password:
passwd: password updated successfully
[1]-  Done                    /usr/sbin/sshd -D
root@0bd702b6fd14:/# passwd -S root
root P 11/12/2026 0 99999 7 -1
root@0bd702b6fd14:/#
```

#### Massnahme 3: Benutzer `student` mit `sudo`-Rechten angelegt

```text
Is the information correct? [Y/n] Y
uid=1000(student) gid=1000(student) groups=1000(student),27(sudo)
student
root@0bd702b6fd14:/#
```

#### Massnahme 4: Prozesskontrolle, keine unnötigen Dienste aktiv

```text
root@0bd702b6fd14:/# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   5152  4324 pts/0    Ss   13:47   0:00 /bin/bash
root        3626  0.0  0.0   4636  1740 pts/0    S    13:51   0:00 /bin/bash
root        3629  0.0  0.0  15456  9200 pts/0    S    13:51   0:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startu
root        3666  0.0  0.0   7080  3120 pts/0    R+   13:55   0:00 ps aux
root@0bd702b6fd14:/#
```

#### Massnahme 5: `ufw status` scheitert an fehlenden Kernel-Rechten in Docker

```text
root@0bd702b6fd14:/# ufw status
ERROR: problem running iptables: iptables v1.8.7 (nf_tables): Could not fetch rule set generation id: Permission denied (you must be root)

root@0bd702b6fd14:/#
```

#### Massnahme 6: Port-Vergleich mit `ss -tuln` (unverändert Port 22)

```text
root@0bd702b6fd14:/# ss -tuln
Netid      State       Recv-Q      Send-Q           Local Address:Port            Peer Address:Port     Process
tcp        LISTEN      0           128                    0.0.0.0:22                   0.0.0.0:*
tcp        LISTEN      0           128                       [::]:22                      [::]:*
root@0bd702b6fd14:/#
```

#### Massnahme 7: Eintrag in `sysctl.conf` gesetzt, Kernel-Reload schreibgeschützt

```text
root@0bd702b6fd14:/# echo "net.ipv6.conf.all.disable_ipv6 = 1" >> /etc/sysctl.conf
sysctl -p
tail -n 2 /etc/sysctl.conf
sysctl: setting key "net.ipv6.conf.all.disable_ipv6", ignoring: Read-only file system

net.ipv6.conf.all.disable_ipv6 = 1
root@0bd702b6fd14:/#
```

#### Massnahme 8: `nano` und `net-tools` sauber deinstalliert

```text
root@0bd702b6fd14:/# apt remove -y nano net-tools && apt autoremove -y
which nano net-tools
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages will be REMOVED:
  nano net-tools
0 upgraded, 0 newly installed, 2 to remove and 0 not upgraded.
After this operation, 1700 kB disk space will be freed.
(Reading database ... 11088 files and directories currently installed.)
Removing nano (6.2-1ubuntu0.3) ...
Removing net-tools (1.60+git20181103.0eebece-1ubuntu5.4) ...
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
root@0bd702b6fd14:/#
```

---

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

**Konsolenausgabe beim Speichern des Images:**

```text
root@0bd702b6fd14:/# exit
docker commit hardeninglab hardened-ubuntu
docker images
exit

What's next:
  Debug this container error with Gordon -> docker ai "help me fix this container error"
PS C:\Users\admin>
```

---

## 5. Reflexion

* **Welche Dienste waren unnötig offen?**  
  Eigentlich braucht ein Docker-Container überhaupt keinen offenen SSH-Dienst auf Port 22, weil man in der Praxis direkt über `docker exec` vom Host aus in den Container wechselt.
* **Welche Konfiguration war besonders sicherheitsrelevant?**  
  Am wichtigsten war es, `PermitRootLogin no` zu setzen und einen normalen Benutzer (`student`) anzulegen. Dadurch kann niemand mehr direkt über das Netzwerk versuchen, das Root-Passwort zu erraten.
* **Wie unterscheidet sich ein gehärteter Container von einem «Vanilla»-Image?**  
  Das unveränderte Basis-Image `ubuntu:22.04` ist komplett minimal gehalten und hat weder SSH noch eine Firewall oder normale Benutzer. Mein fertiges Image bringt zwar SSH mit, ist aber durch das gesperrte Root-Login, den eigenen Sudo-User, starke Passwörter und das Löschen unnötiger Pakete (`nano`, `net-tools`) deutlich besser abgesichert.
