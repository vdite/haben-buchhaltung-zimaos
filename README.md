# Haben: Kostenlose Buchhaltungssoftware auf ZimaOS installieren

Fertige ZimaOS-YAML (Docker Compose) für **[Haben](https://github.com/Speicherstadt/haben)**, die kostenlose Open-Source-Buchhaltung für Freiberufler und Kleinunternehmer. Datei herunterladen, Platzhalter ersetzen, in ZimaOS als eigene App importieren. Ein tägliches, verschlüsseltes Backup aufs NAS ist schon eingebaut.

**Ausführliche Anleitung mit YAML-Generator:** [Haben: Kostenlose Buchhaltungssoftware auf ZimaOS installieren](https://mizine.de/homeserver/haben-buchhaltung-zimaos-installieren/). Der Generator trägt deine IP, deinen NAS-Pfad und deine Schlüssel direkt ein, ohne Suchen und Ersetzen.

> **Haben ist Beta.** Nutze es zum Testen und verlass dich noch nicht allein darauf für deine echte Buchhaltung.

## Was ist Haben?

Haben ist eine selbst gehostete Buchhaltungssoftware und ein Rechnungsprogramm. Es läuft kostenlos auf deinem eigenen Server statt in einer Cloud mit Monatsabo. Haben kann:

- **E-Rechnungen erstellen** (ZUGFeRD, XRechnung) und empfangen
- **Belege** erfassen, auf Wunsch mit KI-Auslesung
- **Bankabgleich**: Umsätze den Rechnungen und Belegen zuordnen
- **EÜR oder Bilanz** und **Steuererklärungen** direkt an **ELSTER** senden
- **DATEV-Export** für die Steuerberatung

Haben ist eine Open-Source-Alternative zu Lexware Office (lexoffice), sevDesk und Accountable. Entwickelt wird es von Tristan Teufel ([firsttris](https://github.com/firsttris)) und einer wachsenden Community unter der GitHub-Organisation [Speicherstadt](https://github.com/Speicherstadt), lizenziert unter der AGPL-3.0.

**Wichtig:** Das hier ist eine **inoffizielle Installationshilfe** für ZimaOS und kein Teil des Haben-Projekts. Doku, Fehlermeldungen und Fragen zu Haben selbst gehören ins [Original-Repo](https://github.com/Speicherstadt/haben).

## Was drin ist

| Container | Aufgabe |
|---|---|
| `haben-init-perms` | setzt einmalig die Rechte der AppData-Ordner und beendet sich |
| `haben-db` | PostgreSQL 18 |
| `haben-app` | die Weboberfläche auf Port 8690 |
| `haben-backup` | tägliches, verschlüsseltes Backup (restic) von Datenbank und Belegen aufs NAS |

Die Daten liegen unter `/DATA/AppData/haben`.

## Installation

### 1. Schlüssel erzeugen

Im Terminal (Mac, Linux oder per SSH auf ZimaOS):

```bash
openssl rand -hex 24      # DB-PASSWORT
openssl rand -base64 32   # AUTH-SECRET
openssl rand -base64 32   # ENCRYPTION-KEY
openssl rand -base64 32   # BACKUP-PASSWORT
```

Speichere **ENCRYPTION-KEY** und **BACKUP-PASSWORT** in deinem Passwortmanager. Ohne sie kannst du ein Backup nicht wiederherstellen.

### 2. Platzhalter ersetzen

Lade [`zimaos-docker-compose.yaml`](zimaos-docker-compose.yaml) herunter und ersetze alles in eckigen Klammern:

| Platzhalter | Wert |
|---|---|
| `[DEINE-IP]` | IP deines ZimaOS-Rechners, z. B. `192.168.178.50` |
| `[DB-PASSWORT]` | kommt **dreimal** vor, überall derselbe Wert |
| `[AUTH-SECRET]`, `[ENCRYPTION-KEY]`, `[BACKUP-PASSWORT]` | deine Werte aus Schritt 1 |
| `[SETUP-CODE]` | ein Einrichtungscode deiner Wahl, mindestens 8 Zeichen |
| `/DEIN/NAS/ORDNER` | Backup-Ordner auf dem NAS, z. B. `/media/192.168.178.100/Backup/Haben` |

Ohne Backup lässt du den Block `backup:` einfach weg.

### 3. Backup-Ziel markieren

Lege im NAS-Ordner eine leere Datei `HABEN-BACKUP-ZIEL.txt` an. Das Backup läuft nur, wenn es diese Datei findet. So landet es nie auf der lokalen Platte, wenn das NAS gerade nicht eingehängt ist.

### 4. In ZimaOS importieren

App-Store, dann **Eigene App** und **Importieren**. Ab ZimaOS 1.8.0 kannst du die Datei direkt hochladen.

### 5. Ersteinrichtung

Öffne `http://[DEINE-IP]:8690/setup` und gib deinen `[SETUP-CODE]` ein. Danach legst du dein Konto an.

## Update

Die YAML nutzt `:latest`, du musst sie für ein Update also nicht ändern.

Zeigt ZimaOS bei der App ein Update an, klickst du darauf. Oder du gehst bei der App-Kachel über die **drei Punkte** auf **Verwalten** und dann auf **Speichern**. In beiden Fällen holt ZimaOS die neuesten Images und startet Haben neu.

Datenbank-Migrationen laufen beim Start automatisch und lassen sich nicht zurückspielen. Wer vorher noch ein Backup will, startet es mit `docker exec haben-backup backup.sh`.

Wenn du eine bestimmte Version festhalten willst, ersetze `latest` an allen drei Stellen durch dieselbe Versionsnummer, z. B. `0.12.0`.

### Umzug zu Speicherstadt (ab Version 0.12)

Haben ist im November 2026 in die GitHub-Organisation [Speicherstadt](https://github.com/Speicherstadt) umgezogen. Die Images heißen jetzt `ghcr.io/speicherstadt/haben`, `haben-db` und `haben-backup`. Unter `ghcr.io/firsttris/...` bleibt Version 0.11 stehen.

Hast du eine ältere Fassung dieser YAML im Einsatz, ersetze einmal in den drei `image:`-Zeilen `firsttris` durch `speicherstadt` (drei Punkte, **Verwalten**, Reiter **YAML**, **Speichern**). Deine Daten bleiben erhalten.

## Änderungen in ZimaOS

Ändere die App später über die **drei Punkte**, dann **Verwalten** und den Reiter **YAML**, statt sie neu zu importieren. Ein zweiter Import legt eine neue App an, und die Containernamen kollidieren.

## Hinweise

- **Healthcheck ohne `$`:** ZimaOS ersetzt `$$variable` beim Import durch einen leeren String. Darum steht der Pfad im Healthcheck von `haben-backup` direkt drin.
- **Images von GHCR:** Der Docker-Hub-Mirror von ZimaOS liefert nicht überall zuverlässig. Deshalb kommen die Images von `ghcr.io`.
- **Backup-Befehle:**
  - Backup jetzt: `docker exec haben-backup backup.sh`
  - Stände anzeigen: `docker exec haben-backup restic snapshots`
  - Test-Restore: `docker exec haben-backup restore-check.sh`
  - Den Status siehst du auch in Haben unter **Einstellungen > Export**.

## Lizenz

Diese YAML und Anleitung: MIT. Haben selbst steht unter der [AGPL-3.0](https://github.com/Speicherstadt/haben/blob/main/LICENSE).
