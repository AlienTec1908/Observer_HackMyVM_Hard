# Observer - HackMyVM - Easy

**Schwierigkeitsgrad:** Easy 🟢

---

## ℹ️ Maschineninformationen

*   **Plattform:** HackMyVM
*   **VM Link:** [https://hackmyvm.eu/machines/machine.php?vm=Observer](https://hackmyvm.eu/machines/machine.php?vm=Observer)
*   **Autor:** DarkSpirit

![Observer Machine Icon](Observer.png)

---

## 🏁 Übersicht

Dieser Bericht dokumentiert den Penetrationstest der virtuellen Maschine "Observer" von HackMyVM. Ziel war die Erlangung von Root-Rechten. Die Maschine wies eine kritische Path Traversal/Local File Inclusion (LFI) Schwachstelle auf einem Golang HTTP-Dienst auf Port 3333 auf, die es mir ermöglichte, die private SSH-Schlüsseldatei des Benutzers `jan` auszulesen. Nach der Erlangung des initialen Zugriffs via SSH als Benutzer `jan` nutzte ich eine Kombination aus erneuter LFI via Symlink auf das `/root`-Verzeichnis und einer unsicheren Passwort-Handhabung des Root-Benutzers in dessen Bash-History, um das Root-Passwort zu erlangen. Dies führte zur erfolgreichen Erlangung einer Root-Shell mittels `su root`.

---

## 📖 Zusammenfassung des Walkthroughs

Der Pentest gliederte sich in folgende Hauptphasen:

### 🔎 Reconnaissance

*   Identifizierung der Ziel-IP (192.168.2.213) im lokalen Netzwerk mittels `arp-scan`.
*   Hinzufügen des Hostnamens `observer.hmv` zur lokalen `/etc/hosts`.
*   Umfassender Portscan (`nmap`), der Port 22 (SSH - OpenSSH 9.2p1) und Port 3333 (HTTP - Golang net/http server) als offen identifizierte.

### 🌐 Web Enumeration

*   Untersuchung des Golang HTTP-Dienstes auf Port 3333 mittels `curl`.
*   Identifizierung der "OBSERVING FILE" Meldung und der Pfadbehandlung, die angefragte Pfade an `/home/` anhängt.
*   Fehlschläge bei Standard-Path Traversal (`../`) Versuchen.
*   Erfolgreiche Ausnutzung einer Path Traversal Schwachstelle mittels `..//` Syntax, nachgewiesen durch eine Weiterleitung auf `/etc/passwd`.
*   Entdeckung des Benutzernamens `jan` durch Brute-Forcing von Benutzernamen in Kombination mit der Dateilese-Schwachstelle auf `.bash_history` Dateien im `/home/` Verzeichnis.
*   Erfolgreiches Auslesen der privaten SSH-Schlüsseldatei `/home/jan/.ssh/id_rsa` mittels der LFI-Schwachstelle.

### 💻 Initialer Zugriff

*   Herunterladen und Vorbereiten der privaten SSH-Schlüsseldatei (`idrsa`) mit `chmod 600`.
*   Erfolgreicher SSH-Login als Benutzer `jan` unter Verwendung der erlangten `idrsa`-Datei.

### 📈 Privilege Escalation

*   System- und Benutzer-Enumeration als `jan`.
*   Entdeckung der Sudo-Regel `(ALL) NOPASSWD: /usr/bin/systemctl -l status`.
*   Auffinden des ausführbaren Golang-Binaries `/opt/observer`.
*   Transfer des `observer`-Binaries zur Offline-Analyse (indirekt über Python HTTP Server).
*   Ausnutzung der Dateilese-Schwachstelle auf Port 3333 in Kombination mit einem lokal erstellten Symlink (`ln -s /root iroot`) im `/home/jan/` Verzeichnis, um den Inhalt von `/root/.bash_history` auszulesen.
*   Analyse der Root-Bash-History und Identifizierung eines wahrscheinlich hartcodierten oder versehentlich eingegebenen Root-Passworts (`fuck1ng0bs3rv3rs`) neben einem `passwd`-Befehl.
*   Erfolgreiche Erlangung einer Root-Shell mittels `su root` und dem gefundenen Passwort. (Alternativer Weg über Sudo `systemctl` im separaten PoC beschrieben).

### 🚩 Flags

*   **User Flag:** Gefunden in `/home/jan/user.txt`
    ` HMVdDepYxsi8VSucdruB3P7 `
*   **Root Flag:** Gefunden in `/root/root.txt`
    ` HMVb6MPDxdYLLC3sxNLIOH1 `

---

## 🧠 Wichtige Erkenntnisse

*   **Path Traversal / LFI:** Implementieren Sie strenge Eingabevalidierung und Pfadnormalisierung, um die Ausnutzung von Dateipfad-Schwachstellen zu verhindern. Vermeiden Sie das Anhängen von Benutzereingaben an interne Pfade.
*   **Schutz sensibler Dateien:** Private SSH-Schlüssel (`id_rsa`) sollten immer mit Passphrase geschützt sein und niemals in öffentlich zugänglichen Verzeichnissen liegen. Überprüfen Sie Dateiberechtigungen.
*   **Sichere Passwort-Handhabung:** Vermeiden Sie die Eingabe von Passwörtern im Klartext in der Kommandozeile. Konfigurieren Sie die Shell-History (`HISTCONTROL`, `HISTIGNORE`) um die Speicherung sensibler Befehle zu verhindern. Überprüfen Sie History-Dateien auf preisgegebene Passwörter.
*   **Sudo Fehlkonfigurationen:** Vermeiden Sie `NOPASSWD` Regeln, wo immer möglich. Überprüfen Sie erlaubte Binaries gegen GTFOBins oder ähnliche Datenbanken auf Missbrauchsmöglichkeiten.
*   **Umfassende Enumeration:** Eine gründliche Enumeration des Systems nach Initial Access, einschließlich Dateisystem, Benutzer und Sudo-Berechtigungen, ist entscheidend für das Finden von Privilegien-Eskalationspfaden.

---

## 📄 Vollständiger Bericht

Eine detaillierte Schritt-für-Schritt-Anleitung, inklusive Befehlsausgaben, Analyse, Bewertung und Empfehlungen für jeden Schritt, finden Sie im vollständigen HTML-Bericht:

[**➡️ Vollständigen Pentest-Bericht hier ansehen**](https://alientec1908.github.io/Observer_HackMyVM_Easy/)

---

*Berichtsdatum: 06. Juni 2025*
*Pentest durchgeführt von Ben Chehade*
