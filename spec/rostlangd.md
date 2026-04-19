# Närvaro och röstlängd

> Del av [Tillsammans-specifikation](../tillsammans.md).

Närvarokontroll och röstlängd är två relaterade men distinkta funktioner som hanteras olika beroende på mötestyp. Den centrala designprincipen: **digitalt om praktiskt och proportionerligt, pappers + skanning där det är norm**.

**Relaterade dokument:**
- [moten.md](moten.md) — mötesstrukturer, dagordningsmallar, `KVORUM`-paragraftypen
- [core-concepts.md](core-concepts.md) — befintlig text om röstlängdsetablering vid stämmoöppnande, samägande, fullmakter
- [roles/meeting-roles.md](roles/meeting-roles.md) — mötesordförande och rösträknare
- [granskningslogg.md](granskningslogg.md) — händelsetyper för närvaro och röstlängd
- [medlemskap.md](medlemskap.md) — AKTIV-medlemmar som underlag för röstlängd

## Två spår enligt mötestyp

| Mötestyp | Närvarokontroll | Röstlängd | Per-röst-data |
|---|---|---|---|
| **Styrelsemöte** | Digital — alla deltagare är systemanvändare | (saknas — närvaro räcker) | `RÖST_AVGIVEN` per ledamot |
| **Stämma** | Pappers — medlemmar skriver in sig vid ankomst | Pappers, skannas in efter mötet som bilaga | Aggregerade siffror i `STÄMMOBESLUT` |

Kärnskillnaden: **styrelseledamöter är aktiva systemanvändare med konton; stämmodeltagare är medlemmar som besöker stämman en gång om året och kan inte rimligen begäras registrera sig digitalt utan tydligt återkommande värde.**

## Närvarokontroll för styrelsemöten — digital

Alla styrelseledamöter och suppleanter har konton i systemet.

### Vid mötesöppning — fasta dagordningspunkter

Som fasta punkter i styrelsemötesmallen ([moten.md](moten.md)):

**Punkt 2 — Närvarokontroll** `[auto]`
- Systemet visar lista över alla styrelseledamöter och suppleanter
- Mötesordförande markerar varje person: `NÄRVARANDE` / `FRÅNVARANDE` / `ANMÄLT_FÖRHINDER`
- För frånvarande ordinarie: systemet flaggar suppleant som kan aktiveras
- Mötesordförande aktiverar suppleant för specifik ledamot — *"agerar som ledamot X"* loggas i audit-trail

**Punkt 3 — Mötets behörighet** `[auto]`
- Systemet räknar närvarande röstberättigade ledamöter (inkl. aktiverade suppleanter) mot stadgans `KVORUM`-paragraf
- Visar status: `BEHÖRIGT` (klart över tröskel) / `BEHÖRIGT_MED_MARGINAL` (precis över) / `EJ_BESLUTSFÖRT` (under)
- Mötesordförande bekräftar behörigheten (eller avbryter mötet vid `EJ_BESLUTSFÖRT`)
- `BEHÖRIGHET_BEKRÄFTAD`-händelse skrivs i granskningsloggen

### Under mötet

Mid-möte-justeringar (sen ankomst, tidig avgång, jäv) registreras digitalt:

- **Sen ankomst:** ledamot loggar in eller mötesordförande markerar i UI
- **Tidig avgång:** motsvarande markering
- **Jäv:** tillfällig avregistrering från specifik fråga; återställs efter
- Varje ändring → `NÄRVARO_REGISTRERAD`-händelse med tidpunkt och typ (`ANKOMST` / `AVGÅNG` / `JÄV_AKTIVERAD` / `JÄV_AVSLUTAD`)

