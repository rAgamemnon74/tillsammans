# Krav: Mötesadministration

> Del av [Tillsammans-specifikation](../../tillsammans.md).

Detta är **kravdokumentet** för mötesadministration. Det specificerar *vad* systemet ska kunna leverera utan att binda upp *hur*. Implementationsvalen detaljeras i `spec/moten.md`, kommande `spec/protokoll.md`, och relaterade filer.

## Bakgrund

Kraven är härledda från:

- **LEF 6** — möten, beslut, protokoll, jäv, röstlängd (LEF 6:13, 6:14, 6:21, 6:25–26, 6:30, 6:48)
- **LFS 36–49§** — samfällighetsstämmor och beslutsförhet
- **Bokföringslagen** — bevarandekrav för räkenskapsrelevant material
- **GDPR** — minimering av persondata i protokoll och bilagor
- **Tillsammans grundprinciper** — [mission.md](../mission.md), [threats.md](../threats.md), [analysis-rules.md](../analysis-rules.md)
- **Studie av Hemmets mötesadministration** (april 2026) — referens för funktionsomfång; beslut om vad vi övertar och vad vi lämnar dokumenteras separat

Kraven är numrerade per kategori (`K-XXX-N`) för spårbarhet i kommande spec-implementation och granskning.

## 1. Mötestyper

- **K-MÖTE-1:** Systemet ska stödja tre mötestyper: **styrelsemöte**, **ordinarie årsstämma**, **extra stämma**. Inga ytterligare typer i MVP.
- **K-MÖTE-2:** Varje mötestyp har egen dagordningsmall (centrala defaults levereras med systemet).
- **K-MÖTE-3:** Föreningen ska kunna anpassa mallar via overlay (lägga till, ta bort, omordna punkter) inom stadgans ramar. Anpassningar audittas.
- **K-MÖTE-4:** Mallar ska förfyllas dynamiskt baserat på (a) stadgens taggade paragrafer enligt [moten.md](../moten.md) och (b) öppna ärenden från [case-types/](../case-types/).

## 2. Mötets livscykel

- **K-LIVSCYKEL-1:** Varje möte har explicit datalivscykel: **UTKAST → SCHEMALAGT → PÅGÅR → SLUTBEHANDLAS → AVSLUTAT** plus sidospår **AVBOKAT**.
- **K-LIVSCYKEL-2:** Övergångar är manuella (utlösta av mötesordförande eller sekreterare); systemet validerar förutsättningar (kallelse-tid, kvorum, signeringar).
- **K-LIVSCYKEL-3:** Varje övergång loggas i granskningsloggen som `MÖTE_STATUS_ÄNDRAT` med från-status, till-status, motivering om relevant.
- **K-LIVSCYKEL-4:** Avbokat möte ska kunna omplaneras (nytt möte med referens till det avbokade); det ursprungliga lever kvar som AVBOKAT i historiken.
- **K-LIVSCYKEL-5:** Mötet kan inte hoppa över livscykel-steg utan särskild motivering registrerad som händelse.

## 3. Dagordning och punkter

