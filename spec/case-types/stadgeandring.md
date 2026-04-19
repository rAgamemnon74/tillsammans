# Stadgeändring

> Del av [Tillsammans-specifikation](../../tillsammans.md). Specialiserar [gemensam ärendehanterings-infrastruktur](../case-types.md#gemensam-infrastruktur-för-ärendehantering).

## Syfte

Förändring av föreningens stadgar — den juridiska grundvalen som styr allt annat governance-arbete. Lagstiftaren har försett stadgeändringen med extra spärrar: **två stämmobeslut** (LEF 6:36, LFS 52§) med specifika majoriteter. Ärendetypen håller dessa två beslut samman som ett sammanhållet flöde.

**Edition:** bägge.

## Primär initiator

`BOARD_*` genom styrelsebeslut, eller `MEMBER` via motion (motionen blir vid bifall styrelseförslag till stadgeändring i nästa cykel).

## Obligatoriska fält

- **Förslagets text** — den nya stadgaformuleringen.
- **Diff mot nuvarande stadga** — strukturerad jämförelse, sida-vid-sida (genereras av stadge-modulen).
- **Berörda paragrafer** — vilka §§ ändras.
- **Motivering** — varför ändringen behövs.
- **Konsekvensbedömning** — vilka beslut, processer, och rättigheter påverkas; särskilt om medlemmars rättigheter eller skyldigheter förändras (utlöser samtyckes-förnyelse, se [medlemskap.md#samtyckes-förnyelse-vid-stadgeändring](../medlemskap.md#samtyckes-förnyelse-vid-stadgeändring)).
- **Kategorisering:** *väsentlig* (rättigheter, skyldigheter, avgifter, röstregler) eller *redaktionell* (språkjusteringar, tekniska förtydliganden).
- **Stadgaförankring** — paragrafen som reglerar stadgeändring (LEF 6:36 / LFS 52§).
- **Bilagor:** ev. juridisk granskning, jämförelse med normalstadga.

## Livscykel

Specialiserad med **två AVGJORT-tillstånd** (en per stämma):

```
INLÄMNAD
  → BEREDNING (styrelsen tar upp)
    → YTTRANDE_FÖRLAGT (om initiator är medlem)
      → PUBLICERAD_FÖRSTA_STÄMMAN
        → PÅ_FÖRSTA_STÄMMAN
          → AVGJORD_FÖRSTA_STÄMMAN
            · BIFALL_PRELIMINÄRT (förslaget går vidare)
            · AVSLAG (slutgiltigt)
            · ÅTERREMISS

[Mellanperiod — minst stadge-bunden tid mellan stämmorna]

  → PUBLICERAD_ANDRA_STÄMMAN
    → PÅ_ANDRA_STÄMMAN
      → AVGJORD_SLUTGILTIGT
        · BIFALL_SLUTGILTIGT (STADGEVERSION_ANTAGEN skrivs)
          → VERKSTÄLLD (samtyckes-förnyelse triggas vid väsentlig ändring)
            → AVSLUTAD
        · AVSLAG (förslaget faller — ingen ny version antas)

Sidospår:
  → ÅTERKALLAD (styrelsen drar tillbaka, kräver nytt styrelsebeslut)
  → AVVISAD_PÅ_FORMALIA (saknar juridiskt giltig formulering)
```

Båda `AVGJORD_*`-händelserna registreras separat och måste båda ske med stadgeenlig majoritet (typiskt 2/3 av avgivna röster, eller högre per stadga).

## RBAC per övergång

| Övergång | Roller |
|---|---|
| skapa INLÄMNAD | `BOARD_*` eller `MEMBER` (via motion) |
| INLÄMNAD → BEREDNING | `BOARD_*` |
| BEREDNING → YTTRANDE_FÖRLAGT | `BOARD_*` (om initiator är medlem) |
| → PUBLICERAD_*_STÄMMAN | automatik vid kallelseutskick |
| PÅ_*_STÄMMAN → AVGJORD_* | stämmans beslut, verkställs av `BOARD_SECRETARY` |
| AVGJORD_SLUTGILTIGT → VERKSTÄLLD | `BOARD_SECRETARY` (uppdaterar stadge-modulen) |

## Tidsregler

- **Mellan stämmorna:** stadge-bunden minimitid (typiskt 1 månad enligt LFS 52§; LEF 6:36 kräver två på varandra följande stämmor utan minimum, men praxis är minst 1 månad).
- **Båda stämmorna inom skälig tid:** rekommendation att andra stämman sker inom 12 månader för att undvika att förslaget åldras.
- **Verkställighet:** stadgeändringen träder i kraft vid `STADGEVERSION_ANTAGEN`-händelsens datum (typiskt direkt efter andra stämman).
- **Samtyckes-förnyelse-fönster:** vid väsentlig ändring får medlemmar stadge-bunden tid (default 90 dagar eller nästa årsstämma) att förnya samtycke, se [medlemskap.md](../medlemskap.md#samtyckes-förnyelse-vid-stadgeändring).

## Relationer

- Stadgeändring ↔ två `STÄMMOBESLUT` (1..2).
- Stadgeändring ↔ `STADGEVERSION_ANTAGEN`-händelse vid slutgiltigt bifall.
- Stadgeändring ↔ tidigare `Stadgeversion` (diff-bas).
- Stadgeändring ↔ ny `Stadgeversion` (resultat).
- Stadgeändring ↔ ev. samtyckes-förnyelseärenden (per medlem) vid väsentlig ändring.
- Stadgeändring ↔ ev. ursprunglig motion (om initiator var medlem).

## Publicering

| Status | Publik | Medlem | Styrelse |
|---|---|---|---|
| INLÄMNAD / BEREDNING | — | — | läs |
| YTTRANDE_FÖRLAGT | — | — | läs |
| PUBLICERAD_FÖRSTA | **läs** | **läs** | läs |
| AVGJORD_FÖRSTA | **läs** | läs | läs |
| PUBLICERAD_ANDRA | **läs** | **läs** | läs |
| AVGJORD_SLUTGILTIGT | **läs** | läs | läs |
| VERKSTÄLLD | **läs** (ny stadgaversion) | läs | läs |

Stadge-modulen visar diff sida-vid-sida; ny version blir aktiv först efter `VERKSTÄLLD`. Versionshistoriken i stadge-modulen reflekterar ändringen först efter slutgiltigt bifall.

## Notifiering

| Övergång | Notifiering till |
|---|---|
| BEREDNING (motion-initiator) | Initiator (kvittens) |
| PUBLICERAD_*_STÄMMAN | Alla medlemmar (via kallelseutskick) |
| AVGJORD_FÖRSTA_BIFALL_PRELIMINÄRT | Alla medlemmar (informativ — andra stämman kommer) |
| AVGJORD_SLUTGILTIGT | Alla medlemmar, revisorn, valberedningen, ev. Bolagsverket (LEF) |
| VERKSTÄLLD | Alla medlemmar (samtyckes-förnyelse om väsentlig), ev. Bolagsverket |

## Textbibliotek

- **Förslags-skelett** — *"Föreslagen formulering av §X: '[ny text]'. Nuvarande lydelse: '[tidigare text]'. Skäl för ändringen: [motivering]. Konsekvenser: [påverkade rättigheter/skyldigheter]."*
- **Yttrande-skelett (om motion)** — *"Styrelsen har granskat förslaget. Stadgeenlighet: [bedömning]. Konsekvensbedömning: [...]. Styrelsen yrkar [bifall / avslag] med följande motivering: [...]."*
- **Bekräftelse efter första stämman (vid bifall preliminärt)** — *"Stadgeändringen om §X har preliminärt antagits av första stämman den DD/MM. Ändringen träder i kraft först efter andra stämmans bekräftande beslut, planerat till [datum]."*
- **Slutgiltig bekräftelse** — *"Stadgeändringen om §X är slutgiltigt antagen av andra stämman den DD/MM. Ny stadgaversion publiceras på [länk] från [datum]. [Vid väsentlig ändring:] Som medlem behöver du bekräfta samtycke till den nya versionen senast DD/MM."*
- **Samtyckes-förnyelse-fråga** — *"Hej [namn], föreningens stadgar är ändrade i §X med påverkan på [rättighet/skyldighet]. Bekräfta att du tagit del av ändringen och samtycker till den nya versionen via [länk] senast DD/MM. Saknad bekräftelse aktiverar VILANDE-status med möjlighet att återgå till AKTIV genom senare bekräftelse."*

## GDPR-hänsyn

Stadgeändringen i sig är inte personrelaterad — låg känslighet. Samtyckes-förnyelse-trigger genererar dock per-medlem-spår med personuppgifter; dessa hanteras enligt [medlemskap.md](../medlemskap.md). Stadgeändringen och dess beslutshistorik bevaras långsiktigt som governance-historik.

## Hot och försvar

Se [threats.md](../threats.md):

- **Smyg-ändring av medlemmars rättigheter:** kategorisering *väsentlig* vs *redaktionell* + samtyckes-förnyelse → medlemmen tvingas se ändringen aktivt.
- **Övermajoritet kringgår minoritet:** två-stämmokravet ger tid för opposition att organisera sig.
- **Ren omformulering döljer reell förändring:** diff-jämförelse i stadge-modulen + revisorns granskning av kategoriseringen ([medlemskap.md#öppna-frågor](../medlemskap.md#öppna-frågor)).
- **Bristande publicering:** publik diff-vy + automatisk inkludering i kallelse är default.

## Öppna frågor

- `[ÖPPEN]` Mellan-stämmo-tid: minst 1 månad eller stadge-bunden? Förslag: stadge-default 1 månad, kan inte sättas under lagminimum.
- `[ÖPPEN]` Vid `BIFALL_PRELIMINÄRT` följt av väsentligt ändrad omvärld: får styrelsen återkalla mellan stämmorna? Förslag: ja, med nytt styrelsebeslut + tydlig kommunikation till medlemmarna; ärendet får sidospår `ÅTERKALLAD`.
- `[ÖPPEN]` Hur hanteras klandertalan mot stadgeändringen (LEF 7:54)? Klandertalan-fönstret löper efter slutgiltigt antagande. Stadgeändringens `VERKSTÄLLD`-status kan behöva pausa om talan väcks. Förslag: sätt `VERKSTÄLLD` direkt; ev. domstolsavgörande hanteras som `TILLÄGG_KLANDERTALAN` enligt [granskningslogg.md](../granskningslogg.md#retroaktiva-tillägg-addenda-till-stängd-epok).
- `[ÖPPEN]` Vid Bolagsverket-registrering (LEF): triggar ändringen automatisk inlämning, eller manuell? Förslag: manuell — sekreteraren genererar dokument från stadge-modulen, sparar Bolagsverket-kvittensen som bilaga.
