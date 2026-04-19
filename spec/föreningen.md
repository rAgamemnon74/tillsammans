# Föreningen — årshjul, mötesplanering och minsta aktivitet

> Del av [Tillsammans-specifikation](../tillsammans.md).

Svenska ekonomiska föreningar — och i praktiken även ideala föreningar — följer räkenskapsåret slaviskt. Det är inte en valfri rytm; det är en konsekvens av bokföringslagen, LEF/LFS, Bolagsverket och Skatteverket som alla ställer tidskrav mätta från räkenskapsårets slut. Tillsammans ser därför årshjulet som föreningens grundtakt — allt stöd kring mötesplanering, påminnelser och minsta aktivitet organiseras runt det.

**Relaterade dokument:**
- [granskningslogg.md](granskningslogg.md) — epok-modellen (räkenskapsår = epok)
- [core-concepts.md](core-concepts.md) — kallelsemodell, bordläggning
- [case-types.md](case-types.md) — motionens tidsfönster
- [roles/board-roles.md](roles/board-roles.md), [roles/oversight-roles.md](roles/oversight-roles.md), [roles/nominating-committee.md](roles/nominating-committee.md) — vilka roller är aktiva i vilka faser
- [legal-context.md](legal-context.md) — lagrum och tidsregler

## Tre nivåer av aktiv förening

Begreppet "aktiv" har flera betydelser. Tillsammans skiljer:

1. **Formellt aktiv** — juridiska existensen består (org-nummer registrerat, ingen likvidation). Lägsta nivån *"föreningen finns"*.
2. **Governance-aktiv** — ansvarsfrågan hanteras varje räkenskapsår, styrelse och revisor är på plats, stadgan är aktuell. Detta är vad Tillsammans kan observera och validera.
3. **Verksamhetsaktiv** — faktiska beslut fattas, pengar rör sig, ärenden hanteras. Ligger utanför granskningsloggens scope.

Tillsammans fokuserar på nivå 2. Minimum nedan är alltså "governance-aktiv-miniminivån". En förening som producerar färre händelser uppfyller inte lagens formkrav; en förening som producerar fler är välskött men inte *mer* aktiv i systemets mening.

## Räkenskapsåret som oavvikbar rytm

Varför kan en förening inte bara välja sitt eget tempo? Fyra drivkrafter:

| Lagkälla | Tvingande tidskrav |
|---|---|
| **Bokföringslagen** | Årsbokslut ska vara klart inom rimlig tid efter räkenskapsårets slut. |
| **LEF 6:14** | Ordinarie årsstämma ska hållas **inom 6 månader** från räkenskapsårets slut. |
| **LFS** | Samma 6-månaders-krav för samfälligheter. |
| **Bolagsverket** | Årsredovisning ska vara inkommen **inom 7 månader** efter räkenskapsårets slut för bokföringsskyldig LEF. |
| **Skatteverket** | Inkomstdeklaration enligt separat tidsschema. |

Varje tidskrav räknas *bakåt* från räkenskapsårets slut. Räkenskapsårets startdatum (kalender, brutet läsår, eller annat stadgebestämt fönster) är det enda föreningen väljer — när det är valt är resten låst.

Mötesplaneringen följer som en konsekvens: stämman måste ligga senast månad 6, vilket betyder att revisionen måste vara klar senast månad 4–5, vilket betyder att bokslutet måste vara klart senast månad 2–3. Allt tidsrelaterat faller ut av den första valda datumgränsen.

## Årshjulet — integrerat flöde

Det här är *det typiska* årshjulet som Tillsammans föreslår i onboarding. Stadgan kan förskjuta datum inom lagens ramar, men ordningen är oföränderlig: bokslut → revision → kallelse → stämma → efterarbete.

### Månad 0 — Räkenskapsårets start

- `EPOK_ÖPPNAD` skrivs automatiskt i granskningsloggen.
- Eventuell styrelse-konstituering om ny styrelse valts vid föregående stämma (val av firmatecknare, fördelning av ansvarsområden).
- Budget för nya räkenskapsåret finns (fastställd av föregående stämma).

### Månad 1–2 — Bokslutsarbete

- **Kassören avslutar böckerna**, upprättar bokslut och balansräkning.
- Löpande styrelsemöten fortsätter; inga formkrav men normalt ett möte för att kvittera bokslutet.
- Uppföljning av föregående räkenskapsårs eventuella utestående ärenden.

### Månad 2–3 — Revision