Vid varje omröstning används aktuell aktiv närvaro. `RÖST_AVGIVEN` registreras per röstande ledamot — detta är väsentligt för det solidariska ansvarets transparens (se [core-concepts.md#reservation-och-solidariskt-ansvar](core-concepts.md#reservation-och-solidariskt-ansvar)).

### Efter mötet

`MÖTE_AVSLUTAT`-händelse registreras med slutgiltig närvarolista i payload. Ingen separat "fastställande"-händelse behövs — närvaron är resultat av kontinuerliga `NÄRVARO_REGISTRERAD`-händelser plus initial `MÖTE_ÖPPNAT`-payload.

### Loggning — sammanställning för styrelsemöte

| Tidpunkt | Händelse | Innehåll |
|---|---|---|
| Mötesöppning | `MÖTE_ÖPPNAT` | Initial närvarolista, mötesordförande |
| Punkt 3 | `BEHÖRIGHET_BEKRÄFTAD` | Behörighetsstatus, KVORUM-tröskel, närvarande antal |
| Mid-möte | `NÄRVARO_REGISTRERAD` | Vem, typ (`ANKOMST`/`AVGÅNG`/`JÄV_*`), tidpunkt |
| Per beslut | `STYRELSEBESLUT` + `RÖST_AVGIVEN` per ledamot | Röstposition, ev. reservation |
| Mötesavslut | `MÖTE_AVSLUTAT` | Slutgiltig närvaro, mötets sluttidpunkt |

## Röstlängd för stämmor — pappers, skannas in efter mötet

Stämmor är sociala/fysiska händelser där medlemmar ankommer, skriver in sig och möter varandra. Att kräva digital incheckning av medlemmar som besöker stämman en gång om året är opraktiskt — och tillför inget värde över den traditionella pappers-rutinen.

### Före mötet — röstlängds-underlag

**Sekreteraren** genererar röstlängds-underlaget i systemet som del av stämmoförberedelsen (se [roles/board-roles.md#sekreterarens-roll-vid-stämmoförberedelse](roles/board-roles.md#sekreterarens-roll-vid-stämmoförberedelse)). Underlaget bygger på `AKTIV`-medlemmar vid kallelsetillfället:

- En PDF eller utskrivbar lista över förväntade röstberättigade
- För **samfällighet:** per fastighetsbeteckning + ägare + andelstal per GA + ev. samägande-info
- För **LEF:** per medlem + insats-status + organisationsnummer för juridiska personer

Underlaget är **inte** en juridisk röstlängd — det är referens för mötesordförande och rösträknare. Kan skrivas ut för bordet vid stämmans ingång eller hållas digitalt på mötesordförandes enhet. Pappers-röstlängden som faktiskt fylls i under mötet är det som blir del av protokollet (skannas in efter mötet).

### Under mötet — pappers-rutin

Traditionell process; systemet är inte involverat i realtid:

1. **Ankomst:** medlemmar skriver in sig på fysisk röstlängds-lista. Fullmakter visas och granskas av rösträknare/mötesordförande.
2. **Mid-möte:** sena ankomster läggs till; tidiga avgångar markeras.
3. **Vid varje omröstning:** rösträknare räknar manuellt enligt aktuell närvaro. Resultat förkunnas av mötesordförande.
4. **Slutligt:** vid mötets avslutning är pappers-röstlängden komplett.

Mötesordförande och rösträknare är ansvariga för rutinerna. Systemet stör inte.

### Efter mötet — skanning och bilage-arkivering

Pappers-röstlängden + alla fullmakter skannas och bifogas protokollet via [bilage-mekaniken](granskningslogg.md#bilagor):

- Skannad röstlängd → `BILAGA_INLÄMNAD` med kategori `RÖSTLÄNGD`, typ `FIL` (PDF)
- Skannade fullmakter → `BILAGA_INLÄMNAD` med kategori `FULLMAKTER`, typ `FIL` (PDF)
- Eventuella kompletterande noteringar (invändningar mot röstlängden, anteckningar från mötesordförande) → `BILAGA_INLÄMNAD` med kategori `RÖSTLÄNGDS_NOTERING`, typ `TEXTNOTERING` eller `FIL`

Aggregerade röstresultat per beslut förs in i protokollet manuellt och registreras i `STÄMMOBESLUT`-händelsens systemdata:

```
STÄMMOBESLUT.systemdata = {
  ärende_id:       <referens till ärendet, om relevant>
  beslutspunkt:    "Beslut om ansvarsfrihet för styrelsen 2025"
  rost_metod:      ACKLAMATION / OMRÖSTNING_HANDUPPRÄCKNING / SLUTEN_OMRÖSTNING / NAMNUPPROP
  rost_för:        27
  rost_emot:       4
  rost_nedlagda:   2
  resultat:        BIFALL / AVSLAG / BORDLAGT
  reservationer:   ["Anna Andersson reserverar sig mot beslutet, motivering: ..."]
  rosträknare:     [<actor_id>, <actor_id>]
}
```

**Per-medlem-röster lagras inte digitalt för stämmor.** De finns bara i den skannade röstlängden vid namnupprop. Sluten omröstning bevarar anonymitet per definition.

### Loggning — sammanställning för stämma

| Tidpunkt | Händelse | Innehåll |
|---|---|---|
| Före möte | (intern) | Röstlängds-underlag genereras, sparas i konfiguration |
| Mötesöppning | `MÖTE_ÖPPNAT` | Mötesordförande + mötessekreterare + justerare valda; närvaro inte fastställd ännu |
| Per beslut | `STÄMMOBESLUT` | Aggregerade röstsiffror, ev. reservationer som textnoteringar i payload |
| Mötesavslut | `MÖTE_AVSLUTAT` | Sluttidpunkt |
| Efter möte | `BILAGA_INLÄMNAD` (× flera) | Skannad röstlängd, skannade fullmakter, ev. noteringar |
| Protokoll-justering | `PROTOKOLL_JUSTERAT` | Slutgiltig protokollversion med alla bilagor |

## Stadgeparagraf-typ: KVORUM

Lägger till befintlig lista i [moten.md](moten.md#stadgeparagraf-typer-med-automatiskt-systemstöd):

| Typ | Vad systemet gör automatiskt |
|---|---|
| `KVORUM` | För styrelsemöte: räknar närvarande mot tröskel; flaggar `BEHÖRIGT`/`EJ_BESLUTSFÖRT`. För stämma: visar tröskel som referens men ingriper inte automatiskt (mötesordförandes ansvar att verifiera). |

`KVORUM`-paragrafen specificerar:

- **Tröskel-typ:** `MAJORITET_AV_LEDAMÖTER` (vanligast för styrelse), `FAST_ANTAL`, `PROCENT_AV_MEDLEMMAR` (för stämma där stadga föreskriver)
- **Tröskel-värde:** numeriskt eller procentuellt
- **Gäller mötestyp:** `STYRELSEMÖTE` / `STÄMMA` / `BÄGGE`
- **Undantag:** högre kvorum för specifika beslut (t.ex. stadgeändring)

**Defaults när stadgan inte specificerar:**
- Samfällighet (LFS 39§): styrelsen är beslutsför när minst halva antalet styrelseledamöter är närvarande
- LEF (LEF 6:30): majoritet av styrelsen
- Stämma: inga lagstadgade kvorum — om inte stadgan föreskriver

## Vad händer om EJ_BESLUTSFÖRT vid styrelsemöte?

Mötet kan inte fatta formella beslut; bara informationspunkter får behandlas. Stadge-konfigurationen styr vad systemet gör:

- **Mjuk hantering (default):** systemet flaggar `EJ_BESLUTSFÖRT` men låter mötet fortsätta. Mötesordförande beslutar om mötet ska hållas för informationsutbyte (kan vara värdefullt även utan beslut).
- **Hård hantering (stadge-bestämt):** mötet avbryts automatiskt; nytt möte måste kallas.

Förslag: mjuk är default; hård kräver explicit `KVORUM`-konfiguration med flaggan `STRIKT: true`.

## Edge cases och öppna frågor

- **Mindre föreningar med 100 % digitala medlemmar.** I sällsynta fall (mindre LEF där alla är aktiva systemanvändare) kan digital närvarokontroll på stämma vara motiverad. Inte för MVP. `[ÖPPEN]`: lägg till stadge-flagga `STÄMMA_NÄRVARO_DIGITAL` om behovet uppstår.
- **Hybrid-stämma.** Vissa stadgar tillåter distansdeltagande. Systemet är agnostiskt — den fysiska/digitala distinktionen ligger i mötesordförandes hantering. Skannad röstlängd förblir auktoritativt dokument.
- **Sluten omröstning på stämma.** Per definition icke-spårbar per medlem; aggregerat resultat är allt. Systemet har ingen särskild stöd-mekanik utöver att registrera `rost_metod = SLUTEN_OMRÖSTNING` i `STÄMMOBESLUT`.
- **Klandertalan på röstlängd.** Den skannade röstlängden är primärt bevis. Vid klandertalan måste föreningen kunna visa: skannad lista (med inskrivna namn), skannade fullmakter, ev. anteckningar från mötesordförande. Systemet bevarar bilagorna; bevisföringen är pappers-baserad.
- **Sen registrering av stämma-bilagor.** Skanning sker typiskt inom dagarna efter stämman, parallellt med protokollsutkastet. `[ÖPPEN]`: ska systemet flagga om `BILAGA_INLÄMNAD` med kategori `RÖSTLÄNGD` saknas inom X dagar efter `MÖTE_AVSLUTAT` för stämma? Förslag: ja, som påminnelse till sekreteraren — inte blockerande för protokoll-justering, men varning om saknad.

## Vad denna fil *inte* hanterar

- **Detaljerad röstmekanik** (huvudmetod, andelsmetod, fullmakts-tak, voteCapPerMember) — finns i [core-concepts.md#röstning](core-concepts.md#röstning).
- **Mötesrolls-RBAC** — finns i [roles/meeting-roles.md](roles/meeting-roles.md).
- **Dagordningsmallarna** — finns i [moten.md](moten.md).
- **Bilagestrukturen** — finns i [granskningslogg.md#bilagor](granskningslogg.md#bilagor).
- **Samägande-mekaniken** (UNIFIED/PROPORTIONAL) — finns i [core-concepts.md#samägda-fastigheter](core-concepts.md).
- **Mandatverifiering för juridisk persons representant** — finns i [medlemskap.md#juridisk-person-som-medlem](medlemskap.md#juridisk-person-som-medlem) och [case-types/mandatverifiering.md](case-types/mandatverifiering.md).
