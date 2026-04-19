# Öppna frågor, framtid och agent-regler

> Del av [Tillsammans-specifikation](../tillsammans.md).

## Framtida / aspirationer

**Inte MVP.** Fångade här för att inte glömmas.

- **AI-driven riskanalys (Grön/Gul/Röd)** på agendapunkter — kräver: svenska juridiska modeller, RAG över stadgar + anläggningslagen + praxis, human-in-the-loop verifiering. Fara: styrelsen lär sig luta på "AI sa så". Ska vara *decision support*, aldrig *decision maker*.
- **Dokumenttolkning av historiska akter** (OCR + juridisk tolkning) — ej moget på svensk marknad idag. Svensk paleografi och arkaisk juridisk svenska är inte lösta problem. Avvakta.
- **Myndighetsintegrationer** (Bolagsverket, Skatteverket API) — genererade underlag räcker länge; direktinsändning är senare.
- **Exportformat mot externa operativa verktyg** — eftersom vi avgränsat bort operativ funktionalitet blir bra export-integrationer (t.ex. SIE-export för bokföring, CSV-underlag för Fortnox) desto viktigare. Governance-kärnan levererar underlag, externa verktyg verkställer.

## Öppna specifikationsfrågor

### ~~`[ÖPPEN]` DecisionLog — immutability-nivå~~ (löst)

Beslutad arkitektur finns i [granskningslogg.md](granskningslogg.md): sex-lager-stack (L1 appkonvention, L2 DB-tvingad immutabilitet, L3 hash-kedja — obligatoriska; L4 extern förankring via OpenTimestamps som default, L5 distribuerade kopior, L6 publik tip-hash — opt-in). Rättelser som nya händelser; räkenskapsårs-epoker med sigill vid ansvarsfrihet; retroaktiva tillägg (addenda) via fem typkodade vägar. Kvarvarande öppna detaljer är listade sist i `granskningslogg.md` — inte här.

### `[ÖPPEN]` Andelstal — teknisk representation

Röstmekaniken är avgjord (se [Kärnkoncept: Röstning](core-concepts.md#röstning)). Kvar är tekniska val, kalibrerade mot empiri från 10 samfälligheter (apr 2026, [verification-reports/samfalligheter-2026-04.md](verification-reports/samfalligheter-2026-04.md)):

- Prisma `Decimal` (inte `Float`) — självklart.
- Precision/skala: **`Decimal(10, 6)`** räcker för alla observerade fall. Exempel:
  - Kyrkmossen: platt `1/97 = 0.010309` (alla fastigheter lika).
  - Hule Vägförening: GA-vikter uttryckta som procent 68/27/5 — lagras som 0.680000 / 0.270000 / 0.050000 på GA-nivå (separat från per-fastighet-andelstal).
  - Historiska andelar (Norrby brygga, Sättra): kan ha udda decimaler från förrättningsbeslut.
- UI: bråk, procent eller decimal? Alla tre som visningsalternativ — samfällighets-onboarding detekterar källformatet och bevarar det i `visualization`-fältet.
- Summerar andelstal alltid till 1.0 per GA? Anläggningslagen säger ja — systemet ska **varna vid avvikelse, inte tvinga**. Åby Nordgårds **dokumenterade konflikt 27 vs 31 andelar** i GA:2 är precis det legacy-fall där strikt validering skulle låsa föreningen.

### `[ÖPPEN]` Autentisering

- NextAuth v5 (e-post + lösenord) som i Hemmet?
- BankID från start? Rimligt för svensk föreningskontext — identitet är central för rösträtt och protokollsignering.
- Hybrid: e-post/lösenord för inloggning, BankID för röster och protokollsignaturer.

### `[ÖPPEN]` Stack

Avgörs efter att domänmodellen är stabil. Sannolikt Next.js + Prisma + tRPC + Tailwind (T3-familjen) i linje med Hemmet, men inget tvång.

### `[ÖPPEN]` Stadgetolkning vs. stadgeändring

Stämman kan i praktiken tolka en formulering i stadgan på ett avvikande sätt (t.ex. läsa *"två suppleanter"* som *minst* två och välja in tre) utan att ändra själva texten. Det är inte en stadgeändring — det är en tolkningsavvikelse.

Stadge-modulen behöver kunna representera:

- **Normalläsning** — ordalydelsen tolkas som skriven.
- **Avvikande tolkning** — stämman har uttryckligen tolkat en formulering på ett specifikt sätt, protokollfört, men inte ändrat texten.
- **Stadgeändring** — texten själv ändras genom formell process.

Modell-valet avgör om avvikande tolkningar blir sökbara över tid (värdefullt — syns om en ny styrelse försöker "tolka om" i motsatt riktning) eller försvinner in i protokoll-brus.

Förslag: avvikande tolkning lagras som en annoterings-entitet kopplad till (a) den stadgeparagraf den gäller och (b) det beslut som etablerade tolkningen. Renderas som varningsikon på stadgeparagrafen i medlem-vyn. Stadgeändring följer sin egen väg.

## Utvecklaragent-regler vs. produktfunktioner

Geminis ursprungsprompt blandar två saker:

1. **Produktfunktioner** — Risk Analysis Mode, Jäv, Bordläggning, Grön/Gul/Röd. Vad plattformen gör för användaren. Specas i övriga sektioner.
2. **Utvecklaragent-regler** — hur Claude ska skriva kod. Lyfts till `CLAUDE.md` när stack och konventioner är beslutade.

De ska inte blandas.