- Kassören överlämnar bokslut och underlag till revisorn.
- **Revisorn granskar** räkenskaper och styrelsens förvaltning. `REVISORSFRÅGA_STÄLLD` + `REVISORSSVAR_AVLÄMNAT` är aktuella händelsetyper om revisorn har kompletterande frågor.
- Revisorn färdigställer revisionsberättelsen med till- eller avstyrkan av ansvarsfrihet.
- `REVISIONSBERÄTTELSE_AVLÄMNAD` skrivs i loggen.

### Månad 3–4 — Kallelse

- **Styrelsemöte** (ofta kallat *"budgetmöte"* eller *"årsmöte-förberedande möte"*): fastställer dagordning för årsstämman, tar ställning till inkomna motioner, förbereder styrelsens yttranden.
- Motionsfönstret stängs (typiskt 2 veckor–1 månad före stämma enligt stadga).
- **Valberedningen** ([nominating-committee.md](roles/nominating-committee.md)) färdigställer sitt samlade förslag.
- `KALLELSE_UTSKICKAD` inom stadgans tidsfönster (default min 2 v, max 4–6 v — se [editions/samfallighet.md](editions/samfallighet.md), [editions/foraldraforening.md](editions/foraldraforening.md)).

### Månad 4–5 — Beredning

- Styrelsens yttranden över motioner publiceras ([case-types.md](case-types.md)).
- Revisionsberättelsen distribueras med kallelsematerialet.
- Publik läsvy öppnas på motioner och valberedningens förslag (L6 publicerad tip-hash ingår).
- Medlemmar läser igenom och förbereder sig.

### Månad 5–6 — Ordinarie årsstämma

Stämmans formella genomförande är **lagens slutpunkt** för räkenskapsåret. Se [core-concepts.md](core-concepts.md), [meeting-roles.md](roles/meeting-roles.md) och granskningsloggens *"Praktiskt flöde"*.

Minsta uppsättning beslut som måste fattas:

1. Val av mötesordförande, mötessekreterare, protokolljusterare, rösträknare.
2. Godkännande av kallelsens formalia.
3. Fastställande av årsredovisning.
4. Ansvarsfrihet (beviljad / avslagen / uppskjuten).
5. Val av styrelse (även om samma personer väljs om).
6. Val av revisor (även om samma person väljs om).
7. Val av valberedning.
8. Beslut om eventuella motioner.

Mötets avslutning sätter tip-hash som blir del av protokollet (L6). `EPOK_FÖRSEGLAD` skrivs med sigill-variant beroende på ansvarsfrihetsutfallet.

### Månad 6–7 — Efterarbete

- Protokollet cirkuleras till justerare; `PROTOKOLL_JUSTERAT` när signerat (typiskt inom 2–4 veckor).
- Fullständig protokollstext med tip-hash distribueras till medlemmar och arkiveras i dokumentregistret.
- För LEF: **årsredovisning inlämnas till Bolagsverket** senast månad 7. Registreras som `DOKUMENT_BIFOGAT` i nästa epoks granskningslogg.
- Inkomstdeklaration hanteras enligt Skatteverkets tidsschema.

### Löpande under året

Mellan de fasta milstolparna driver styrelsen förening i vardagen. Typisk frekvens:

- **Styrelsemöten:** 4–8 per år är vanligt, minimum 1–2. Registreras som `MÖTE_ÖPPNAT` + `MÖTE_AVSLUTAT` med `STYRELSEBESLUT` för varje ärende.
- **Ärenden:** motioner, medlemsansökningar, utläggsgodkännanden hanteras löpande enligt [case-types.md](case-types.md).
- **Extra stämmor:** vid behov (stadgeändring mid-år, uteslutningsärende, styrelsens avsägande). Egen kallelsecykel (typiskt 1 vecka enligt LFS).
- **Kvartalsvis ekonomi-uppföljning:** inte lagkrav men starkt rekommenderat. Kassören rapporterar till styrelsen så att bokslutet inte blir en överraskning i månad 1.

## Minsta aktivitet per räkenskapsår

För att epoken ska kunna förseglas *korrekt* (inte med `STÄMMA_UTEBLEV`- eller `ANSVARSFRIHET_UPPSKJUTEN`-sigill) krävs följande minimum:

### Händelser i granskningsloggen

