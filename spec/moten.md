# Möten — strukturer och dagordningsmallar

> Del av [Tillsammans-specifikation](../tillsammans.md).

Möten är där föreningens governance manifesteras: styrelsen beslutar, stämman avgör, protokollet bevarar. Den här filen specar **strukturen** för tre mötestyper, deras **dagordningsmallar**, och hur föreningens stadgar — via taggade paragrafer — driver automatiskt systemstöd som förenklar styrelsens administration.

**Relaterade dokument:**
- [core-concepts.md](core-concepts.md) — kallelsemodell, röstlängdsetablering, jäv, reservation, bordläggning
- [roles/meeting-roles.md](roles/meeting-roles.md) — mötesordförande, mötessekreterare, justerare, rösträknare
- [granskningslogg.md](granskningslogg.md) — möteshändelser i loggen
- [föreningen.md](föreningen.md) — möten i årshjulet och två-perioders-modellen
- [case-types.md](case-types.md) — ärenden som behandlas på möten
- [case-types/extrastamma.md](case-types/extrastamma.md) — kallelsebegäran för extra stämma

## Tre mötestyper

| Mötestyp | Förekomst | Beslutsfattare | Beslutsmandat | Stadge-bundet |
|---|---|---|---|---|
| **Styrelsemöte** | 4–8 per år vanligt; min 1–2 | Styrelsen | Inom styrelsens befogenheter enligt stadga | Frekvens, ev. minimi-möten |
| **Ordinarie årsstämma** | En gång per räkenskapsår (LEF 6:14, LFS 47§) | Medlemmar (eller ombud) | Övergripande, inkl. ansvarsfrihet | Tidpunkt, kallelseregler, obligatoriska punkter |
| **Extra stämma** | Vid behov (LEF 6:13, LFS 47§) | Medlemmar (eller ombud) | Bara punkter som angivits i kallelsen | Initieras av styrelse, medlemsminoritet eller revisor |

Styrelsemötens dagordning är flexibel; stämmodagordningar har lagligt obligatoriska punkter som varje föreningsstadga preciserar i detalj.

## Dagordningsmallar

### Styrelsemöte — standardmall

Inga lagkrav på struktur, men närvarokontroll och behörighet är fasta första punkter eftersom alla deltagare är systemanvändare och digital validering är effektiv.

```
1.  Mötets öppnande (ordförande eller vice ordförande)
2.  Närvarokontroll                                          [auto, digital]
3.  Mötets behörighet                                        [auto, mot KVORUM]
4.  Val av justerare (om inte stadga föreskriver fast process)
5.  Godkännande av föregående mötesprotokoll
6.  Genomgång av åtgärder från föregående möte
7.  Ekonomisk rapport (kassör)
8.  Inkomna ärenden under behandling (per ärendetyp)
9.  Beslutspunkter
10. Övriga frågor (informationspunkter, ej beslut)
11. Nästa möte och eventuella förberedelser
12. Mötets avslutande
```

