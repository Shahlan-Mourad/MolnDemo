# Incident Management System – Projektöversikt

**Version:** 1.0 (MVP)  
**Senast uppdaterad:** 2025-01-12  
**Status:** Under utveckling

---

## 1. Projektmål och syfte

### 1.1 Affärsmål
Detta projekt syftar till att bygga en molnbaserad incidenthanteringslösning som:
- Möjliggör effektiv hantering av incidenter (skapa, spåra, uppdatera)
- Följer GDPR-principer och säkerhetsbestämmelser
- Är skalbar och förberedd för framtida utökningar (prioritering, notifieringar, rapportering)
- Kan deployas och drivas i Azure-miljöer med CI/CD

### 1.2 Tekniska mål
- Använda .NET 8 för backend (Web API)
- Följa enkel men tydlig lagerarkitektur (Endpoint → Service → Repository)
- Förbereda för Azure Functions (asynkrona jobb, notifieringar)
- Bygga en modern frontend (t.ex. React)
- Säkerställa hög kodkvalitet med enhetstester och integrationstester
- Automatisera bygg, test och deploy via Azure DevOps

---

## 2. Systemarkitektur (högnivå)

### 2.1 Komponentöversikt

```
┌─────────────────┐
│   Frontend      │  (React - planerat)
│   (Web UI)      │
└────────┬────────┘
         │ HTTP/REST
         ▼
┌─────────────────┐
│   Web API       │  (.NET 8 - MVP)
│   (Controllers) │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   Service       │  (Affärslogik)
│   Layer         │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   Repository    │  (Dataåtkomst - MVP: In-Memory)
│   Layer         │
└─────────────────┘

┌─────────────────┐
│ Azure Functions │  (Asynkrona jobb - planerat)
│   (Notifiering) │
└─────────────────┘
```

### 2.2 Lagerarkitektur (detaljerad)

#### Endpoint Layer (API Controllers)
- **Ansvar:** HTTP-hantering, inputvalidering, statuskoder
- **Teknologi:** ASP.NET Core Web API Controllers
- **Plats:** `Src/web/IncidentManagement.Api/Controllers/`
- **Exempel:** `IncidentController` med endpoints för CRUD