| Händelse | Kommentar |
|---|---|
| `EPOK_ÖPPNAD` | Automatiskt vid räkenskapsårets start. |
| `KALLELSE_UTSKICKAD` | För ordinarie årsstämma inom stadgans tidsfönster. |
| `MÖTE_ÖPPNAT` + `MÖTE_AVSLUTAT` | Stämman måste genomföras. |
| `STÄMMOBESLUT` — val av mötesroller | Mötesordförande, sekreterare, justerare (LEF 6:19, LFS 49§). |
| `STÄMMOBESLUT` — fastställande av årsredovisning | Räkenskaperna godkända. |
| `STÄMMOBESLUT` — ansvarsfrihet | Beviljad, avslagen eller uppskjuten — men stämman måste ta ställning. |
| `STÄMMOBESLUT` — val av styrelse | Omval är val. |
| `STÄMMOBESLUT` — val av revisor | Omval är val. |
| `REVISIONSBERÄTTELSE_AVLÄMNAD` | Förutsättning för ansvarsfrihetsbeslutet. |
| `PROTOKOLL_UPPRÄTTAT` + `PROTOKOLL_JUSTERAT` | Stämmoprotokollet måste signeras. |
| `EPOK_FÖRSEGLAD` | Epoken stängs med sigill-variant. |

### Aktiva rolltilldelningar

- **Ordförande** — utan ordförande saknas signeringskapacitet och mötesledning.
- **Kassör** — utan kassör saknas ekonomisk kontroll.
- **Minst en revisor** — utan revisor kan ansvarsfrihet inte baseras på granskning. Revisorssuppleant är *inte* ett substitut — ordinarie måste finnas.
- **Stadgeenligt minimum-antal ledamöter** — LFS kräver minst 3, LEF minst 3 (6:1), föräldraföreningars stadgar varierar.

### Aktuell stadga

En `STADGEVERSION_ANTAGEN` måste vara i kraft. Vid föreningens första `EPOK_ÖPPNAD` måste stadgan ha antagits först.

## Edition-specifika tillägg till minimum

- **Samfällighet:** `DEBITERINGSLÄNGD_FASTSTÄLLD` per räkenskapsår är **lagkrav enligt LFS**. Ingår i minsta uppsättningen.
- **LEF (inkl. kooperativ-förskola):** årsredovisningen ska inlämnas till Bolagsverket för bokföringsskyldig förening. Registreras som `DOKUMENT_BIFOGAT`.
- **Ideal föräldraförening (ej LEF):** minimalt utöver grundkraven. Bokföringsskyldighet gäller bara om särskilt rekvisit uppfylls (anställda, över 3 Mkr tillgångar, e.d.).

## Räkenskapsårs-varianter

Tre empiriska mönster:

| Variant | Årscykel | Förekomst |
|---|---|---|
| **Kalenderår** (1/1–31/12) | Bokslut jan–feb, revision feb–mar, stämma mar–apr | Dominerande hos samfälligheter, ekonomiska föreningar, LEF, kommunal-associerade skol-FF |
| **Brutet läsårsföljande** (1/7–30/6) | Bokslut jul–aug, revision aug–sep, stämma okt–nov | Friskole- och Waldorf-föräldraföreningar, föräldrakooperativ-förskolor |
| **Fritidshus-säsong** (1/5–30/4) | Bokslut maj–jun, revision jun–jul, stämma jul–aug | Sommarstuge-samfälligheter — stämma i augusti då sommarboende är på plats |

Edition-onboarding föreslår variant baserat på föreningstyp; stadgan bestämmer slutgiltigt.

## Aktivitetsstatus — härledning från loggen

Tillsammans kan härleda föreningens aktivitetsstatus direkt ur granskningsloggen:

| Status | Kriterium |
|---|---|
| **Aktiv** | Senaste epoken förseglad med `ANSVARSFRIHET_BEVILJAD` eller `ANSVARSFRIHET_AVSLAGEN`; inga kritiska vakanser (ordförande, kassör, revisor); aktuell stadga finns. |
| **Pending** | Löpande epok; tidigare epok korrekt försegladad. Normalläget mellan stämmor. |
| **Vilande** | Senaste epoken förseglad `ANSVARSFRIHET_UPPSKJUTEN` — stämman har inte tagit ställning. Tekniskt aktiv men med påminnelse-flagga. |
| **Stadgebrott** | Senaste epoken förseglad `STÄMMA_UTEBLEV` eller kritisk vakans > stadgad tidsgräns. |
| **Nystartad** | Första epoken pågår — mjukare flaggor tillåts under första räkenskapsåret. |
| **Under avveckling** | Egen flagga när upplösnings-process påbörjats. Utanför normal aktivitetslogik. |

Statusen visas:

- I styrelsevyn med åtgärdsförslag (*"Ordförande-vakans i 72 dagar — överväg extra stämma"*).
- I medlemmens publika vy som status-etikett (*"Aktiv 2026"* / *"Vilande"* / *"Stadgebrott föreligger"*).
- I revisorns vy med prioriterade granskningspunkter.
- I den publika tip-hash-publiceringen (`ANSVARSFRIHET_UPPSKJUTEN` är synligt för alla).

