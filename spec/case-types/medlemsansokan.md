# Medlemsansökan

> Del av [Tillsammans-specifikation](../../tillsammans.md). Specialiserar [gemensam ärendehanterings-infrastruktur](../case-types.md#gemensam-infrastruktur-för-ärendehantering).

## Syfte

LEF-formell ansökan om inträde i föreningen. Ärendet bär ansökan från inlämning genom granskning, beslut och eventuell överklagan till stämma (LEF 4:3). Hela medlemsregister-flödet och tillstånden specas i [medlemskap.md](../medlemskap.md); detta dokument beskriver ärende-aspekten.

**Edition:** primärt LEF. Samfällighet använder inte denna ärendetyp — där triggas medlemskap av ägarbyte (se [agarovergang.md](agarovergang.md)).

## Primär initiator

Sökande person eller juridisk person — inte ännu medlem, så ingen `MEMBER`-roll. Identifiering via kontaktuppgifter; ärendet skapas av sökanden själv via publik ansökningsblankett.

## Obligatoriska fält

- **Sökandens namn** + kontaktkanal (e-post / postadress).
- **Aktör-typ** — `NATURAL_PERSON` / `LEGAL_PERSON` / `ESTATE` / `MUNICIPALITY` (se [medlemskap.md#juridisk-person-som-medlem](../medlemskap.md#juridisk-person-som-medlem)).
- **Samtycke till stadgar** — version + tidpunkt.
- **Ev. motiveringsfält** om stadgan föreskriver — *aldrig* fritt "orsak"-fält enligt [medlemskap.md](../medlemskap.md) (diskrimineringsrisk).
- **Bilagor:** för juridisk person obligatoriska representations-handlingar (se [mandatverifiering.md](mandatverifiering.md)).

## Livscykel

Specialiserar standard-livscykeln med medlemskaps-tillstånd från [medlemskap.md#livscykel](../medlemskap.md#livscykel):

```
INLÄMNAD
  → GODKÄND (medlemsansvarig eller styrelsebeslut)
    → AKTIV (efter insats-betalning, om stadgan kräver)
  → AVSLAGEN
    → ÖVERKLAGAD_TILL_STÄMMA (sökanden själv, inom stadgad frist, LEF 4:3)
      → AKTIV (stämman bifaller, ev. via GODKÄND om insats kvarstår)
      → AVSLAGEN (stämman fastställer — slutgiltigt)
  → ÅTERKALLAD (sökanden drar tillbaka)
```

## RBAC per övergång

| Övergång | Roller |
|---|---|
| skapa INLÄMNAD | sökanden själv (publik blankett) |
| INLÄMNAD → ÅTERKALLAD | sökanden |
| INLÄMNAD → GODKÄND / AVSLAGEN (rutin) | `medlemsansvarig` (med obligatorisk jäv-deklaration, se [other-roles.md#medlemsansvarig](../roles/other-roles.md#medlemsansvarig)) |
| INLÄMNAD → GODKÄND / AVSLAGEN (eskalering) | `BOARD_*` genom styrelsebeslut |
| AVSLAGEN → ÖVERKLAGAD_TILL_STÄMMA | sökanden själv (LEF 4:3) |
| ÖVERKLAGAD → AVGJORD | stämmans beslut |
| GODKÄND → AKTIV | automatik vid bekräftad insats (`MEDLEMSKAP_AKTIVERAT`) |

## Tidsregler

- **Beslut inom rimlig tid** — stadgan kan föreskriva. Default: 30 dagar från `INLÄMNAD`.
- **Överklagan-fönster:** stadge-bundet, typiskt 1 månad efter avslaget enligt LEF 4:3.
- **Insats-fönster** efter `GODKÄND`: stadge-bundet, typiskt 30 dagar.
- **Gallring** av personuppgifter vid `AVSLAGEN`/`ÅTERKALLAD`: 6 månader (se [medlemskap.md#gallring](../medlemskap.md#gallring)).

## Relationer

- Medlemsansökan ↔ `Medlem` (skapas vid `AKTIV`).
- Medlemsansökan ↔ `STYRELSEBESLUT` eller medlemsansvarigs beslut (`MEDLEMSKAP_GODKÄNT` / `MEDLEMSKAP_AVSLAGET`).
- Medlemsansökan ↔ `Stadgaparagraf` — minst medlemskriterierna.
- Medlemsansökan ↔ `STÄMMOBESLUT` vid överklagan.
- Vid juridisk person: ↔ [mandatverifiering.md](mandatverifiering.md) som parallellt ärende.

## Publicering

| Status | Publik | Medlem | Styrelse | Sökande |
|---|---|---|---|---|
| INLÄMNAD | — | — | läs | läs (egen) |
| GODKÄND / AVSLAGEN | — | — | läs | läs |
| ÖVERKLAGAD_TILL_STÄMMA | — | **läs**¹ | läs | läs |
| AKTIV | — | namn syns i medlemslista² | läs | — |

¹ Som agenda-punkt i kallelsen — beslutsunderlaget är publicerat enligt stämmans transparens-default.
² GDPR-filtrerat — kontaktuppgifter aldrig publika utan samtycke.

Personuppgifter under beredning är endast synliga för styrelsen och medlemsansvarig.

## Notifiering

| Övergång | Notifiering till |
|---|---|
| INLÄMNAD | Sökande (kvittens), medlemsansvarig (ny ansökan) |
| GODKÄND | Sökande (välkomst + ev. insats-instruktioner), styrelsen (informativ) |
| AVSLAGEN | Sökande (besvärshänvisning + saklig grund), styrelsen |
| ÖVERKLAGAD_TILL_STÄMMA | Styrelsen (förbered yttrande), valberedning (informativ) |
| AKTIV | Sökande (välkomst), styrelsen, kassör |

## Textbibliotek

- **Inlämningskvitto** — *"Hej [namn], din ansökan om medlemskap är registrerad och tilldelad ärendenummer NN. Föreningen kommer att behandla den inom 30 dagar. Du får besked via [kanal]."*
- **Kompletteringsförfrågan** — *"Tack för din ansökan. För att vi ska kunna fatta beslut behöver vi komplettera med [konkret saknad uppgift]. Skicka uppgiften via [länk] senast DD/MM."*
- **Välkomst (GODKÄND)** — *"Vi välkomnar dig som medlem. För att medlemskapet ska aktiveras ska insats om [belopp] betalas senast DD/MM enligt §X i stadgarna. Instruktioner finns i bilaga."*
- **Avslag** — *"Föreningen kan tyvärr inte godkänna din ansökan i nuläget. Skäl: [saklig hänvisning till stadgaparagraf eller medlemskriterium]. Du har rätt att begära att frågan prövas av föreningsstämman enligt LEF 4 kap. 3 §. Begäran lämnas in senast DD/MM via [länk]."*
- **Aktivering** — *"Hej [namn], din insats är registrerad och du är nu aktiv medlem. Här är dina inloggningsuppgifter: [...]."*

## GDPR-hänsyn

Hög känslighetsklass under beredning. Hela ansökningsdata gallras automatiskt 6 månader efter `AVSLAGEN`/`ÅTERKALLAD` — endast beslutsreferens + datum bevaras i granskningsloggen för audit. Personnummer hanteras enligt [medlemskap.md#personnummer](../medlemskap.md#personnummer--explicit-opt-in-aldrig-default). Endast styrelse, medlemsansvarig och revisor ser personuppgifter under beredning.

## Hot och försvar

Se [threats.md](../threats.md):

- **Asymmetrier vid avslag:** sökanden som juridiskt resursstark kan utnyttja oklara avslagsgrunder → mallar kräver saklig stadgehänvisning, besvärshänvisning är obligatorisk.
- **Diskrimineringsrisk vid godtycklig granskning:** "orsak/motivation"-fält undviks helt; medlemsansvarigs delegering har obligatorisk jäv-deklaration.
- **Maktkoncentration:** medlemsansvarig kan inte ensidigt avslå utan att övriga styrelse + revisor ser beslutet i granskningsloggen.

## Öppna frågor

- `[ÖPPEN]` Vid juridisk person-ansökan: ska medlemsansökan och mandatverifiering vara samma ärende eller två separata? Förslag: två separata, kopplade via `references` — mandatverifieringen är återkommande över tid (representant-byten), ansökan är engångs.
- `[ÖPPEN]` Hur lång preskriptionstid för rätten att överklaga avslag enligt LEF 4:3? Lagen är otydlig; stadgan brukar sätta 1 månad. Förslag: stadge-default, men systemet flaggar om stadgan saknar regel.