- **K-DAGORDNING-1:** Dagordningen är strukturerad data — varje punkt är en egen entitet (inte bara markdown-rader).
- **K-DAGORDNING-2:** Punkter har **specialtyper** som styr UI-rendering: `MÖTETS_ÖPPNANDE`, `VAL_AV_MÖTESORDFÖRANDE`, `VAL_AV_MÖTESSEKRETERARE`, `VAL_AV_JUSTERARE`, `NÄRVAROKONTROLL`, `KVORUM_KONTROLL`, `KALLELSE_GODKÄNNANDE`, `DAGORDNING_GODKÄNNANDE`, `RÖSTLÄNGD_FASTSTÄLLANDE`, `VERKSAMHETSBERÄTTELSE`, `REVISIONSBERÄTTELSE`, `RESULTAT_BALANS`, `ANSVARSFRIHET`, `RESULTATDISPOSITION`, `BUDGET_AVGIFTER`, `UNDERHÅLLSFOND`, `DEBITERINGSLÄNGD`, `MOTION_BEHANDLING`, `STYRELSEFÖRSLAG`, `VAL_STYRELSE`, `VAL_REVISOR`, `VAL_VALBEREDNING`, `ALLMÄN_DISKUSSION`, `MÖTETS_AVSLUTANDE`. Listan kan utökas.
- **K-DAGORDNING-3:** Sekreteraren ska kunna anteckna per punkt under mötet (real-time eller efteråt). Anteckningar är del av punkten, inte separat dokument.
- **K-DAGORDNING-4:** Ordning kan ändras före publicering (kallelse). Efter publicering krävs explicit motivering registrerad i granskningsloggen.
- **K-DAGORDNING-5:** Mötesordförande kan föra upp punkt mid-möte (typiskt under "Övriga frågor") som registreras som tillägg till dagordningen.
- **K-DAGORDNING-6:** Dagordningspunkt kan ha attributet `responsibleActorId` som pekar på ansvarig aktör för punktens underlag (t.ex. kassör för ekonomisk rapport, ordförande för verksamhetsberättelse, revisor för revisionsberättelse, utskottets kontaktperson för utskottsrapport). Attributet är valfritt; punkter utan ansvarig har ingen saknad-flagga.
- **K-DAGORDNING-7:** Bilagor kan bifogas per dagordningspunkt via `BILAGA_INLÄMNAD` med `references = [agenda_item_id]`. Två flöden: utkast i styrelsens arbetsyta (rollgrupp-scope, se [grunddata.md#arbetsytans-scope](../grunddata.md#arbetsytans-scope)) under beredning; publicering till granskningsloggen vid kallelseutskick eller löpande. Bilagor följer bilage-mekaniken i K-INTEGRATION-4.
- **K-DAGORDNING-8:** RBAC för punkt-bilagor: ansvarig ledamot publicerar för sin punkt; sekreteraren och ordförande har rätt att publicera för alla punkter (backup). Övriga styrelseledamöter kan redigera utkast i arbetsytan men publicerar inte andra ledamöters rapporter.
- **K-DAGORDNING-9:** Saknad-flagga: dagordningspunkt med `responsibleActorId` men utan publicerad bilaga vid kallelseutskick flaggas *"förväntad rapport saknas"* i styrelsens mötesvy. Flaggan är deskriptiv — den pekar på var rapporten saknas, inte ut vem (per [policy 8](../mission.md#grund-policies)).

## 4. Roller och behörigheter

- **K-ROLL-1:** **Sekreteraren** är operativ huvudaktör för förberedelse — konfigurerar mötet, förbereder dagordning, genererar röstlängds-underlag, hanterar bilagor, skickar kallelse. Per [roles/board-roles.md](../roles/board-roles.md#sekreterarens-roll-vid-stämmoförberedelse).
- **K-ROLL-2:** **Mötesordförande** styr genomförandet (öppnar, leder, förkunnar beslut, avslutar). På styrelsemöte är detta normalt föreningens ordförande; på stämma är det stämmans val.
- **K-ROLL-3:** **Justerare** granskar och signerar protokollet. Två justerare på stämma (LFS-praxis); en eller två på styrelsemöte enligt stadga.
- **K-ROLL-4:** **Mötesordförande på stämma** kan vara icke-medlem (LEF 6:19, LFS 49§ ger fri valrätt) — med konsekvensen att hen inte kan avge utslagsröst. Systemet flaggar valet med varning men hindrar inte. Per [roles/meeting-roles.md](../roles/meeting-roles.md).
- **K-ROLL-5:** **Suppleant** kan aktiveras dynamiskt vid frånvaro av ordinarie ledamot. Aktivering loggas i audit-trail med tidsstämpel.
- **K-ROLL-6:** **Revisor** har permanent läsrätt på all mötesdokumentation (per [roles/oversight-roles.md](../roles/oversight-roles.md)) men ingen aktiv roll i mötets exekvering.

## 5. Närvaro och röstlängd

Detaljerat i [rostlangd.md](../rostlangd.md). Kärnkrav:

- **K-NÄRVARO-1:** För **styrelsemöten**: digital närvarokontroll med automatisk kvorum-validering mot stadgens `KVORUM`-paragraf.
- **K-NÄRVARO-2:** För **stämmor**: pappers-rutin är norm. Systemet genererar röstlängds-underlag före mötet och tar emot skannad röstlängd + fullmakter som bilagor efter mötet. Ingen digital incheckning av medlemmar krävs.
- **K-NÄRVARO-3:** Mid-möte-byten (sen ankomst, tidig avgång, jäv-aktivering) kan registreras under styrelsemöten; för stämmor hanteras de pappersbaserat och syns i den skannade röstlängden.

## 6. Beslut och röstning

- **K-BESLUT-1:** Varje beslut kopplas till (a) en agenda-punkt, (b) minst en stadgaparagraf, (c) berörda ärenden från case-types-systemet om relevant.
- **K-BESLUT-2:** Röstmetod ska kunna sparas: `ACKLAMATION` / `OMRÖSTNING_HANDUPPRÄCKNING` / `SLUTEN_OMRÖSTNING` / `NAMNUPPROP`.
- **K-BESLUT-3:** **Per-ledamot-röster** (`RÖST_AVGIVEN` per styrelseledamot) registreras för **styrelsemöten** — viktigt för det solidariska ansvarets transparens.
- **K-BESLUT-4:** **Aggregerade siffror** (för/emot/nedlagda) registreras för **stämmor** i `STÄMMOBESLUT`-payload. Per-medlem-data lagras inte digitalt för stämmor (sluten omröstning bevarar anonymitet).
- **K-BESLUT-5:** **Reservation** kan registreras per ledamot vid styrelsebeslut, eller som textnotering vid stämmobeslut. Reservation visas i protokollet enligt etablerad praxis. Per [core-concepts.md#reservation-och-solidariskt-ansvar](../core-concepts.md#reservation-och-solidariskt-ansvar).
- **K-BESLUT-6:** **Jäv-deklaration** ska kunna ges före varje ekonomiskt beslut. Jäv-deklarerad person utesluts automatiskt från det specifika beslutet (deltar inte, röstar inte, dokumenteras separat). Per [core-concepts.md#jäv](../core-concepts.md#jäv-conflict-of-interest).
- **K-BESLUT-7:** Om beslutet kräver kvalificerad majoritet (stadgeändring, upplösning) ska systemet validera att tröskeln nåtts innan `BIFALL` kan registreras.

## 7. Protokoll

- **K-PROTOKOLL-1:** Protokollet ska kunna **autogenereras** från mötesloggen — närvaro, beslut, röstresultat, jäv-deklarationer, mötesroller, sekreterarens anteckningar per punkt.
- **K-PROTOKOLL-2:** Protokollet har egen livscykel: **UTKAST → SLUTBEHANDLAT → SIGNERAT → ARKIVERAT**.
- **K-PROTOKOLL-3:** Sekreteraren slutbehandlar (UTKAST → SLUTBEHANDLAT). Sekreteraren kan dra tillbaka från SLUTBEHANDLAT om korrigeringar behövs.
- **K-PROTOKOLL-4:** Mötesordförande och alla justerare måste signera (SLUTBEHANDLAT → SIGNERAT). Signering loggas med tidsstämpel per signer i audit-trail.
- **K-PROTOKOLL-5:** Bilagor (skannad röstlängd, fullmakter, övriga underlag) bifogas via standard-bilage-mekanik enligt [granskningslogg.md#bilagor](../granskningslogg.md#bilagor).
- **K-PROTOKOLL-6:** Efter SIGNERAT är protokollet låst. Rättelser efter signering sker som **addenda** enligt [granskningslogg.md#retroaktiva-tillägg-addenda-till-stängd-epok](../granskningslogg.md) — aldrig som tyst edit.
- **K-PROTOKOLL-7:** Protokollet inkluderar tip-hash från granskningsloggen vid signeringstidpunkten (L6 enligt granskningslogg.md). Detta möjliggör senare bevisning av integritet.
- **K-PROTOKOLL-8:** För årsstämma: protokollets sigill triggar `EPOK_FÖRSEGLAD` med rätt sigill-variant baserat på ansvarsfrihetsbeslutet.
- **K-PROTOKOLL-9:** Digital signering i MVP: klick-baserad bekräftelse i systemet med tidsstämpel. BankID-integration är *valfritt fas-2-utvidgning*, inte MVP-krav.

## 8. Notifieringar och påminnelser

- **K-NOTIFIERING-1:** **Kallelse** skickas till alla berörda enligt mötestypens distributionsregel ([case-types.md#distribution-och-notifiering](../case-types.md)).
- **K-NOTIFIERING-2:** **Påminnelse till sekreteraren** när protokoll-utkast är försenat (default: 14 dagar efter mötet).
- **K-NOTIFIERING-3:** **Påminnelse till justerare** för pending signeringar (default: 7 dagar efter SLUTBEHANDLAT).
- **K-NOTIFIERING-4:** **Påminnelser till styrelsen** vid kommande deadlines: motionsfrist, kallelse-deadline, revisionsberättelse-deadline (kopplat till årshjulet i [föreningen.md](../föreningen.md)).
- **K-NOTIFIERING-5:** Kanal är aktörens registrerade kontaktkanal — typiskt e-post för medlemmar, in-app för styrelseledamöter. Inte SMS, inte extern app.

## 9. Integration med andra moduler

- **K-INTEGRATION-1:** Pågående case-type-ärenden ska **auto-länkas** till lämpliga möten:
  - Motioner inom motionsfrist → nästa ordinarie årsstämma
  - Öppna styrelsebeslut, utlägg, medlemsansökningar → nästa styrelsemöte
  - Extra stämma — kallelsebegäran → resulterande extrastämma
- **K-INTEGRATION-2:** Stämmobeslut om ansvarsfrihet triggar `EPOK_FÖRSEGLAD` i granskningsloggen med rätt sigill-variant ([föreningen.md](../föreningen.md)).
- **K-INTEGRATION-3:** Stadgeparagrafer (kommande stadge-modul) styr automatiska aspekter av mötet enligt 17 paragraftyper specificerade i [moten.md](../moten.md).
- **K-INTEGRATION-4:** Bilagor hanteras via samma bilage-mekanik som ärenden (tre typer: FIL, URL, TEXTNOTERING) enligt [granskningslogg.md#bilagor](../granskningslogg.md#bilagor).
- **K-INTEGRATION-5:** Alla möteshändelser skrivs till granskningsloggen enligt fyra-delars-modellen (metadata + fritext + systemdata + bilagor).

## 10. Lag- och stadgekrav

### Per mötestyp

| Mötestyp | Lagkrav | Stadge-typiska krav |
|---|---|---|
| **Styrelsemöte** | Beslutsförhet via majoritet (LFS 39§, LEF 6:30) | Frekvens, kvorum-tröskel, kallelseprocess |
| **Ordinarie årsstämma** | Inom 6 månader efter räkenskapsår (LEF 6:14, LFS 47§); röstlängd förs (LEF 6:21); protokoll förs och signeras (LEF 6:25–26) | Specifika dagordningspunkter; obligatoriska beslut (ansvarsfrihet, val) |
| **Extra stämma** | Kallelse av styrelse, medlemsminoritet eller revisor (LEF 6:13, LFS 47§); bara behandling av kallelsens punkter | Tröskel för medlemsminoritet (typiskt 1/10) |

### Generella

- **K-LAG-1:** Kallelse-tider enligt stadgan, aldrig under lagens minimi-gränser.
- **K-LAG-2:** Röstlängd förs vid stämmor (LEF 6:21); för samfälligheter kan stadgan stiga till alltid eller enbart vid begärd omröstning (LFS 49§).
- **K-LAG-3:** Protokoll signeras av mötesordförande och minst en justerare (LEF 6:26).
- **K-LAG-4:** Klandertalan inom 3 månader (LEF 6:48). Protokollet och granskningsloggen ska räcka som juridiskt bevisunderlag.
- **K-LAG-5:** Bokföringslagen kräver bevarande av räkenskapsmaterial 7 år. Protokoll med ekonomi-relevans bevaras enligt detta.
- **K-LAG-6:** GDPR-minimering: protokoll ska inte innehålla mer persondata än nödvändigt för juridisk dokumentation.

## 11. Integritets- och spårbarhetskrav

- **K-INTEGRITET-1:** Alla möteshändelser skrivs till granskningsloggen enligt fyra-delars-modellen. Inget mötesdata lever utanför loggen.
- **K-INTEGRITET-2:** Protokollets innehåll är **härlett från granskningsloggen** — inte en separat sanning. Vid divergens är loggen auktoritativ.
- **K-INTEGRITET-3:** Systemet ska kunna visa "vem ändrade vad och när" för varje del av mötesdokumentationen. Implementeras via granskningsloggens vyer.
- **K-INTEGRITET-4:** Stämmoprotokollets tip-hash ska kunna verifieras mot OpenTimestamps eller motsvarande extern förankring (L4 enligt granskningslogg.md).
- **K-INTEGRITET-5:** Rättelser i färdigt protokoll sker alltid som addenda — aldrig som modifikation av originalet.

## 12. Edition-specifika krav

### Samfällighet

- **K-SAMF-1:** Röstlängd förs bara när omröstning begärs (LFS 49§); systemet stödjer båda lägen via stadge-flagga.
- **K-SAMF-2:** Frågespecifik röstlängd vid multi-GA — olika GA har olika medlemsomfång; röstlängden räknas per fråga.
- **K-SAMF-3:** Andelstal och samägande respekteras vid röstning (UNIFIED/PROPORTIONAL — se [core-concepts.md#samägda-fastigheter](../core-concepts.md)).
- **K-SAMF-4:** Debiteringslängd kan fastställas som obligatorisk dagordningspunkt på årsstämma.
- **K-SAMF-5:** Mandatverifiering för juridiska personers representanter — se [medlemskap.md](../medlemskap.md) och [case-types/mandatverifiering.md](../case-types/mandatverifiering.md). Verifieras före stämmoöppning av mötesordförande/sekreterare.

### LEF (övriga ekonomiska föreningar)

- **K-LEF-1:** Bolagsverket-inlämning av årsredovisning efter sigill — manuell process; systemet flaggar deadline.
- **K-LEF-2:** Insats-frågor vid medlemsansökningar kan tas på styrelsemöte enligt mandat ([medlemskap.md](../medlemskap.md)).
- **K-LEF-3:** KU-rapportering till Skatteverket vid utdelning är manuell process; systemet flaggar att den behövs.
- **K-LEF-4:** Stadgeändring-flödet kräver två stämmobeslut (LEF 6:36) — separat livscykel hanteras av [case-types/stadgeandring.md](../case-types/stadgeandring.md), möten är förmedlare.

## 13. Bortvalt — vad systemet uttryckligen INTE ska göra i MVP

- **BankID-signering** — ej i MVP. Klick-baserad bekräftelse räcker för digital audit-trail. BankID är fas-2.
- **Digital röstning under stämma** — pappers-rutin är beslut. Per `K-NÄRVARO-2` och [rostlangd.md](../rostlangd.md).
- **Automatisk publicering av protokoll** — sekreterarens slutbehandling krävs alltid.
- **Hård avbrottshantering vid EJ_BESLUTSFÖRT** — mjuk hantering: flagga, tillåt mötet fortsätta för informationsutbyte. Stadge-flagga för hård hantering är möjlig (`K-NÄRVARO-strikt`).
- **Boende-kommunikation, bokningar, mätarinsamling, andra operativa funktioner** — out-of-scope per [architecture.md#utanför-scope-externa-verktyg](../architecture.md).
- **Komplex valberedningsprocess** med separata `NominationPeriod`-entiteter — den enkla [nominating-committee.md](../roles/nominating-committee.md)-modellen räcker.

## 14. Öppna frågor

- **K-ÖPPEN-1:** Behöver Tillsammans en `AgendaItem`-entitet med separata specialtyper, eller räcker det att mallen renderas som markdown med `[auto]`-taggning? Förslag: ja, separata entiteter — ger UI auto-rendering, valideringsmöjligheter, och bättre koppling till case-type-ärenden.
- **K-ÖPPEN-2:** Ska protokoll-justering ha särskilt tidsfönster (t.ex. inom 14 dagar), eller är det bara en påminnelse? Förslag: påminnelse-baserat; stadge-flagga möjlig om föreningen vill ha hård gräns.
- **K-ÖPPEN-3:** Vid hybrid-stämma (digitala + fysiska deltagare): är digital närvaro tillåten på stämma? Just nu pappers-default; stadge-flagga `STÄMMA_NÄRVARO_DIGITAL` finns reserverad.
- **K-ÖPPEN-4:** Hur hanteras motioner som inkommer efter motionsfrist men före stämman? Förslag: kan föras upp som "övriga ärenden" men inte ge formell behandling som motion. Behöver detaljeras i [case-types/](../case-types/).
- **K-ÖPPEN-5:** Ska systemet stödja "preliminär dagordning" som sekreteraren kan justera fram till kallelse, kontra "publicerad dagordning" som är låst efter kallelse? Förslag: ja — UTKAST-status hanterar detta.
- **K-ÖPPEN-6:** Hur fungerar protokoll-justering vid digital signering? Sekventiell (en åt gången) eller parallell (alla signerar oberoende)? Förslag: parallell — alla får tillgång samtidigt, kan signera oberoende; protokollet blir SIGNERAT när alla har signerat.
- **K-ÖPPEN-7:** Behöver systemet stöd för **mötesordförandens utslagsröst** vid lika röstetal — registreras som vanlig RÖST_AVGIVEN med flagga `is_tiebreaker`, eller separat händelse? Förslag: flagga räcker.

## 15. Kravens uppfyllelse i kommande spec

| Krav-grupp | Implementeras i |
|---|---|
| 1–3 (mötestyper, livscykel, dagordning) | Utvidgning av `spec/moten.md` |
| 4 (roller) | Befintlig `spec/roles/*` är tillräcklig; minor utvidgningar för sekreterare/mötesordförande |
| 5 (närvaro/röstlängd) | Befintlig `spec/rostlangd.md` täcker det mesta; mindre justeringar |
| 6 (beslut och röstning) | Befintlig `spec/core-concepts.md` + utvidgning av moten.md |
| 7 (protokoll) | Ny `spec/protokoll.md` |
| 8 (notifieringar) | Utvidgning av `spec/case-types.md` distribution-sektion + möte-specifika tillägg i moten.md |
| 9 (integration) | Cross-refs och utvidgningar av befintliga filer |
| 10 (lag/stadge) | Cross-refs till `spec/legal-context.md` och `spec/moten.md` stadgeparagraf-typer |
| 11 (integritet) | Befintlig `spec/granskningslogg.md` täcker det |
| 12 (edition) | Cross-refs till `spec/editions/samfallighet.md` (LEF-edition väntas) |
| 13 (bortvalt) | Detta dokument är förankringspunkten; kommande spec refererar tillbaka |
| 14 (öppna) | Klargörs när designen formaliseras |
