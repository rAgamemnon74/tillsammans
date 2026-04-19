# Tillsammans för samfälligheter — edition-spec

> Del av [Tillsammans-specifikation](../../tillsammans.md). Den här filen dokumenterar vad som är *specifikt* för `AssociationType = samfällighet`. Kärnans governance-flöden ligger i övriga spec-filer; här finns edition-specifika defaults, stadge-alternativ och domänmönster.
>
> Empirisk bakgrund: [verification-reports/samfalligheter-2026-04.md](../verification-reports/samfalligheter-2026-04.md) (10 föreningar analyserade).

## Vad editionen täcker

Tillsammans för samfälligheter är rollstöd för förtroendevalda i svenska samfällighetsföreningar — juridiska personer under [LFS (1973:1150)](../legal-context.md) som förvaltar en eller flera gemensamhetsanläggningar bildade enligt [Anläggningslagen (1973:1149)](../legal-context.md).

Primärexponerade roller: **ordförande och kassör parallellt** (se [audience.md](../audience.md#implikation-för-mvp-roadmap-per-edition)). Ordföranden bär governance-skölden vid sakägartvister och asymmetriska påtryckningar; kassören bär den juridiskt snåriga debiteringslängden. Båda är primära.

## Sub-typer

Editionen täcker fem strukturellt olika samfällighetstyper. Alla lyder under LFS men har olika komplexitet i andelstal, stämma och ekonomi:

| Sub-typ | Typisk skala | Vad förvaltas | Röstning-komplexitet |
|---|---|---|---|
| **Ren vägsamfällighet** | 20–200 fastigheter | En GA (vägen) | Låg — platt eller andelstal per fastighet |
| **Multi-GA radhus-tätort** | 50–250 fastigheter | Värme, vatten, väg, grönytor (ofta flera GA) | Medel — platt per fastighet dominerar |
| **Skärgård/sommarstuge** | 50–250 fritidshus | Vatten, bryggor, väg, allmänningar, delrätter | Hög — delmängds-GA, olika medlemskategorier |
| **Vattenförening** | 20–100 fastigheter | Gemensamt VA, pumpstationer | Medel — ofta en fastighet utanför (servitut-fall) |
| **Fjäll-/landsbygds-väg** | 30–500 fastigheter | Vägnät med blandat permanent/fritid | Medel — distansdeltagande vanligt |

Distinktionen styr onboarding-wizard, röstmodell-defaults och kassörs-funktioner (t.ex. hur debiteringslängden genereras när olika GA har olika medlemsomfång).

## Juridiskt ramverk

Stadgar för samfälligheter är **starkt lagstyrda**. LFS tvingar formalia för:

- Stämmor och kallelsetider
- Röstning (huvudmetod default, andelstal på begäran, 1/5-tak)
- Majoritetskrav för stadgeändring och upplösning
- Styrelsens solidariska ansvar (LFS 47§)
- Jäv vid beslut (LFS 36/48§)
- Debiteringslängd som årlig output

**Lantmäteriets normalstadgar** är de facto mall — alla 10 granskade föreningar följer dem. Stadge-modulen i editionen levererar normalstadgorna som **parametriserad wizard** — inte fritext-stadga.

## Onboarding via normalstadge-wizard

Nya samfälligheter bygger stadgan genom att välja värden i normalstadgans parametrar:

- Föreningens säte (kommun)
- Antal styrelseledamöter (typiskt 5, intervall 3–8) + suppleanter (0–3)
- Mandatperiod per roll (ordf 1 eller 2 år; ledamot 1 eller 2 år; rullande)
- Antal revisorer (1–2) + suppleanter
- Kallelsetid min (default 2v) och max (default 4v) — max-gränsen skyddar mot glömska
- Stämmoperiod (kalendermånad för ordinarie stämma)
- Räkenskapsår (kalender / sommar-säsong 1/5–30/4 / läsår 1/7–30/6 / annat)
- Fondavsättning-regel (procent av anläggningsvärde eller fast krontal, min/max tak)
- Kallelsemetod (hemsida / e-post / lokaltidning / kombination)
- Motioner-tidsfönster (typiskt 2v före stämma)

Stadge-output är en pdf/signeringsbar text som motsvarar normalstadgorna med valda värden — direkt användbar hos Lantmäteriet.

### Äldre samfälligheter — stadga laddas in som kopia

Samfälligheter bildade före 2000-talet har ofta 20+ år gamla stadgar som motsvarar ett pappersoriginal. Onboarding ska låta föreningen *ladda upp sin existerande stadga* utan att "moderniseras" — spec-modellen ([verification.md](../verification.md) Spår 2, scenario 11). Systemet behandlar den uppladdade stadgan som sanning, även om den avviker från dagens normalstadga.

## Röstmodeller

**Default: huvudmetod som grundregel (LFS 49§).**

- `MAJORITY_BY_HEAD` — 1 medlem = 1 röst, oavsett fastighetens andelstal (default)
- `MAJORITY_BY_SHARE_ON_REQUEST` — vid ekonomisk fråga kan medlem begära andelstalsröstning; då tillämpas `Participation.andelstal` för den frågan, med tak `voteCapPerMember = 0.2` (20% av totala röstetalet per medlem, SFL 49§)
- `MAJORITY_BY_HEAD_ENFORCED` — stadgeändring och vissa LFS-tvingade frågor kräver alltid huvudmetod (SFL 52§)

### Alternativ modell: `PER_GA_SHARE` (på stadgestöd)

Vissa föreningar vill att GA-specifika beslut alltid använder andelstal för just den GA:n, utan att någon behöver begära det. Detta är stadge-beslut, inte default. Används när en multi-GA-förening vill att vägfrågor automatiskt viktas efter väg-GA:s andelstal.

### Ägarformer — fysisk, juridisk, dödsbo, kommun

Samfällighetens fastighetsägare är en **blandad kår**. Juridisk person (aktiebolag, stiftelse, kommun) är normalt förekommande, inte undantag:

- **Privatpersoner** dominerar antalsmässigt.
- **Aktiebolag** som äger kontors-, industri- eller uthyrningsfastigheter.
- **Ekonomiska föreningar** som äger kooperativa hyresrättshus.
- **Stiftelser** som äger institutionsfastigheter (skola, hemvärnsgård).
- **Kommuner** — vanligt när allmänna byggnader eller ytor ingår i området.
- **Bank** som tillfälligt innehar efter exekutiv auktion.
- **Dödsbo** som övergångsform mellan ägare.

Modellering (Actor-abstraktion, Representation, mandat-verifiering) specificeras i [medlemskap.md#juridisk-person-som-medlem](../medlemskap.md#juridisk-person-som-medlem). Mandatverifikation vid stämma är del av röstlängdsetableringen — se [core-concepts.md#röstlängdsetablering](../core-concepts.md#röstlängdsetablering--processen-vid-stämmans-öppnande). Samägande mellan fysisk och juridisk person hanteras enligt [samägda fastigheter](#samägda-fastigheter) nedan.

### Samägda fastigheter

- **`UNIFIED` (default)** — fastigheten avger en samlad röst med hela sitt andelstal. Enas inte ägarna internt avges ingen röst. Matchar Torpa Skogs §19 *"en per fastighet"*.
- **`PROPORTIONAL`** — `Ownership.sharePercent` splittar röstvikten. Sällsynt; kräver explicit stadge-stöd.

### Fullmakter

**Default: `maxProxiesPerAgent = 1`** (SFL 49§). Ett ombud får företräda högst en medlem utöver sig själv.

Vissa stadgar tillåter mer eller förbjuder fullmakter helt — konfigurerbart i stadge-data. Fullmakt måste vara skriftlig och registreras före stämman.

## Andelstal och debiteringslängd

### Andelstal som data

Varje `Participation` (PropertyUnit ↔ GA) har:

- `andelstal: Decimal(10, 6)` — lagrat som decimal oavsett hur det presenteras
- `source` — anläggningsbeslut (förrättningsdatum), stämmobeslut, historisk uppgift
- `visualization` — bråk (1/80), procent (1.25%), decimal (0.0125) — alla tre kan visas; lagringen är alltid decimal

Systemet *varnar* om andelstalen inte summerar till 1.0 per GA — men tvingar inte. Historisk data kan vara felaktig (se [lapptäcket-modellen](../domain-model.md#lapptäcket)).

### Debiteringslängd som automatisk output

Debiteringslängden är lagkrav (LFS) och årlig produkt. Den genereras automatiskt från:

1. Budget per GA (styrelsens förslag, stämmans beslut)
2. Andelstal per fastighet per GA (`Participation.andelstal`)
3. Fakturerings-period och förfallodagar (stadge-parameter)

Output är en lista per fastighet med rad per GA och summa. Fastigheten kan ha olika `faktureringsmottagare` från formell ägare (viktigt för dödsbon, arvsskiften) — se [domain-model.md#samfälligheter](../domain-model.md#samfälligheter--fastighetsanknutet-informellare-än-brf).

Systemet genererar debiteringslängden; kassören granskar och stämman fastställer (lagligt bindande fastställande).

### Multi-GA och kostnadsfördelning

Multi-GA-föreningar fördelar ofta kostnader mellan GA enligt fastslagna procent-nycklar (Hule 68/27/5). Detta är *inte* andelstal — det är **GA-vikter** vid fördelning av gemensamma styrelsekostnader eller administrativa omkostnader.

Modelleras som `Association.gaCostWeights: Map<GAId, Decimal>` separat från `Participation.andelstal`.

## Extern ekonomi-förvaltning

Empiriskt vanligt (Kyrkmossen → HSB, Fröåvägen → CarLe Ekonomi AB): föreningen delegerar bokföring, fakturering och årsredovisning till extern partner med avtal och arvode.

Modelleras som `ExternalAdministrator`-roll:

- Tidsbegränsat mandat (avtalets löptid)
- Stadge-bestämt scope (bokföring / fakturering / årsredovisning / hela)
- Läs/skriv-rättigheter mot ekonomiska vyer, läsrätt mot övriga
- Inget beslutsmandat — alla beslut sker fortsatt av styrelsen

Extern administratör är inte medlem, har inte rösträtt. Upphör när avtalet gör det.

## Fondavsättning / underhållsfond

Samfälligheter har **lagkrav på underhållsfond** för större anläggningar. Empiriska modeller:

- **Procent av anläggningsvärde** (Äskestock: min 0,3%/år)
- **Fast krontal per år med tak** (PSV: min 10 000 kr/år, max 700 000 kr total)
- **Avsättning beslutat per stämma** (Fröåvägen: höjt till 100 000 kr/år via stadgeändring 2025)

Modelleras som `Fund`-entitet enligt [architecture.md](../architecture.md) underfonder-konceptet: fördelningsregel, tak/golv, beslutsgång.

## Räkenskapsår

Tre empiriska mönster:

- **Kalenderår** (1/1–31/12) — dominerar för permanent-bebodda
- **Sommar-säsong** (1/5–30/4) — fritidshus-föreningar (Äskestock)
- **Augusti–augusti** — stämma i augusti då sommarboende är på plats (Rörumstrand)

Stadge-inställning, inte hårdkodad.

## Stämma och kallelse

- Kallelsetid min 2v ordinarie stämma, 1v extra. Normalstadgar.
- Max-gräns konfigurerbar (default 4v; Torpa Skogs *"tidigast 4 och senast 2 veckor"*). Skyddar mot glömska.
- Kallelsemetod enligt stadgarna — hemsida, e-post, lokaltidning, eller kombination. Rörumstrand använder anslag på tavlor + skriftligt till medlemmar + e-post vid samtycke.
- Stämmoperiod (kalendermånad) konfigurerbart. Sommarkommuner kör ofta juli–augusti.
- Motioner in senast 2v–1 mån före stämma.

## Solidariskt ansvar och reservation

LFS 47§ solidariskt ansvar nämns aldrig i stadgarna — det är implicit via lagen. Detta är precis det mönster [core-concepts.md#reservation-och-solidariskt-ansvar](../core-concepts.md) adresserar: ansvaret är latent, aktualiseras bara vid rättsprocess, och styrelseledamöter upptäcker det ofta för sent.

Spec-mekanismen (synlig individuell röstning + enkel reservation + personlig exponerings-vy) är särskilt värdefull i samfällighetskontext där ekonomiska beslut kan ha miljon-konsekvenser över tid (väg-renovering, VA-ombyggnad).

## Jäv — onboarding-berättelse

Noll av 10 granskade stadgar har obligatorisk jävsdeklaration. Detta **är** hotet [threats.md](../threats.md) adresserar — *"asfaltera vägen fram till ordförandens hus"*.

Onboarding för samfällighets-editionen ska aktivt motivera jäv-modulen:

- *"Er förening har troligen inget strukturerat jäv-stöd idag. Här är varför det är värdefullt: [konkret exempel från verification-rapporten]."*
- *"Alla beslut där en styrelseledamot har ekonomiskt intresse blir synliga i revisorns vy. Det skyddar både styrelsen och medlemmen."*

Detta är försäljningstesen, inte teknisk feature-skillnad mot kärnan.

## Vad som inte tillför värde i denna edition

Explicit bortvalt från edition-scope:

- **Operativ anläggnings-förvaltning** (vägunderhåll, asfaltering, grävschakt) — använd externa verktyg. Vi dokumenterar beslut och avtal, inte logistiken.
- **Bokföring** — extern administratör eller separat bokföringssystem. Kassörsstödet exporterar underlag.
- **Vattenmätning/energimätning** — externt. Debiteringslängden läser *resultatet*, inte rå-mätvärdena.
- **Vägbokningssystem, släpvagnsbokning, badplats-reservation** — inte governance-frågor.

Gränsdragningen mot operativ funktionalitet enligt [mission.md](../mission.md) gäller oförändrat.

## Referenser

- [verification-reports/samfalligheter-2026-04.md](../verification-reports/samfalligheter-2026-04.md) — empirisk bakgrund
- [domain-model.md](../domain-model.md) — PropertyUnit, Ownership, Participation, GA, lapptäcket
- [core-concepts.md#röstning](../core-concepts.md#röstning) — huvudmetod/andelstalsmetod
- [legal-context.md](../legal-context.md) — LFS, Anläggningslagen
- [open-questions.md](../open-questions.md) — andelstal-representation, autentisering