#### Service Layer
- **Ansvar:** Affärslogik (t.ex. default-status, statusflöden, prioritering)
- **Teknologi:** .NET Services (C#)
- **Plats:** `Src/web/IncidentManagement.Api/Services/`
- **Exempel:** `IncidentService` som hanterar incidentlogik

#### Repository Layer
- **Ansvar:** Dataåtkomst, abstraktion mot lagring
- **Teknologi:** Interface + Implementation (C#)
- **Plats:** `Src/web/IncidentManagement.Api/Repositories/`
- **MVP:** `InMemoryIncidentRepository`
- **Framtid:** Kan bytas mot SQL/NoSQL/Entity Framework utan att påverka övriga lager

---

## 3. Datamodell

### 3.1 Incident Entity

**Domänmodell:**
```csharp
public class Incident
{
    public Guid Id { get; set; }
    public string Titel { get; set; }
    public string Beskrivning { get; set; }
    public DateTime SkapadDatum { get; set; }
    public string Status { get; set; }  // t.ex. "Open", "InProgress", "Closed"
}
```

**GDPR-tänk:**
- Ingen PII (personuppgifter) i modellen
- Endast tekniska/operativa fält
- Framtida utökningar (t.ex. Prioritet, Kategori) ska också undvika PII

### 3.2 DTO:er (Data Transfer Objects)

- **`CreateIncidentRequest`** – Input för POST /api/incidents
- **`IncidentResponse`** – Output för GET-endpoints

---

## 4. API Endpoints (MVP)

| Metod | Endpoint | Beskrivning | Status |
|-------|----------|-------------|--------|
| POST | `/api/incidents` | Skapa ny incident | Planerad |
| GET | `/api/incidents` | Lista alla incidenter | Planerad |
| GET | `/api/incidents/{id}` | Hämta specifik incident | Planerad |

**Framtida endpoints (ej MVP):**
- `PUT /api/incidents/{id}` – Uppdatera incident
- `PATCH /api/incidents/{id}/status` – Ändra status
- `DELETE /api/incidents/{id}` – Ta bort incident (soft delete?)

---

## 5. Säkerhet och GDPR

### 5.1 Loggning (GDPR-kompatibel)

**Vad ska loggas:**
- Correlation ID (för spårning av requests)
- HTTP-metod och endpoint
- Statuskoder (200, 404, 500, etc.)
- Tekniska felmeddelanden (utan PII)
- Timestamps

**Vad ska INTE loggas:**
- Personuppgifter (namn, e-post, personnummer)
- Secrets, tokens, API-nycklar (i klartext)
- Känslig affärsdata som kan identifiera personer

### 5.2 Konfiguration

- Alla secrets ska läsas från **Environment Variables**
- Använd `IConfiguration` för att läsa settings
- Lokal utveckling: `dotnet user-secrets` eller lokala env vars
- Produktion: Azure App Configuration / Key Vault (framtida)

### 5.3 Autentisering och auktorisering (framtida)

- MVP: Ingen autentisering (endast lokal utveckling)
- Framtid: Azure AD / OAuth2 / JWT tokens
- Rollbaserad åtkomst (t.ex. Admin, User, Viewer)

---

## 6. Teststrategi

### 6.1 Enhetstester
- **Plats:** `Tests/unit/IncidentManagement.Tests/`
- **Teknologi:** xUnit, Moq, FluentAssertions
- **Fokus:** Service-lagret, affärslogik, repository-implementationer
- **Mål:** Minst 70% code coverage (MVP: 2+ tester för IncidentService)

### 6.2 Integrationstester
- **Plats:** `Tests/integration/` (planerat)
- **Teknologi:** `Microsoft.AspNetCore.Mvc.Testing`, `WebApplicationFactory`
- **Fokus:** API-endpoints, hela flöden (POST → GET)
- **Mål:** Testa alla MVP-endpoints

### 6.3 E2E-tester (framtida)
- Testa frontend + backend tillsammans
- Använd t.ex. Playwright eller Cypress

---

## 7. CI/CD Pipeline

### 7.1 Azure DevOps Pipeline

**Fil:** `azure-pipelines.yml` (i projektroten)

**Steg (MVP):**
1. Checkout repository
2. Installera .NET 8 SDK
3. `dotnet restore`
4. `dotnet build -c Release`
5. `dotnet test`

**Triggers:**
- Push till `main` branch
- Pull requests mot `main`

**Framtida steg:**
- Deploy till Azure App Service (API)
- Deploy till Azure Functions
- Deploy frontend (Static Web Apps eller App Service)

### 7.2 Miljöer (planerade)

- **Dev** – Utvecklingsmiljö (kontinuerlig deploy från `dev` branch)
- **Test** – Testmiljö (manuell eller automatisk deploy från `main`)
- **Prod** – Produktionsmiljö (manuell godkännande + deploy)

---

## 8. Deployment och infrastruktur (planerat)

### 8.1 Azure-tjänster (planerade)

- **API:** Azure App Service (Linux, .NET 8)
- **Functions:** Azure Functions (Consumption eller Premium plan)
- **Frontend:** Azure Static Web Apps eller App Service
- **Logging:** Application Insights
- **Config:** Azure App Configuration / Key Vault

### 8.2 Databas (framtida)

- **MVP:** In-Memory (mockad databas)
- **Framtid:** Azure SQL Database eller Cosmos DB
- Repository-lagret är förberedd för att bytas ut

---

## 9. Projektstruktur (detaljerad)

```
MolnDemo/
├── Src/
│   ├── web/
│   │   └── IncidentManagement.Api/
│   │       ├── Controllers/          # Endpoint Layer
│   │       ├── Services/             # Service Layer
│   │       ├── Repositories/         # Repository Layer
│   │       ├── Models/               # Entities, DTOs
│   │       ├── Program.cs
│   │       └── appsettings.json
│   └── functions/                    # (planerat) Azure Functions
│
├── Tests/
│   ├── unit/                         # Enhetstester
│   │   └── IncidentManagement.Tests/
│   └── integration/                  # (planerat) Integrationstester
│
├── docs/
│   ├── overview.md                   # Denna fil
│   └── architecture.md               # Detaljerad arkitektur
│
├── azure-pipelines.yml               # CI/CD pipeline
└── README.md                         # Snabbstart och översikt
```

---

## 10. Teknisk stack

### Backend
- **.NET 8** – Huvudplattform
- **ASP.NET Core Web API** – REST API
- **xUnit** – Testramverk
- **Moq** – Mocking för tester
- **FluentAssertions** – Assertions i tester

### Frontend (planerat)
- **React** eller **Next.js** (beslut krävs)
- **TypeScript** (rekommenderas)
- **Axios** eller **fetch** för API-anrop

### DevOps
- **Azure DevOps** – CI/CD
- **Git** – Versionshantering
- **Azure CLI** – Hantering av Azure-resurser

---

## 11. Utvecklingsprocess och workflow

### 11.1 Branch-strategi
- **`main`** – Huvudbranch (produktionsredo kod)
- **`dev`** – Utvecklingsbranch (valfritt)
- **Feature branches** – `feature/issue-XX-beskrivning`

### 11.2 Commit-messages
- Använd beskrivande meddelanden
- Länka till GitHub Issues när möjligt (t.ex. `Fixes #5`)

### 11.3 Code Review
- Alla PR:er ska granskas innan merge till `main`
- CI-pipeline måste vara grön innan merge

---

## 12. Framtida utökningar (backlog)

### 12.1 Funktioner
- [ ] Prioritering av incidenter (Hög, Medel, Låg)
- [ ] Statusflöden (Open → InProgress → Closed)
- [ ] Notifieringar via e-post/SMS (Azure Functions)
- [ ] Sök och filtrering av incidenter
- [ ] Rapportering och dashboards
- [ ] Användarhantering och roller
- [ ] Kommentarer på incidenter
- [ ] Bifogade filer

### 12.2 Tekniska förbättringar
- [ ] Ersätt In-Memory repository med riktig databas
- [ ] Caching (t.ex. Redis)
- [ ] API-versionering
- [ ] Rate limiting
- [ ] GraphQL endpoint (valfritt)
- [ ] WebSocket för real-time updates

---

## 13. Referenser och dokumentation

- **README.md** – Snabbstart och översikt
- **docs/architecture.md** – Detaljerad arkitektur och designbeslut
- **docs/overview.md** – Denna fil (projektöversikt)

---

## 14. Kontakt och ansvar

**Projektägare:** [Namn/Team]  
**Teknisk lead:** [Namn]  
**GitHub Repository:** [Länk till repo]

---

**Senast uppdaterad:** 2025-01-12  

