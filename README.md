# DEA-C01 Floci Hands-on Missions

Lokale, kostenfreie Lernplattform für einen Data-Engineering-Missionenplan von **M00 bis M43**. Sie nutzt AWS-kompatible APIs über Floci, eine lokale Ressourcen-UI und eine eigene Mission-Control-Oberfläche für persistenten Lernfortschritt.

> **10-Sekunden-Zusammenfassung:** Dieses Repository ist ein praktisches AWS-kompatibles Lernlabor, kein Nachweis produktiver AWS-Berufserfahrung. Lokale Emulation, reale AWS-Semantik und geplante Missionsinhalte werden bewusst getrennt dokumentiert.

## Aktueller Stand

**Stand:** 20. Juli 2026

| Bereich | Status |
|---|---|
| Plattform-Core | eingerichtet und für S3-Missionen nutzbar |
| M01 — S3 Data Lake Fundamentals | abgeschlossen; Artefakte unter [`missions/M01-s3-data-lake/`](missions/M01-s3-data-lake/) |
| M02 — S3 Lifecycle, Versioning und Resilienz | aktuelle Mission |
| Gesamtplan | M00–M43 definiert |

Mission Control bleibt die operative Fortschrittsanzeige; Repository-Artefakte dokumentieren den versionierten Lernstand.

## Was dieses Repository tatsächlich beweist

| Bereich | Evidenz |
|---|---|
| **Lokale AWS-kompatible APIs** | Floci-Endpunkt für CLI/SDK/IaC-Übungen |
| **S3-Grundlagen** | praktisch umgesetzte M01-Artefakte |
| **Reproduzierbare Lernumgebung** | Docker Compose, Skripte, Health-/Verifikationsschritte |
| **Mission Tracking** | persistente Mission-Control-Web-UI |
| **Tooling** | AWS CLI/`awslocal`, Boto3/SDK-fähiger lokaler Endpoint |
| **Infrastructure Thinking** | getrennte Compose-Profile für schwere Komponenten |
| **Testbarkeit** | Node-Contract-Tests, Compose-Validierung und PowerShell-Verifikationsskripte |
| **Evidenzgrenze** | lokale Emulation und reale AWS-Fähigkeiten werden ausdrücklich getrennt |

## Aktuelle Mission: M02

M02 erweitert den in M01 aufgebauten Bucket `northstar-data-lake` um Schutz- und Lifecycle-Mechanismen.

Schwerpunkte:

- S3 Versioning und Version IDs
- Delete Marker und Wiederherstellung
- Lifecycle Rules: Transition und Expiration
- Storage-Class-Entscheidungen
- Replication als Resilienzmechanismus, nicht als Backup-Ersatz
- Object Lock, Verschlüsselung sowie RPO/RTO
- Backup-/Restore-Runbook und Failure Injection

### Wichtige Evidenzgrenze

M02 ist als Emulationsklasse A/B eingeordnet. Versionierung und grundlegende S3-Operationen können lokal praktisch geprüft werden. Reale Storage-Class-Übergänge, Glacier-Abrufzeiten, Multi-Region-Replikation, Object-Lock-Compliance und vollständige AWS-Security-Semantik müssen separat in echtem AWS validiert werden.

Die verbindlichen Aufgaben stehen im [`DEA-C01 Floci Hands-on Missionenplan`](content/DEA-C01_Floci_Hands-on_Missionenplan.md).

## Architektur des Lernlabors

```text
Mission Control UI ───────→ Lernfortschritt
        │
        └────────────────→ persistentes Volume

AWS CLI / SDK / IaC
        ↓
AWS_ENDPOINT_URL
        ↓
Floci AWS-compatible API
        ↓
lokal emulierte Ressourcen
        ↓
Floci UI zur Inspektion
```

Schwere Zusatzkomponenten werden nur über explizite Compose-Profile gestartet.

## Schnellstart unter Windows

