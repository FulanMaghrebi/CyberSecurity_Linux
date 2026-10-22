# Nmap – die wichtigsten Befehle

Nmap untersucht erreichbare Geräte, Ports und Netzwerkdienste. Diese Übersicht dient als kompakter Spickzettel für eigene Systeme und autorisierte Tests.

> Scanne nur Systeme, die dir gehören oder für die du eine ausdrückliche Erlaubnis hast. Ersetze die Beispiel-IP `192.168.1.10` und das Beispielnetz durch deine freigegebenen Ziele.

## Voraussetzungen und Syntax

Installiere Nmap über die [offizielle Downloadseite](https://nmap.org/download.html). Unter Windows wird für viele Scanarten Npcap benötigt; das Nmap-Installationsprogramm bietet es mit an.

```bash
nmap --version
nmap [Optionen] ZIEL
```

Die Beispiele funktionieren in PowerShell sowie in üblichen Linux-/macOS-Shells. SYN-Scans, UDP-Scans und Betriebssystemerkennung benötigen üblicherweise erhöhte Rechte: unter Linux/macOS `sudo` voranstellen, unter Windows ein Terminal als Administrator öffnen.

## Schnellübersicht

| Zweck | Befehl | Erklärung |
|---|---|---|
| Standardscan | `nmap 192.168.1.10` | Prüft standardmäßig die 1.000 häufigsten TCP-Ports. |
| Geräte entdecken | `nmap -sn 192.168.1.0/24` | Führt Host-Erkennung ohne Portscan durch. |
| Weniger Ports prüfen | `nmap -F 192.168.1.10` | Prüft normalerweise die 100 häufigsten Ports. |
| Einzelne Ports | `nmap -p 22,80,443 192.168.1.10` | Prüft nur die angegebenen TCP-Ports. |
| Portbereich | `nmap -p 1-1024 192.168.1.10` | Prüft den angegebenen Bereich. |
| Alle TCP-Ports | `nmap -p- 192.168.1.10` | Prüft TCP-Ports 1–65535. |
| Häufigste Ports | `nmap --top-ports 20 192.168.1.10` | Prüft die 20 häufigsten TCP-Ports. |
| Dienste und Versionen | `nmap -sV 192.168.1.10` | Untersucht erreichbare Dienste zur Versionserkennung. |
| TCP-Connect-Scan | `nmap -sT 192.168.1.10` | Verwendet vollständige TCP-Verbindungsversuche. |
| TCP-SYN-Scan | `nmap -sS 192.168.1.10` | Prüft TCP-Ports mittels SYN-Paketen; erhöhte Rechte nötig. |
| UDP-Scan | `nmap -sU -p 53,123,161 192.168.1.10` | Prüft ausgewählte UDP-Ports; kann länger dauern. |
| Betriebssystem erkennen | `nmap -O 192.168.1.10` | Versucht, das Betriebssystem anhand von Netzwerkantworten zu bestimmen. |
| Host-Erkennung überspringen | `nmap -Pn -p 80,443 192.168.1.10` | Behandelt das Ziel als erreichbar, auch wenn Erkennungsproben unbeantwortet bleiben. |
| IPv6 | `nmap -6 ::1` | Scannt die lokale IPv6-Loopback-Adresse. |

`-Pn` umgeht keine Firewall und kann Scans deutlich verlängern. Ohne explizite Scanart verwendet Nmap bei passenden Rechten normalerweise einen SYN-Scan, sonst einen TCP-Connect-Scan.

## Mehrere Ziele

```bash
# Mehrere einzelne Geräte
nmap 192.168.1.10 192.168.1.20

# Zusammenhängender Adressbereich
nmap 192.168.1.10-20

# Geräte in einem /24-Netz entdecken
nmap -sn 192.168.1.0/24

# Ziele aus einer Textdatei einlesen
nmap -iL targets.txt

# Ein Gerät beim Scan ausschließen
nmap -sn 192.168.1.0/24 --exclude 192.168.1.1
```

In `targets.txt` kannst du beispielsweise eine IP-Adresse oder einen Hostnamen pro Zeile eintragen.

## Genauere Untersuchung

```bash
# Dienste auf ausgewählten Ports bestimmen
nmap -sV -p 22,80,443 192.168.1.10

# Standard-NSE-Skripte und Versionserkennung
nmap -sC -sV -p 22,80,443 192.168.1.10

# Betriebssystem, Dienste, Standard-Skripte und Traceroute
nmap -A 192.168.1.10

# Hilfe zu einem Skript anzeigen, ohne das Ziel zu scannen
nmap --script-help http-title

# Seitentitel eines Webdienstes ermitteln
nmap -p 80,443 --script http-title 192.168.1.10
```

`-sC` entspricht `--script=default`. Prüfe eingesetzte Skripte vorab: Einige Standard-Skripte gelten als eingreifend. `-A` aktiviert mehrere Erkennungsverfahren, scannt aber nicht automatisch alle Ports und ändert nicht automatisch das Timing.

## Ausgabe speichern und Fortschritt anzeigen

```bash
# Lesbare Textdatei
nmap -sV -oN scan.txt 192.168.1.10

# XML zur Weiterverarbeitung
nmap -sV -oX scan.xml 192.168.1.10

# Normal-, XML- und Grepable-Ausgabe gleichzeitig
nmap -sV -oA scan 192.168.1.10

# Ausführlichere Ausgabe mit Fortschrittsmeldungen
nmap -v --stats-every 10s 192.168.1.10

# Nur offene bzw. möglicherweise offene Ports anzeigen
nmap --open 192.168.1.10

# Gründe für Port- und Hostzustände anzeigen
nmap --reason 192.168.1.10
```

`-oA scan` erzeugt `scan.nmap`, `scan.xml` und `scan.gnmap` im aktuellen Verzeichnis. Für automatisierte Auswertung ist XML vorzuziehen; das Grepable-Format ist veraltet.

## Nützliche Zusatzoptionen

| Option | Wirkung |
|---|---|
| `-n` | Deaktiviert Reverse-DNS-Auflösung; vermeidet zusätzliche Namensabfragen. |
| `-T3` | Standard-Timing. |
| `-T4` | Schnelleres Timing für zuverlässige, schnelle Netze; bei Verlusten vorsichtig einsetzen. |
| `--version-light` | Weniger intensive Versionserkennung in Verbindung mit `-sV`. |
| `--host-timeout 5m` | Bricht die Untersuchung eines Hosts nach fünf Minuten ab; Ergebnisse können unvollständig sein. |
| `-h` | Zeigt die integrierte Kurzhilfe. |

## Typischer Ablauf

```bash
# 1. Erreichbare Geräte im freigegebenen Netz suchen
nmap -sn 192.168.1.0/24

# 2. TCP-Ports eines ausgewählten Geräts prüfen
nmap -p- --open -oN ports.txt 192.168.1.10

# 3. Gefundene Ports gezielt genauer untersuchen
# Beispiel: Schritt 2 hat die Ports 22, 80 und 443 gefunden.
nmap -sV -p 22,80,443 -oA services 192.168.1.10
```

## Ergebnisse verstehen

| Zustand | Bedeutung |
|---|---|
| `open` | Ein Dienst nimmt auf diesem Port Verbindungen bzw. Pakete entgegen. |
| `closed` | Der Port ist erreichbar, aber dort wurde kein lauschender Dienst erkannt. |
| `filtered` | Nmap kann wegen Filterung oder fehlender Antworten nicht eindeutig zwischen offen und geschlossen unterscheiden. |
| `open\|filtered` | Nmap kann offen und gefiltert nicht unterscheiden; häufig bei UDP. |

Ein offener Port ist kein automatischer Nachweis einer Schwachstelle. Ohne `-sV` kann die Dienstbezeichnung lediglich aus der üblichen Zuordnung zur Portnummer stammen. Auch Versions- und Betriebssystemerkennung sind nicht immer eindeutig.

## Offizielle Dokumentation

- [Nmap-Referenzhandbuch](https://nmap.org/book/man.html)
- [Deutsche Übersicht der Optionen](https://nmap.org/man/de/man-briefoptions.html)
- [NSE-Skripte und Dokumentation](https://nmap.org/nsedoc/)

