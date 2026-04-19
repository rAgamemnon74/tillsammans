# Revisorsfråga

> Del av [Tillsammans-specifikation](../../tillsammans.md). Specialiserar [gemensam ärendehanterings-infrastruktur](../case-types.md#gemensam-infrastruktur-för-ärendehantering).

## Syfte

Revisorns kanal för löpande frågor till styrelsen under räkenskapsåret — innan revisionsberättelsen färdigställs. Tröskeln är medvetet låg: revisorn ska kunna ställa frågor utan att det känns som kritik. Iterativ kommunikation, inget formellt beslut. Frågorna och svaren bidrar till revisionsberättelsens underlag och bevarar revisorns granskningsspår över tid.

**Edition:** bägge.

## Primär initiator

`AUDITOR` (revisor) eller `AUDITOR_DEPUTY` (revisorssuppleant när hen agerar som revisor).

## Obligatoriska fält

- **Adressat** — vilken styrelseroll förväntas svara (`BOARD_TREASURER` för ekonomi, `BOARD_CHAIR` för förvaltningsfrågor, `BOARD_*` för generella).
- **Frågetext** — fri text.
- **Kategori** — `EKONOMI` / `FÖRVALTNING` / `STADGAEFTERLEVNAD` / `JÄV` / `INTERNKONTROLL` / `ANNAT`.
- **Berör räkenskapsår** — vilken epok frågan rör (default innevarande, kan vara föregående under sigilleringsperioden).
- **Ev. referens** — till specifikt `STYRELSEBESLUT`, utlägg, debiteringslängd-post eller annan händelse i loggen.
- **Brådskandegrad** — `RUTIN` / `INFÖR_REVISIONSBERÄTTELSE` / `KRITISK`. Defaultar till `RUTIN`.

## Livscykel

Specialiserad — iterativ kommunikation, inget formellt beslut:

```
STÄLLD (revisorn skriver REVISORSFRÅGA_STÄLLD)
  → ANSVAR_TILLDELAT (styrelsen tar emot, väljer svarande)
    → SVAR_AVLÄMNAT (REVISORSSVAR_AVLÄMNAT)
      → AVSLUTAT (revisorn nöjd)
      → FÖLJDFRÅGA (revisorn vill veta mer)
        → ANSVAR_TILLDELAT (ny iteration)
      → INKLUDERAT_I_REVISIONSBERÄTTELSE (revisorn lyfter frågan i sin årsberättelse)

Sidospår:
  → ÅTERKALLAD (revisorn drar tillbaka — frågan visade sig irrelevant)
```

Iterationer (`SVAR_AVLÄMNAT → FÖLJDFRÅGA → SVAR_AVLÄMNAT`) är obegränsade — varje runda är egen händelse i loggen.

## RBAC per övergång

| Övergång | Roller |
|---|---|
| skapa STÄLLD | `AUDITOR`, `AUDITOR_DEPUTY` |
| STÄLLD → ANSVAR_TILLDELAT | `BOARD_*` (typiskt sekreterare eller ordförande tilldelar) |
| ANSVAR_TILLDELAT → SVAR_AVLÄMNAT | utpekad svarande (`BOARD_*`) |
| SVAR_AVLÄMNAT → AVSLUTAT / FÖLJDFRÅGA | `AUDITOR`, `AUDITOR_DEPUTY` |
| → INKLUDERAT_I_REVISIONSBERÄTTELSE | `AUDITOR` (vid revisionsberättelsens färdigställande) |

## Tidsregler

- **Svarstid default:** 14 dagar från `ANSVAR_TILLDELAT` → påminnelse till svarande.
- **Brådskande (`KRITISK`):** 3 dagar; flagga till hela styrelsen.
- **Inför revisionsberättelse:** revisorn anger deadline; default 7 dagar före revisionsberättelsens leveransdatum.
- **Ingen preskription** — frågor kan ställas löpande under hela räkenskapsåret och även efter sigill (men då via addendum-mekanismen, se [granskningslogg.md](../granskningslogg.md#retroaktiva-tillägg-addenda-till-stängd-epok)).

## Relationer

- Revisorsfråga ↔ ev. specifika `STYRELSEBESLUT`, `UTLÄGG_*`, `DEBITERINGSLÄNGD_FASTSTÄLLD`, eller annan händelse via `references`.
- Revisorsfråga ↔ `Epok` — frågan tillhör en specifik epok.
- Revisorsfråga ↔ ev. `Revisionsberättelse` — om `INKLUDERAT_I_REVISIONSBERÄTTELSE`.
- Revisorsfråga ↔ uppföljnings-revisorsfrågor i kommande räkenskapsår.

## Publicering

| Status | Publik | Medlem | Styrelse | Revisor |
|---|---|---|---|---|
| STÄLLD | — | — | läs (informativ) | läs + skriv |
| ANSVAR_TILLDELAT | — | — | läs (svarande får skriv) | läs |
| SVAR_AVLÄMNAT | — | — | läs | läs |
| AVSLUTAT / FÖLJDFRÅGA | — | — | läs | läs |
| INKLUDERAT_I_REVISIONSBERÄTTELSE | (via revisionsberättelsens publicering) | (samma) | läs | läs |

Revisorsfrågor är **interna** — publik insyn sker först vid revisionsberättelsens publicering, och bara om revisorn väljer att inkludera frågan där.

## Notifiering

| Övergång | Notifiering till |
|---|---|
| STÄLLD | Adressat-roll (typiskt kassör eller ordförande), styrelsen (informativ) |
| ANSVAR_TILLDELAT | Utpekad svarande |
| SVAR_AVLÄMNAT | Revisorn |
| FÖLJDFRÅGA | Tidigare svarande, styrelsen (informativ) |
| INKLUDERAT_I_REVISIONSBERÄTTELSE | Styrelsen, ev. medlemmar (via revisionsberättelse-publicering) |

## Textbibliotek

Tonen är samarbetsvillig och ödmjuk på bägge sidor — undvik allt som kan upplevas som anklagelse. Frågan är revisorns informationsbehov, inte mistroendevotum.

- **Frågemall (revisor)** — *"Hej styrelsen, jag vill förstå följande: [konkret fråga]. Bakgrunden är [varför detta är relevant för min granskning]. Inget brådskande — svar inom [tidsfrist] räcker. Tack."*
- **Tilldelnings-bekräftelse** — *"Hej [revisor], frågan är registrerad. [Svarande-namn] tar fram svar och återkommer inom [tidsfrist]."*
- **Svarsmall (styrelse)** — *"Hej [revisor], svar på din fråga om [ämne]: Följande beslut låg till grund för [...] (referens: [STYRELSEBESLUT NN]). Underlag bifogas. Säg till om något behöver kompletteras."*
- **Följdfråga (revisor)** — *"Tack för svaret. Jag behöver komplettera bilden i en punkt: [konkret fråga]. Resten är klart."*
- **Avslutnings-bekräftelse (revisor)** — *"Tack — frågan är besvarad. Jag noterar svaret som del av min granskning."*
- **Inkluderings-meddelande** — *"Hej styrelsen, jag har valt att lyfta denna fråga i revisionsberättelsen som [observation / anmärkning / rekommendation]. Texten finns i utkastet på [länk] för er möjlighet att kommentera innan färdigställande."*

## GDPR-hänsyn

Revisorsfrågor är interna styrelse–revisor-kommunikation. Personuppgifter förekommer bara om frågan rör en specifik medlem (t.ex. utlägg-relaterat) — då gäller samma roll-baserade åtkomst som källhändelsen. Bevaras i loggen så länge granskningsperioden + ev. preskriptionstid kräver (typiskt 5 år efter epokens sigill enligt LEF 8:13).

## Hot och försvar

Se [threats.md](../threats.md):

- **Försvarsmekanismen *är* uppdraget:** systemet får inte skapa friktion som gör att revisorn slutar fråga. Låg tröskel + samarbetsvillig ton i mallar.
- **Asymmetrier:** styrelsen kan inte avfärda eller dölja frågor — alla revisorsfrågor är synliga för hela styrelsen.
- **Historierevision:** iterativa svar bevaras evigt i loggen; senare omtolkning av styrelsens svar är spårbar.
- **Passiv erosion:** påminnelser vid passerad svarstid säkerställer att frågor inte hamnar i styrelse-glömskan.

## Öppna frågor

- `[ÖPPEN]` Får revisor ställa frågor till medlem (t.ex. om en specifik motion-historia)? Förslag: nej — revisorn granskar styrelsen, inte medlemmarna. Vid behov av medlemsperspektiv går frågan via styrelsen.
- `[ÖPPEN]` Återkommer obesvarade revisorsfrågor som varningsflaggor i nästa stämma? Förslag: ja, *"Antal obesvarade revisorsfrågor vid räkenskapsårets slut: X"* visas i revisionsberättelsens översikt.
- `[ÖPPEN]` Får tidigare revisor (efter mandatets slut) ställa frågor om sin egen mandatperiod? Förslag: ja, men bara på existerande frågor (`FÖLJDFRÅGA`) — nya frågor är innevarande revisors ansvar. Konsistent med [oversight-roles.md#byte-av-revisor-mid-år](../roles/oversight-roles.md#byte-av-revisor-mid-år).
