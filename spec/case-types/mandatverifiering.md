# Mandatverifiering (juridisk person)

> Del av [Tillsammans-specifikation](../../tillsammans.md). Specialiserar [gemensam ärendehanterings-infrastruktur](../case-types.md#gemensam-infrastruktur-för-ärendehantering).

## Syfte

Granskning och bekräftelse av en fysisk persons bemyndigande att representera en juridisk medlem (AB, stiftelse, ekonomisk förening, dödsbo, kommun). Verifieringen är **förutsättning** för att representanten får rösta vid stämma eller fatta andra beslut å den juridiska personens vägnar. Rutinen är en **mänsklig handling** stödd av systemet — föreningen avgör vad som är giltigt, men spårbarheten är teknikens.

Hela mandat-mekaniken specas i [medlemskap.md#juridisk-person-som-medlem](../medlemskap.md#juridisk-person-som-medlem); detta dokument beskriver ärende-aspekten.

**Edition:** primärt samfällighet (där juridisk person är normalt förekommande enligt LFS), även LEF där stadgan tillåter.

## Primär initiator

Den juridiska personens representant — typiskt vid första registreringen, vid representant-byte, eller på begäran av styrelsen om ett tidigare verifierat mandat ifrågasatts.

`BOARD_*` eller `medlemsansvarig` kan också initiera ärendet vid behov av nytt verifikationsunderlag.

## Obligatoriska fält

- **Juridisk aktör** — `Actor` av typ `LEGAL_PERSON` / `ESTATE` / `MUNICIPALITY`.
- **Föreslagen representant** — `Actor` av typ `NATURAL_PERSON`.
- **Mandattyp** — `FIRMATECKNING` / `FULLMAKT` / `STYRELSEBESLUT` / `DELEGATIONSORDNING`.
- **Mandatets omfattning** — `ENSAM` / `I_FÖRENING` / `SPECIFIK_FRÅGA`.
- **Giltighetsperiod** — från-datum + ev. till-datum (default: tillsvidare).
- **Bilaga (obligatorisk):** bemyndigandedokument enligt mandattyp:
  - `FIRMATECKNING` → Bolagsverket-utdrag (ej äldre än stadge-bunden tid, default 6 månader).
  - `FULLMAKT` → undertecknad fullmakt med behörig signering.
  - `STYRELSEBESLUT` → protokollutdrag från den juridiska personens styrelse.
  - `DELEGATIONSORDNING` → kommunens delegationsordning (för `MUNICIPALITY`).
- **Bouppteckning** (för `ESTATE`) eller motsvarande grundande handling.

## Livscykel

```
INLÄMNAT (representant lämnar dokument)
  → GRANSKAS (medlemsansvarig/ordförande tar upp)
    → KOMPLETTERAS (saknad eller oklar handling)
      → GRANSKAS
    → MANDAT_VERIFIERAT
      → AKTIVT (representationen gäller)
        → UPPHÖRT (vid till-datum, byte, eller återkallelse)
        → IFRÅGASATT (mötesordförande eller styrelse begär ny verifikation)
          → GRANSKAS (omverifikation)
    → AVVISAT (dokumentet är otillräckligt eller ogiltigt)

Sidospår:
  → ÅTERKALLAT (representant eller juridisk person drar tillbaka)
```

`MANDAT_VERIFIERAT`-händelse skrivs i granskningsloggen vid övergång till `AKTIVT`. `REPRESENTATION_REGISTRERAD` skrivs parallellt; `REPRESENTATION_UPPHÖRD` vid `UPPHÖRT`.

## RBAC per övergång

| Övergång | Roller |
|---|---|
| skapa INLÄMNAT | representant själv, `BOARD_*`, `medlemsansvarig` |
| INLÄMNAT → GRANSKAS | `BOARD_CHAIR` eller `medlemsansvarig` (med jäv-deklaration vid släktskap/affärskoppling) |
| GRANSKAS → KOMPLETTERAS | granskaren |
| GRANSKAS → MANDAT_VERIFIERAT | granskaren |
| GRANSKAS → AVVISAT | granskaren med saklig motivering |
| AKTIVT → IFRÅGASATT | mötesordförande (vid stämma), `BOARD_*` |
| AKTIVT → UPPHÖRT | automatik vid till-datum, eller `BOARD_*` vid återkallelse |

## Tidsregler

- **Bemyndigandedokumentets ålder:** Bolagsverket-utdrag default max 6 månader, fullmakter default max 12 månader (stadge-bart).
- **Granskningstid:** 14 dagar (default) — påminnelse till granskaren.
- **Inför stämma:** alla aktiva mandat genomgår automatisk åldersgranskning vid kallelseutskick — för gamla flaggas för omverifikation.
- **Vid `IFRÅGASATT` mid-stämma:** mötesordförande kan tillåta att stämman fortsätter på övriga frågor; den fastighetens röst avges inte tills frågan är avgjord (se [core-concepts.md#röstlängdsetablering](../core-concepts.md#röstlängdsetablering--processen-vid-stämmans-öppnande)).

## Relationer

- Mandatverifiering ↔ `Actor` (juridisk person).
- Mandatverifiering ↔ `Actor` (representant, fysisk person).
- Mandatverifiering ↔ `Representation`-post.
- Mandatverifiering ↔ ev. `Medlemsansökan` (vid första registreringen av juridisk medlem).
- Mandatverifiering ↔ tidigare verifikationer för samma representation (kronologisk kedja).
- Mandatverifiering ↔ ev. `RoleAssignment` om representanten också tilldelas roll i föreningen.

## Publicering

| Status | Publik | Medlem | Styrelse | Representant | Juridisk aktör |
|---|---|---|---|---|---|
| INLÄMNAT | — | — | läs | läs | läs |
| GRANSKAS | — | — | läs + skriv | läs | läs |
| MANDAT_VERIFIERAT | (intern) | (representantens namn vid stämma) | läs | läs | läs |
| AKTIVT | (representantens namn vid stämma) | läs | läs | läs | läs |
| UPPHÖRT | (intern) | — | läs | läs | läs |
| AVVISAT | — | — | läs | läs (saklig motivering) | läs |

Bemyndigandedokument (Bolagsverket-utdrag, fullmakt) är persondata och visas endast för styrelsen och revisorn — inte publikt eller för andra medlemmar.

## Notifiering

| Övergång | Notifiering till |
|---|---|
| INLÄMNAT | Representant (kvittens), granskare (medlemsansvarig/ordförande) |
| KOMPLETTERAS | Representant (vad som saknas) |
| MANDAT_VERIFIERAT | Representant (välkomst som ombud), juridisk persons kontaktperson |
| AVVISAT | Representant (saklig grund), juridisk persons kontaktperson |
| IFRÅGASATT | Representant, juridisk persons kontaktperson, styrelsen |
| UPPHÖRT | Representant, juridisk persons kontaktperson; nytt mandat kan behövas |

## Textbibliotek

- **Inlämningskvitto** — *"Tack för insänd dokumentation. Granskningen sker inom 14 dagar. När mandatet är verifierat kan du representera [juridisk person] vid stämma och i andra sammanhang."*
- **Kompletteringsförfrågan** — *"För att kunna verifiera ditt mandat behöver vi: [konkret saknad uppgift, t.ex. nyare Bolagsverket-utdrag, fullmakt-original, signerat protokoll]. Skicka via [länk]."*
- **Verifierings-bekräftelse** — *"Hej [namn], ditt mandat att representera [juridisk person] är verifierat. Mandatet är giltigt [från-datum] till [till-datum eller 'tillsvidare']. Vid representant-byte eller om mandatet behöver förnyas, kontakta oss i god tid."*
- **Avvisning** — *"Tyvärr kan vi inte verifiera mandatet på inlämnat underlag. Skäl: [saklig grund, t.ex. utdraget är äldre än 6 månader, fullmakten saknar firmatecknarens behörighet]. Konstruktiv väg framåt: [konkret förslag]."*
- **Ifrågasättande mid-stämma** — *"Vid stämmans öppning den DD/MM uppstod osäkerhet kring mandatet. Mötesordförande har bett om förnyad dokumentation. Tills frågan är klar avges ingen röst för [fastighet/aktör]. Vänligen återkom med [specifik handling]."*

## GDPR-hänsyn

Måttlig–hög känslighet. Bemyndigandedokument innehåller personuppgifter om både firmatecknare och föreslagen representant. Bevaras så länge representationen är aktiv + 7 år (bokföringsmotsvarande), därefter gallras dokumentinnehåll men metadata och hashar bevaras. Vid `AVVISAT` gallras inom 6 månader (samma som avslagen medlemsansökan).

## Hot och försvar

Se [threats.md](../threats.md):

- **Skuggrepresentation:** krav på dokumenterat bemyndigande + ålderskrav minskar risken att rätt person inte är behörig.
- **Verifikation som teaterföreställning:** mänsklig granskning är obligatorisk; systemet får inte auto-godkänna baserat på filuppladdning.
- **Civilrättsliga tvister inom juridisk person:** "Tillsammans löser inga civilrättsliga tvister" ([core-concepts.md](../core-concepts.md#röstlängdsetablering--processen-vid-stämmans-öppnande)) — vid tvist mellan firmatecknare avges ingen röst, parterna hänvisas till sin egen governance.
- **Stämpling/diskriminering:** per [policy 8](../mission.md#grund-policies) flaggar systemet inte juridiska personer som "misstänkta" — det registrerar fakta. Förtroendevaldas moraliska kompass avgör vid tveksamhet ([medlemskap.md#vad-systemet-uttryckligen-inte-gör](../medlemskap.md#vad-systemet-uttryckligen-inte-gör)).

## Öppna frågor

- `[ÖPPEN]` Bolagsverket-API-integration för automatisk firmatecknarkontroll? Post-MVP — manuell granskning av PDF-utdrag är default. När integrationen finns kan systemet flagga avvikelser (t.ex. firmateckning ändrades efter dokumentets datum).
- `[ÖPPEN]` Vid mandattyp `STYRELSEBESLUT`: ska systemet kräva specifik formulering ("[Person] är behörig att rösta för [Aktör] vid samfällighetsföreningens stämmor")? Förslag: nej — föreningens granskare bedömer protokollutdragets innehåll.
- `[ÖPPEN]` Vid `IFRÅGASATT` mid-stämma: får mötesordförande ensidigt avgöra, eller behöver stämman ta ställning? Förslag: mötesordförande avgör operativt; stämman kan överröstas via ordningsfråga om någon medlem invänder.
- `[ÖPPEN]` Hur lång varning ges innan ett tidsbegränsat mandat upphör? Förslag: 90 + 30 + 7 dagar enligt [case-types.md#tidsregler-och-deadlines](../case-types.md#tidsregler-och-deadlines).
