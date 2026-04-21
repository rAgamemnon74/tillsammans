# Medlemskap

> Del av [Tillsammans-specifikation](../tillsammans.md).

Medlemsregistret är grundunderlaget för nästan allt systemet gör — röstlängd, kallelseutskick, ärendeinitiering, rättigheter till insyn. Det är också den mest personuppgiftstyngda datastrukturen i plattformen, och kräver därför skarpt beslut om vad som faktiskt ska samlas in.

Grundhållningen: **samla inte mer än absolut nödvändigt**, och håll editionerna strukturellt åtskilda eftersom "medlem" betyder olika saker i dem.

**Relaterade dokument:**
- [föreningen.md](föreningen.md) — medlemskapets plats i årshjulet
- [core-concepts.md](core-concepts.md) — röstning, röstlängd, och mandatverifiering för juridisk-person-representanter vid [stämmoöppning](core-concepts.md#röstlängdsetablering--processen-vid-stämmans-öppnande)
- [case-types.md](case-types.md) — medlemsansökan som ärendetyp
- [granskningslogg.md](granskningslogg.md) — medlemskapshändelser i loggen
- [roles/other-roles.md#medlemsansvarig](roles/other-roles.md) — delegerat godkännande av ansökningar
- [domain-model.md](domain-model.md) — entitetsrelationer, edition-specifika kopplingar

## Två editioner, två processer, ett register

Båda processer slutar i samma medlemsregister — samma `Medlem`-entitet — men vägen dit skiljer sig:

| Edition | Process | Trigger | Typisk beslutfattare |
|---|---|---|---|
| **Samfällighet** | Ägarövergång | Ny ägare till fastigheten | Styrelsen (registrerar) |
| **LEF (övriga ekonomiska)** | Formell ansökan | Person ansöker | Styrelsen |

Registret är gemensamt men de föregående processer skiljer sig i data som samlas in, vem som beslutar, och om insats krävs.

## Samfällighet — medlemskap följer fastigheten

Medlemskap är lagstyrt kopplat till fastighetsägarskap (LFS). Ingen ansökan — nya ägare blir medlemmar automatiskt när ägarbytet är klart. Ägaren kan vara **fysisk eller juridisk person** (aktiebolag, stiftelse, kommun, dödsbo) — LFS begränsar inte ägarform. Juridisk person är normalt förekommande, inte undantag; se [Juridisk person som medlem](#juridisk-person-som-medlem) nedan för mandat- och representationsmekaniken.

Praktiskt flöde:

- Fastighetsägare (typiskt den som säljer) eller styrelsen meddelar systemet om ägarbytet.
- Styrelsen uppdaterar registret manuellt: tidigare ägares medlemskap går till `VILANDE`, ny ägare skapas som `AKTIV`.
- `MEDLEMSKAP_ÖVERGÅTT`-händelse skrivs med båda medlemmarnas id och fastighetsbeteckningen som referens.

**Epok-tillhörighet.** `MEDLEMSKAP_ÖVERGÅTT` skrivs till **innevarande epok** (den öppna epoken vid registreringstillfället), oavsett när det faktiska ägarbytet skedde. Systemet lagrar två distinkta datum: `at` (metadata — tidpunkt för registrering i Tillsammans) och `actual_transfer_date` (systemdata — när ägarbytet tekniskt ägde rum, från kontraktsdatum eller Lantmäteriets registrering). Kassörens debiteringsberäkningar utgår från `actual_transfer_date` — inte registreringstidpunkten. Om det faktiska bytet skedde under en redan förseglad epok kan det aktualisera ett addendum till den stängda epoken; se [granskningslogg.md#retroaktiva-tillägg-addenda-till-stängd-epok](granskningslogg.md#retroaktiva-tillägg-addenda-till-stängd-epok).

### Data för samfällighetsmedlem

- **Fastighetsbeteckning** (unique id — vem är medlem bestäms av vem som äger fastigheten).
- **Ägarens namn** + kontaktkanal.
- **Andelstal per GA** där fastigheten ingår (se [editions/samfallighet.md](editions/samfallighet.md)).
- **Personnummer är opt-in** — bara om stadgan kräver eller för namnmatchning mot Lantmäteriets ägarregister. Se GDPR-sektionen nedan.

Samägda fastigheter har flera personer kopplade till samma fastighetsbeteckning — hanteras via `Ownership` med `sharePercent` ([core-concepts.md#samägda-fastigheter](core-concepts.md)).

## LEF (övriga ekonomiska föreningar) — formell ansökan

Fullständigt ansökningsflöde:

1. Person fyller i ansökan (namn, kontakt, motivering om stadgan kräver, samtycke till stadgar).
2. Styrelsen eller medlemsansvarig granskar.
3. Beslut: `GODKÄND` eller `AVSLAGEN`.
4. Vid `GODKÄND`: insats betalas enligt stadgans belopp.
5. Vid bekräftad insats: `MEDLEMSKAP_AKTIVERAT` — personen är medlem.

### Data för LEF-medlem

- Namn, kontaktkanaler.
- Samtycke till stadgar + stadgeversion.
- Insats-bekräftelse (belopp, betaldatum).
- Personnummer *om* stadgan kräver eller om utdelning/skatterättslig hantering förekommer (KU-rapportering, utbetalningar).
- Inget "orsak/motivation"-fält (se nedan).

## Juridisk person som medlem

Kontexten är primärt **samfällighet** (där LFS inte begränsar ägarform och juridisk person är normalt förekommande) och vissa **LEF** där stadgan tillåter.

### Ägarformer och actor-abstraktionen

Medlem är inte nödvändigtvis en fysisk person. Medlemsregistret bygger på en abstrakt `Actor`-entitet som kan vara:

- `NATURAL_PERSON` — fysisk person (standardmodellen ovan)
- `LEGAL_PERSON` — aktiebolag, stiftelse, ekonomisk förening, handelsbolag, kommanditbolag
- `ESTATE` — dödsbo (övergångsform mellan två ägarställen)
- `MUNICIPALITY` — kommun (specialfall med egen beslutsordning)

### Organisationsnummer som unik id

För `LEGAL_PERSON` är **organisationsnumret** (format `NNNNNN-NNNN`) obligatorisk unique id. Det är den auktoritativa identifieraren som används i alla externa register (Bolagsverket, Skatteverket, Lantmäteriet) och garanterar stabil identifiering oberoende av firmanamnsändring.

För `ESTATE` finns inget organisationsnummer. Systemet genererar internt UUIDv7 som stabil id; extern referens kan vara den avlidnes personnummer + bouppteckningsdatum.

För `MUNICIPALITY` används kommunkoden som externt id.

### Representation — fysisk person som röstar för juridisk

En juridisk person röstar aldrig själv. En fysisk person måste representera. Representation är en egen relation i modellen, åtskild från medlemskapet:

```
Representation
  actor_id                den juridiska personen (Actor)
  representative_actor_id fysisk person som representerar (Actor av typ NATURAL_PERSON)
  mandate_type            FIRMATECKNING / FULLMAKT / STYRELSEBESLUT / DELEGATIONSORDNING
  mandate_document_ref    uppladdad bilaga (Bolagsverket-utdrag, fullmakt, protokoll)
  valid_from, valid_until null = tillsvidare
  verified_at             tidpunkt för senaste verifikation
  verified_by             actor_id (styrelseledamoten som verifierade)
```

### Verifikation av mandat är styrelserutin

Verifieringen av representantens bemyndigande är en **styrelserutin** — typiskt delegerad till medlemsansvarig eller ordförande. Rutinen:

1. Representanten laddar upp bemyndigandedokument (Bolagsverket-utdrag, fullmakt, styrelseprotokoll eller delegationsordning — beroende på `mandate_type`).
2. Medlemsansvarig/ordförande granskar dokumentet och bekräftar behörigheten.
3. `MANDAT_VERIFIERAT`-händelse skrivs i granskningsloggen med referens till bilagan.
4. Representationen är giltig tills annat registreras.

Verifikationen är en **mänsklig handling** — systemet stödjer processen men avgör inte själv vad som är giltigt. Vid stämman kan mötesordföranden begära ny verifikation om dokumentet är gammalt eller om någon ifrågasätter det.

### Byte av representant

Hanteras **unikt per situation**. Ny representant = ny `Representation`-post som genomgår egen verifikation. Föregående representation får `valid_until` vid bytet och avslutas med `REPRESENTATION_UPPHÖRD`-händelse. Systemet automatiserar inte något — varje byte är en medveten registrering.

### Flera samtidiga representanter

Vissa AB har firma tecknad *"av två i förening"*. Praktiskt på stämma:

- Bemyndigandet specificerar *"i förening"* som mandatets attribut.
- Båda representanter måste närvara för att rösten ska kunna avges.
- Båda signerar röstlängden vid stämmans öppning.

**Om de är oeniga om en röst kan rösten inte avges** för den fastigheten. Föreningen avgör den specifika frågan utan den fastighetens röst.

**Om oenigheten blockerar en för föreningen viktig punkt:** extra stämma kallas. Den juridiska personen får då internt lösa oenigheten via sin egen styrelse eller ledningsfunktion innan den extra stämman hålls. Det är inte samfällighetsföreningens uppgift att medla i den juridiska personens interna governance-strider.

### Dödsbo som övergångsform

Formellt en juridisk person men med tydlig övergångskaraktär (från avliden ägare till en eller flera arvingar).

- Styrelsen informeras vid dödsfall, typiskt av närstående eller boutredningsman.
- `Estate`-actor registreras med referens till den avlidne och till fastigheten.
- Representation: boutredningsman om förordnad, annars dödsbodelägare med fullmakt från övriga delägare.
- Vid arvskifte: `MEDLEMSKAP_ÖVERGÅTT` för dödsboet → en eller flera nya ägare. Dödsboets status → `VILANDE`.

Under övergångstiden (typiskt månader till år) kan dödsboet rösta via sin representant enligt normala regler.

### Kommun

- Representant är typiskt en tjänsteperson vid den förvaltning som ansvarar för fastigheten (fastighetskontoret, fritidsförvaltningen eller liknande).
- `mandate_type = DELEGATIONSORDNING` — kommunallagens delegering ersätter firmateckning.
- Vissa äldre samfällighetsstadgar har historiska röstbegränsningar för kommun (för att förhindra att kommunens andelstal dominerar privatägare). Stadgekonfigurerade begränsningar respekteras; systemet ifrågasätter inte — det är föreningens juridiska fråga.

### Samägande mellan fysisk och juridisk person

Samma mekanik som för fysiskt samägande ([core-concepts.md#samägda-fastigheter](core-concepts.md)):

- `UNIFIED` (default): fastigheten avger en samlad röst. Ägarna enas internt; om inte → ingen röst.
- `PROPORTIONAL`: röstvikten splittras enligt `sharePercent`. AB röstar exempelvis 60 % av fastighetens andelstal; privatpersonen 40 %.

Juridisk och fysisk person kan fritt samäga — ingen begränsning.

### Vad systemet uttryckligen *inte* gör

Per [policy 8](mission.md#grund-policies) ligger identifiering av skuggföretag, brevlådebolag eller strukturer där verklig huvudman döljs utanför uppdraget. Systemet:

- Registrerar fakta: organisationsnummer, legal_form, representant, bemyndigandedokument.
- Tillgängliggör underlaget för förtroendevalda och revisor.
- Loggar mandatverifikationer och representationsbyten spårbart.

Det som avgör är **verifikationsrutinen** (någon har granskat dokumentet) och **förtroendevaldas moraliska kompass** (om något ser fel ut, agera på det). Systemet levererar underlag; människan granskar.

## Grunddata — översikt per edition

| Fält | Samfällighet | LEF |
|---|---|---|
| Namn | ✓ | ✓ |
| E-post | rekommenderat | ✓ |
| Telefon | frivilligt | frivilligt |
| Postadress | rekommenderat | ofta |
| Fastighetsbeteckning | ✓ (unique id) | — |
| Andelstal per GA | ✓ | — |
| Personnummer | opt-in per stadga | opt-in per stadga |
| Födelsedatum | frivilligt | frivilligt |
| Insats | — | ✓ |
| Samtycke stadgar | ✓ | ✓ |
| Orsak/motivation | — | **aldrig** |

**Varför inte "orsak/motivation"?** Två skäl: (1) blir sällan meningsfull, (2) öppnar diskrimineringsrisk om styrelsen avslår baserat på olämplig motivation (stadge-brott mot likabehandling, potentiell diskrimineringslagen). Ersätts av *samtycke till stadgarna* + *insats-bekräftelse*.

## Personnummer — explicit opt-in, aldrig default

Båda editionerna är opt-in med motivering.

| Situation | Motiverar opt-in? |
|---|---|
| Stadgan kräver personnummer-matchning | Ja |
| LEF har skatterättsligt relevanta utbetalningar (utdelning, KU-rapportering) | Ja |
| Samfällighet behöver matcha Lantmäteriets ägarregister *internt* | Ja |
| Föreningen har bankkontakter som kräver personnummer | Ja |
| Bara för att "det är praktiskt att ha" | Nej |

Aktivering sker i onboarding med registrerad motivering. Vid aktivt:

- Alltid maskat i vyer som standard (`YYMMDD-****`).
- "Visa fullständigt"-klick åtkomstloggas (samma mönster som Hemmets GDPR-design).
- Synligt endast för ordförande, kassör och revisor.
- Aldrig i publika vyer eller medlemslistor som visas för andra medlemmar.

## Livscykel

Bygger på `AKTIV` / `VILANDE` / `UTESLUTEN` från [domain-model.md](domain-model.md). De två editionerna når `AKTIV` via olika vägar — LEF genom ansöknings-fas, samfällighet direkt via ägarbyte.

### LEF-ansöknings-flöde

Ansökningsfasen adderar fem tillstånd före `AKTIV`, inklusive en överklagan-gren enligt LEF 4:3:

```
INLÄMNAD → GODKÄND → AKTIV
         → AVSLAGEN → ÖVERKLAGAD_TILL_STÄMMA → AKTIV     (stämman bifaller)
                                              → AVSLAGEN (stämman fastställer)
         → ÅTERKALLAD   (sökanden drar tillbaka före beslut)

AKTIV → VILANDE   (utebliven avgift eller annan extern trigger)
      → UTESLUTEN (stadgeenligt förfarande)

VILANDE → AKTIV   (reconfirmation eller ny avgift)
        → (arkiveras efter stadgad tid)
```

**Överklagan till stämman (endast LEF, LEF 4:3).** En sökande som fått `AVSLAGEN` kan begära att stämman prövar beslutet inom ett stadge-bundet tidsfönster (typiskt 1 månad efter avslaget). Övergången `AVSLAGEN → ÖVERKLAGAD_TILL_STÄMMA` triggas av sökanden själv — styrelsens avslag kan inte hindra begäran. Styrelsen förbereder yttrande på samma sätt som för motion, och ärendet publiceras som del av kallelsen till nästa ordinarie eller extra stämma. Stämman avgör: bifall → `AKTIV` (eller `GODKÄND` om insats ännu inte är betald); avslag → återgång till `AVSLAGEN` slutgiltigt (ingen ytterligare överklagan inom systemet). Efter ett slutgiltigt avslag gallras personuppgifter enligt samma 6-månaders-regel som ordinarie avslag.

### Samfällighets-flöde (ägarbyte)

Ingen ansökningsfas — styrelsens registrering av ägarbytet aktiverar medlemskapet direkt:

```
(ingen ansökan) → AKTIV   (styrelsen registrerar ägarbytet via MEDLEMSKAP_ÖVERGÅTT)

AKTIV → VILANDE   (tidigare ägare vid försäljning — MEDLEMSKAP_ÖVERGÅTT
                   refererar båda aktörer: ny ägare AKTIV, gamla ägare VILANDE)
      → UTESLUTEN (stadgeenligt förfarande — sällan tillämpligt i samfällighet
                   eftersom medlemskap följer fastigheten enligt LFS)

VILANDE → AKTIV   (vid tillbakaköp eller liknande — ovanligt men möjligt)
        → (arkiveras efter stadgad tid)
```

`INLÄMNAD`, `GODKÄND`, `AVSLAGEN` och `ÅTERKALLAD` aktualiseras aldrig för samfällighetsmedlemskap — de saknar ansökningssteg.

### Tillstånd

- **INLÄMNAD** — ansökan inlämnad, väntar på beslut.
- **GODKÄND** — godkänd, väntar eventuell insats-betalning (LEF).
- **AKTIV** — fullständig medlem, alla rättigheter aktiva.
- **VILANDE** — tidigare aktiv men har fallit ur (ej betald avgift, fastighet såld och ägarbyte registrerat, själv-avsagd utan formell uteslutning).
- **UTESLUTEN** — utesluten via stadgeenligt förfarande.
- **AVSLAGEN** — ansökan avslagen.
- **ÖVERKLAGAD_TILL_STÄMMA** — avslagen ansökan där sökanden begärt stämmoprövning enligt LEF 4:3 (endast LEF).
- **ÅTERKALLAD** — sökanden drog tillbaka.

### Gallring

- `AVSLAGEN` och `ÅTERKALLAD`: personuppgifter gallras **6 månader** efter beslut. Grundmetadata (beslutsreferens, datum) bevaras i granskningsloggen för audit.
- `VILANDE`: personuppgifter bevaras tills stadgestyrd gallringstid (typiskt 2–5 år); ekonomi-anknuten data bevaras 7 år enligt bokföringslagen.
- `UTESLUTEN`: samma som VILANDE, plus uteslutningsbeslutets fulltext bevaras som del av föreningens formella historik.
- `AKTIV`: ingen gallring (medan medlemmen är aktiv).

## Samtyckes-förnyelse vid stadgeändring

Stadgan kan ändras på sätt som påverkar medlemmens rättigheter och skyldigheter. Då krävs förnyat samtycke:

- **Väsentlig stadgeändring** (rättigheter, skyldigheter, avgifter, röstregler): existerande medlemmar får notifiering vid nästa inloggning samt via registrerad kontaktkanal. Obligatorisk bekräftelse inom stadgestyrd tidsfrist (default: nästa årsstämma eller 90 dagar, beroende på vilket kommer först).
- **Redaktionell ändring** (språkjusteringar, tekniska förtydliganden): ingen förnyelse; medlemmen informeras men ingen aktion krävs.

Om medlemmen inte förnyar inom fristen aktiveras `VILANDE` med motivering *"samtycke till stadgeversion X ej förnyat"*. Medlemmen kan när som helst bekräfta och återgå till `AKTIV`.

Kategorisering (*väsentlig* vs *redaktionell*) avgörs av styrelsen i samband med stadgeändringsbeslutet och är del av stadgeändringens protokoll.

## Delegerat godkännande — medlemsansvarig

För att undvika fullständig styrelsebehandling av varje rutinansökan finns **medlemsansvarig** som attribut på en styrelseledamot. Se [roles/other-roles.md#medlemsansvarig](roles/other-roles.md#medlemsansvarig) för RBAC-detaljer.

Sammanfattningsvis:

- Medlemsansvarig får godkänna *normala* ansökningar inom delegeringsmandatet.
- Avslag och eskalering till fullständig styrelse gäller vid tveksamhet eller kontroversiell ansökan.
- **Obligatorisk jäv-deklaration vid varje beslut** — medlemsansvarig måste aktivt bekräfta att inget intressekopplat förhållande finns till sökanden (make, sambo, släkting i rakt upp- eller nedstigande led eller syskon, affärspartner, annan närstående). Deklarationen lagras i beslutshändelsens systemdata och granskas av revisorn löpande. Vid jäv eller misstänkt jäv eskaleras ärendet till fullständig styrelse där den berörda ledamoten avstår enligt vanliga jäv-regler ([core-concepts.md#jäv](core-concepts.md#jäv-conflict-of-interest)).
- Alla beslut loggas med `actor_id = medlemsansvariges_id` i granskningsloggen.
- Delegeringsmandatet fastställs som stadgebeslut eller permanent styrelsemandat; kan återkallas.

## Register-struktur — referensmodell

Grundbygget är en `Actor`-abstraktion som kan vara fysisk, juridisk, dödsbo eller kommun. `Medlem` är då en länk mellan en `Actor` och en förening.

```
Actor (bas)
  actor_id                UUIDv7 (intern stabil identifierare)
  actor_type              NATURAL_PERSON / LEGAL_PERSON / ESTATE / MUNICIPALITY
  name                    personnamn eller firmanamn
  
  # Typ-specifika fält (populerade beroende på actor_type)
  
  -- För NATURAL_PERSON
  förnamn, efternamn
  födelsedatum             null om ej lämnat (för åldersbaserade stadgeregler)
  personnummer             null om inte aktivt i föreningen
  
  -- För LEGAL_PERSON
  organisationsnummer      obligatorisk unique id (NNNNNN-NNNN)
  legal_form               AKTIEBOLAG / STIFTELSE / EKONOMISK_FÖRENING / 
                           HANDELSBOLAG / KOMMANDITBOLAG / …
  registered_address       Bolagsverkets registrerade adress
  contact_person           vem på organisationen som tar emot post (inte röstbemyndigad)
  
  -- För ESTATE
  deceased_person_id       den avlidne
  probate_deed_date        bouppteckningsdatum
  dispositive_until        förväntat arvskifte (null om okänt)
  
  -- För MUNICIPALITY
  municipality_code        kommunkoden
  responsible_department   vem i kommunen som hanterar fastigheten

Medlem
  medlem_id                UUIDv7 — äkta primärnyckel, intern och stabil
  actor_id                 FK → Actor
  primär_id                söknyckel/visningsnyckel, inte nödvändigtvis unik:
                            - samfällighet + valfri typ: fastighetsbeteckning
                            - LEF + NATURAL_PERSON: person_id (intern)
                            - LEF + LEGAL_PERSON: organisationsnummer
  kontakt_epost
  kontakt_telefon
  kontakt_postadress
  preferred_channel        EMAIL / PHONE / POST
  samtycke_stadgar_vid     timestamp
  samtycke_stadgar_version stadgeversionen vid samtycke
  insats_betald_vid        null om ingen insats krävs
  status                   INLÄMNAD / GODKÄND / AKTIV / VILANDE / UTESLUTEN / AVSLAGEN / ÅTERKALLAD
  vilande_orsak            null, eller textkod för varför VILANDE

Representation  (bara relevant för LEGAL_PERSON / ESTATE / MUNICIPALITY)
  actor_id                 den juridiska entiteten
  representative_actor_id  fysisk person som representerar (Actor av NATURAL_PERSON)
  mandate_type             FIRMATECKNING / FULLMAKT / STYRELSEBESLUT / DELEGATIONSORDNING
  mandate_scope            ENSAM / I_FÖRENING / SPECIFIK_FRÅGA
  mandate_document_ref     uppladdad bilaga
  valid_from, valid_until  (null = tillsvidare)
  verified_at              tidpunkt för senaste verifikation
  verified_by              styrelseledamot som verifierade
```

Data-relationen till fastigheter (Samfällighet: PropertyUnit via Ownership med samägande möjligt) hanteras utanför detta register — se edition-filerna och [domain-model.md](domain-model.md).

**Unikhet och samägande.** `medlem_id` (UUIDv7) är den äkta primärnyckeln och är alltid unik per Medlem-post. `primär_id` är en mänskligt läsbar söknyckel som *inte* garanteras vara unik — vid samfällighets-samägande delar två eller flera Medlem-poster samma fastighetsbeteckning, en post per ägare. Disambiguering i UI sker via `actor.name` och `Ownership.sharePercent` i relaterad PropertyUnit-data. För LEF är `primär_id` i praktiken unik eftersom samma aktör inte kan ha två aktiva medlemskap i samma förening.

## Händelser i granskningsloggen

Medlemskapshändelser som tillhör registret:

**Aktörer:**
- `AKTOR_REGISTRERAD` — ny aktör (fysisk, juridisk, dödsbo, kommun) registrerad
- `AKTOR_UPPDATERAD` — ändring av aktörens kontaktuppgifter, firmanamn eller motsvarande

Aktör-händelsen föregår alltid första medlemskapshändelsen för aktören — en aktör måste existera i systemet innan ett medlemskap kan skapas. Samma aktör kan ha flera medlemskap (t.ex. ett AB som äger två fastigheter i samma samfällighet, eller en person som är medlem i både en samfällighet och en LEF): `AKTOR_REGISTRERAD` skrivs en gång per aktör, `MEDLEMSKAP_*`-händelser över tid per medlemsrelation.

**Medlemskap:**
- `MEDLEMSKAP_ANSÖKAN_INLÄMNAD` — INLÄMNAD
- `MEDLEMSKAP_GODKÄNT` — GODKÄND
- `MEDLEMSKAP_AVSLAGET` — AVSLAGEN
- `MEDLEMSKAP_ÖVERKLAGAT` — sökande begär stämmoprövning av avslag (LEF 4:3, endast LEF)
- `MEDLEMSKAP_ÅTERKALLAT` — ÅTERKALLAD
- `MEDLEMSKAP_AKTIVERAT` — AKTIV (efter ev. insats eller efter stämmans bifall av överklagan)
- `MEDLEMSKAP_VILANDE` — VILANDE (automatiskt eller manuellt)
- `MEDLEMSKAP_UTESLUTET` — UTESLUTEN
- `MEDLEMSKAP_BEKRÄFTAT` — förnyat samtycke eller reconfirmation
- `MEDLEMSKAP_ÖVERGÅTT` — samfällighetens ägarbyte (två aktörer berörda)

Kontaktuppgift-uppdatering (ny e-post, ny telefon, ny postadress) hanteras via `AKTOR_UPPDATERAD` — inte egen medlemskapshändelse, eftersom kontaktuppgifter hör till aktören snarare än till medlemsrelationen.

**Representation (juridisk person / dödsbo / kommun):**
- `REPRESENTATION_REGISTRERAD` — ny representant bemyndigad för juridisk aktör
- `REPRESENTATION_UPPHÖRD` — representant byter eller mandatet återkallas
- `MANDAT_VERIFIERAT` — bemyndigandedokumentet granskat och godkänt av styrelseledamot

Alla händelser skrivs enligt [granskningslogg.md](granskningslogg.md) med fyra delar (metadata, fritext, systemdata, ev. bilagor).

## Öppna frågor

- **Samtyckes-förnyelse vid mindre stadgeändringar.** Gränsdragningen mellan "väsentlig" och "redaktionell" är subjektiv. Förslag: stadgan kan föreskriva en kategorisering vid varje ändring; revisorn granskar kategoriseringen.
- **Medlemsansvarig-mandatets räckvidd.** Ska ett specifikt belopp/kvantitet av delegerat mandat sättas (t.ex. "max 50 godkännanden per månad innan återrapportering till styrelsen")? Förslag: stadgan/föreningen bestämmer; systemet rapporterar siffror löpande till styrelsen oavsett.
- **Lantmäteri-integration.** Tillsammans importerar inte från Lantmäteriet (per [mission.md](mission.md)), men för samfälligheter är det en reell friktion att ägarbyten måste uppdateras manuellt. Framtida opt-in för API-läsning? `[ÖPPEN]` — tydligt post-MVP.
- **Historisk läsrätt för medlemmar.** Tidigare förtroendevalda har permanent läsrätt till sina epoker ([föreningen.md#historisk-läsrätt-efter-mandatet](föreningen.md)). Vad gäller för *vanliga medlemmar* som lämnat föreningen? Förslag: tillgång upphör vid `VILANDE`/`UTESLUTEN` förutom till de publika vyerna under epokerna de var aktiva medlemmar i. De har inte samma juridiska skyddsbehov som förtroendevalda.
