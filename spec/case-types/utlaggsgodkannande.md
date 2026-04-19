# Utläggsgodkännande

> Del av [Tillsammans-specifikation](../../tillsammans.md). Specialiserar [gemensam ärendehanterings-infrastruktur](../case-types.md#gemensam-infrastruktur-för-ärendehantering).

## Syfte

Kassörens kärnflöde: en medlem eller styrelseledamot lägger ut egna pengar för föreningens räkning och begär ersättning. Ärendet binder utgiften till **både** ett underliggande styrelsebeslut (auktorisation) **och** en stadgaparagraf (mandat) — utan dessa två får utlägget inte godkännas. Mekaniken försvarar mot otillåtna utgifter och mot jäv vid självgodkännande.

**Edition:** bägge — relevant både för LEF och samfällighet.

## Primär initiator

`MEMBER` (status `AKTIV`) eller `BOARD_*`. Personen som lagt ut pengarna initierar själv.

## Obligatoriska fält

- **Belopp** + valuta (default SEK).
- **Datum för utlägget**.
- **Beskrivning** — kort, vad pengarna gick till.
- **Underliggande styrelsebeslut** — referens till `STYRELSEBESLUT` som auktoriserade utgiften (1..1, obligatorisk). Saknas — ärendet kan inte gå förbi `KOMPLETTERAS`.
- **Stadgaförankring** — vilken paragraf gör utgiften legitim.
- **Bilagor:** kvitto / faktura (`FIL` obligatorisk för belopp över ev. tröskelvärde).
- **Mottagarkonto** — clearing + nummer eller IBAN.

## Livscykel

```
INLÄMNAT
  → KASSÖR_GRANSKAR
    → KOMPLETTERAS (saknad bilaga, oklar koppling till beslut)
      → KASSÖR_GRANSKAR (efter komplettering)
    → GODKÄNT (kassören godkänner inom mandat)
      → BETALNING_EXPORTERAD (utbetalningsfil till bokföring/bank)
        → AVSLUTAT
    → ESKALERAT_TILL_STYRELSE (jäv eller över belopp)
      → STYRELSEBESLUT
        · GODKÄNT → BETALNING_EXPORTERAD → AVSLUTAT
        · AVSLAGET → AVSLUTAT
    → AVSLAGET (kassören avslår med saklig motivering)
      → AVSLUTAT
  → ÅTERKALLAT (initiator drar tillbaka)
```

`STYRELSEBESLUT` skrivs bara vid eskalering eller tvist.

## RBAC per övergång

| Övergång | Roller |
|---|---|
| skapa INLÄMNAT | `MEMBER`, `BOARD_*` |
| INLÄMNAT → ÅTERKALLAT | initiator (tills `BETALNING_EXPORTERAD`) |
| INLÄMNAT → KASSÖR_GRANSKAR | automatik |
| KASSÖR_GRANSKAR → KOMPLETTERAS | `BOARD_TREASURER` |
| KASSÖR_GRANSKAR → GODKÄNT | `BOARD_TREASURER` (med obligatorisk jäv-deklaration) |
| KASSÖR_GRANSKAR → ESKALERAT_TILL_STYRELSE | `BOARD_TREASURER` (vid jäv eller över mandat-belopp) |
| KASSÖR_GRANSKAR → AVSLAGET | `BOARD_TREASURER` |
| ESKALERAT → AVGJORT | `BOARD_*` genom styrelsebeslut |
| GODKÄNT → BETALNING_EXPORTERAD | `BOARD_TREASURER` eller `ExternalAdministrator` (när scope tillåter) |

**Kassören kan inte godkänna sitt eget utlägg.** Sådana ärenden eskaleras automatiskt till styrelsen där kassören avstår enligt jäv-regler ([analysis-rules.md](../analysis-rules.md) grupp 5).

## Tidsregler

- **Granskningstid:** kassören granskar inom 14 dagar (default) — påminnelse till kassören vid passerad frist.
- **Stadge-bestämt tröskelbelopp:** över beloppet krävs styrelsebeslut, inte enbart kassör-mandat. Stadgan sätter beloppet (typiskt 5 000 – 50 000 kr).
- **Kvitto-krav:** stadgan eller styrelse-praxis sätter belopp över vilket kvitto är obligatoriskt (default 200 kr enligt skatteverkets rekommendation).
- **Preskription:** utlägg som inte begärs ersatta inom 12 månader avvisas på formalia (default; stadgan kan justera).

## Relationer

- Utlägg ↔ underliggande `STYRELSEBESLUT` (1..1, obligatorisk).
- Utlägg ↔ `Stadgaparagraf` (1..n).
- Utlägg ↔ ev. `BUDGET_POST` om budget-modul finns.
- Utlägg ↔ `BETALNING_EXPORTERAD`-händelse vid utbetalning.
- Utlägg ↔ ev. `Fund` om utlägget belastar specifik underfond.

## Publicering

| Status | Publik | Medlem | Styrelse | Kassör | Initiator | Revisor |
|---|---|---|---|---|---|---|
| INLÄMNAT | — | — | läs | läs | läs (egen) | läs |
| KASSÖR_GRANSKAR | — | — | läs | läs + skriv | läs | läs |
| ESKALERAT | — | — | läs + skriv | läs | läs | läs |
| GODKÄNT / AVSLAGET | aggregerad¹ | — | läs | läs | läs | läs |
| BETALNING_EXPORTERAD | aggregerad | — | läs | läs | läs | läs |
| AVSLUTAT | aggregerad | — | läs | läs | läs | läs |

¹ **Publik kvartalsrapport:** belopp + kategori + beslutslänk + stadgeparagraf — ingen personidentifierande data om initiator. Säkerställer ekonomisk transparens utan integritets-läckage.

Revisor har permanent läsrätt enligt [oversight-roles.md](../roles/oversight-roles.md#revisor).

## Notifiering

| Övergång | Notifiering till |
|---|---|
| INLÄMNAT | Initiator (kvittens), kassör (nytt ärende) |
| KOMPLETTERAS | Initiator (vad som saknas) |
| ESKALERAT_TILL_STYRELSE | Styrelsen, initiator (informativ) |
| GODKÄNT | Initiator (ersättning utbetalas inom X dagar) |
| AVSLAGET | Initiator (saklig grund + besvärshänvisning) |
| BETALNING_EXPORTERAD | Initiator (kvittens med datum), kassören |

## Textbibliotek

- **Inlämningskvitto** — *"Hej [namn], ditt utlägg om [belopp] är registrerat med ärendenummer NN. Kassören granskar inom 14 dagar."*
- **Kompletteringsförfrågan** — *"Tack för ditt utlägg. För att kunna godkänna det behöver vi: [konkret saknad uppgift, t.ex. kvitto, koppling till styrelsebeslut]. Skicka uppgiften via [länk]."*
- **Godkännande** — *"Ditt utlägg om [belopp] är godkänt. Utbetalning sker till [konto] inom [X] arbetsdagar."*
- **Avslag** — *"Vi kan tyvärr inte godkänna utlägget i nuvarande form eftersom [saklig grund]. Stadgehänvisning: §X. Konstruktiv väg framåt: [konkret förslag, t.ex. omformulera koppling till annat beslut]."*
- **Eskalering** — *"Ditt utlägg är över kassörens beslutsmandat och har eskalerats till styrelsen. Beslut fattas vid styrelsemötet DD/MM."*
- **Betalningsbekräftelse** — *"Utbetalning av [belopp] för utlägg NN är genomförd den DD/MM. Tack."*

## GDPR-hänsyn

Måttlig känslighet — kontonummer är personlig finansiell uppgift. Kontonummer visas endast för kassör, ev. ExternalAdministrator och revisor; aldrig publikt eller för andra medlemmar. Bokförings-relaterad data bevaras 7 år enligt bokföringslagen — ingen tidigare gallring även av personuppgifter knutna till utgiften.

## Hot och försvar

Se [threats.md](../threats.md):

- **Egenintressen och jäv:** kassören som godkänner egna utlägg → systemspärr + automatisk eskalering.
- **Utgift utan auktorisation:** krav på underliggande styrelsebeslut → utan beslut, ingen godkännande-väg.
- **Smyg-utgifter via många små utlägg:** publik kvartalsrapport gör volym-mönster synliga; revisor ser löpande.
- **Mandatläckage från utskott:** utskotts-utlägg över utskottets takbelopp eskaleras automatiskt enligt [other-roles.md#utskotts-och-kommittéledamot](../roles/other-roles.md#utskotts--och-kommittéledamot).

## Öppna frågor

- `[ÖPPEN]` Hur hanterar kassören utlägg från sig själv? Förslag: alla utlägg där kassören är initiator eskaleras automatiskt till hela styrelsen utan kassörens deltagande. Vice ordförande eller annan ledamot driver granskningen.
- `[ÖPPEN]` Tröskelbelopp för obligatoriskt kvitto: Skatteverket säger 200 kr för representation; stadgan brukar inte vara explicit. Förslag: stadge-default 200 kr, ändras via stadgekonfiguration.
- `[ÖPPEN]` Vid `ExternalAdministrator` med `scope: FINANCIAL_*` — får administratören godkänna utlägg under tröskeln, eller är godkännande alltid kassörens? Förslag: administratören kan *granska* (förbereda) men kassören (eller styrelsen vid jäv) godkänner formellt — administratören har inget governance-mandat.
- `[ÖPPEN]` Återbetalningstid: stadge-bunden eller systemdefault? Förslag: ingen hård regel; kassören anger i godkännande, system rapporterar utfall till revisor.
