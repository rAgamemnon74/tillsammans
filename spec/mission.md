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
- **En plattform, flera editions.** `AssociationType` är förstaklassig konfiguration. Marknadsförs utåt som *Tillsammans för samfälligheter*, *Tillsammans för föräldraföreningar* etc. — samma produkt, typ-anpassad onboarding, terminologi och domänmodell. Framtida splitning är möjlig om vertikalerna bevisligen drar isär; gå samlad från start eftersom motsatt riktning är opraktisk.