## Systemets mötesplanerings-stöd

### Årshjul-planerare

Vid onboarding konfigurerar föreningen räkenskapsår + stadgans kallelsetider. Systemet beräknar och visualiserar årshjulet:

- Datum då revisor senast måste få bokslutet.
- Senaste datum för styrelsemöte som fastställer kallelsen.
- Tidigaste och senaste datum för kallelseutskick.
- Stämmoperiodens tillåtna intervall.

Styrelsen kan justera specifika datum inom lagens ramar. Årshjulet lagras som konfiguration — inte som händelser — men händelser valideras mot det.

### Påminnelser

Systemet genererar påminnelser baserat på årshjulet:

| Tidspunkt | Påminnelse |
|---|---|
| 90 dagar före stämma | Kallelsefönster öppnas; styrelsemöte om dagordning kan planeras |
| 60 dagar före | Revisor: revisionsarbetet bör påbörjas |
| 45 dagar före | Kassör: bokslut bör vara klart |
| 30 dagar före | Styrelsemöte för att fastställa kallelse |
| Stadgans min-gräns före | Kallelse måste skickas nu |
| 7 dagar efter stämma | Protokollsjustering bör påbörjas |

Påminnelserna skickas till berörda roller i deras arbetsyta, inte via extern e-post (den kanalen hör till externa kommunikationsverktyg). Revisorn har egna påminnelser i sin arbetsyta.

### Vakans-flaggor

- **Kritisk vakans** (ordförande, kassör, revisor) > 60 dagar → röd flagga i styrelsevyn.
- **Mindre vakans** (sekreterare, suppleant, valberedningsledamot) → gul flagga.
- Flaggan visas tills rollen besätts eller stadgeändring tar bort den.

## Avvikelser — systemet flaggar, blockerar aldrig

Om föreningen underproducerar i förhållande till minimum:

- Händelser kan fortfarande skrivas (systemet är inte självhämmande).
- Epoken får ett sigill som reflekterar verkligheten (`STÄMMA_UTEBLEV`, `ANSVARSFRIHET_UPPSKJUTEN`).
- Statusen förändras i publika och interna vyer.
- Revisorn får prioriterade varningar.
- Nästa års planering startar med kvardröjande flaggor från förra årets brister.

Systemet stämplar aldrig enskilda personer som ansvariga. Signaler tolkas enligt [analysis-rules.md#diagnostiska-signaler](analysis-rules.md) — en av tre rot-orsaker (engagemang, system-miss, CX/UX).

## Vad som uttryckligen *inte* är minimum

För att undvika rolförvirring:

- **Flera styrelsemöten per år.** Ett, eller inget mellan stämmorna räcker juridiskt om stadgan tillåter.
- **Motioner.** Ingen måste lämnas in.
- **Utläggs-, faktura- eller betalningsflöden.** Vissa föreningar har knappt ekonomi.
- **Anslagstavla-aktivitet.**
- **Utskott eller kommittéer.**
- **Publikations-disciplin utöver lagkravet** (men verifikationsrapporterna visar att bristande publicering är en tydlig "passiv erosion"-signal — se [threats.md](threats.md)).

En förening som bara gör minimum är formellt korrekt — inte nödvändigtvis väl driven. Skillnaden är observerbar (få händelser per år) men inte disciplinär från systemets håll.

## Öppna frågor

- **Styrelsens konstituering** — val av kassör, sekreterare: stämmans eller styrelsens beslut? Olika stadgar löser det olika. Förslag: systemet tillåter bägge; onboarding-wizarden sätter default per edition.
- **Tidsfönster för kritisk vakans** innan röd flagga utlöses? Förslag: 60 dagar för ordförande/kassör, 90 dagar för revisor. Stadgan kan justera.
- **Verksamhetsberättelse som krav för epok-sigill?** Formellt krav finns för LEF men inte alltid för ideal förening. Förslag: krav för LEF-edition, rekommendation för övriga.
- **Nystartad-tröskel** — de första 12 månaderna eller första epoken till ansvarsfrihet? Förslag: första epoken fram till och med dess `EPOK_FÖRSEGLAD`.
- **Påminnelse-kanal.** I MVP bara i systemets arbetsytor? Eller även via e-post som opt-in? Förslag: MVP bara i arbetsytor, e-post är senare utvidgning.
- **Årshjul-justering mid-år.** Om stadgan ändras och kallelsetiden förändras under pågående år — hur uppdateras årshjulet? Förslag: aktiveras vid nästa epok; nuvarande epok behåller sitt fastställda schema.
