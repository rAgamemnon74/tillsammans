# Ägarövergång

> Del av [Tillsammans-specifikation](../../tillsammans.md). Specialiserar [gemensam ärendehanterings-infrastruktur](../case-types.md#gemensam-infrastruktur-för-ärendehantering).

## Syfte

Registrerar ägarbyte av en fastighet i samfälligheten. **Edition: samfällighet** (LFS — medlemskap följer fastigheten). Inget formellt godkännandebeslut — ägarbytet är en juridisk realitet utanför föreningens kontroll. Ärendet är registrerings-fokuserat: båda aktörers status uppdateras, debiteringen anpassas, och historiken bevaras.

Hela mekaniken specas i [medlemskap.md#samfällighet--medlemskap-följer-fastigheten](../medlemskap.md#samfällighet--medlemskap-följer-fastigheten).

## Primär initiator

`BOARD_*` eller `medlemsansvarig` — typiskt ordförande eller medlemsansvarig som registrerar bytet. Underrättelsen kommer normalt från säljare, köpare eller från Lantmäteriets uppgift.

`MEMBER` (säljaren) kan själv anmäla ägarbytet via formulär; styrelsen verifierar och registrerar.

## Obligatoriska fält

- **Fastighetsbeteckning** — den berörda `PropertyUnit`.
- **Avgående aktör** — referens till befintlig `Medlem`.
- **Tillträdande aktör** — ny aktör (`AKTOR_REGISTRERAD` skrivs först om aktören saknas).
- **Faktiskt tillträdesdatum** (`actual_transfer_date`) — kontraktsdatum eller Lantmäteriets registreringsdatum.
- **Källa till uppgiften** — köpekontrakt, Lantmäteri-utdrag, säljarens anmälan eller köparens anmälan.
- **Ev. samäganderelation** — nya ägares andelar om flera ägare.
- **Bilagor** (rekommenderat): köpekontrakt eller Lantmäteri-utdrag.

## Livscykel

Specialiserad — kortare än normala ärenden eftersom inget beslutsmoment finns:

```
INKOMMEN (uppgiften om ägarbyte registrerad)
  → VERIFIERAD (styrelsen har bekräftat uppgiften)
    → REGISTRERAD (MEDLEMSKAP_ÖVERGÅTT skrivs)
      → AVSLUTAT (debiteringsändring genomförd, ärendet stängt)

Sidospår:
  → AVVISAT_PÅ_FORMALIA (uppgiften kan inte verifieras — inget bevis på överlåtelse)
  → ÅTERKALLAT (anmälaren drar tillbaka, t.ex. om köpet hävts)
```

Vid `REGISTRERAD` skrivs `MEDLEMSKAP_ÖVERGÅTT` med båda aktörers id som referens. Avgående medlems status → `VILANDE`; tillträdande → `AKTIV`.

## RBAC per övergång

| Övergång | Roller |
|---|---|
| skapa INKOMMEN | `BOARD_*`, `medlemsansvarig`, eller `MEMBER` (säljaren själv) |
| INKOMMEN → VERIFIERAD | `BOARD_*` eller `medlemsansvarig` (med obligatorisk jäv-deklaration vid släktskap eller affärsrelation) |
| VERIFIERAD → REGISTRERAD | `BOARD_*` eller `medlemsansvarig` |
| REGISTRERAD → AVSLUTAT | automatik när debiteringen är uppdaterad |

## Tidsregler

- **Registrering snarast** efter underrättelse — debiteringsperiodens nästa start är hård deadline för korrekt fakturering.
- **Faktiskt tillträdesdatum** styr debiteringen, inte registreringsdatumet (se [medlemskap.md](../medlemskap.md#epok-tillhörighet)).
- **Stängd epok-frågan:** om `actual_transfer_date` ligger i en redan förseglad epok kan ett addendum bli aktuellt — se [granskningslogg.md#retroaktiva-tillägg-addenda-till-stängd-epok](../granskningslogg.md#retroaktiva-tillägg-addenda-till-stängd-epok).

## Relationer

- Ägarövergång ↔ `PropertyUnit` (1..1, central).
- Ägarövergång ↔ avgående `Medlem` (1..1).
- Ägarövergång ↔ tillträdande `Medlem` (1..n vid samägande).
- Ägarövergång ↔ `MEDLEMSKAP_ÖVERGÅTT`-händelse.
- Ägarövergång ↔ ev. tidigare ägarövergång på samma fastighet (kronologisk kedja).
- Ägarövergång ↔ `DEBITERINGSLÄNGD_FASTSTÄLLD` — påverkar nästa lista.

## Publicering

| Status | Publik | Medlem | Styrelse | Berörda |
|---|---|---|---|---|
| INKOMMEN | — | — | läs | läs (avg./tillträd.) |
| VERIFIERAD | — | — | läs | läs |
| REGISTRERAD | fastighetsbeteckning + datum¹ | läs | läs | läs |
| AVSLUTAT | samma | läs | läs | läs |

¹ Fastighetsbeteckning är en publik uppgift via Lantmäteriet; ägarbyte är registrerbart fakta. Ägarens namn kan vara publikt eller endast medlemssynligt enligt föreningens stadge-inställning.

## Notifiering

| Övergång | Notifiering till |
|---|---|
| INKOMMEN | Avgående och tillträdande aktör (bekräftelse) |
| VERIFIERAD | (intern) |
| REGISTRERAD | Avgående medlem (status `VILANDE`, sluträkning), tillträdande medlem (välkomst + andelstal + nästa debitering), kassören |
| AVSLUTAT | (intern) |

## Textbibliotek

- **Bekräftelse till säljare** — *"Vi har mottagit uppgift om ägarbyte för fastighet [beteckning], tillträdesdatum DD/MM. När styrelsen verifierat uppgiften kommer ditt medlemskap att avslutas formellt och slutavräkning skickas. Tack för din tid som medlem."*
- **Välkomst till köpare** — *"Hej [namn], välkommen som ny ägare av [beteckning] och därmed medlem i [föreningen]. Ditt andelstal är [X], och nästa debitering om [belopp] avser perioden [...]. Stadgar finns på [länk]; styrelsen kontaktas via [kanal]."*
- **Verifieringsfråga** — *"Vi har mottagit anmälan om ägarbyte för [beteckning] men behöver komplettera med [köpekontrakt / Lantmäteri-utdrag] för att registrera bytet. Skicka dokumentet via [länk] eller post."*
- **Slutavräkning till säljare** — *"Här är slutavräkningen för ditt medlemskap fram till tillträdesdatum DD/MM: [periodiserad debitering]. Eventuell återbetalning sker till [konto]."*

## GDPR-hänsyn

Måttlig känslighet. Avgående medlems data övergår till `VILANDE` och gallras enligt stadgad tid (typiskt 2–5 år; se [medlemskap.md#gallring](../medlemskap.md#gallring)). Bokföringsanknutna uppgifter (debitering) bevaras 7 år enligt bokföringslagen. Personnummer hanteras inte annorlunda än för aktiv medlem.

## Hot och försvar

Se [threats.md](../threats.md):

- **Felregistrering / oklart ägarskap:** styrelsen registrerar inte utan verifierbart underlag → `INKOMMEN → VERIFIERAD` är medvetet steg, inte automatik.
- **Civilrättsliga tvister mellan säljare och köpare:** "Tillsammans löser inga civilrättsliga tvister" ([core-concepts.md#röstlängdsetablering](../core-concepts.md#röstlängdsetablering--processen-vid-stämmans-öppnande)) — ärendet pausas vid `VERIFIERAD` om dokumentationen är tvistig; parterna hänvisas till Lantmäteriet/domstol.
- **Jäv vid registrering:** medlemsansvarig som är släkt med köpare/säljare måste deklarera jäv och eskalera till annan styrelseledamot.

## Öppna frågor

- `[ÖPPEN]` Lantmäteri-integration för automatisk uppgift om ägarbyte är post-MVP enligt [medlemskap.md#öppna-frågor](../medlemskap.md#öppna-frågor) — tills dess är ärendet manuellt.
- `[ÖPPEN]` Vad händer om både säljare och köpare anmäler ägarbyte med olika datum eller motstridiga uppgifter? Förslag: ärendet stannar vid `INKOMMEN`, styrelsen begär in köpekontrakt och avgör baserat på det objektiva dokumentet.
- `[ÖPPEN]` Vid samägande som splittras (en delägare köper ut den andra): ny `Ownership`-konfiguration, eller ny `MEDLEMSKAP_ÖVERGÅTT`? Förslag: `MEDLEMSKAP_ÖVERGÅTT` även här — `Ownership` är intern struktur, händelsen är den auktoritativa loggen.
