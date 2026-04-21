# Uppdrag, förtroendemodell och fastslagna beslut

> Del av [Tillsammans-specifikation](../tillsammans.md).

## Uppdrag

Skydda lekmän (ideella styrelser) från maktobalanser och juridiska risker i svenska föreningar. Plattformen är en **governance-sköld** — den kodifierar demokratiska processer så att enskilda påverkansförsök, informella överenskommelser och asymmetrier mellan medlem och styrelse inte kan leda till beslut som avviker från majoritet och stadgar.

Försäljningsargumentet är *beslutstrygghet*, inte fastighetsadministration.

## Förtroendemodellen — två mottagare

Systemet byggs *för* de förtroendevalda — det är de som står formellt ansvariga, dagligen arbetar i plattformen, och vars juridiska skydd hänger på att processen håller. Men plattformens *värde mot medlemmarna* är något annat — och minst lika viktigt.

Medlemmen behöver inte själv granska varje beslut. Hon behöver veta att **det system styrelsen arbetar i är konstruerat att följa majoriteten och stadgarna — inte enskilda påtryckningar eller informella överenskommelser.** Det är en separat typ av förtroende än styrelsens egen, och lätt att underskatta.

### Varför open source är mer än ett tekniskt val

Open source (MIT) är inte bara en hygien-preferens. Det är det enda sättet att göra anspråket "*systemet behandlar alla medlemmar lika enligt stadgarna*" granskningsbart av någon annan än oss själva:

- *"Läs koden om ni inte tror oss"* är ett starkare försvar än någon proprietär plattform kan erbjuda.
- Ingen central aktör (HSB, Riksbyggen, oss själva) kan bakvägen justera logiken utan att det syns.
- Det gör plattformen juridiskt robust mot framtida anklagelser om algoritmisk partiskhet.

### "Vår förening kör Tillsammans" som governance-signal

På samma sätt som *"vi har en auktoriserad revisor"* är ett trovärdighetsmärke utåt blir valet av governance-plattform en signal. För att det ska fungera måste plattformen själv leverera på löftet — inte bara påstå det.

### Designprinciper som följer av detta

- **Medlem-facing vyer.** Beslutslogg, stadge-index, stämmoresultat, (anonymiserad) jäv-historik ska ha publika läsvyer inom GDPR-ramarna — inte bara styrelsens interna arbetsyta.
- **Synlig stadge-anknytning.** När ett beslut visas för en medlem ska hon kunna klicka sig till den paragraf det vilar på.
- **Process-transparens före utfall-påverkan.** Medlemmen ska inte kunna ändra beslut, men ska alltid kunna se *hur* beslutet togs.
- **Publik versionshistorik av stadgarna.** Ändringar över tid blir spårbara — paragrafer som lagts till eller tagits bort kan granskas i efterhand.

### Vad detta inte betyder

- **Inte full insyn i pågående ärenden.** Personärenden, pågående juridiska processer och liknande har fortsatt avgränsad insyn enligt GDPR och föreningspraxis.
- **Inte röstning-as-a-service till medlemmar.** Den formella makten ligger kvar hos stämman och styrelsen enligt stadgarna. Systemet replikerar föreningsdemokratin — det omdefinierar den inte.

## Grund-policies

Principer som styr all spec-utveckling. Fördjupningar i de filer som refereras.

### Filosofisk grund

1. **Synlighet framför tvingning.** Granskningsloggen är tolknings-underlag för medlemmar, revisor och eventuellt domstol — inte en tamper-proof enforcement-mekanism. Skyddet är att handlingar blir läsbara bakåt, inte att fel blir omöjliga framåt. Se [granskningslogg.md](granskningslogg.md).
2. **Förtroendevald = förhandsgivet förtroende.** Styrelsen har mandat från medlemmarna. Systemet utgår från den tilliten och synliggör förvaltningen — ersätter den inte med tekniska garantier. Förtroende förnyas eller dras tillbaka med evidens från loggen, inte genom strukturella spärrar i förväg.
3. **Förening = pågående förhandling.** Stadga, praxis, roller omförhandlas löpande. Systemet dokumenterar förhandlingens förlopp, låser inte fast dess position. Det som var "fel" kan bli "sed"; systemet bär hela spåret.
4. **Årlig periodfrekvens som grundrytm.** Räkenskapsåret är den primära tidsenheten. Kallelse → stämma → ansvarsfrihet → budget → debiteringslängd → revision. Den takt LEF, LFS och Anläggningslagen föreskriver. Se [föreningen.md](föreningen.md), [granskningslogg.md](granskningslogg.md).

### Scope och ambition

