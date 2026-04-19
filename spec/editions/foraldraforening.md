# Tillsammans för föräldraföreningar — edition-spec

> Del av [Tillsammans-specifikation](../../tillsammans.md). Den här filen dokumenterar vad som är *specifikt* för `AssociationType = föräldraförening`. Kärnans governance-flöden ligger i övriga spec-filer; här finns edition-specifika defaults, alternativ och domänmönster.
>
> Empirisk bakgrund: [verification-reports/foraldraforeningar-2026-04.md](../verification-reports/foraldraforeningar-2026-04.md) (12 föreningar analyserade).

## Vad editionen täcker

Tillsammans för föräldraföreningar är rollstöd för förtroendevalda i svenska föräldraföreningar och föräldrakooperativ knutna till en skola/förskola. Medlemmen är vårdnadshavaren; barnet är bäraren av klasstillhörighet.

Primärexponerad roll: **kassör** (se [audience.md](../audience.md)). Kassören är mer utsatt för governance-hot än ordföranden — social misstro kring klasskassa, swish, eventfondsmedel, "vem snodde pengarna"-risken.

## Sub-typer

Editionen täcker tre strukturellt olika föreningstyper som alla kallas "föräldraförening" i vardagligt tal:

| Sub-typ | Juridisk form | Typisk storlek | Driver skolan? |
|---|---|---|---|
| **Skol-FF** | Ideell förening | 100–700 vårdnadshavare | Nej — stödförening |
| **FF-med-representationsmandat** | Ideell förening | Som skol-FF | Utser ledamot i stiftelsen/ab som driver skolan |
| **Föräldrakooperativ-förskola** | Ekonomisk förening (LEF) | 15–40 familjer | Ja — föräldrarna är huvudman |

Distinktionen styr vilka moduler som prioriteras. Ett kooperativ är en *arbetsgivare med arbetsplikt*; en skol-FF är en *stödförening med fondfördelning*.

## Medlemskapsmodell

### Defaults per sub-typ

