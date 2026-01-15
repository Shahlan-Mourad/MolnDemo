# Data Overview - Incidenthanteringslösning

[cite_start]Detta dokument beskriver hur data hanteras, klassificeras och skyddas i systemet för att uppfylla kraven på säkerhet och GDPR[cite: 11, 15].

## 1. Dataklassificering
Systemet hanterar följande kategorier av data:

* [cite_start]**Incidentdata:** Titel, beskrivning, prioritet och statuskod[cite: 31, 32].
* [cite_start]**Personuppgifter (PII):** Namn, e-post och telefonnummer till rapportören[cite: 52, 56, 70].
* [cite_start]**Bilagor:** Skärmbilder och loggfiler lagrade i Blob Storage[cite: 56, 70].
* [cite_start]**Systemloggar:** Telemetri, statuskoder och Correlation IDs[cite: 63, 77].

## 2. GDPR och Integritet
Vi följer principen om dataminimering och säker lagring enligt följande regler:

### [cite_start]Loggningsregler (Vad vi INTE loggar) [cite: 57, 71]
* [cite_start]**Ingen PII i klartext:** Namn, e-post eller telefonnummer skrivs aldrig till applikationsloggar[cite: 58, 72].
* [cite_start]**Inga hemligheter:** Access tokens, API-nycklar eller lösenord loggas aldrig[cite: 58, 72].
* [cite_start]**Inga råa request bodies:** Hela meddelandekroppar loggas inte då de kan innehålla PII[cite: 59, 73].

### [cite_start]Vad vi loggar för driftbarhet [cite: 61, 75]
* [cite_start]**Correlation ID:** För att spåra ett ärende genom hela flödet[cite: 62, 76].
* [cite_start]**Statuskoder & Endpoints:** För att se om tjänsten mår bra[cite: 63, 77].
* [cite_start]**Anonymiserade ID:n:** Vi loggar IncidentId men inte vem som skapade det[cite: 65, 79].

## [cite_start]3. Roller och Behörighet (RBAC) [cite: 14, 25]
Åtkomst till data styrs via roller för att säkerställa att endast behörig personal ser känslig information:

| Roll | Beskrivning | Behörighet |
| :--- | :--- | :--- |
| **User (Rapportör)** | Slutanvändare | [cite_start]Skapa incident, se egna ärenden[cite: 32]. |
| **Operator (IT-drift)** | Handläggare | [cite_start]Läsa alla ärenden, ändra status och prioritet[cite: 41]. |
| **Admin** | Systemadministratör | Full åtkomst, hantera användare och retention-policy. |
| **System Identity** | Managed Identity | [cite_start]Applikationens rättighet att läsa från SQL/Blob[cite: 48, 49]. |

## 4. Lagring och Retention
* [cite_start]**Databas (SQL):** Lagrar incidenter och metadata[cite: 42].
* [cite_start]**Blob Storage:** Lagrar bilagor[cite: 43].
* **Retention-policy:** * Incidenter raderas eller anonymiseras 12 månader efter avslut.
    * [cite_start]Loggar i Application Insights sparas i 90 dagar[cite: 50].
    * Tillfälliga filer rensas automatiskt via Lifecycle Management.

## 5. Säkerhetshantering
* [cite_start]**Inga hemligheter i koden:** Alla anslutningssträngar hanteras via miljövariabler eller Key Vault[cite: 44, 45, 83, 85].
* [cite_start]**Kryptering:** All data är krypterad vid lagring (Encryption at rest) och under transport (HTTPS)[cite: 27].