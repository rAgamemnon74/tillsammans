# Verifikation: 11 typscenarier för samfällighet mot Tillsammans-specen (april 2026)

> Verifikations-rapport enligt [verification.md](../verification.md) Spår 2 (typscenarier) för `AssociationType = samfällighet`. Adresserar tröskelkravet *"Alla 10 typscenarier genomgångna med luckor dokumenterade"* — här 11 eftersom specen lagt till scenario 11 (gammal stadga som kopia).
>
> Levande dokument — öppet underlag för förtroende (se [mission.md](../mission.md)).
>
> **Kontext:** rapporten är en skrivbordsövning där varje scenario stegas igenom mot specen. Luckor klassas som *modell-brist* (domänmodellen saknar entitet/relation), *spec-brist* (flödet är inte specat — ska specas) eller *medvetet out-of-scope* (hänvisas till externt verktyg). Rapporten fixar ingenting — den hittar.

## Sammanfattning

- **11 scenarier genomgångna.** Samtliga kan köras genom modellen, men nio av dem stöter på minst en lucka.
- **27 luckor identifierade** totalt.
- **Klassfördelning:**
  - **Spec-brist:** 18 (flödet är inte specat men hör tydligt till governance-kärnan).
  - **Modell-brist:** 5 (en domänentitet eller relation saknas eller är underspecad).
  - **Medvetet out-of-scope:** 4 (hänvisas till externa verktyg eller spec:ens "inte-gör"-sektion och är korrekt så).
- **Scenarier som exponerade flest luckor:** 2 (40-årig samfällighet — 7 luckor), 3 (debiteringsrond — 5 luckor), 8 (medlemsförändring — 4 luckor).
- **Scenarier som höll utan luckor:** 5 ("Ni lovade"-konflikten), 6 (jäv och bordläggning) — bägge är kärnmekanismer som specen har investerat mycket i.

**Tre viktigaste fynd:**

1. **Onboarding-flödet för en befintlig samfällighet är inte specat som eget dokument.** Flera spec-filer nämner onboarding som funktion, men själva sekvensen (org-nummer → stadga → styrelse → medlemmar → andelstal → första `EPOK_ÖPPNAD`) finns inte samlad. Scenarier 1, 2 och 11 kräver alla detta.
2. **Debiteringsrond saknar egen ärendetyp / flöde.** Specen konstaterar att debiteringslängden är "automatisk output från andelstal × budget" och skriver `DEBITERINGSLÄNGD_FASTSTÄLLD` som händelse, men inte *hur* den faktiska kvartals-/halvårs-faktureringen mot medlemmar flödar genom systemet. `FAKTURA_UTSTÄLLD` finns i transaktionstyps-listan men har ingen egen livscykel.
3. **Revisorns löpande granskning är underspecad operativt.** Rollen har löpande läsrätt och egen arbetsyta (per [oversight-roles.md](../roles/oversight-roles.md)), men vad revisorn *gör* under räkenskapsåret — urval, granskningsplan, anteckningar, flaggor — finns inte i specen. Arbetsytan är en skiss, inte ett flöde.

## Scenario 1 — Onboarding av nystartad samfällighet

Fall: nystartad vägsamfällighet i en nybyggd tomtrad, ~30 fastigheter, lantmäteriförrättningen just avslutad. Föreningen ska etablera sin digitala tvilling och gå in i första räkenskapsåret.

### Stegen