- **Skol-FF (gratis):** automatiskt medlemskap när barn börjar på skolan. `LAPSED` vid extern händelse — barnet ej längre inskriven. Backatorp-modellen ([verification-reports/foraldraforeningar-2026-04.md#mönster-1](../verification-reports/foraldraforeningar-2026-04.md#mönster-1--medlemskapsmodell)).
- **Skol-FF (avgift):** medlemsavgift per läsår, beslutas av stämman. Avgiften fungerar som årlig aktivering. Nya Elementar-modellen.
- **Föräldrakooperativ:** ansökan + styrelseinval + insats + arbetsplikt. Formell uppsägningstid (1–2 månader). Minigiraffen/Svanen-modellen.

### Val av trigger för `LAPSED`

Tre alternativ erbjuds i stadge-modulen:

1. **Externt faktum** (default för skol-FF) — barnet ej längre inskriven → automatisk `LAPSED`. Kräver koppling till skolans elevregister (kan vara manuell uppdatering per termin).
2. **Årlig reconfirmation** (tillägg) — aktiv bekräftelse en gång per år. Spec-originalmodellen.
3. **Avgift-cykel** (för avgiftsbaserade skol-FF) — utebliven årsavgift → `LAPSED`.

Föreningen kan kombinera: t.ex. externt faktum + reconfirmation vid varje ägarbyte i klassen.

### Föräldrakooperativ — arbetsplikt (`[ÖPPEN]`)

Föräldrakooperativ har kvantifierad arbetsplikt i stadgarna (Svanen 20 dagar/år, Minigiraffen 15–17). Styrelseledamöter kan få reduktion. Uteblivna dagar kan leda till uteslutning.

Se [open-questions.md](../open-questions.md#-öppen-arbetsplikt-i-föräldrakooperativ) — är detta i scope för Tillsammans eller hänvisas till externt verktyg?

## Röstmodeller

**Default: 1 medlem = 1 röst** (huvudmetod enligt LEF 6:3-mönster). Empiriskt dominerande — används av 8 av 12 observerade föreningar i någon form.

**Alternativ (stadge-val):**

| Modell | Beskrivning | Empirisk förekomst |
|---|---|---|
| `ONE_MEMBER` | 1 medlem = 1 röst (default) | Rudolf Steiner GBG, Nya Elementar, Backatorp, Minigiraffen |
| `ONE_FAMILY` | 1 familj = 1 röst oavsett antal barn | KG Parents, Minigiraffen §12 |
| `BOTH_PARENTS` | Båda vårdnadshavare är medlemmar → familjen får 2 röster | Svanen |
| `ONE_PRESENT_PARENT` | 1 närvarande förälder = 1 röst (handuppräckning) | Skol-FF A (styrdokument) |
| `PER_CHILD_SPLIT` | 1 barn = 1 röst, splittat på närvarande vårdnadshavare | **Ingen observerad** |

`PER_CHILD_SPLIT` finns kvar som alternativ eftersom den är mer jämlik per barn (skyddar ensamstående mot kärnfamiljs övervikt), men ska inte marknadsföras som default — den skulle vara en praxisförändring, inte konfiguration.

**Stadgeändring krävs för byte av röstmodell** — röstregeln är stadge-bestämmelse, inte styrelsens tekniska val.

### Fullmakt

LEF 6:3 tillåter fullmakter om inte stadgarna säger annat. Flera observerade stadgar **förbjuder** ombudsröstning (Minigiraffen §12). Stöd båda: `proxyVotingAllowed: boolean` i stadge-data.

## Klasskassor

`Class`-entiteten ([domain-model.md#föräldraföreningar](../domain-model.md#föräldraföreningar-vårdnadshavare-minimal-barndata)) kan ha egen kassa. Implementationsvarianter:

- **Intern konto-rad** — klasskassan är en rad i föreningens huvudbok. Skol-FF A-modellen.
- **Separat bankkonto under föreningens juridiska paraply** — HAFF-modellen. Kräver separat redovisningsspår men skyddar blandning.

Kassörsvyn ska kunna projicera klass-saldo oavsett vilken modell föreningen kör. Kassör-stödet måste stödja båda.

**Klassrepresentant** (klassförälder) är en roll, inte en formell styrelseposition. Två per klass är vanligt. Har kommunikationsansvar, ej beslutsmandat. Rollen modelleras separat från styrelseroller.

## Underfonder

Skol-FF har ofta **strukturerade underfonder** inom föreningens ekonomi. Empiri från Skol-FF A:

- **Kulturfonden** (20% av årets intäkter, max innestående 50 000 kr, löpande styrelsebeslut)
- **Elevfonden** (fast belopp, konfidentiell hantering av rektor)
- **Bibliotek, elevkår** (fast belopp, ingen ansökan)
- **Projektbidrag** (ansökningsfönster, beslut på öppet möte)
- **Klassresebidrag** (reserverat tak, löpande ansökningar)

### Generisk modell

Varje fond har:
- **Fördelningsregel:** fast belopp / procent av totalen / ansökningsbaserat
- **Beslutsgång:** styrelsebeslut / öppet-möte-omröstning / årsmötesbeslut
- **Tak/golv:** max innestående (tvingar utdelning), min reserv (får ej delas ut)
- **Konfidentialitet:** `ConfidentialBudget`-flagga — pengar synliga, mottagare ej (elevfond-mönster, GDPR-skyddad)

Se [architecture.md](../architecture.md) för kärnabstraktionen "underfond under förening."

## Utskott

Delegerad beslutsrätt under styrelsen. Empiri: Skol-FF A:s **Eventkommittén** (operativ ledning av årets stora insamlingsevent) är utskott under styrelsen, eget mandat för mindre inköp, budget-preliminär 1 mån före event, slutrapport 14 dagar efter.

Generiskt utskotts-koncept:
- Valda separat på årsmötet (inte av styrelsen)
- Eget mandatperiod (kan avvika från styrelsens)
- Delegerad beslutsrätt inom definierat mandat och takbelopp
- Rapporterar till styrelsen via namngiven kontaktperson

Skilj från *arbetsgrupp* (informell, utan beslutsmandat) och *kommitté* (ofta synonym med utskott — välj en term i UI).

## Huvudmanna-relation

I vissa fall har föreningen **formellt representationsmandat** gentemot huvudmannen som driver skolan:

- **Backatorp § 5**: FF utser en ledamot till Stiftelsen Backatorpsskolans styrelse.
- **Skol-FF A**: stämman och valberedningen utser 5 ledamöter i huvudmannaskolans styrelse.

Modelleras som `ExternalMandate`: förening → extern entitet → antal platser, valprocess, mandatperiod.

**Detta är inte samma sak som att föreningen är styrelsen.** I kooperativ-förskolor *är* föräldraföreningen huvudmannen — det är en annan struktur som inte behöver `ExternalMandate`.

## Räkenskapsår

Två empiriskt observerade varianter:

- **Kalenderår** (1/1–31/12) — Minigiraffen, Rudolf Steiner GBG.
- **Brutet läsårsföljande** (1/7–30/6) — KG Parents, Svanen.

Backatorp specificerar *"per läsår"* utan datum — hanteras som 1/7–30/6.

**Stadge-konfiguration**, inte hårdkodad. Rapporterings-mallar måste kunna konfigureras för både varianter.

## Kallelsetid

Default: **minst 2 veckor** före årsstämma, **minst 1 vecka** före extra stämma.

**Max-gräns på kallelsetid stöds** (Minigiraffen-mönstret, §17: tidigast 6 veckor före). Skyddar mot att kallelsen kommer så tidigt att medlemmar glömmer datum. Konfigureras i stadge-data som `noticeMaxDays`.

## Stadgeändring

Empiriskt variationsintervall:

| Tröskel | Antal stämmor | Föreningar |
|---|---|---|
| 2/3 majoritet | 2 följande | Minigiraffen, Rudolf Steiner GBG |
| 3/4 majoritet | 2 följande, min 6 mån mellan | Backatorp |
| Enhälligt eller 2/3 | 2 följande | Svanen (upplösning) |
| 4 veckors mellanrum | 2 följande | Rudolf Steiner GBG |

**Alla har tröskel** (ingen tillåter stadgeändring med enkel majoritet på en stämma). Stadge-modul måste stödja två-stämmors-flöde med konfigurerbar tröskel och mellanrum.

### Stadgetolkning (mjuk-tolknings-mönstret)

Ibland tolkar stämman stadgans ord **mjukt** utan att ändra den (empiriskt fall 2019: "två suppleanter" tolkades som *minst* två). Detta är inte stadgeändring. Spec behöver separat mekanism: **dokumenterad tolkningsavvikelse**, länkad till beslutet men inte uppgraderad till stadgeändring. Se [core-concepts.md](../core-concepts.md) — öppen punkt.

## Vad som inte tillför värde i denna edition

Explicit bortvalt från edition-scope, trots att vissa föreningar gör det:

- **Swish-insamling för klasskassan** — använd Swish direkt; vi dokumenterar inflöden, hanterar dem inte.
- **Facebook-gruppmoderering** — föreningens FB-grupp är inte samma register som medlemsregistret. Håll åtskilda.
- **Insamlings-event-operativt** (bokning, schema, säkerhet) — externt verktyg. Vi dokumenterar beslut och fördelning, inte logistiken.
- **Event-inbjudningar och RSVP** — använd kalenderverktyg.

Gränsdragningen mot operativ funktionalitet enligt [mission.md](../mission.md) gäller oförändrat i denna edition.

## Onboarding för förtroendevalda — berättelsen

Eftersom jäv-deklaration och aktiv publikationsdisciplin är **utanför dominerande praxis**, måste onboarding aktivt motivera dem:

- *"Din förening har troligen ingen formell jäv-process idag. Här är varför det är värdefullt: [exempel från verification-report]."*
- *"Efter din mandatperiod kommer någon annan styrelse att ärva systemet. Allt du gör här kommer vara sökbart för dem utan extra arbete."*

Detta är **inte tekniska feature-skillnader** mot kärnan — det är försäljningstesen. Men den hör hemma i edition-filen eftersom den är typ-specifik.

## Referenser

- [verification-reports/foraldraforeningar-2026-04.md](../verification-reports/foraldraforeningar-2026-04.md) — empirisk bakgrund, 12 föreningar analyserade.
- [open-questions.md](../open-questions.md) — arbetsplikt, stadgetolkning.
- [domain-model.md#föräldraföreningar](../domain-model.md#föräldraföreningar-vårdnadshavare-minimal-barndata) — kärnans medlemsmodell.
- [core-concepts.md#röstning](../core-concepts.md#röstning) — röstmodell-grundlogik.
