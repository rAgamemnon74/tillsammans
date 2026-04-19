# Ärendetyper

> Del av [Tillsammans-specifikation](../tillsammans.md).

Nästan varje governance-handling som plattformen stödjer är ett *ärende* — en motion, en medlemsansökan, en jävsdeklaration, ett utlägg. Istället för att uppfinna varje ärendeflöde separat definieras en gemensam struktur (livscykel, roller, texter, audit, publicering) som varje ärendetyp ärver från. Det här dokumentet är biblioteket.

**Fokus:** styrelseprocessstöd — att ärenden glider genom styrelsens arbete utan att fastna i formalia, samtidigt som varje steg är spårbart.

## Principer för ärendetyperna

1. **Transparens är default för de ärendetyper MVP börjar med.** Motion, styrelseförslag, utlägg, stadgeändring, kandidatnominering, revisorsfråga har publik eller medlem-läsvy efter publiceringstillfället. Undantag måste motiveras per ärendetyp — inte per ärende.
2. **Personärenden är *explicita* undantag.** Uteslutning, medlemsansökan under beredning, disciplinära varningar har avgränsad insyn. Regeln från [mission.md](mission.md): *"Inte full insyn i pågående ärenden."*
3. **GDPR — fältfilter per roll.** Inga publika kontaktuppgifter utan samtycke, gallringsregler kopplade till ärendestatus. `[ÖPPEN]`: importera motsvarande struktur som Hemmets 8 GDPR-principer till egen sektion i [architecture.md](architecture.md).
4. **God och samarbetsvillig ton.** Varje ärendetyp har ett *textbibliotek* med välformulerade mallar — bekräftelser, kompletteringsfrågor, avslag, yttranden. UI föreslår, styrelsen anpassar. Principen är saklig konstruktiv ton, inte byråkrati, inte konflikteskalation. Se [Textbiblioteket](#textbiblioteket) nedan.
5. **Stadgeförankring är obligatorisk.** Varje ärende måste peka på minst en stadgaparagraf. Detta är samma krav som redan gäller beslut ([architecture.md#kärnmoduler](architecture.md#kärnmoduler)) — drar ut det en nivå tidigare så att ärendet *föds* med sin juridiska förankring.
6. **Beslut är terminus.** Ett ärende avslutas via beslut på möte/stämma eller via formellt avvisande/återkallelse. Beslutet slutloggas i `DecisionLog`; ärendet är vägen dit. Ingen ärendetyp får sin egen parallella beslutsmekanism.
7. **Enhetlig livscykel.** Alla ärendetyper använder samma grundmönster: `CaseStatus` enum + tillåtna övergångar per typ + RBAC per övergång + audit-logg per övergång. Jäv, bordläggning och reservation är *övergripande mekanismer* ([core-concepts.md](core-concepts.md)) som fungerar på alla ärendetyper utan duplicering.

## Gemensam struktur — mall för varje ärendetyp

Varje ärendetyp i biblioteket dokumenteras med följande fält:

- **Syfte** — vad ärendetypen finns för.
- **Primär initiator** — vilken roll påbörjar typiskt ett ärende.
- **Obligatoriska fält** — minsta datauppsättning.
- **Livscykel** — statusvärden + tillåtna övergångar (specialiserar [livscykel-motorn](#livscykel-motor) nedan).
- **RBAC per övergång** — vilka roller får driva ärendet framåt.
- **Tidsregler** — deadlines, fönster, stadgebestämda tidskrav (bygger på [tidsregler](#tidsregler-och-deadlines)).
- **Relationer** — till möte, beslut, protokoll, andra ärenden, stadgaparagrafer.
- **Publicering** — publik / medlem / styrelse / berörd person, per status.
- **Notifiering** — vilka aktörer informeras vid vilka övergångar (bygger på [distribution](#distribution-och-notifiering)).
- **Textbibliotek** — mallar för de meddelanden ärendet genererar.
- **GDPR-hänsyn** — känslighetsklass, gallring, särskild hantering per edition.
- **Hot och försvar** — vilka hot mot governance ärendetypen särskilt utsätts för, och mekanismerna som möter dem.

---

## Gemensam infrastruktur för ärendehantering

Alla ärendetyper bygger på samma underliggande mekanik. Att specificera infrastrukturen separat besparar duplikation — varje ärendetyp anger bara sina specifikt avvikande detaljer mot denna grund.

### Nio gemensamma aktiviteter

Alla ärendetyper kombinerar en delmängd av:

1. **Initiera** — aktör skapar ärendet; obligatoriska fält + bilagor registreras; ärendenummer tilldelas
2. **Formaliakontroll** — automatisk validering (obligatoriska fält, tidsfrister) + ev. manuell granskares bedömning
3. **Distribuera** — berörda aktörer notifieras
4. **Bereda** — handläggare/styrelse granskar, diskuterar, förbereder beslutsunderlag
5. **Publicera för beslut** — ärendet blir tillgängligt för beslutsorganet (styrelse eller stämma)
6. **Besluta** — beslutsorganet avgör med ev. röstning + jäv-deklaration
7. **Verkställa** — effekt på register/system (medlem aktiveras, stadga uppdateras, utlägg exporteras)
8. **Kommunicera utfall** — berörda parter får besked
9. **Avsluta / arkivera** — ärendet markeras slutbehandlat; historik bevaras

Infrastrukturen stödjer alla nio. Ärendetyp-specifikationen anger vilka som ingår och vem som utför dem.

### Livscykel-motor

Standard-tillstånd som alla ärendetyper kan använda:

```
Standard-tillstånd:
- ÖPPET     — initierat, formalia ej kontrollerad
- BEREDNING — formalia OK, handläggning pågår
- BESLUTAT  — beslut fattat, verkställighet pågår eller klar
- AVSLUTAT  — historik

Gemensamma sidospår (från ÖPPET eller BEREDNING):
- ÅTERKALLAT           — initiator drar tillbaka
- AVVISAT_PÅ_FORMALIA  — formaliakrav uppfylls inte
- BORDLAGT             — flyttas till senare möte/stämma
```

Ärendetyper specialiserar denna mall:

- *Motion* lägger till `YTTRANDE_FÖRLAGT`, `PUBLICERAD`, `PÅ_STÄMMA` mellan BEREDNING och BESLUTAT.
- *Medlemsansökan* har egna `INLÄMNAD/GODKÄND/AVSLAGEN`-tillstånd som motsvarar BEREDNING/BESLUTAT-utfallen, plus LEF-specifik `ÖVERKLAGAD_TILL_STÄMMA`.
- *Utläggsgodkännande* lägger till `KASSÖR_GRANSKAR` och `EXPORTERAD_TILL_BOKFÖRING`.
- *Uteslutningsärende* har obligatorisk `VARNING` + `SVARSFÖNSTER` före BEREDNING.

Övergångar är typ-specifika och deklareras per ärendetyp. Varje övergång loggas med `ÄRENDE_STATUS_ÄNDRAT`-händelse i granskningsloggen, där systemdata bär från-status och till-status.

Vid övergångar med större betydelse skrivs **specialiserad händelsetyp parallellt** med `ÄRENDE_STATUS_ÄNDRAT`:

| Övergång | Specialiserad händelse |
|---|---|
| Avvisning på formalia | `ÄRENDE_AVVISAT_PÅ_FORMALIA` |
| Bordläggning | `ÄRENDE_BORDLAGT` |
| Återkallelse av initiator | `ÄRENDE_ÅTERKALLAT` |
| Beslut (styrelse) | `STYRELSEBESLUT` + `RÖST_AVGIVEN` per ledamot + ev. `JÄV_DEKLARERAT` |
| Beslut (stämma) | `STÄMMOBESLUT` + `RÖST_AVGIVEN` per medlem + ev. `JÄV_DEKLARERAT` + `RESERVATION_INGIVEN` |
| Slutavgörande | `ÄRENDE_AVGJORT` med resultat-kod |
| Typ-specifik effekt | t.ex. `MEDLEMSKAP_GODKÄNT`, `UTLÄGG_GODKÄNT`, `STADGEVERSION_ANTAGEN` |

`ÄRENDE_STATUS_ÄNDRAT` är den generella fallback-händelsen för alla övergångar; specialiserade händelser används parallellt när de tillför mening (beslut, gallring, register-effekt).

### Bilagor på ärenden

Bilage-strukturen specas i [granskningslogg.md#bilagor](granskningslogg.md#bilagor). Tre typer (`FIL`, `URL`, `TEXTNOTERING`) gäller alla ärendetyper.

På ärende-nivå **aggregeras** bilagor från alla händelser med samma `aggregate_id`: en motion som inkommit med två offert-PDF:er, fått en URL-bilaga vid styrelsens beredning, och en kompletterande textnotering vid yttrandet — alla fyra visas i ärendets bilage-vy oavsett vilken event de tillhör.

Visibility per bilaga ärver respektive event-visibility (Väg A från [granskningslogg.md](granskningslogg.md#visibility--strikt-inheritance-väg-a)). Om en känslig bilaga (läkarintyg, kontraktsuppgifter) behöver strängare access än ärendet i övrigt registreras den som **egen `BILAGA_INLÄMNAD`-händelse** med strängare visibility, och refereras från huvudärendet via `references`.

### Ärendets aggregerade vy

UI visar varje ärende som en sammanhållen vy projicerad ur händelseströmmen. Komponenter:

- **Aktuellt tillstånd** — räknas ut via replay av ärendets händelser (filter: `aggregate_id = ärendet`)
- **Historikspår** — kronologisk lista över alla händelser som rört ärendet
- **Bilagor** — aggregerat över alla händelser, grupperat per typ (FIL, URL, TEXTNOTERING)
- **Berörda aktörer** — initiator, handläggare, beslutsfattare, ev. berörd part
- **Relaterade ärenden** — via `references` (motioner som grupperats, ärenden som denna bordlagts till)
- **Stadgaförankring** — länkar till de paragrafer ärendet refererar
- **Resultat** — om AVSLUTAT, vad blev utfallet; om pågående, nästa förväntade steg + deadline

Vyn är en **projektion**, inte sann data. Vid tveksamhet är granskningsloggens händelseström sanningen. Vyn kan caches (materialized view) för prestanda men byggs alltid från events.

Per användarroll renderas vyn olika:

- **Initiator** ser allt offentligt + sin egen registrering, men inte styrelsens interna beredning förrän publicering.
- **Berörd part** (vid uteslutning, allmänt ärende-klagomål) ser sin del + relevanta beslut, inte annan kommunikation om sig.
- **Styrelseledamot** ser hela ärendets historik inom epok-rättigheterna.
- **Revisor** ser allt löpande.
- **Medlem** ser den publika vyn (varierar per ärendetyp och status).

### Tidsregler och deadlines

Ärendetypers tidsregler kommer från tre källor:

1. **Lag** — kallelseminimum för stämma (LEF/LFS), preskriptionstid för klandertalan, formkrav för uteslutning.
2. **Stadga** — föreningens egna tidsfönster (motionsfrist, kallelse-max, svarsfönster vid uteslutning, överklagan-fönster).
3. **Ärendetyp-default** — om varken lag eller stadga säger något (gallringstid för avslag, påminnelse-frekvens).

Systemet räknar deadlines automatiskt från nyckeldatum (kallelsedatum, beslutsdatum, mötesdatum) och genererar **påminnelser** till berörda aktörer:

- **30 dagar före deadline:** informativ påminnelse
- **7 dagar före:** varning
- **Vid passerad deadline:** statusövergång enligt ärendetypens regler — typiskt `AVVISAT_PÅ_FORMALIA` (för missade inlämningsfrister), `LAPSED` (för missade förnyelser), eller flagga till handläggaren (för missade granskningstider)

Tidsregler är konfigurerbara per förening i stadge-modulen. Defaults är konservativa — Lantmäteriets normalstadgor för samfällighet, generella mönster för LEF.

### Distribution och notifiering

Varje ärendetyp har en lista av **vilka aktörer som notifieras vid vilka statusövergångar**. Inte vid alla övergångar — bara där notifieringen är meningsfull för respektive aktör.

Exempel för motion:

| Övergång | Notifiering till |
|---|---|
| INLÄMNAD | Initiator (kvittens), styrelsen (ny motion att bereda) |
| YTTRANDE_FÖRLAGT | Initiator (yttrande klart, möjligt att läsa) |
| PUBLICERAD (kallelse) | Alla medlemmar (via kallelseutskick) |
| PÅ_STÄMMA | Initiator (om ej närvarande) |
| AVGJORD | Initiator (resultat) |

**Kanal:** aktörens registrerade kontaktkanal — typiskt e-post för medlemmar, in-app för styrelseledamöter. Inte SMS, inte extern app. Föreningen kan välja (via stadge-konfiguration) om vissa notifieringar dessutom ska ske via brev — typiskt vid uteslutning där rättssäkerhet kräver dokumenterat mottagande.

Notifieringar är **inte** ett separat kommunikationssystem; de återanvänder textbibliotek-mallarna (se [Textbiblioteket](#textbiblioteket) nedan) och loggas inte i granskningsloggen som egen händelse — de är konsekvenser av statusövergångarna och bär samma trovärdighet som ärendets egen audit-trail.

---

## Referens-ärendetyp: Motion

Motionen är den kanoniska governance-bärande ärendetypen och den mest utsatta för taktiska angrepp ([threats.md](threats.md)). Den dokumenteras fullt ut här; övriga ärendetyper återanvänder mönstret.

### Syfte

En medlems formella yrkande till stämman. Motionen är garantin att makten ligger kvar hos majoriteten — ingen styrelse kan tysta en ordentligt formulerad fråga från att nå stämmobordet.

### Primär initiator

`MEMBER` (status `AKTIV`). Styrelsen kan lägga fram *styrelseförslag* som följer samma livscykel men utan eget yttrande-spår — styrelseförslaget *är* styrelsens position.

### Obligatoriska fält

- **Yrkande** — en mening, formen *"Jag yrkar att ..."*.
- **Motivering** — fri text.
- **Stadgaförankring** — vilken paragraf aktualiseras? Stadge-modulens sökfunktion föreslår kandidater när yrkandet skrivs.
- **Bilagor** — valfria dokument (underlag, skisser, offerter).
- **Inlämnare** — medlem, plus eventuella medundertecknare.

### Livscykel

```
INLÄMNAD
  → BEREDNING (styrelsen tar upp ärendet)
    → YTTRANDE_FÖRLAGT (styrelsen har tagit ställning)
      → PUBLICERAD (del av kallelse till stämma)
        → PÅ_STÄMMA
          → AVGJORD
            · BIFALL
            · AVSLAG
            · ÅTERREMISS (ytterligare beredning till nästa stämma)

Sidospår (från vilken status som helst före AVGJORD):
  → ÅTERKALLAD (initiator drar tillbaka)
  → AVVISAD_PÅ_FORMALIA (saknar yrkande/stadgaförankring; motiveras sakligt)
  → GRUPPERAD_MED (slås ihop med annan motion som rör samma fråga)
  → BORDLAGD_TILL_STAMMA (se [core-concepts.md#bordläggning](core-concepts.md#bordläggning))
```

### RBAC per övergång

| Övergång | Roller som får utföra |
|---|---|
| skapa INLÄMNAD | `MEMBER` (status `AKTIV`) |
| INLÄMNAD → ÅTERKALLAD | initiator (tills PUBLICERAD) |
| INLÄMNAD → BEREDNING | `BOARD_*` |
| INLÄMNAD / BEREDNING → AVVISAD_PÅ_FORMALIA | `BOARD_CHAIR`, `BOARD_SECRETARY` (kräver motivering + paragrafhänvisning) |
| BEREDNING → YTTRANDE_FÖRLAGT | `BOARD_*` genom styrelsebeslut |
| BEREDNING / YTTRANDE_FÖRLAGT → GRUPPERAD_MED | `BOARD_SECRETARY` (alla inlämnare informeras) |
| YTTRANDE_FÖRLAGT → PUBLICERAD | automatik vid kallelseutskick |
| PÅ_STÄMMA → AVGJORD | stämmans beslut, verkställs av `BOARD_SECRETARY` |

### Tidsregler

- **Inlämningsfrist:** per stadga. Samfällighet typiskt 2v–1 mån före stämma ([editions/samfallighet.md](editions/samfallighet.md)); LEF enligt stadgan.
- **Yttrande-frist:** styrelsen måste ha förlagt yttrande senast X dagar före kallelse. Stadge-styrt; default = så att PUBLICERAD hinner ske innan kallelsens min-gräns.
- **Återkallelse-fönster:** tillåtet t.o.m. PUBLICERAD. Efter att kallelse gått ut hör motionen till stämman.

### Relationer

- Motion ↔ `AgendaItem` — skapas automatiskt när PUBLICERAD inträffar för kommande stämma.
- Motion ↔ `Decision` — resultatet av stämmans behandling.
- Motion ↔ `Stadgaparagraf` — 1..n; minst en.
- Motion ↔ `Protocol` — stämmans protokoll refererar motionen och beslutet.
- Motion ↔ Motion — via `GRUPPERAD_MED` eller *"samma fråga har tidigare avgjorts"*-referens.

### Publicering

| Status | Publik | Medlem | Styrelse | Initiator |
|---|---|---|---|---|
| INLÄMNAD | — | — | läs | läs + skriv (återkalla) |
| BEREDNING | — | — | läs + skriv | läs |
| YTTRANDE_FÖRLAGT | — | — | läs + skriv | läs |
| PUBLICERAD | **läs** | **läs** | läs | läs |
| PÅ_STÄMMA | läs | läs | läs | läs |
| AVGJORD | **läs** | läs | läs | läs |

Publik läsvy visar motionstext, styrelsens yttrande och resultatet — inte beredningshistoriken. Följer [mission.md#process-transparens-före-utfall-påverkan](mission.md).

Initiatorns namn är synligt vid publicering (lagstadgat för motioner i svensk föreningsrätt). Kontaktuppgifter (e-post, telefon, adress) är *aldrig* publika.

### Textbibliotek

Varje mall nedan finns som redigerbar standardtext. Styrelsen kan anpassa per ärende, men defaultmallen är konstruerad för att inte eskalera konflikt.

- **Inlämningskvitto till initiator** — *"Din motion är registrerad och tilldelad ärendenummer NN. Styrelsen tar upp den på sitt nästa möte den DD/MM. Du får besked när yttrande är förlagt."*
- **Kompletteringsförfrågan** — *"Tack för din motion. För att stämman ska kunna ta ställning behöver vi förtydliga [yrkandet / stadgaförankringen]. Skulle du vilja omformulera den på följande sätt? [förslag]"* — alltid med konkret förslag, aldrig bara *"komplettera"*.
- **Formalia-avvisning** — *"Motionen kan inte tas upp i nuvarande form eftersom [saklig grund + paragrafhänvisning]. Styrelsen välkomnar en omformulering som [konkret väg framåt]."* — aldrig tonen *"avslås"*; alltid *"kan inte tas upp i nuvarande form"* + väg framåt.
- **Yttrande-skelett** — mall som uppmanar saklighet och stadge-anknytning, slutar med tydligt *"Styrelsen yrkar bifall / avslag / bifall med tillägg"*.
- **Grupperings-meddelande** — *"Din motion rör samma fråga som motion NN från [inlämnare]. Styrelsen föreslår att de behandlas gemensamt på stämman. Bägge ursprungstexter bevaras. Kontakta styrelsen om du motsätter dig grupperingen."*
- **Stämmoresultat till initiator** — *"Stämman avgjorde din motion den DD/MM: [resultat]. Protokollsutdrag finns här: [länk]."*

### GDPR

- Inlämnare-identitet är nödvändig enligt föreningsrätt — kan inte anonymiseras.
- Publik vy visar endast namn; aldrig kontaktuppgifter, fastighetsadress eller personnummer.
- Motioner + beslut bevaras långsiktigt (governance-historik); inga särskilda gallringstider.

### Hot och försvar

Se [threats.md#taktisk-motionsgivning](threats.md). Specifika motion-försvar:

- **Gruppering** — snarlika motioner slås ihop → motstår drunknings-taktik (*"10 motioner för förhandlingstryck"*).
- **Formalia-krav** (yrkande + stadgaförankring) → irrelevanta motioner kan avvisas sakligt, inte politiskt.
- **Publik motionshistorik** — en medlem som år efter år inkommer med samma avgjorda motion blir synlig över tid. Det dämpar taktisk upprepning utan att förbjuda legitim omformulering.

### `[ÖPPEN]`-frågor

- Får styrelsen ändra sitt yttrande efter att PUBLICERAD inträffat? Förslag: nej, men kan lägga till *tillägg* (synligt som addendum, daterad, ursprungsyttrandet bevarat).
- Systemet flaggar när motion återkommer efter tidigare avslag — som information till stämman, inte blockering. Hur många gånger innan flaggan? Förslag: alltid, men med förtydligande *"Stämman har tidigare behandlat liknande fråga år X med resultat Y"*.
- Vem får grupperas ihop med vem? Konsent från bägge inlämnare? Förslag: styrelsen får gruppera, bägge inlämnare informeras med 7 dagars invändningsfönster.

---

## Stubbar — övriga ärendetyper

Dessa följer samma mall som motionen men är ännu inte utarbetade. Fångade här för att inte glömmas och för att visa ärendebibliotekets räckvidd.

### Styrelseförslag

Som motion men med styrelsen som initiator. Inget yttrande-spår — styrelsens ställning *är* förslaget. Samma publik transparens. Stubb.

### Medlemsansökan

Ny medlem söker inträde → styrelse- eller medlemsansvarig-beslut. Primärt för LEF (samfällighet triggas av ägarbyte, inte ansökan). Stadgaförankring: medlemskriterier. Livscykel, datauppsättning och gallringsregler specas i [medlemskap.md](medlemskap.md). **GDPR:** persondata under beredning är endast synlig för styrelsen. **Ton:** välkomstmallar, avslag med saklig grund och besvärshänvisning. Stubb — textbiblioteket utarbetas när ärendetypen fullt detaljeras.

### Uteslutningsärende

**Personärende — starkaste GDPR-känsligheten i biblioteket.** Initiator: styrelsen (eller stadgekrav vid utebliven avgift). Stadgeförankring obligatorisk. Livscykel med flera spärrar: `VARNING → SVARSFÖNSTER → BEREDNING → STYRELSEBESLUT → (ev. STÄMMOPRÖVNING)`. **Publicering: ej publik.** Endast berörd medlem, styrelse och revisor. **Ton i mallar:** sakliga, proportionerliga, med tydlig besvärshänvisning — *aldrig* anklagande formuleringar. Hanteringen ska följa god sed även när ärendet gäller allvarlig konflikt. Stubb.

### Utläggsgodkännande

Kassörens kärnflöde ([architecture.md#kärnmoduler](architecture.md#kärnmoduler) punkt 3). Initiator: medlem eller styrelseledamot som lagt ut egna pengar. Obligatorisk koppling till (a) underliggande styrelsebeslut som auktoriserar utgiften, (b) stadgaparagraf. Revisor har permanent läsrätt. Livscykel: `INLÄMNAD → KASSÖR_GRANSKAR → (BESLUT_SAKNAS → tillbaka till styrelse) → GODKÄND → EXPORTERAD_TILL_BOKFÖRING`. **Publicering:** aggregerad publik rapport per kvartal (belopp + kategori + beslutslänk, inte personuppgifter). Stubb.

### Stadgeändring

Två-stämmo-krav enligt LEF 6:36 / LFS 52§ (specifika majoriteter per lagrum). Egen livscykel med *två* beslutssteg, vart och ett en AVGJORD-status. Relation till stadge-modulen via textdiff — ändringen visas sida-vid-sida med befintlig text. Särskilt viktigt att *båda* stämmobeslut audittas + att stadge-modulens versionshistorik (se [mission.md](mission.md)) reflekterar ändringen först efter andra stämmobeslutet. Stubb.

### Revisorsfråga

Initiator: `AUDITOR`. Adressat: styrelsen. Används löpande under räkenskapsåret, inte bara i revisionsberättelsen. Låg tröskel för revisor att ställa frågor utan att det känns som kritik. **Ton:** mallar för både revisor ("jag vill förstå ...") och styrelsesvar ("följande beslut låg till grund för ..."). Relation till revisionsberättelse vid räkenskapsårets slut. Stubb.

### Kandidatnominering

Initiator: valberedning, eller medlem som självnominerar. Publiceringsregler: publik inför stämman (namn + position, samtyckesgrundat). Livscykel kopplad till `NominationPeriod` (Hemmets modellnamn, anpassas). **Ton:** varken tryck ("vi räknar med dig") eller avrådan — neutralt förfrågningsspråk. Stubb.

### Bordläggning

*Inte egen ärendetyp — tillståndsövergång som fungerar på alla ärendetyper.* Se [core-concepts.md#bordläggning](core-concepts.md#bordläggning). Refererad här för navigerbarhet.

### Jävsdeklaration

*Inte egen ärendetyp — attribut på deltagande i beslut.* Se [core-concepts.md#jäv](core-concepts.md#jäv). Refererad här för navigerbarhet.

---

## Textbiblioteket

Mallbiblioteket är i sig ett governance-bidrag — en förening ärver *hur man kommunicerar* när den börjar använda Tillsammans, inte bara *vilka flöden som finns*. Detta påverkar föreningskulturen mer än funktionaliteten gör.

### Tonprinciper

- **Saklig, inte byråkratisk.** Normal svenska, inte kanslisvenska. *"Din motion behandlas på styrelsemötet 14 maj"* slår *"Er inkomna skrivelse föranleder styrelsens behandling vid sammanträde 2026-05-14"*.
- **Konstruktiv vid avslag.** Visa väg framåt, inte bara stängd dörr. *"Kan inte tas upp i nuvarande form — en omformulering enligt X skulle uppfylla stadgans krav."*
- **Ingen rätts-retorik.** Undvik *"det är fel att ..."*, *"du måste ..."*, *"kravet är ..."*. Använd *"stadgan föreskriver ..."* + paragrafhänvisning.
- **Personlig tilltal.** *"Hej Anna"* slår *"Bäste medlem"*. En förening är en samling människor, inte en myndighet.
- **Stadgehänvisning istället för auktoritetspåbud.** Alla restriktiva formuleringar grundas i stadgan eller lag — inte i styrelsens vilja.

### Struktur

- **Centrala defaults** underhålls i Tillsammans-projektet (community-bidrag välkomnas enligt kommande [mission.md#community-och-bidrag](mission.md) — pending edit från tidigare session).
- **Föreningens overrides** lagras per förening. Skillnader från defaults är spårbara i audit.
- **Granskning:** varje central mall ska vara läst av minst en aktiv föreningsfunktionär innan den blir default — motsvarar *sakkunnig läsning*-tröskel för stadgemallar ([verification.md](verification.md)).

### `[ÖPPEN]`-frågor kring textbiblioteket

- Ska mallarna vara flerspråkiga från start? Förslag: nej, svenska som MVP (matchar spec-språket); översättning är senare fas.
- Hur paketeras mallbibliotek — samma git-repo som koden, eller eget repo som parametriserade YAML-filer? Kopplas till pending edit i [open-questions.md](open-questions.md) om stadgemallar som parametriserade YAML/JSON.
- Versionering: följer en mall föreningens version eller den centrala? Förslag: förening låser version per mall, får meddelande när central mall uppdateras och kan välja att migrera.

---

## Vad denna fil *inte* gör

- **Inte datamodell.** Fält och relationer beskrivs schematiskt. Konkret schema tillhör [domain-model.md](domain-model.md) när det är dags att definiera tabeller.
- **Inte UI-spec.** Hur ett ärende visas och redigeras hör till en kommande UX-sektion.
- **Inte uttömmande.** Biblioteket växer. En ny ärendetyp ska bevisa sitt existensberättigande mot governance-scopet ([architecture.md](architecture.md)) — operativa ärendetyper (skaderapport, bokning, störningsanmälan) hör i externa verktyg, inte här.