Punkt 2-3 är digital-stödda enligt [rostlangd.md#närvarokontroll-för-styrelsemöten---digital](rostlangd.md). Punkt 8 förfylls automatiskt med ärenden som har status `BEREDNING` eller väntar på styrelsebeslut (från [case-types/](case-types/)). Punkt 9 förfylls med pågående ärenden där beslutsdeadline närmar sig.

### Ordinarie årsstämma — standardmall

Bygger på LEF 6-kapitel och Lantmäteriets normalstadgor för samfällighet. Lagligt obligatoriska punkter markerade `[lag]`; auto-stöd-aktiva punkter markerade `[auto]`.

```
1.  Mötets öppnande
2.  Val av mötesordförande                                          [lag] [auto]
3.  Val av mötessekreterare                                         [lag] [auto]
4.  Val av två justeringspersoner tillika rösträknare              [lag] [auto]
5.  Fråga om stämman kallats enligt stadgarna                      [lag] [auto]
6.  Godkännande av dagordning
7.  Fastställande av röstlängd                                     [lag — pappers-rutin, skannas till protokollet efter mötet]
8.  Styrelsens verksamhetsberättelse                               [bilaga obligatorisk]
9.  Revisionsberättelse                                            [auto]
10. Fastställande av resultat- och balansräkning                   [lag]
11. Beslut om ansvarsfrihet för styrelsen                          [lag] [auto]
12. Beslut om disposition av årets resultat
13. Beslut om budget och avgifter (kommande år)
14. Beslut om underhållsfond-avsättning (samfällighet)             [auto, om §]
15. Beslut om debiteringslängd (samfällighet)                      [lag, om samf] [auto]
16. Behandling av inkomna motioner och styrelseförslag             [auto]
17. Val av styrelseledamöter och suppleanter                       [lag]
18. Val av revisor och revisorssuppleant                           [lag]
19. Val av valberedning
20. Övriga ärenden (informationspunkter, ej beslut)
21. Mötets avslutande
```

Punkter och ordning är vanligast; specifik förenings-stadga kan föreskriva annan ordning eller ytterligare obligatoriska punkter (t.ex. arvoden, fondering).

### Extra stämma — standardmall

Begränsad till de frågor som angivits i kallelsen — extra stämma får inte behandla annat. Övriga punkter är formaliarutiner identiska med ordinarie:

```
1. Mötets öppnande
2. Val av mötesordförande                                           [lag] [auto]
3. Val av mötessekreterare                                          [lag] [auto]
4. Val av justeringspersoner                                        [lag] [auto]
5. Fråga om kallelse skett enligt stadgarna                         [lag] [auto]
6. Godkännande av dagordning
7. Fastställande av röstlängd                                       [lag] [auto]
8. Behandling av den/de specifika frågor som angivits i kallelsen   [auto från ärendet]
9. Mötets avslutande
```

Punkt 8 förfylls automatiskt från kallelsebegäran-ärendet ([case-types/extrastamma.md](case-types/extrastamma.md)) — extra stämman kan bara ta upp det som motiverade kallelsen.

## Stadgeparagraf-typer med automatiskt systemstöd

Föreningens stadga är inte bara ett dokument — när dess paragrafer **taggas** med en typkod kan systemet *agera* på dem. Detta är den centrala kopplingen mellan stadgan och Tillsammans automatik.

Vid onboarding genomgås föreningens stadga paragraf för paragraf. För varje paragraf väljer den onboardande personen en av följande typer (eller "ingen — fri text"):

| Stadgeparagraf-typ | Vad systemet gör automatiskt |
|---|---|
| `RÄKENSKAPSÅR` | Öppnar epok vid räkenskapsårets början; flaggar sigill-tidpunkt |
| `KALLELSETID` | Räknar deadline för kallelseutskick; varnar för min/max-överträdelse |
| `STÄMMOPERIOD` | Föreslår stämmodatum inom föreskrivet fönster |
| `MOTIONSFRIST` | Stänger automatiskt motions-mottagningen vid rätt tid |
| `RÖSTMETOD` | Konfigurerar röstlängd-genereringen (huvudmetod / andelsmetod / kombination) |
| `MAJORITET` | Sätter godkännandetröskel per beslutstyp (enkel / kvalificerad 2/3 eller 3/4) |
| `KVORUM` | Räknar närvarande mot beslutsförhets-tröskel; flaggar `BEHÖRIGT`/`EJ_BESLUTSFÖRT` vid styrelsemöte. För stämma: visar tröskel som referens. Se [rostlangd.md#stadgeparagraf-typ-kvorum](rostlangd.md#stadgeparagraf-typ-kvorum) |
| `STYRELSESAMMANSÄTTNING` | Validerar antal vid val; flaggar avvikelser |
| `MANDATPERIOD` | Beräknar mandatslut per ledamot; flaggar omval i god tid |
| `FIRMATECKNING` | Markerar beslut som kräver firmatecknares signering |
| `INSATS` (LEF) | Kräver insats-bekräftelse innan medlemskap aktiveras |
| `DEBITERINGSLÄNGD` (samfällighet) | Genererar årlig debiteringslängd från andelstal × budget |
| `UNDERHÅLLSFOND` | Beräknar årlig avsättning enligt formel |
| `STADGEÄNDRINGSMAJORITET` | Aktiverar två-stämmo-flöde med rätt tröskel ([case-types/stadgeandring.md](case-types/stadgeandring.md)) |
| `UTESLUTNINGSPROCESS` | Föreslår varning + svarsfönster-mekanik enligt stadgad tidsfrist |
| `EXTRA_STÄMMA_TRÖSKEL` | Sätter % av medlemmar som kan begära extra stämma |
| `ANSVARSFRIHET` | Triggar `EPOK_FÖRSEGLAD` med rätt sigill-variant baserat på stämmobeslut |

### Hur taggning fungerar

Vid onboarding:

1. Föreningen laddar upp sin gällande stadga som källdokument (PDF eller text).
2. Stadgan delas upp i paragrafer i stadge-modulen (manuellt eller via parser-stöd).
3. Per paragraf väljer onboarderen en typ ur listan ovan, eller "ingen". Otypade paragrafer är ren text utan auto-effekt.
4. För typade paragrafer fyller onboarderen i typens parametrar (t.ex. `KALLELSETID`: min 14 dagar, max 28 dagar).
5. Resultatet är att stadgan inte bara *finns* utan *fungerar* — systemet vet vad varje paragraf betyder operativt.

### Värdet — investering en gång, frukt över tid

Onboarding-arbetet att tagga stadgaparagraferna är påtagligt — kan ta några timmar för en stadga med 30 paragrafer. Värdet är att styrelsen *aldrig mer* behöver påminnas om kallelsetider, fondavsättningar eller motionsfrister manuellt. Systemet räknar, varnar och förfyller. Detta är skillnaden mellan ett dokument och en levande regelmotor.

### Otaggade paragrafer är fortfarande värdefulla

Inte alla paragrafer behöver auto-stöd. *"Föreningens säte är Falkenberg"* har ingen aktiv funktion — det är bara fakta. Sådana paragrafer förblir ren text, sökbar och refererbar men utan systemkonsekvens. Auto-stöd är för paragrafer som **driver återkommande administration**.

## Mötets livscykel i granskningsloggen

Använder transaktionstyper från [granskningslogg.md](granskningslogg.md):

### Före möte

- `KALLELSE_UTSKICKAD` — kallelse + dagordning + bilagor + tip-hash. För stämma: del av föregående epok (avslutar den).
- För årsstämma: tidsfristerna räknas baklänges från mötesdatum via `KALLELSETID`-paragrafen; systemet flaggar om kallelsen är försenad eller för tidig.

### Vid mötet

- `MÖTE_ÖPPNAT` — mötet startar, datum och tid registreras
- `STÄMMOBESLUT` (eller `STYRELSEBESLUT`) per dagordningspunkt med beslutsmoment
- `RÖST_AVGIVEN` per röstande aktör (medlem på stämma; ledamot på styrelsemöte)
- `JÄV_DEKLARERAT` före varje ekonomisk fråga
- `RESERVATION_INGIVEN` när relevant
- `MÖTE_AVSLUTAT` — mötet avslutas, slut-tidpunkt registreras
- För årsstämma: `EPOK_FÖRSEGLAD` med sigill-variant från ansvarsfrihetsbeslutet (se [föreningen.md](föreningen.md))

### Efter mötet

- `PROTOKOLL_UPPRÄTTAT` — utkast färdigställt av sekreteraren
- `PROTOKOLL_JUSTERAT` — justerare har signerat (typiskt 2–4 veckor efter mötet)
- För årsstämma: tip-hash publiceras enligt L6 (i protokollet, på anslagstavlan, i årsredovisningens bilagor)

## Stadge-driven anpassning — sammanfattning

Förenings-stadgan styr (via taggade paragrafer):

| Aspekt | Stadgeparagraf-typ | Default när ej stadgereglerat |
|---|---|---|
| Räkenskapsår | `RÄKENSKAPSÅR` | Kalenderår |
| Kallelse-tider | `KALLELSETID` | 2v min / 4v max ordinarie; 1v min / 2v max extra |
| Stämmoperiod | `STÄMMOPERIOD` | Mars–maj (kalenderår-föreningar) |
| Motionsfrist | `MOTIONSFRIST` | 2 veckor före stämma |
| Röstmetod | `RÖSTMETOD` | Huvudmetod (LEF/LFS-default) |
| Antal styrelseledamöter | `STYRELSESAMMANSÄTTNING` | Min 3, ofta 5 (samfällighet) |
| Mandatperiod | `MANDATPERIOD` | 2 år ordinarie, 1 år suppleant |
| Antal revisorer | — (per stadga) | 1 lekmanna + 1 suppleant |
| Underhållsfond-avsättning | `UNDERHÅLLSFOND` | Per stadga; ingen default |
| Stadgeändrings-tröskel | `STADGEÄNDRINGSMAJORITET` | 2/3 på två följande stämmor |

Vid onboarding går wizarden igenom dessa parametrar; defaults används för otydliga fall och kan justeras i efterhand när stadgan tolkas mer precist.

## Systemets förberedelsestöd per möte

**Sekreteraren är operativ huvudaktör** för förberedelsen — driver det praktiska arbetet i systemet medan ordförande beslutar formellt och kallar. Detaljerat ansvar specas i [roles/board-roles.md#sekreterarens-roll-vid-stämmoförberedelse](roles/board-roles.md#sekreterarens-roll-vid-stämmoförberedelse).

Inför varje möte erbjuder Tillsammans (sekreteraren använder):

1. **Dagordnings-mall** — föreslagen baserat på mötestyp + stadgad tagning + öppna ärenden i case-types-systemet
2. **Röstlängds-snapshot** — beräknad enligt stadgad röstmetod, med fullmakter inräknade
3. **Beslutsunderlag** — alla ärenden med status `PUBLICERAD` eller `BEREDNING` listade som agendapunkter med länk till ärendets aggregerade vy
4. **Tidsregler-validering** — kallelseperiod, motionsfrist, kvorum-krav
5. **Stadge-checklist** — obligatoriska punkter markerade enligt paragraf-tagningar; varning om dagordningen saknar något
6. **Påminnelser** — till berörda aktörer enligt mötestypens distributionsregel ([case-types.md#distribution-och-notifiering](case-types.md#distribution-och-notifiering))
7. **Förberedande dokument** — verksamhetsberättelse, revisionsberättelse, ekonomisk rapport — länkas in om de finns i dokumentarkivet

## Möten i edition-kontext

**Samfällighet:**
- Underhållsfond-avsättning är obligatorisk diskussionspunkt om stadgan föreskriver
- Debiteringslängd fastställs som del av årsstämman ([editions/samfallighet.md#andelstal-och-debiteringslängd](editions/samfallighet.md))
- Ofta extern administratör närvarande utan rösträtt; presenterar ekonomi
- Sub-typer (väg, vatten, multi-GA) påverkar inte mötesstrukturen, bara innehållet i ekonomi-rapporten

**LEF (övriga ekonomiska föreningar):**
- Insats-frågor vid nya medlemmar — ny medlem-godkännande är vanlig dagordningspunkt på styrelsemöten
- Bolagsverket-inlämning som efterarbete (ej dagordningspunkt) — sker av sekreterare/ordförande inom 7 månader efter sigill
- KU-rapportering till Skatteverket om utdelning sker; kassörens uppgift, inte dagordningspunkt

## Hybrid- och digitala möten

Möten kan hållas:

- **Fysiskt** — alla deltagare på plats
- **Digitalt** — alla deltagare via video/telefon
- **Hybrid** — kombination

Systemet förhåller sig neutralt till mötesform — röstlängd, beslutsregistrering och protokoll fungerar identiskt. Mötesordförande verifierar närvaro oavsett deltagandeform.

För digital deltagare: identifiering vid mötesöppning kan ske via inloggad session i systemet. Vid hybrid-möten är denna identifiering särskilt viktig så att man inte räknas dubbelt (närvarande både fysiskt och inloggat).

`[ÖPPEN]`: stadgekrav för digital närvaro varierar — vissa stadgar tillåter inte distansröstning. `RÖSTMETOD`-paragrafen kan utökas med `DIGITAL_RÖSTNING_TILLÅTEN: bool`.

## Mall-anpassning per förening

Centralt levererade mallar är **defaults**. Föreningen kan:

- **Lägga till punkter** — t.ex. "Information om sommarens gemensamma städning" som återkommande punkt
- **Ta bort punkter** — t.ex. om föreningen inte har valberedning som separat post (ovanligt)
- **Omordna punkter** — om stadgan föreskriver annan ordning
- **Lägga till lokal text** — t.ex. specifika formuleringar för rösträkning

Föreningens lokala mall-overlay sparas i konfigurationen; ändringar audittas i granskningsloggen som `STADGEVERSION_ANTAGEN`-relaterade händelser eller separata `MALL_ANPASSAD`-händelser.

`[ÖPPEN]`: behövs en separat händelsetyp för mallanpassningar, eller räcker det att de versioneras tillsammans med stadgan? Förslag: separat lättviktshändelse, eftersom mallar ändras oftare än stadgan och inte alltid som följd av stadgeändring.

## Öppna frågor

- **Stadge-modulen är ännu inte fullt specad.** Auto-stöd-paragraftyperna förutsätter dess existens. Behöver eget specdokument (`spec/stadge-modul.md`) som detaljerar paragraf-modellering, parser-stöd, versionering och taggnings-UI.
- **Onboarding-arbetsflödet för stadge-tagning** — hur ser UI:t ut? Förslag: guidad wizard som går igenom paragrafer en i taget med dropdown för typ-val. Kan ta 2–4 timmar för en stadga, men bara en gång.
- **Stadgar som blandar flera typer i en paragraf** — vissa paragrafer reglerar både kallelsetid och röstmetod. Lösning: tillåt flera typer per paragraf eller dela upp i underparagrafer i modulen.
- **Lantmäteriets normalstadgor som mall** — för samfällighet finns redan etablerad mall som kan levereras förtaggad. Förslag: leverera defaultkonfiguration så att en samfällighet som följer normalstadgan får 80 % av tagningsarbetet gjort vid onboarding.
- **Hybrid-möte och teknisk svaghet** — om internet faller under mötet, hur hanteras det formellt? Förslag: stadge-rekommendation om reservplan; systemet varnar inte men loggar förlorad anslutning som metadata på `MÖTE_ÖPPNAT`.
- **Mötesordförande som ej är ordförande** — frekvent vid jäv-läge eller större förenings-konflikter. Mötet väljer någon annan; systemet stödjer det redan via mötesroller, men dagordningen behöver kunna förmedla *varför* (kort motivering att läsa upp).
