# Allmänt ärende

> Del av [Tillsammans-specifikation](../../tillsammans.md). Specialiserar [gemensam ärendehanterings-infrastruktur](../case-types.md#gemensam-infrastruktur-för-ärendehantering).

## Syfte

Diariefört kanal för det som inte passar i de andra ärendetyperna men ändå ska hanteras formellt: klagomål, förslag, frågor, synpunkter, ersättningsanspråk. Sekreteraren tar emot, kategoriserar och slussar vidare. Ärendetypen bär den **administrativa lägstanivå** som stadgar och förvaltningstradition förutsätter — utan den blir alla informella ärenden osynliga eller hamnar fel.

**Edition:** bägge.

## Primär initiator

`MEMBER` (status `AKTIV` — undantag: ersättningsanspråk kan komma från icke-medlem som drabbats av föreningens verksamhet, då registreras initiator som extern aktör).

## Obligatoriska fält

- **Underkategori (obligatorisk):**
  - `KLAGOMÅL_VERKSAMHET` — klagomål på föreningens drift, beslut, förvaltning.
  - `KLAGOMÅL_ANNAN_MEDLEM` — klagomål mot en specifik annan medlem (personärende — se publicering).
  - `FÖRSLAG` — informellt förslag (ej formell motion).
  - `FRÅGA` — informationsbegäran.
  - `SYNPUNKT` — feedback utan handlingsbehov.
  - `ERSÄTTNINGSANSPRÅK` — krav om ekonomisk ersättning.
- **Beskrivning** — fri text.
- **Stadgaförankring** — om relevant; obligatorisk för `ERSÄTTNINGSANSPRÅK` och `KLAGOMÅL_*`.
- **Berörd part** — vid `KLAGOMÅL_ANNAN_MEDLEM`: medlems-referens; vid `ERSÄTTNINGSANSPRÅK`: belopp + grund.
- **Bilagor** — frivilliga.

## Livscykel

```
INKOMMET (medlem registrerar)
  → DIARIEFÖRT (sekreterare tilldelar diarienummer + kategori)
    → SLUSSAT (ansvarig roll utpekad — kassör, ordförande, ledamot, utskott)
      → BEHANDLAS (utpekad roll arbetar med ärendet)
        → BESVARAT (svar lämnat till initiator)
          → AVSLUTAT (initiator nöjd eller ingen vidare aktion)
        → ESKALERAT_TILL_STYRELSE (kräver styrelsebeslut)
          → STYRELSEBESLUT (om beslut behövs)
            → BESVARAT
              → AVSLUTAT
        → AVSKRIVET (faller på formalia eller saknad respons från initiator)

Sidospår:
  → ÅTERKALLAT (initiator drar tillbaka)
  → ÖVERFÖRT_TILL_ANNAN_ÄRENDETYP (visar sig vara motion, utlägg, klagomål mot medlem som ska bli uteslutningsärende, etc.)
```

## RBAC per övergång

| Övergång | Roller |
|---|---|
| skapa INKOMMET | `MEMBER`, ev. extern aktör vid `ERSÄTTNINGSANSPRÅK` |
| INKOMMET → DIARIEFÖRT | `BOARD_SECRETARY` |
| DIARIEFÖRT → SLUSSAT | `BOARD_SECRETARY` (utpekar ansvarig) |
| SLUSSAT → BEHANDLAS | utpekad ansvarig |
| BEHANDLAS → BESVARAT | utpekad ansvarig |
| BEHANDLAS → ESKALERAT_TILL_STYRELSE | utpekad ansvarig (vid behov av styrelsebeslut) |
| ESKALERAT → STYRELSEBESLUT | `BOARD_*` |
| → AVSLUTAT | utpekad ansvarig eller sekreterare |
| → ÖVERFÖRT_TILL_ANNAN_ÄRENDETYP | sekreterare (med information till initiator) |

## Tidsregler

- **Diarieföring:** inom 7 dagar från `INKOMMET`.
- **Slussning:** inom 14 dagar från `DIARIEFÖRT`.
- **Svarstid default:** 30 dagar från `SLUSSAT` — påminnelse till ansvarig vid passerad frist.
- **Ersättningsanspråk:** styrelsens svarstid 30 dagar enligt allmän förvaltningspraxis; preskription enligt civilrätt.
- **Vid utebliven respons från initiator:** 60 dagars tystnad vid kompletteringsfråga → automatisk `AVSKRIVET`.

## Relationer

- Allmänt ärende ↔ ev. utpekad `Medlem` (vid `KLAGOMÅL_ANNAN_MEDLEM`).
- Allmänt ärende ↔ ev. `STYRELSEBESLUT` vid eskalering.
- Allmänt ärende ↔ ev. överfört ärende (motion, utlägg, uteslutningsärende) via `references`.
- Allmänt ärende ↔ ev. tidigare allmänna ärenden från samma initiator (mönster).

## Publicering

Varierar **per underkategori** — det är ärendetypens karakteristikum:

| Underkategori | Publik | Medlem | Styrelse | Berörd part | Initiator |
|---|---|---|---|---|---|
| `KLAGOMÅL_VERKSAMHET` | (statistik) | (anonymiserad) | läs | n/a | läs |
| `KLAGOMÅL_ANNAN_MEDLEM` | — | — | läs | **läs** + svarsmöjlighet¹ | läs |
| `FÖRSLAG` | — | (vid AVSLUTAT) | läs | n/a | läs |
| `FRÅGA` | — | — | läs | n/a | läs |
| `SYNPUNKT` | — | — | läs | n/a | läs |
| `ERSÄTTNINGSANSPRÅK` | (statistik vid AVSLUTAT) | — | läs | n/a | läs |

¹ `KLAGOMÅL_ANNAN_MEDLEM` är **personärende** — avgränsad insyn enligt [mission.md](../mission.md), samma princip som [uteslutning.md](uteslutning.md). Berörd part har rätt till information om klagomål och möjlighet att bemöta innan ev. eskalering till uteslutningsärende.

## Notifiering

| Övergång | Notifiering till |
|---|---|
| INKOMMET | Initiator (kvittens), sekreterare (nytt ärende) |
| DIARIEFÖRT | Initiator (diarienummer + förväntad svarstid) |
| SLUSSAT | Utpekad ansvarig, initiator (vem som handlägger) |
| BESVARAT | Initiator (svar) |
| ESKALERAT_TILL_STYRELSE | Initiator (informativ), styrelsen |
| ÖVERFÖRT_TILL_ANNAN_ÄRENDETYP | Initiator (vart ärendet flyttats + nytt ärendenummer) |
| Vid `KLAGOMÅL_ANNAN_MEDLEM` `DIARIEFÖRT` | Berörd medlem (om klagomålet) |

## Textbibliotek

- **Inlämningskvitto** — *"Tack för ditt ärende. Det har registrerats med diarienummer NN. Vi diariefor och svarar inom [7 / 14 / 30] dagar."*
- **Diarieförings-bekräftelse** — *"Hej [namn], ditt ärende NN är diariefört som [kategori] och kommer att handläggas av [ansvarig roll]. Förväntat svar inom [tidsfrist]."*
- **Klagomål mot medlem — bekräftelse** — *"Tack för ditt ärende. Vi har diariefört det som klagomål mot annan medlem. Den berörda personen kommer att informeras och få möjlighet att bemöta. Styrelsen behandlar därefter."*
- **Klagomål mot medlem — informering till berörd** — *"Hej [namn], styrelsen har mottagit ett klagomål som rör dig. Klagomålet handlar om: [neutralt sammandrag]. Du har möjlighet att lämna synpunkter senast DD/MM. Inget formellt beslut har fattats."*
- **Förslag — beslut att ej gå vidare** — *"Tack för ditt förslag. Styrelsen har övervägt det och valt att inte gå vidare i nuläget av följande skäl: [saklig grund]. Om förslaget rör en stadgebunden fråga är motion till stämman alternativ väg framåt — vi hjälper gärna med formulering."*
- **Ersättningsanspråk — bedömning** — *"Vi har bedömt ditt ersättningsanspråk om [belopp]. Beslut: [bifall / delvis / avslag]. Skäl: [saklig grund + stadge-/laghänvisning]. [Om avslag:] Du har möjlighet att gå vidare via civilrättslig process."*
- **Överföring** — *"Ditt ärende NN visade sig vara av typen [motion / utlägg / annan]. Det har överförts till nytt ärende MM med rätt handläggning. Du behöver inte göra något — vi hör av oss därifrån."*

## GDPR-hänsyn

Varierar per underkategori:

- `KLAGOMÅL_ANNAN_MEDLEM`: hög känslighet — personärende, samma försiktighet som [uteslutning.md](uteslutning.md). Bilagor om berörd person registreras med strängare visibility.
- `ERSÄTTNINGSANSPRÅK`: bokföringsrelaterat — bevaras 7 år enligt bokföringslagen om ersättning utbetalats.
- Övriga: måttlig — gallras 3 år efter `AVSLUTAT` om ärendet inte lett till annat governance-beslut.

## Hot och försvar

Se [threats.md](../threats.md):

- **Påtryckning mellan möten via "informellt"-flagga:** alla ärenden går via diarieföring; sekreterare tilldelar svarstid.
- **Klagomål som personangrepp:** `KLAGOMÅL_ANNAN_MEDLEM` kräver berörd parts svarsmöjlighet och förhindrar att klagomål utan grund eskaleras till formellt ärende.
- **Volym-spam:** sekreterare kan markera serie av identiska ärenden som dubletter; mönster över tid synligt för styrelsen.
- **Maktkoncentration:** att alla ärenden diariefors av sekreterare (inte selektivt) säkerställer likabehandling — ingen styrelseledamot kan ensidigt välja vilka ärenden som syns.
- **Ej-medlems-anspråk (ersättning):** registreras med extern aktör, hanteras med samma formalia som medlems-ärenden.

## Öppna frågor

- `[ÖPPEN]` Vid `KLAGOMÅL_ANNAN_MEDLEM` som upprepas mot samma person: ska systemet flagga mönstret för styrelsen? Förslag: ja, sekreterare ser flaggan vid `DIARIEFÖRT`; ingen automatisk eskalering — bedömning är styrelsens.
- `[ÖPPEN]` Får anonym anmälan göras? Förslag: nej för formellt ärende — anonyma synpunkter får skickas via anslagstavla; allmänt ärende kräver identifierbar initiator för att kunna besvara.
- `[ÖPPEN]` Kan medlem stänga sitt eget ärende före `BESVARAT`? Förslag: ja, via `ÅTERKALLAT`-sidospår.
- `[ÖPPEN]` Diarienummer-format: löpnummer per år (`2026-NNN`), per kategori, eller globalt? Förslag: löpnummer per år (svensk förvaltningstradition).
- `[ÖPPEN]` Får sekreteraren ändra kategori efter `DIARIEFÖRT`? Förslag: ja, men ändringen loggas som rättelse-post med motivering.