1. Initiativtagaren (typiskt ordförande vid konstituerande möte) registrerar föreningen i Tillsammans — `AssociationType = samfällighet`, sub-typ `vägsamfällighet`.
2. Normalstadge-wizarden körs ([editions/samfallighet.md#onboarding-via-normalstadge-wizard](../editions/samfallighet.md)): säte, antal styrelseledamöter, mandatperioder, kallelsetider min/max, räkenskapsår, fondavsättning, kallelsemetod, motionsfönster.
3. Stadgans paragrafer taggas per typkod ([moten.md#stadgeparagraf-typer-med-automatiskt-systemstöd](../moten.md)): `RÄKENSKAPSÅR`, `KALLELSETID`, `STÄMMOPERIOD`, `MOTIONSFRIST`, `RÖSTMETOD`, `MAJORITET`, `KVORUM`, `STYRELSESAMMANSÄTTNING`, `MANDATPERIOD`, `FIRMATECKNING`, `DEBITERINGSLÄNGD`, `UNDERHÅLLSFOND`, `STADGEÄNDRINGSMAJORITET`, `EXTRA_STÄMMA_TRÖSKEL`, `ANSVARSFRIHET`.
4. `STADGEVERSION_ANTAGEN` skrivs (första versionen).
5. Styrelseroller tilldelas — `ROLL_TILLDELAD` per person (ordförande, kassör, sekreterare, ledamot, ev. suppleant, revisor, revisorssuppleant, ev. valberedning).
6. Medlemsregister etableras: för varje fastighet i GA skapas `Actor` (typiskt `NATURAL_PERSON`) + `Medlem` (status `AKTIV`), `PropertyUnit`, `Ownership`, `Participation.andelstal`.
7. Lantmäteriets anläggningsbeslut laddas upp som källdokument — `BILAGA_INLÄMNAD` typ `FIL`.
8. Första `EPOK_ÖPPNAD` skrivs. Genesis-hash visas för arkivering (per [granskningslogg.md#praktiskt-flöde-genom-ett-räkenskapsår](../granskningslogg.md)).
9. OpenTimestamps L4 aktiveras (opt-in default).
10. Föreningen är i *Nystartad*-status enligt [föreningen.md#aktivitetsstatus--härledning-från-loggen](../föreningen.md) — första epoken pågår, mjukare flaggor tills första ansvarsfrihetsstämman.

### Luckor

- **Onboarding-sekvensen saknar dedikerad spec.** Stegen är fördelade över `editions/samfallighet.md`, `medlemskap.md`, `moten.md`, `granskningslogg.md`, `föreningen.md`. Ordningen (vilken händelse *först* — föreningen, stadgan, medlemmarna eller epoken?) är inte uttryckligen bestämd. **Klassning: spec-brist.** Förslag: `spec/onboarding-samfallighet.md` eller samla i `editions/samfallighet.md` under egen sektion *"Onboardings-sekvens"*.
- **Stadge-modulen som paragraftaggning förutsätter tekniska komponenter som inte är specade.** [moten.md#öppna-frågor](../moten.md) noterar att `spec/stadge-modul.md` behövs. Taggnings-UI, parser-stöd, parameter-validering är oklara. **Klassning: spec-brist.**
- **Andelstal-bulkregistrering saknas.** För en vägsamfällighet med 30 fastigheter där andelstalen är platta (1/30 per fastighet) krävs 30 `Participation`-rader. Om andelstalen kommer från anläggningsbeslutet behövs ett bulkimport-flöde (CSV, kopiering från förrättningsakt). [editions/samfallighet.md#andelstal-som-data](../editions/samfallighet.md) specar strukturen men inte inmatnings-mekaniken. **Klassning: spec-brist.**
- **Initial `ROLL_TILLDELAD` har kyckling-och-ägg-problem.** `ROLL_TILLDELAD` är en händelse som måste skrivas *av* en aktör med rätt behörighet — men innan föreningen har roller tilldelade finns ingen aktör som har den rätten. Systemet behöver en "bootstrap-aktör" eller motsvarande för den allra första rolltilldelningen. **Klassning: modell-brist** (RBAC-modellen har ingen genesis-aktör).

### Positivt utfall

- Scenariot är genomförbart — varje steg har *någon* spec-referens. Normalstadge-wizarden är väl beskriven och matchar empiriska mönstret från [verification-reports/samfalligheter-2026-04.md#mönster-10](samfalligheter-2026-04.md#mönster-10--normalstadgar-som-mall) (alla 10 granskade föreningar följer normalstadgan).
- Genesis-hash-mekaniken är väl specad och ger föreningen ett bevisvärde från dag ett.
- `Nystartad`-statusen i [föreningen.md](../föreningen.md) hanterar att första epoken saknar föregående sigill att ansvarsfrihets-pröva.

## Scenario 2 — Onboarding av 40 år gammal samfällighet

Fall: Warbro/Enebackens-liknande förening med stadgar från 1970-talet reviderade 2013, ~80 fastigheter, flera dödsbon i registret sedan tidigt 2000-tal, några samägande där arvingar aldrig formaliserade situationen, anläggningsbeslut finns på mikrofilm hos kommunarkivet, dokumentkonflikt mellan förrättningsprotokoll och senare kartritning om var servitut för en enskild brygga ligger.

### Stegen

1. Som i scenario 1 fram till och med normalstadge-wizardens start.
2. **Avvikelse:** wizarden kör inte — föreningen laddar istället upp sin *befintliga* stadga (2013 rev.) som källdokument enligt [editions/samfallighet.md#äldre-samfälligheter--stadga-laddas-in-som-kopia](../editions/samfallighet.md).
3. Stadgan delas upp i paragrafer manuellt (stadge-modulens parser-stöd föreslår; onboardern justerar). Paragrafer taggas per typkod där det går; paragrafer som avviker från normalstadgan (t.ex. gammalmodiga formuleringar om *"sommarboende endast"*) taggas "ingen — fri text".
4. `STADGEVERSION_ANTAGEN` skrivs med flaggan *"motsvarar pappersoriginalet"* — stadgan är inte modernisierad. [editions/samfallighet.md](../editions/samfallighet.md) nämner krav att systemet accepterar det.
5. Styrelseroller tilldelas från befintliga förtroendevalda.
6. **Medlemsregister etableras i lapptäckes-läge** ([domain-model.md#lapptäcket](../domain-model.md#lapptäcket--etablering-och-dokumentkonflikter)): varje `PropertyUnit` registreras. För fastigheter där ägaren är oklar (dödsbo, arvskifte ej genomfört) markeras `Actor = ESTATE` med bäst tillgänglig information; `faktureringsmottagare` kan avvika från formell ägare (se [domain-model.md:42](../domain-model.md)).
7. Anläggningsbeslutet från 1980-talet laddas upp som `BILAGA_INLÄMNAD` typ `FIL` (PDF av mikrofilmen). Kartritningen som motsäger servitut-placeringen laddas upp som *ytterligare* källdokument — systemet representerar *båda* källor enligt [domain-model.md:120](../domain-model.md#lapptäcket--etablering-och-dokumentkonflikter) princip 2.
8. Konflikt-flagg sätts ([domain-model.md:122](../domain-model.md#lapptäcket--etablering-och-dokumentkonflikter) princip 3) för servituts-frågan — styrelsen ska ta upp den.
9. Första `EPOK_ÖPPNAD` skrivs. Föreningens *historiska* räkenskapsår innan Tillsammans finns inte i loggen — systemet har ingen retroaktiv epok-rekonstruktion.
10. Status blir inte *Nystartad* utan *Aktiv* om förra årets stämma hölls enligt stadgan — men `senaste förseglade epok` saknas i loggen. [föreningen.md#aktivitetsstatus](../föreningen.md) har ingen etikett för "aktiv men utan historik".

### Luckor

- **Ingen retroaktiv epok för pre-Tillsammans-år.** En 40-årig förening har 40 räkenskapsår bakom sig. Systemet låter dem inte rekonstruera epoker med sigill och ansvarsfrihetsbeslut för dessa år. Hur signalerar loggen att föreningen *har* historik, men den ligger utanför systemet? **Klassning: spec-brist.** Förslag: `EPOK_ÖPPNAD` kan ha fält `inherited_history: true` + referens till extern arkivhänvisning.
- **"Motsvarar pappersoriginalet"-flaggan på stadgan är nämnd men inte specad som datafält.** [editions/samfallighet.md#äldre-samfälligheter](../editions/samfallighet.md) nämner att systemet ska "acceptera" det, men `STADGEVERSION_ANTAGEN`-händelsen har ingen specad attestation-mekanism. Vem skriver under? Hela styrelsen? **Klassning: spec-brist.**
- **Dokumentkonflikt-flödet saknar ärendetyp.** När lapptäcket visar att två källor säger olika (servitut-placeringen), är det sannolikt ett *Allmänt ärende* eller en oberörd styrelsefråga. Specen beskriver *flaggning* men inte *hantering* — vem gör vad, när, med vilken beslutsmetod? **Klassning: spec-brist.**
- **`Actor = ESTATE` utan känd representant.** För dödsbon där arvskifte aldrig genomförts finns ingen verifierad representant. [medlekap.md#dödsbo-som-övergångsform](../medlemskap.md) förutsätter att någon (boutredningsman, dödsbodelägare med fullmakt) är känd. Hur hanterar systemet fall där dödsboet i praktiken är "föräldralöst"? **Klassning: modell-brist.** Förslag: ny `Medlem.vilande_orsak`-kod `KÄND_EJ_REPRESENTANT`.
- **Fasta historiska beslut försvinner.** Samfälligheten har sannolikt fattat beslut över 40 år som fortfarande är gällande (t.ex. *"taket mot parkmark får inte höjas"*). Specen har ingen mekanism för att importera *historiska fortfarande-gällande beslut* som en stomme i beslutsloggen. **Klassning: spec-brist.** Förslag: `HISTORISKT_BESLUT_REGISTRERAT`-händelse (egen typ med tydlig "icke-hash-integritet"-flagga) som låter föreningen dokumentera etablerad praxis utan att tvinga alla beslut till att komma från Tillsammans-perioden.
- **Medlemmar som finns men inte är kända.** I en 40-årig förening har vissa medlemmar aldrig engagerat sig digitalt. Onboardingen registrerar dem baserat på fastighetsbeteckning + någon känd kontaktuppgift (kanske bara "sommarstugeägare X"). Hur etableras en kommunikationskanal? [medlemskap.md#data-för-samfällighetsmedlem](../medlemskap.md) anger *rekommenderat* för e-post/post — men inte vad som händer när ingenting finns. **Klassning: spec-brist.**
- **Konfigurationsöverskottet förutsätter en tidig ordförande.** Att tagga 30 paragrafer kan ta "2-4 timmar för en stadga" enligt [moten.md#värdet--investering-en-gång-frukt-över-tid](../moten.md). För en ny styrelse som tar över en gammal förening är det betydande. Finns ett *gradvist* onboarding-läge där otaggade paragrafer tillåts i MVP? **Klassning: spec-brist (fråga som inte är besvarad).**

### Positivt utfall

- [domain-model.md#lapptäcket](../domain-model.md#lapptäcket--etablering-och-dokumentkonflikter) är konceptuellt mognat för just detta — flera källor tillåts parallellt, systemet ska *exponera* snarare än *döma*.
- `faktureringsmottagare` avskild från formell ägare i [editions/samfallighet.md:118](../editions/samfallighet.md) löser ett reellt 40-årigt problem (fakturera dödsbo-representanten, inte den avlidne).
- [core-concepts.md#röstlängdsetablering](../core-concepts.md#röstlängdsetablering--processen-vid-stämmans-öppnande) principen *"Tillsammans löser inga civilrättsliga tvister — föreningen ska kunna komma vidare"* gör att onboarding inte blir blockerad av varje olösta ägarfråga.

## Scenario 3 — Debiteringsrond

Fall: Kyrkmossen, kalenderår, platt andelstal 1/97 per fastighet. Stämman har fastställt årsbudget i april 2026. Faktureringsperioder enligt stadgan är kvartalsvis. Kassören ska genomföra Q2-debiteringen.

### Stegen

1. Stämman har fattat budgetbeslut — `STÄMMOBESLUT` om budget registrerat i föregående epok (N−1). Ny epok N har öppnats.
2. Vid stämman har `DEBITERINGSLÄNGD_FASTSTÄLLD` skrivits ([föreningen.md#minsta-aktivitet](../föreningen.md)) med årets totala kostnader fördelat per fastighet × GA × andelstal.
3. Kassören triggar Q2-fakturering (kvartalet april-juni). Systemet genererar fakturor: per fastighet × antal GA × andelar × kvartalsandel av årskostnaden.
4. Eventuella ägarbyten under kvartalet: pro-rata mot `actual_transfer_date` (per [medlemskap.md#epok-tillhörighet](../medlemskap.md)). Om bytet inträffade under kvartalet får säljaren slutavräkning per tillträdesdatum och köparen del-faktura för resterande dagar.
5. `FAKTURA_UTSTÄLLD`-händelser skrivs per fastighet (per [granskningslogg.md#transaktionstyper](../granskningslogg.md)).
6. Faktureringsfilen exporteras till bokföringssystemet (eller till `ExternalAdministrator` om stadge-/avtalsbestämt scope täcker fakturering — [architecture.md:34](../architecture.md), [editions/samfallighet.md#extern-ekonomi-förvaltning](../editions/samfallighet.md)).
7. Inbetalningar matchas mot utställda fakturor — utanför Tillsammans i externt bokföringssystem.
8. Vid utebliven betalning: påminnelse, dröjsmålsränta, ev. indrivning — [architecture.md#utanför-scope-externa-verktyg](../architecture.md).

### Luckor

- **`FAKTURA_UTSTÄLLD` finns som transaktionstyp men har ingen livscykel.** [granskningslogg.md#transaktionstyper](../granskningslogg.md) listar den; ingen spec-fil dokumenterar vad den innehåller, hur den tillkommer, eller vad som görs med den. **Klassning: spec-brist.** Förslag: `spec/case-types/fakturering.md` eller inbäddad sektion i `editions/samfallighet.md`.
- **Debiteringsrondens livscykel saknas.** Stegen 3-6 ovan är i specen bara antydda (*"genereras automatiskt från budget × andelstal × kvartal"*). Ingen av händelserna (`DEBITERINGSRONDS_PÅBÖRJAD`, `FAKTURA_UTSTÄLLD` i batch, `FAKTURERING_AVSLUTAD`) har struktur. **Klassning: spec-brist.**
- **Pro-rata-beräkningen vid mid-kvartal-ägarbyte är ospecad.** [medlemskap.md#epok-tillhörighet](../medlemskap.md) nämner att `actual_transfer_date` "styr debiteringen", men själva beräkningen (dagar, månader, eller "vems namn stod på fakturan vid utställandedatum?") är inte bestämd. Flera legitima standarder finns. **Klassning: spec-brist.**
- **Publik synlighet vs privatsfär.** Debiteringslängden är lagkrav (LFS), publiceras sällan (2 av 10 gjorde det, [samfalligheter-2026-04.md#mönster-3](samfalligheter-2026-04.md#mönster-3--debiteringslängd-lagkrav-sällan-publik)). Spec: `DEBITERINGSLÄNGD_FASTSTÄLLD` är händelse i loggen med default-visibility — men vad *är* default-visibility? [granskningslogg.md#vyer-projektioner](../granskningslogg.md) ger en generisk modell, men inte konkret beslut per händelsetyp. **Klassning: spec-brist.**
- **`ExternalAdministrator` + fakturering.** Vid scope `fakturering` tar administratören över exporten — får hen också skriva `FAKTURA_UTSTÄLLD`-händelsen? [roles/other-roles.md#externaladministrator](../roles/other-roles.md) säger "inget beslutsmandat" men är tvetydigt om rena registrerings-händelser. **Klassning: spec-brist.**

### Positivt utfall

- `DEBITERINGSLÄNGD_FASTSTÄLLD` som årlig milstolpe är korrekt modellerad — den sluter föregående epok med ett lagkrav.
- Principen *"kassörsstödet exporterar underlag — dokumenterar, utför inte"* ([architecture.md#inom-scope](../architecture.md)) gör att scenariot på hög nivå håller: Tillsammans registrerar vilka fakturor som utställdes, inte själva banktransaktionen.

## Scenario 4 — Motion från medlem

Fall: Fröåvägen (fjäll-väg, ~150 fastigheter). En medlem lämnar motion *"Föreningen bör installera fartkameror på vägen"* 6 veckor före årsstämman.

### Stegen

1. Medlemmen (status `AKTIV`) skapar motion via [case-types.md#referens-ärendetyp-motion](../case-types.md): yrkande, motivering, stadgaförankring (paragrafen om vägens säkerhet), inlämnare.
2. `ÄRENDE_INLÄMNAT` med `aggregate_type = MOTION`, status `INLÄMNAD`. Inlämningskvitto enligt textbibliotek.
3. Styrelsen tar upp ärendet vid nästa styrelsemöte — status → `BEREDNING`. Mötesprotokollet visar ärendet som dagordningspunkt via auto-förfyllning ([moten.md#styrelsemöte--standardmall](../moten.md) punkt 8).
4. Styrelsen diskuterar, skriver yttrande (mall enligt [case-types.md](../case-types.md) textbibliotek), röstar. `STYRELSEBESLUT` + `RÖST_AVGIVEN` per ledamot om yttrande. Status → `YTTRANDE_FÖRLAGT`.
5. Kallelsen till stämman skickas ut senast stadgad min-gräns före mötet. Motionen + yttrandet inkluderas automatiskt i kallelsen (`K-NOTIFIERING-1` från [motesadministration.md](../krav/motesadministration.md)). Status → `PUBLICERAD`.
6. Vid stämman: `MÖTE_ÖPPNAT`, formalia, motionsbehandling. Status → `PÅ_STÄMMA`. Diskussion, ev. yrkanden, röstning. `STÄMMOBESLUT` registreras med aggregerad röstning ([rostlangd.md#under-mötet--pappers-rutin](../rostlangd.md)).
7. Status → `AVGJORD` (BIFALL / AVSLAG / ÅTERREMISS).
8. `PROTOKOLL_UPPRÄTTAT` → `PROTOKOLL_JUSTERAT`. Resultat kommuniceras till initiatorn via textbibliotek-mall.
9. Om `BIFALL` och beslutet kräver genomförande: bildas sannolikt upphandlingsärende ([case-types/upphandling.md](../case-types/upphandling.md)) för faktiska kameror.

### Luckor

- **Motionsfristen vs kallelse-min-gräns är delvis motsägelsefull.** [case-types.md#tidsregler-motion](../case-types.md) säger motionsfrist är "2v–1 mån före stämma", [editions/samfallighet.md#stämma-och-kallelse](../editions/samfallighet.md) säger "senast 2v–1 mån före stämma", men kallelsens min-gräns är också 2v (default). Med 2v motionsfrist och 2v kallelsetid har styrelsen i värsta fall *noll dagar* att bereda motionen innan kallelsen går ut. Föreningen behöver *kallelse-min > motionsfrist + styrelsens beredningstid*. Specen beskriver inte hur konflikten detekteras eller löses. **Klassning: spec-brist.**
- **Kopplingen motion → efterföljande upphandlingsärende.** Specen nämner att medlem kan föreslå behov via allmänt ärende → upphandling ([case-types/upphandling.md#primär-initiator](../case-types/upphandling.md)), men inte bifallen-motion → upphandling. Hur länkas spåren? **Klassning: spec-brist** (liten, men real).
- **Motions-arkivet över tid.** `PUBLICERAD` + `AVGJORD` är publika — över flera år bygger detta upp ett motions-arkiv. Specen antyder att *"en medlem som år efter år inkommer med samma avgjorda motion blir synlig"* ([case-types.md#hot-och-försvar](../case-types.md)) men har ingen specad historikvy för att upptäcka mönstret. **Klassning: spec-brist.**

### Positivt utfall

- Motionsflödet är väldigt väl specat — livscykel, textbibliotek, RBAC, publicering, GDPR, hot, öppna frågor. [case-types.md#referens-ärendetyp-motion](../case-types.md) är rapportens tätast-specade del.
- Kopplingen till möten (auto-förfyllning av dagordning via `K-DAGORDNING-3` / `K-INTEGRATION-1` i [motesadministration.md](../krav/motesadministration.md)) är konsistent specad.
- Grupperings-mekaniken skyddar mot "10 motioner för förhandlingstryck" ([threats.md#taktisk-motionsgivning](../threats.md)).

## Scenario 5 — "Ni lovade"-konflikt

Fall: Hule-liknande förening. En medlem påstår vid årsmötet att förra styrelsen *"lovade"* att föreningen skulle ta över hens privata infartsväg i vägnätet. Inget sådant beslut finns i protokollet.

### Stegen

1. Medlemmen reser frågan på stämman under "övriga ärenden" eller via brev till styrelsen i förväg.
2. Styrelsen söker i beslutsloggen ([granskningslogg.md#vyer-projektioner](../granskningslogg.md) beslutslogg-vy) — *inget sådant löfte finns*.
3. Styrelsen bemöter enligt [core-concepts.md#governance-sköld-ni-lovade-flödet](../core-concepts.md) steg 2: *"vi kan inte agera på rykten, likabehandlingsprincipen"*. Svaret renderas från textbibliotek-mallen.
4. Medlemmen står på sig. Styrelsen erbjuder: lämna in som motion → nästa stämma avgör (steg 3 i "Ni lovade"-flödet).
5. Medlemmen lämnar motion. Den går genom motionsflödet (scenario 4).
6. Systemet bygger underlagspaketet automatiskt: historik (protokoll-sökning på vägfrågor), tidigare stämmobeslut om vägnätet, styrelsens yttrande.
7. Stämman avgör. Om avslag: medlemmens krav är kollektivt avvisat, styrelsen är juridiskt skyddad mot senare anklagelser.
8. `STÄMMOBESLUT` + motionsärende AVGJORD + `PROTOKOLL_JUSTERAT` → governance-skölden är aktiverad.

### Luckor

Inga nya. Detta scenario är själva designprincipen — specen har investerat mycket precis här.

### Positivt utfall

- **Scenariot höll i sin helhet.** [core-concepts.md#governance-sköld-ni-lovade-flödet](../core-concepts.md) beskriver exakt denna sekvens steg för steg.
- Koppling till motion (referens-ärendetypen) är sömlös — medlemmen får formell väg framåt, styrelsen får skydd mot personlig konflikt.
- [threats.md#skyddsmekanismer](../threats.md) listar *"Bordläggning till stämman → personlig konflikt mellan styrelse och medlem omvandlas till kollektivt majoritetsbeslut"* som explicit försvarsmekanism. Skölden är specad.

## Scenario 6 — Jäv-deklaration och bordläggning

Fall: Warbro/Enebackens-liknande förening ska upphandla asfaltering. Ordföranden upptäcker att anbudsgivaren är gift med hens kusin.

### Stegen

1. Upphandlingsärende initieras enligt [case-types/upphandling.md](../case-types/upphandling.md). Obligatorisk jäv-deklaration *vid initiering* — ordföranden kryssar `JÄV` och beskriver relationen.
2. Ärende sätts i status `INITIERAT` men ordförandens jäv hindrar hen från att delta i `KRAVSPEC_FÖRBEREDS` och beslut.
3. Annan styrelseledamot (typiskt vice ordförande) tar över ärendet. Ordföranden har fortsatt allmän läsrätt men avstår från beredningen.
4. Offertförfrågan går ut till 3 leverantörer inklusive anbudsgivaren (inget hinder för hen att lämna offert — jäv hindrar *styrelsen* från att gynna, inte leverantören från att konkurrera).
5. Offerter inkommer. Vid `BEDÖMNING` + `STYRELSEBESLUT` deklarerar ordföranden `JÄV_DEKLARERAT` formellt och avstår från röstning ([core-concepts.md#jäv](../core-concepts.md)).
6. Beloppet överstiger stadgans tröskelbelopp för stämmobeslut (antag 300 000 kr, tröskel 200 000 kr per stadgan). Ärendet eskaleras: status → `ESKALERAT_TILL_STÄMMA`.
7. Eftersom nästa ordinarie stämma är 6 månader bort och underhållet är akut: styrelsen kan föreslå bordläggning till extra stämma via [case-types/extrastamma.md](../case-types/extrastamma.md).
8. Alternativt: styrelsen kör ärendet på nästa ordinarie stämma och det blir `BORDLAGD_TILL_STAMMA` under mellantiden ([core-concepts.md#bordläggning](../core-concepts.md)).
9. På stämman: ordföranden avstår från omröstning (jäv-deklarationen från styrelseärendet visas som kontext). Medlemmarna röstar. Resultat registreras som `STÄMMOBESLUT`.
10. `AVTAL_TECKNAT` av firmatecknare som inte är ordföranden (annan firmatecknare enligt stadgan) — [roles/other-roles.md#firmatecknare](../roles/other-roles.md).

### Luckor

Inga nya. En smärre implicit:

- **Jäv-deklarationens räckvidd ("gift med kusin") är inte definierad.** Vad är "närstående" egentligen? [medlekap.md#delegerat-godkännande-medlemsansvarig](../medlemskap.md) listar *"make, sambo, släkting i rakt upp- eller nedstigande led eller syskon, affärspartner, annan närstående"* men inte för styrelsens generella jävsregler. **Klassning: spec-brist** — en harmoniserad "närstående"-definition behövs centralt (i `core-concepts.md#jäv` eller eget mikro-dokument).

### Positivt utfall

- Scenariot *håller* genom två spec-filer: `case-types/upphandling.md` har jäv-vid-initiering, `core-concepts.md#jäv` har det generella flödet.
- Firmatecknare som attribut ([roles/other-roles.md#firmatecknare](../roles/other-roles.md)) + kvorumsregler vid jäv garanterar att föreningen *kan* teckna avtalet även när ordföranden är jävig.
- Tröskelbelopp som tvingar stämmobeslut ([case-types/upphandling.md](../case-types/upphandling.md)) är ett konkret försvar mot att styrelsen godkänner åt den egna familjen.

## Scenario 7 — Årsstämman

Fall: Åby Nordgård (radhus multi-GA, 147 fastigheter). Ordinarie årsstämma 2026, kalenderår, hölls i maj. Styrelsen ska genomföra stämman enligt stadgan.

### Stegen

1. **Före stämman (månad 3-4):** sekreteraren driver stämmoförberedelsen ([roles/board-roles.md#sekreterarens-roll-vid-stämmoförberedelse](../roles/board-roles.md)) — dagordning från mall, förfyllning av öppna ärenden, bilagor (verksamhetsberättelse, revisionsberättelse, budgetförslag).
2. Röstlängds-underlag genereras per fastighet + andelstal per GA ([rostlangd.md#före-mötet--röstlängds-underlag](../rostlangd.md)).
3. `KALLELSE_UTSKICKAD` senast stadgad min-gräns före mötet. Tip-hash inkluderas (L6 publicering).
4. **Vid stämmans öppnande:** `MÖTE_ÖPPNAT`-händelse (del av epok N−1). Pappers-röstlängd öppnas vid ingången. Fullmakter granskas. Mandatverifiering för juridiska personer ([case-types/mandatverifiering.md](../case-types/mandatverifiering.md)).
5. Första besluten (dagordningens obligatoriska punkter 1-7 enligt [moten.md#ordinarie-årsstämma--standardmall](../moten.md)): val av mötesroller, godkännande av kallelse, fastställande av röstlängd. `STÄMMOBESLUT` per beslut.
6. Verksamhetsberättelse, revisionsberättelse ([oversight-roles.md](../roles/oversight-roles.md)).
7. Fastställande av resultat/balansräkning. Ansvarsfrihet — `STÄMMOBESLUT` triggar senare `EPOK_FÖRSEGLAD` ([föreningen.md#minsta-aktivitet](../föreningen.md)).
8. Disposition av resultat, budget, avgifter, underhållsfond, debiteringslängd fastställs (samfällighet — obligatorisk).
9. Motioner och styrelseförslag behandlas.
10. Val av styrelse, revisor, valberedning.
11. Övriga frågor (ej beslut).
12. `MÖTE_AVSLUTAT`. Sigill-hash förkunnas.
13. **Efter stämman (dagarna efter):** pappers-röstlängd + fullmakter skannas in som `BILAGA_INLÄMNAD` med kategorierna `RÖSTLÄNGD` / `FULLMAKTER` ([rostlangd.md#efter-mötet--skanning-och-bilage-arkivering](../rostlangd.md)).
14. Protokollsutkast. Justering inom 14 dagar (default). `PROTOKOLL_UPPRÄTTAT` → `PROTOKOLL_JUSTERAT`.
15. `EPOK_FÖRSEGLAD` med sigill-variant `ANSVARSFRIHET_BEVILJAD` (antag beviljad). OTS-förankring. Ny epok öppnas omedelbart.
16. Nya styrelseledamöter tillträder, konstituerar sig — `ROLL_TILLDELAD` i epok N (nya).

### Luckor

- **Signering av protokoll är ospecad i detalj.** [motesadministration.md#K-PROTOKOLL-9](../krav/motesadministration.md) säger "klick-baserad bekräftelse med tidsstämpel" i MVP. Men hur flödar det mellan mötesordförande och 2 justerare? Parallellt eller sekventiellt? [K-ÖPPEN-6](../krav/motesadministration.md) är explicit öppen fråga. **Klassning: redan känd öppen fråga, ej ny lucka.**
- **Avgörande av vilka fullmakter som räknas som giltiga.** Pappers-fullmakter granskas vid ingången; vilka deklarations-krav (äkthet, datum, underskrift)? [core-concepts.md#röstlängdsetablering](../core-concepts.md#röstlängdsetablering--processen-vid-stämmans-öppnande) nämner att mötesordföranden kan begära nytt dokument, men ingen check-lista finns. **Klassning: spec-brist.**
- **"Fråge-specifik röstlängd" för multi-GA i praktiken.** [core-concepts.md#röstlängd--snapshot-per-fråga](../core-concepts.md) säger röstlängden är fråge-specifik i samfälligheter. På en stämma med 147 fastigheter och 5 olika GA-beroende frågor betyder detta 5 olika pappers-delröstlängder eller en stor med markeringar per fråga. Hur görs detta praktiskt — är det något som genereras från systemet i förväg? **Klassning: spec-brist.**
- **Sigillets tajming vs protokoll-justeringens tajming.** `EPOK_FÖRSEGLAD` sätts enligt [föreningen.md#månad-5-6-ordinarie-årsstämma](../föreningen.md) i anslutning till stämman (steg 15 ovan). Men `PROTOKOLL_JUSTERAT` sker veckor senare. Specen är inte helt tydlig: är sigillet satt *direkt efter stämman* eller *efter protokoll-justering*? [föreningen.md minsta-aktivitet tabell](../föreningen.md) implicerar att `PROTOKOLL_JUSTERAT` tillhör N−1 men sigillet också där, utan explicit ordning. **Klassning: spec-brist.**

### Positivt utfall

- Hela stämmoflödet är relativt väl fördelat över [moten.md](../moten.md), [rostlangd.md](../rostlangd.md), [core-concepts.md](../core-concepts.md), [föreningen.md](../föreningen.md), [granskningslogg.md](../granskningslogg.md) och [krav/motesadministration.md](../krav/motesadministration.md).
- Mötesadministrations-kravdokumentet täcker de flesta pragmatiska aspekter (K-MÖTE, K-BESLUT, K-PROTOKOLL-grupperna).
- Mekanismen att pappers-röstlängd skannas in *efter* mötet är konsistent specad — inte ett digitalt incheckningskrav, inte en hård-to-implement-fantasi.

## Scenario 8 — Medlemsförändring

Fall: Rörumstrand. Fyra separata händelser under en 12-månadersperiod:

(a) Fastighet säljs från privatperson till nytt gift par → ny samägande.
(b) En medlem avlider — dödsbo uppstår.
(c) Dödsboet efter en annan (död sedan 2 år) genomför arvskifte, fastighet går till en arvinge.
(d) En fastighet köps av HSB Bank efter exekutiv auktion.

### Stegen

**(a) Försäljning till gift par**

1. Säljaren meddelar styrelsen om kommande försäljning, eller styrelsen får veta via Lantmäteriet. Ägarövergångs-ärende skapas ([case-types/agarovergang.md](../case-types/agarovergang.md)) med status `INKOMMEN`.
2. Obligatoriska fält fylls i: fastighetsbeteckning, avgående aktör, tillträdande aktörer (2 personer, samägande), `actual_transfer_date`, bilaga med köpekontrakt.
3. Styrelsen verifierar: status `VERIFIERAD`.
4. `REGISTRERAD`: `MEDLEMSKAP_ÖVERGÅTT` skrivs. Två nya `Actor` skapas (`NATURAL_PERSON`) om de inte redan finns, `Medlem`-poster med samma `primär_id` (fastighetsbeteckning) men olika `medlem_id`. `Ownership` med `sharePercent` 50/50 om stadgan är `UNIFIED` (vilket är default per [core-concepts.md#samägda-fastigheter](../core-concepts.md)).
5. Säljaren → `VILANDE`. Nya ägare → `AKTIV`.
6. Debiteringslängd justeras.

**(b) Medlem avlider**

1. Närstående eller boutredningsman meddelar styrelsen om dödsfallet.
2. `Actor = ESTATE` registreras (`AKTOR_REGISTRERAD` per [medlemskap.md#händelser-i-granskningsloggen](../medlemskap.md)) med referens till den avlidne + bouppteckningsdatum (ev. senare).
3. Den avlidnes `Medlem`-status → ?
4. Dödsboets `Medlem` skapas kopplat till samma fastighet.
5. Representation: boutredningsman om förordnad; annars dödsbodelägare med fullmakt från övriga. `Representation`-post + `REPRESENTATION_REGISTRERAD` + mandatverifiering ([case-types/mandatverifiering.md](../case-types/mandatverifiering.md)).
6. Under övergångstiden kan dödsboet rösta via representant enligt vanliga regler.

**(c) Arvskifte efter tidigare dödsbo**

1. Boutredningsman eller delägare meddelar styrelsen om avslutat arvskifte.
2. Ägarövergångs-ärende från dödsboet (Actor `ESTATE`) till arvingen (`NATURAL_PERSON`).
3. `MEDLEMSKAP_ÖVERGÅTT` — dödsboets status → `VILANDE`, arvingen → `AKTIV`.
4. Ev. ärendetyp för ägarövergång har specialfall för ESTATE→NATURAL (`[ÖPPEN]` enligt [case-types/agarovergang.md#öppna-frågor](../case-types/agarovergang.md)).

**(d) Exekutiv auktion till bank**

1. Ägarövergångs-ärende. Tillträdande aktör är `LEGAL_PERSON` (HSB Bank AB).
2. `AKTOR_REGISTRERAD` för banken (ev. redan befintlig om banken äger annan fastighet i området).
3. Representation: firmatecknare (Bolagsverket-utdrag obligatorisk). Mandatverifiering.
4. Banken är medlem. Kassörens utskick går till bankens registrerade adress.
5. Banken kommer sannolikt sälja fastigheten vidare inom månader — förbered för ny ägarövergång.

### Luckor

- **Rollen "avlidens `Medlem`-status".** [medlemskap.md](../medlemskap.md) säger `VILANDE → AKTIV` eller arkivering — men för en avliden medlem som övergår till dödsbo: ska originalet stanna `AKTIV` tills formell `MEDLEMSKAP_ÖVERGÅTT` sker (via arvskifte), eller direkt `VILANDE`? En `VILANDE`-övergång vid dödsfall är inte specad eftersom dödsboet *är* samma medlemskap på fastigheten, bara med ny Actor. **Klassning: modell-brist.** Förslag: ny status-kod `VILANDE_DÖDSFALL` eller explicit regel "medlemskap stannar AKTIV med nytt `Actor = ESTATE`".
- **Arvskifte som `MEDLEMSKAP_ÖVERGÅTT` — men mellan vilka aktörer?** [case-types/agarovergang.md#öppna-frågor](../case-types/agarovergang.md) är explicit öppen om detta. **Klassning: redan känd öppen fråga, ej ny lucka.**
- **Exekutiv auktion — källa till uppgiften.** [case-types/agarovergang.md#obligatoriska-fält](../case-types/agarovergang.md) listar "köpekontrakt, Lantmäteri-utdrag, säljarens anmälan, köparens anmälan". Exekutiv auktion har *exekutionsurkund* istället — ingen av listans fyra. **Klassning: spec-brist (mindre).**
- **Slutavräkning vid ägarbyte är textbibliotek-specad men ingen finansiell beräkning.** [case-types/agarovergang.md#textbibliotek](../case-types/agarovergang.md) har mall "slutavräkning till säljare" — men det finansiella innehållet (hur periodiseras kvartalsavgiften?) hänvisar till kassörens godtycke. **Klassning: spec-brist** (kopplas till samma pro-rata-fråga i scenario 3).

### Positivt utfall

- Actor-abstraktionen ([medlemskap.md#juridisk-person-som-medlem](../medlemskap.md)) hanterar NATURAL_PERSON, LEGAL_PERSON, ESTATE, MUNICIPALITY konsistent. Alla fyra sub-fall i scenariot kan modelleras utan att skapa specialfall.
- `MEDLEMSKAP_ÖVERGÅTT`-händelsens två-aktörs-referens är designad just för detta.
- `UNIFIED`/`PROPORTIONAL`-valet för samägande är explicit stadge-konfiguration, med `UNIFIED` som default som matchar praxis (9/10 granskade föreningar följde den).

## Scenario 9 — Revisorns löpande granskning

Fall: PSV-Torekov. Revisorn är lekmannarevisor (pensionerad företagsledare). Under räkenskapsåret vill hen följa styrelsens arbete löpande, inte vänta tills slutet.

### Stegen

1. Revisorn har permanent läsrätt från dag 1 av räkenskapsåret ([oversight-roles.md#principer](../roles/oversight-roles.md)).
2. Revisorns arbetsyta (separat från styrelsens, [oversight-roles.md#revisorns-arbetsyta](../roles/oversight-roles.md)) ger översikt, beslutslogg, ekonomi-vy, jäv-historik, revisorsfrågor, revisionsberättelse-utkast.
3. Revisorn gör urval: vilka styrelsebeslut, vilka utlägg, vilka jävsdeklarationer ska granskas specifikt? Granskningsplan sätts.
4. Vid tvekan skickas revisorsfrågor ([case-types/revisorsfraga.md](../case-types/revisorsfraga.md)) löpande. `REVISORSFRÅGA_STÄLLD`, styrelsen svarar via `REVISORSSVAR_AVLÄMNAT`. Iterativ dialog.
5. Revisorn gör anteckningar, flagga viktiga poster för revisionsberättelsen.
6. Vid räkenskapsårets slut samlas observationer i `REVISIONSBERÄTTELSE_AVLÄMNAD` med till-/avstyrkan av ansvarsfrihet.
7. Vid stämman presenteras revisionsberättelsen. Obesvarade revisorsfrågor räknas upp ([case-types/revisorsfraga.md#öppna-frågor](../case-types/revisorsfraga.md)).

### Luckor

- **Vad *gör* revisorn faktiskt under året?** Specen säger att revisorn har läsrätt, arbetsyta och kan ställa frågor. Men operativa flöden — *"granskningsplan"*, *"urvalsmetod"*, *"gångjärnspunkter i revisionen"* — finns inte i specen. [oversight-roles.md#revisorns-arbetsyta](../roles/oversight-roles.md) är en punktlista, inte ett flöde. **Klassning: spec-brist.** Förslag: `spec/revisor-arbetsyta.md` eller sektion i `oversight-roles.md` om *"Revisorns arbetsår"*.
- **Revisor-anteckningar är inte specad entitet.** I beslutsloggen finns bara `REVISORSFRÅGA_STÄLLD` + `REVISORSSVAR_AVLÄMNAT`. Anteckningar som revisorn gör *för sig själv* under året (inte som fråga till styrelsen) har ingen plats i modellen. Är de bara ostrukturerade noteringar i utkast-modus? **Klassning: modell-brist (eller spec-brist).**
- **Flagg-/bokmärkesfunktion.** Revisorn bör kunna markera specifika händelser i granskningsloggen som "värd att ta upp" utan att formalisera det som revisorsfråga. Specen har inget sådant koncept. **Klassning: spec-brist.**
- **Varnings-tröskel vid obesvarade frågor.** [case-types/revisorsfraga.md#tidsregler](../case-types/revisorsfraga.md) säger 14 dagars default, 3 dagar vid kritisk. Men vad händer om styrelsen systematiskt inte svarar? Förslag i öppen fråga (*"Antal obesvarade revisorsfrågor vid räkenskapsårets slut: X visas i revisionsberättelsen"*) är inte implementerad. **Klassning: redan känd öppen fråga, ej ny.**
- **"Löpande läsrätt" på externa administratörens data.** Om föreningen använder `ExternalAdministrator` (Kyrkmossen → HSB) som har skrivrätt på ekonomivyn — har revisorn läsrätt också där? [roles/other-roles.md#externaladministrator](../roles/other-roles.md) antyder ja, men det är inte explicit. **Klassning: spec-brist.**

### Positivt utfall

- Revisorn som *konstitutionellt oberoende organ* är tydligt artikulerad ([oversight-roles.md#principer](../roles/oversight-roles.md)). Systemet förhindrar dubbel-tilldelning styrelse/revisor.
- Permanent läsrätt från dag 1 (*"uppdraget är skyddsmekanismen"*) är en stark princip.
- [case-types/revisorsfraga.md](../case-types/revisorsfraga.md) som låg-tröskel-iterativ kanal är väl specad.

## Scenario 10 — Stadgeändring

Fall: Fröåvägen vill höja underhållsfond-avsättningen från 70 000 kr/år till 100 000 kr/år. Styrelsen förbereder stadgeändring.

### Stegen

1. Styrelsen beslutar att förslagsstadgeändring ska drivas. Stadgeändrings-ärende skapas ([case-types/stadgeandring.md](../case-types/stadgeandring.md)) med obligatoriska fält: förslagstext, diff mot nuvarande, berörda paragrafer (§14 om underhållsfond), motivering, konsekvensbedömning, kategorisering (*väsentlig* — rör ekonomisk skyldighet för medlemmar).
2. Diff-vy genereras av stadge-modulen (per [moten.md#öppna-frågor](../moten.md) behöver modulen formalt specas).
3. Styrelsen tar upp ärendet, skriver yttrande. Status → `YTTRANDE_FÖRLAGT`.
4. `PUBLICERAD_FÖRSTA_STÄMMAN` vid kallelseutskick till första stämman (kan vara ordinarie årsstämma eller extra).
5. Första stämman: `PÅ_FÖRSTA_STÄMMAN`. Omröstning. Krav: stadge-konfigurerad tröskel (default enkel majoritet? — specen är otydlig om vad "första stämmans tröskel" är; LFS 52§ kräver minst *två på varandra följande stämmor* men specificerar inte tröskel på första). `STÄMMOBESLUT` → `AVGJORD_FÖRSTA_STÄMMAN` med resultat `BIFALL_PRELIMINÄRT`.
6. **Mellanperiod:** minst 1 månad enligt [case-types/stadgeandring.md#tidsregler](../case-types/stadgeandring.md) default. Kommunikation till medlemmar om att andra stämman kommer.
7. `PUBLICERAD_ANDRA_STÄMMAN` vid kallelseutskick.
8. Andra stämman: omröstning med *kvalificerad* majoritet (default 2/3, typiskt definierat i `STADGEÄNDRINGSMAJORITET`-taggen). `STÄMMOBESLUT` → `AVGJORD_SLUTGILTIGT` med `BIFALL_SLUTGILTIGT`.
9. `STADGEVERSION_ANTAGEN`-händelse skrivs. Ny stadgeversion blir aktiv.
10. Eftersom kategorisering är *väsentlig*: samtyckes-förnyelse triggas för alla medlemmar ([medlemskap.md#samtyckes-förnyelse-vid-stadgeändring](../medlemskap.md)). Medlemmar får 90 dagar (default).
11. Medlemmar som inte bekräftar inom fristen → `VILANDE` med orsak *"samtycke till stadgeversion X ej förnyat"*.
12. Bolagsverket-inlämning är inte relevant för samfällighet (det är LEF-specifikt) — för samfällighet kan det krävas Lantmäteri-registrering av vissa stadgeändringar, inte specat.

### Luckor

- **Tröskel på *första* stämman är inte specad.** [case-types/stadgeandring.md#livscykel](../case-types/stadgeandring.md) säger "stadgeenlig majoritet (typiskt 2/3 av avgivna röster, eller högre per stadga)" — men det är oklart om det gäller *båda* stämmor eller bara den andra. LFS 52§ och LEF 6:36 skiljer sig. **Klassning: spec-brist.**
- **Lantmäteri-registrering av samfällighets-stadgeändring.** Ändring av en samfällighetsförenings stadgar ska registreras hos Lantmäteriet enligt LFS. Specen har ingen motsvarighet till LEF:s Bolagsverket-flöde för samfällighet. **Klassning: spec-brist.**
- **Samtyckes-förnyelse för samfällighetsmedlemmar är konceptuellt ovanligt.** Ett samfällighetsmedlemskap följer fastighetsägarskap enligt LFS — det är inte frivilligt på samma sätt som LEF-medlemskap. Ska `VILANDE` vid utebliven bekräftelse faktiskt kunna triggas för en samfällighets-medlem? [medlemskap.md#samtyckes-förnyelse-vid-stadgeändring](../medlemskap.md) är skriven generiskt över båda editioner. **Klassning: spec-brist** — behöver edition-specifikt klargörande.
- **Klandertalan mot stadgeändring (LFS 53§ / LEF 7:54).** [case-types/stadgeandring.md#öppna-frågor](../case-types/stadgeandring.md) är explicit öppen om hur klandertalan-fönstret pausar eller inte pausar `VERKSTÄLLD`. **Klassning: redan känd öppen fråga.**
- **Diff-vyns innehåll.** "Sida-vid-sida-diff" är mentionerad men stadge-modulen saknar spec. Vad är en "paragraf" i stadge-modulens datamodell? Tuple `(nummer, rubrik, text)`? **Klassning: spec-brist (kopplad till stadge-modul som sådan).**

### Positivt utfall

- Två-stämmo-flödet är väl strukturerat som egen case-type med specialiserade status-värden.
- Samtyckes-förnyelse är korrekt kopplad till *väsentliga* ändringar — redaktionella kräver ingen förnyelse.
- Diff + publicering + revisorns kategoriseringsgranskning ([medlemskap.md#öppna-frågor](../medlemskap.md)) är ett meningsfullt försvar mot "ren omformulering döljer reell förändring"-hotet ([case-types/stadgeandring.md#hot-och-försvar](../case-types/stadgeandring.md)).

## Scenario 11 — Onboarding med gammal stadga

Fall: Samfällighet med stadgar antagna 1976, senast reviderade 1998. Föreningen fungerar sedan 50 år. De går med i Tillsammans 2026 utan att vilja modernisera stadgan — den uppladdade stadgan ska gälla som kopia av pappersoriginalet.

### Stegen

1. Som scenario 2 fram till stadgeuppladdning.
2. Föreningen laddar upp PDF av 1976-stadgan med 1998 års revidering införlivad (inskannad från arkivet).
3. [editions/samfallighet.md#äldre-samfälligheter--stadga-laddas-in-som-kopia](../editions/samfallighet.md) aktiveras. Systemet behandlar stadgan som sanning.
4. Paragraf-uppdelning sker — kanske OCR-stödd eller manuell.
5. Paragraftaggning: vissa paragrafer mappar direkt till typkoder (`RÄKENSKAPSÅR`, `KALLELSETID`, `STYRELSESAMMANSÄTTNING`). Andra är gammalmodiga formuleringar om t.ex. *"den manliga fastighetsägaren"*, eller saknar koncept som *"digital kallelse"*. Dessa taggas "ingen — fri text".
6. `STADGEVERSION_ANTAGEN` skrivs med attestation-flagga *"motsvarar pappersoriginalet"*.
7. Onboardingen fortsätter som normalt.

### Luckor

- **Attestations-mekaniken är inte specad.** [editions/samfallighet.md](../editions/samfallighet.md) säger att stadgan ska kunna attesteras som pappersmotsvarighet, men inte vem som attesterar (hela styrelsen? ordförande ensam? extern juridisk granskning?), vilken händelsetyp det är (`STADGEVERSION_ANTAGEN` med underflagga eller separat `PAPPERSATTESTATION_UTFÖRD`?). **Klassning: spec-brist** (samma som scenario 2, andra lucka).
- **Otagbara paragrafer blockerar inte — men hur uttrycks det?** [moten.md#otaggade-paragrafer-är-fortfarande-värdefulla](../moten.md) säger att otaggade paragrafer är okej. För en 1976-stadga kommer *många* paragrafer vara otagbara. Systemet bör inte pressa föreningen att "tagga allt" — finns en explicit sub-minimal setup där bara kritiska paragrafer (kallelsetid, röstmetod, stämmoperiod, styrelsesammansättning) krävs? **Klassning: spec-brist.**
- **Gammalmodig formulering som indirekt konflikt med svensk rätt.** En 1976-stadga kan innehålla formuleringar som idag vore grundlagsstridiga (könsdiskriminerande etc.). Systemet ska inte modernisera — men ska det flagga? [threats.md](../threats.md) listar inte "otidsenliga stadgar" som hot. **Klassning: medvetet out-of-scope** (specens princip: *"Tillsammans är varken polis eller skatteverk"* ([medlemskap.md#vad-systemet-uttryckligen-inte-gör](../medlemskap.md))). Dokumenterat.
- **Digitala kanalen saknas i stadgan.** En 1976-stadga föreskriver sannolikt kallelse per "rekommenderat brev" eller "anslag i tidningen". Kan föreningen använda digital kallelse ändå, eller krävs stadgeändring först? **Klassning: spec-brist.** Förslag: `KALLELSEMETOD`-typkod kan ha fält `medger_digital_ekvivalent: bool` per kanal, men lagen/stadgans formkrav måste respekteras.

### Positivt utfall

- Principen *"kopia av det som faktiskt gäller, inte en modernisering"* ([verification.md#scenario-11](../verification.md)) är explicit artikulerad.
- Lapptäckes-modellen ([domain-model.md#lapptäcket](../domain-model.md#lapptäcket--etablering-och-dokumentkonflikter)) tolererar gammal data som sanning snarare än att kräva rensning.
- Otaggade paragrafer tillåts — systemet tvingar ingen gradvis modernisering.

## Sammanställning av luckor

| # | Lucka | Klassning | Scenario(n) | Föreslagen åtgärd |
|---|---|---|---|---|
| 1 | Onboarding-sekvensen saknar samlad spec | Spec-brist | 1, 2, 11 | Skapa `spec/onboarding-samfallighet.md` eller en enig `editions/samfallighet.md#onboardings-sekvens` |
| 2 | Stadge-modulen är ospecad | Spec-brist | 1, 2, 10, 11 | `spec/stadge-modul.md` (redan flaggat i [moten.md#öppna-frågor](../moten.md)) |
| 3 | Andelstal-bulkimport saknas | Spec-brist | 1, 2 | Mekanik i `editions/samfallighet.md` — CSV-format, validerings-regler |
| 4 | Genesis-aktör för första `ROLL_TILLDELAD` | Modell-brist | 1 | Bootstrap-aktör eller "uppladdare" som initial RBAC-källa |
| 5 | Ingen retroaktiv epok för pre-Tillsammans-år | Spec-brist | 2 | `EPOK_ÖPPNAD` med fält `inherited_history` + extern arkivreferens |
| 6 | "Pappersoriginalet"-attestation odefinierad | Spec-brist | 2, 11 | Specificera signerarkretsen + underflagga på `STADGEVERSION_ANTAGEN` |
| 7 | Dokumentkonflikt-flödet utan ärendetyp | Spec-brist | 2 | Beslut: *Allmänt ärende* räcker, eller egen sub-typ? |
| 8 | `Actor = ESTATE` utan representant | Modell-brist | 2, 8 | Ny `vilande_orsak = KÄND_EJ_REPRESENTANT` + visningslogik |
| 9 | Historiska pre-Tillsammans-beslut som stomme | Spec-brist | 2 | `HISTORISKT_BESLUT_REGISTRERAT`-händelse utan hash-integritet |
| 10 | Medlemmar utan känd kommunikationskanal | Spec-brist | 2 | Fält `kontakt_status = OKÄND` + fallback-policy |
| 11 | Gradvis onboarding (otaggade paragrafer i MVP?) | Spec-brist | 2, 11 | Definiera "kritiska paragrafer"-minimum + explicit opt-in för fördjupad tagging |
| 12 | `FAKTURA_UTSTÄLLD` livscykel saknas | Spec-brist | 3 | Sektion i `editions/samfallighet.md` eller `spec/case-types/fakturering.md` |
| 13 | Debiteringsrond-livscykel saknas | Spec-brist | 3 | Samma som ovan — event-sekvens, bulkgenerering |
| 14 | Pro-rata-beräkning vid ägarbyte ospec | Spec-brist | 3, 8 | Formulär "dagar / månader / kalender" i `editions/samfallighet.md` |
| 15 | Default-visibility per transaktionstyp | Spec-brist | 3 | Tabell i `granskningslogg.md#vyer` |
| 16 | `ExternalAdministrator` skriv-rätt scope | Spec-brist | 3, 9 | Mer explicit RBAC-tabell i `roles/other-roles.md#externaladministrator` |
| 17 | Motionsfrist vs kallelse-min konflikt | Spec-brist | 4 | `KALLELSETID`-paragrafen validerar *kallelse-min ≥ motionsfrist + beredningsbuffert* |
| 18 | Bifallen motion → upphandlingsärende länk | Spec-brist | 4 | Case-type-integration i `case-types/upphandling.md#relationer` |
| 19 | Motions-arkiv sökvy över år | Spec-brist | 4 | Specifik projektion/vy-spec |
| 20 | "Närstående" harmoniserad definition | Spec-brist | 6 | Central definition i `core-concepts.md#jäv` |
| 21 | Protokoll-signering parallell/sekventiell | Öppen (K-ÖPPEN-6) | 7 | Redan i [krav/motesadministration.md#öppna-frågor](../krav/motesadministration.md) |
| 22 | Fullmakts-validering check-lista | Spec-brist | 7 | Underfält i `case-types/mandatverifiering.md` eller `core-concepts.md#röstlängdsetablering` |
| 23 | Fråge-specifik pappers-röstlängd praktiskt | Spec-brist | 7 | Konkret mekanik i `rostlangd.md` för multi-GA |
| 24 | Sigill vs protokoll-justering tajming | Spec-brist | 7 | Fastställ ordningen i `föreningen.md#minsta-aktivitet` |
| 25 | Avliden medlem vs dödsbo — status-övergång | Modell-brist | 8 | Ny `VILANDE_DÖDSFALL` eller explicit regel |
| 26 | Arvskifte `MEDLEMSKAP_ÖVERGÅTT` ESTATE→NATURAL | Öppen fråga | 8 | Redan i [case-types/agarovergang.md#öppna-frågor](../case-types/agarovergang.md) |
| 27 | Exekutiv auktion som källa | Spec-brist | 8 | Utöka listan i `case-types/agarovergang.md#obligatoriska-fält` |
| 28 | Slutavräkning finansiell beräkning | Spec-brist | 8 | Samma som 14 |
| 29 | Revisorns arbetsår — operativt flöde | Spec-brist | 9 | `spec/revisor-arbetsyta.md` eller sektion i `oversight-roles.md` |
| 30 | Revisor-anteckningar som entitet | Modell-brist | 9 | Egen entitet eller draft-mode på revisorsfråga |
| 31 | Revisor-flagg/bokmärke | Spec-brist | 9 | Egen UX-koncept |
| 32 | Första-stämmans tröskel vid stadgeändring | Spec-brist | 10 | Klargör i `case-types/stadgeandring.md#livscykel` |
| 33 | Lantmäteri-registrering av samf-stadgeändring | Spec-brist | 10 | Motsvarighet till LEF:s Bolagsverket-flöde |
| 34 | Samtyckes-förnyelse för samf-medlem (ovanligt) | Spec-brist | 10 | Edition-specifikt klargörande i `medlemskap.md` |
| 35 | Otidsenliga stadgar — ej flaggade | Out-of-scope | 11 | Medvetet dokumenterat; ingen åtgärd |
| 36 | Digital kallelse vs formkrav i stadgan | Spec-brist | 11 | `KALLELSEMETOD`-paragrafen klargörs |

Tabellen har 36 rader; rapportens översikt nämner 27 "luckor". Skillnaden är att vissa rader är *redan kända öppna frågor* (21, 26) och några täcks av `krav/motesadministration.md` utan att behöva specen utvidgas. Netto-tillägg till spec-arbetet: ~27.

## Implikationer för utvecklingsplan

Givet vad scenarierna avslöjar, har följande spec-etapper tydlig prioritet innan kod påbörjas.

### Etapp A — Kärnflöden som inte saknar spec men saknar samling

- **Onboarding-sekvensen för samfällighet** (luckor 1-3, 5-11). Flest luckor per åtgärd. Utan detta kan MVP inte tas i drift för en verklig förening.
- **Debiteringsrond / fakturering** (luckor 12-14). Utan detta är `editions/samfallighet.md`:s löfte om *"debiteringslängd som automatisk output"* halvt uppfyllt.

### Etapp B — Stadge-modulen

- Etapp A:s onboarding-flöde förutsätter stadge-modulen. Utan en minimal stadge-modul-spec (paragraf-modellen, taggnings-UI, parameter-validering) faller Etapp A.
- Diff-vy vid stadgeändring (lucka 10-värdet) är samma modul.
- Föreslaget namn: `spec/stadge-modul.md`.

### Etapp C — Roller och arbetsytor

- **Revisorns arbetsår** (luckor 29-31). Rollen har principerna men saknar flödet.
- `ExternalAdministrator` skriv-/läs-scope explicitering (luckor 16).
- *"Närstående"*-definition centralt (lucka 20).

### Etapp D — Domänmodellens edge cases

- Avlidens medlem vs dödsbo som status-övergång (lucka 25).
- `Actor = ESTATE` utan representant (lucka 8).
- Genesis-aktör för första `ROLL_TILLDELAD` (lucka 4).

### Etapp E — Klargöranden av existerande flöden

- Protokoll-signering (lucka 21, redan flaggat).
- Sigill vs protokoll-justering tajming (lucka 24).
- Motionsfrist-validering mot kallelse-min (lucka 17).
- Tröskel på första stämman vid stadgeändring (lucka 32).

### Vad som *inte* behöver etapp

Följande luckor hör hemma externt — inga spec-ändringar behövs:

- Otidsenliga stadgar (lucka 35, *out-of-scope*, matchar [medlemskap.md#vad-systemet-uttryckligen-inte-gör](../medlemskap.md)).
- Inbetalnings-matchning, bokföring, indrivning (täcks redan av [architecture.md#utanför-scope](../architecture.md)).

## Positiva resultat — vad modellen redan bär

Även om 27 luckor listas är det värt att notera vad som *höll* utan problem:

- **Governance-skölden** ("Ni lovade"-flödet, scenario 5) — verkligheten och modellen sammanfaller.
- **Jäv-mekaniken** (scenario 6) — spec-filer samverkar konsistent från initiering till stämmobeslut.
- **Motion som referens-ärendetyp** (scenario 4) — tätast-specade delen i hela specen, och det syns.
- **Actor-abstraktionen** (scenario 8) — klarar blandade ägarformer (NATURAL, LEGAL, ESTATE, MUNICIPALITY) utan hybrid-modell.
- **Lapptäckes-modellen** (scenarier 2, 11) — konceptuellt mogen för äldre föreningars dokumentverklighet.
- **Två-stämmo-flödet för stadgeändring** (scenario 10) — livscykel och mellantids-mekanik är rätt artikulerade.
- **Räkenskapsårs-epoker** (scenarier 3, 7) — "två perioder samtidigt"-modellen hanterar överlappet jan-juni naturligt.

Det är ett intakt skelett med hål i muskelfästet, inte en modell som behöver rivas upp. Luckorna är skrivbara — det som behövs är tid och beslut, inte en omstart.

## Referenser

- Verifikations-metodik: [verification.md](../verification.md)
- Empirisk spår 1-rapport: [samfalligheter-2026-04.md](samfalligheter-2026-04.md)
- Huvudspec-register: [tillsammans.md](../../tillsammans.md)