5. **Governance-kärna, inte operativ plattform.** Detaljer i *Fastslagna beslut* nedan och i [architecture.md](architecture.md).
6. **Normal människor, lekmannastyrelser, fritidsaktivitet.** Komplexiteten ligger i systemet, inte hos användarna. Initial installation ska rymmas på ~2 timmar med minimal dokumentuppsättning.
7. **Gradvis nyttoackumulering.** Föreningen betalar komplexitets-kostnaden när funktionen faktiskt behövs. Varje installations-fas är självstående användbar — ingen allt-eller-inget-onboarding.
8. **Inte polis, inte skatteverk, inte domstol, inte debatt-arena.** Systemet är verksamhetsstödjande, inte övervakande — utgångspunkten är att användare är vettiga. Systemet *bygger inte* dedikerade features för missbruk, men censurerar inte heller innehåll i legitima fritextfält. Missbruk är föreningens problem (GDPR-ansvarig), inte systemets. Fyra aspekter av denna *arkitektoniska återhållsamhet*:
   - *Inte tillsynsmyndighet* — validerar inte civilrättsliga frågor, moderniserar inte gamla stadgar, tvingar inte fram "rätt" beteende.
   - *Rättslig agnostisism* — juridiska referenser (polisanmälnings-, åtals-, domnummer) lagras som data (offentliga handlingar), inte som betyg. Inga dedikerade varnings-/flaggsystem för "misstänkta överträdelser" innan domstol avgjort.
   - *Ingen åsiktsregistrering som feature* — Regeringsformen 2:3 + GDPR Art. 9 är föreningens ansvar (personuppgiftsansvarig). Systemet bygger inga strukturerade fält för "medlemsbedömning", "problemmedlem-tagg" eller liknande dedikerade strukturer.
   - *Inte debatt-as-a-service* — kommentartrådar, forum, "likes" på förslag är inte funktioner. Föreningsdemokratin sker på stämman.

### Hot- och skydds-modell

9. **Hotet är internt.** Styrelser, medlemmar, entreprenörer inom föreningen — inte externa hackare. Tre klasser: informella påtryckningar, asymmetrier, historierevision. Se [threats.md](threats.md).
10. **Revisorn som konstitutionellt oberoende motvikt.** Permanent läsrätt från dag 1, egen arbetsyta, inte valbar tillsammans med styrelseuppdrag. Se [roles/oversight-roles.md](roles/oversight-roles.md).
11. **Bevarande med bevisvärde.** Hash-kedja, OpenTimestamps, kanonisering. Syfte: att granskare 2046 ska kunna lita på att 2026 års logg är den som skrevs 2026 — inte att förhindra illegitima skrivningar i realtid.

### Data- och design-hållning

12. **Lapptäcket — exponerar, dömer inte.** Flera källor tillåts parallellt. Dokumentkonflikter flaggas, löses inte automatiskt. Föreningen bygger sanning genom beslut. Se [domain-model.md](domain-model.md).
13. **Typ-anpassning utan typ-läckage i kärnan.** `AssociationType` styr konfiguration och terminologi, aldrig if-grenar i governance-logiken. Edition-specifika mönster i `spec/editions/`. Se [architecture.md](architecture.md).

### Projekt-institution

14. **Open source som förtroende-mekanism.** MIT. Verifikations-rapporter är offentliga. Transparens inte bara i loggens händelser utan i själva spec-arbetet. Fördjupas i *Förtroendemodellen* ovan.
15. **Svensk domän hela vägen.** UI, URL-sökvägar, terminologi på svenska. Kod på engelska. Lagrum (LFS, Anläggningslagen, LEF) som första-klass-referenser.
16. **Analys-regler-disciplin.** Varje spec-ändring granskas mot [analysis-rules.md](analysis-rules.md) — projektets systematiska försvar mot att governance-öppningar smyger in via välmenande funktioner.

## Fastslagna beslut

- **Open source, MIT-licens** (samma som Hemmet).
- **Fristående kodbas** — inte monorepo med Hemmet, inte delade paket. Syskon, inte delad kärna.
- Svensk UI, svenska URL-sökvägar, engelsk kod.
- **Specifikationen går före tekniken** — stack väljs när domänmodellen är stabil.
- `CLAUDE.md` är tunn och pekar hit.
- **MVP = port av Hemmets förtroendevald-stöd**, se [arkitektur](architecture.md).
- **Inga importer från officiella fastighetsregister** (Lantmäteri). Föreningen bygger manuellt upp sin "digitala tvilling" — det är enda sättet att få korrekt data för äldre samfälligheter. `[ÖPPEN]`: gäller detta alla externa datakällor, eller specifikt register med opålitlig data?
- **Tillsammans är en governance-plattform — ingen operativ funktionalitet i scope.** Bokningar, mätarinsamling, insamling, event, inventarier, bokföring, betalningsinfrastruktur hör till externa verktyg. Motivering: (a) operativa behov skiljer sig kraftigt *inom* samma föreningstyp, (b) externa verktyg finns redan (Swish, Fortnox, kalenderappar), (c) vår unique value är governance-sköld.
- **Kassörsstöd (utbetalningar mot styrelsebeslut + stadgeparagraf) är kärna.** Det *dokumenterar* pengaflöden mot beslut — det utför inte betalningar och är inte bokföring. Underlag exporteras till externa system.
- **En plattform, flera editions.** `AssociationType` är förstaklassig konfiguration. Marknadsförs utåt som *Tillsammans för samfälligheter*, *Tillsammans för LEF* — samma produkt, typ-anpassad onboarding, terminologi och domänmodell. Framtida splitning är möjlig om vertikalerna bevisligen drar isär; föräldraförenings-editionen splittades ut april 2026 till det fristående projektet [Framtiden](../../../framtiden) när vertikalen visade sig dra tillräckligt långt från samfällighets- och LEF-mönstret.