Voraussetzungen: Windows 10/11, Docker Desktop mit WSL2/Linux-Containern, PowerShell 7 und Git.

```powershell
Copy-Item .env.example .env
.\scripts\doctor.ps1
.\scripts\start-core.ps1
```

Danach:

- **Mission Control:** `http://localhost:3000`
- **Floci UI:** `http://localhost:4500`
- **Floci API:** `http://localhost:4566`

Erster AWS-kompatibler Aufruf:

```powershell
docker compose exec floci awslocal sts get-caller-identity
docker compose exec floci awslocal s3api list-buckets
```

Anwendungscode nutzt den konfigurierten Endpoint und keine proprietäre Floci-API:

```text
Host:        AWS_ENDPOINT_URL=http://localhost:4566
Container:   AWS_ENDPOINT_URL=http://floci:4566
Region:      eu-central-1
Credentials: test / test (nur lokal)
```

## Optionale Profile

```powershell
.\scripts\start-profile.ps1 -Profile lab
.\scripts\start-profile.ps1 -Profile spark
.\scripts\start-profile.ps1 -Profile cdc
.\scripts\start-profile.ps1 -Profile airflow
```

Verfügbare Profile: `tools`, `lab`, `spark`, `cdc`, `airflow`, `flink`, `vector`, `rag`, `bi`, `observability`.

**Wichtig:** Das Vorhandensein eines Profils ist kein Beleg dafür, dass die zugehörige Technologie bereits praktisch beherrscht oder in einer Mission abgeschlossen wurde.

## Verifikation

```powershell
node --test mission-tracker/test/missions.test.mjs tests/contracts/mission-plan.test.mjs
docker compose config --quiet
.\scripts\verify-core.ps1
.\scripts\verify-profiles.ps1
```

`verify-core.ps1` prüft unter anderem Health, UIs, AWS CLI/STS, temporären S3-Write/Read mit Cleanup und Persistenz über einen Floci-Neustart.

## Engineering-Learnings und Grenzen

- **Emulation ist nicht Produktion.** Lokale API-Kompatibilität beweist nicht automatisch identische AWS-Semantik.
- **Reality Checks müssen geplant sein.** Funktionen wie Multi-Region-Replikation oder echte Storage-Class-Übergänge brauchen spätere AWS-Validierung.
- **Lerninfrastruktur darf Lernziele nicht vortäuschen.** Ein Compose-Profil oder Service im Lab ist noch keine Kompetenz-Evidenz.
- **Reproduzierbarkeit ist Teil des Lernens.** Doctor-, Start-, Verify- und Cleanup-Skripte reduzieren Umgebungsfehler und machen Missionen wiederholbar.
- **Cleanup muss begrenzt sein.** Die Skripte verwenden keine globalen Docker-Prune-Operationen und greifen keine fremden Compose-Projekte an.
- **Progress Tracking ist kein Kompetenznachweis.** Abgehakte Missionen werden durch konkrete Missionsartefakte ergänzt.

## Projektführung

- Source of Truth: `content/DEA-C01_Floci_Hands-on_Missionenplan.md`
- Infrastrukturzuordnung: `docs/platform/mission-infrastructure-matrix.md`
- Plattformbetrieb: `docs/platform/`
- eigene Lösungen: `missions/MXX-name/`
- eigene IaC-Artefakte: `infra/`
- Tests: `tests/`

Der Core provisioniert absichtlich keine Missionsressourcen wie Buckets, Queues, Tabellen, Funktionen, Rollen oder Datenbanken. Diese entstehen beim praktischen Lernen.

## TL;DR

Dieses Repository zeigt einen strukturierten, reproduzierbaren **AWS-kompatiblen Hands-on-Lernpfad** mit klarer Trennung zwischen lokaler Emulation und echter Cloud-Evidenz. Es soll praktische Kompetenz aufbauen, aber keine produktive AWS-Erfahrung simulieren.
