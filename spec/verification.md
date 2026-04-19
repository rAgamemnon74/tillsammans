# Teoriverifikation — testa modellen mot verkligheten före kod

> Del av [Tillsammans-specifikation](../tillsammans.md).

Specen är en **teoretisk modell** över governance-behoven. Den är rimlig och intern-konsistent — men risken är att vi bygger en elegant lösning på fel problem. Innan MVP-kod skrivs bör modellen verifieras mot verkligheten enligt en strukturerad metodik. Fyra parallella spår.

Första instansen är för **samfällighet** (där komplexiteten är störst och användarens egen domänkännedom djupast). Metodiken är generisk och återanvänds för LEF när dess tid kommer.

## Spår 1: Stadgejämförelse (empirisk bredd)

- Samla 3–5 verkliga samfällighetsstadgar att börja med. Användarens egen förening som bas; kompletteras med anonyma kontakter eller offentliga exempel.
- Kartlägg variation per dimension: kallelsetider, röstmetoder (huvudmetod/andelsmetod), styrelsesammansättning, ekonomiska befogenheter, stadgeändringsmajoritet, samägande-regler (`UNIFIED`/`PROPORTIONAL`), motionshantering, jäv-formuleringar.
- **Test:** Kan vår stadge-modul representera dem alla utan tvång eller förlust av information?
- **Utfall:** avvikelser → antingen modell-lucka att täppa, eller medvetet "out of scope" att dokumentera.

## Spår 2: Typscenarier (funktionella flöden)

Gå igenom 10 konkreta scenarier i modellen, steg för steg, från start till färdig beslutslogg:

1. **Onboarding av nystartad samfällighet** — initial digital tvilling-etablering
2. **Onboarding av 40-år-gammal samfällighet** — lapptäckes-hantering (gamla förrättningsakter, servitut, oklara ägarbilder)
3. **Debiteringsrond** — andelstal × kostnader per GA → utgående avgifter per fastighet
4. **Motion från medlem** — inlämning, granskning, styrelsens yttrande, beredning till stämma
5. **"Ni lovade"-konflikt** — medlem hävdar informellt löfte som saknar spår, styrelsen kontrar med loggen, bordläggning
6. **Jäv-deklaration och bordläggning** — ordföranden är svåger till entreprenören som ska upphandlas
7. **Årsstämman** — kallelse → röstlängdssnapshot → etablering vid öppnande → omröstningar → protokoll → signering
8. **Medlemsförändring** — försäljning av fastighet, dödsbo som består, arvskifte, samägande som uppstår eller upplöses
9. **Revisorns löpande granskning** — över verksamhetsåret, inte bara vid årsslutet
10. **Stadgeändring** — förslag, beredning, stämmobeslut (krav på kvalificerad majoritet), Bolagsverket-underlag
11. **Onboarding med gammal stadga** — 10+ år gamla stadgar är vanliga även i samfällighets-domänen (PSV-Torekov 2019, Äskestock 2013, Warbro/Enebackens 2013 rev., flera samfälligheter med stadgar från 1970-talet fortfarande formellt gällande). Systemet måste acceptera att "senaste versionen" kan vara gammal och att digitaliseringen är en *kopia av det som faktiskt gäller*, inte en modernisering. Testet: kan föreningen lägga in en 15-årig stadga oförändrad, få den attesterad som "motsvarar pappersoriginalet", och därefter börja använda systemet?

**Test:** flödet håller utan tvetydigheter eller steg som kräver ad-hoc manuell intervention.
**Utfall:** oklara steg → modell-brist eller spec-brist att åtgärda.

## Spår 3: Juridisk strukturverifiering

Mappa modellen mot svensk rätt och dokumentera att formalia uppfylls:

- **Anläggningslagen (1973:1149)** — definitioner av gemensamhetsanläggning, andelstal, förrättning
- **LFS (1973:1150)** — samfällighetsförvaltningens formalia: kallelsetider, röstning, protokoll, majoritetsregler, jäv
- **LEF (2018:672)** — för övriga ekonomiska föreningar som kan köra Tillsammans

**Test:** Uppfyller modellen lagens krav på kallelsetider, röstformalia, protokollkrav, majoritetsbeslut, jäv-bestämmelser?
**Utfall:** avvikelse från lag = bugg i modellen som måste fixas (lagen trumfar designen).

## Spår 4: Konflikt-simulering (red-team mot egen produkt)

För varje hot i [hotbilden](threats.md), konstruera konkret attack och spåra den genom modellen:

- Medlem hävdar informellt löfte + hotar stämma personligen → stoppas av beslutslogg + bordläggning?
- Taktisk motionsgivning (10 motioner för förhandlingstryck) → formaliakraven avvisar eller grupperar?
- Historierevision (*"det har vi alltid gjort"*) → audit-trail levererar motbevis?
- Jäv som döljs som *"familjeaffär"* → obligatorisk deklaration + revisor-åtkomst fångar det?
- Ex-ägare som hävdar kvarvarande rätt → aktivt medlemskap (`LAPSED`) avvisar?
- Samägande som obstruktionsverktyg → UNIFIED/PROPORTIONAL-modellen hanterar?
- **Passiv erosion efter personalväxling** — engagerad kohort cyklar ut, publiceringsrutiner faller → ärver efterträdaren ett sökbart arkiv eller ett frusen fragment? Fall: en av de verifierade föreningarna där öppen dokumentation de facto upphörde ~2020 när dåvarande styrelse gick vidare. Testet: är dokumentationen en *biprodukt* av systemet (överlever) eller en *manuell rutin* (dör med kohorten)?

**Test:** kan skyddsmekanismen bryta attacken, eller finns en kringgång?
**Utfall:** kringgång → förstärk modellen innan kod skrivs.

## Spår 5: Teknisk robusthet — integritets- och renderings-mekanik

Utöver governance-verifikation ska den tekniska bärlagren testas separat och kontinuerligt (i CI, inte bara vid större releaser). Teknisk robusthet är försvaret bakom governance-integriteten.

### Template-rendering

Varje transaktionstyp i [granskningslogg.md](granskningslogg.md) har en mall som renderar fritext från systemdata. Testmatrisen:

- Alla typer × alla giltiga systemdata-varianter renderar utan oväntade undantag.
- Kantfall: tomma fält, oväntade fält, mycket långa texter, unicode-normaliseringar, null/undefined-värden, gränsvärden.
- `template_incomplete`-flaggan sätts korrekt när mallen inte kan rendera alla fält.
- Historiska mall-versioner testas via regressionstest — template v1 ska producera samma output 2046 som 2026 givet samma systemdata.
- Svenska teckenkodning (åäö), långa meningar, bindestreck, citattecken hanteras korrekt i kanonisering.

Denna testkategori minimerar risken att `template_incomplete` blir ett vanligt tillstånd på grund av defekt mall snarare än genuin ofullständighet i källdata.

### Hash-kedjans integritet

- Manipulation av metadata, fritext, systemdata eller bilaga-innehåll detekteras av verifieringskommandot.
- Rättelse-poster och addenda bryter aldrig originalkedjan.
- OTS-förankring är verifierbar mot Bitcoin-blockchain (mockad i testmiljö med lokal testblockchain).
- Kanonisering är deterministisk — samma indata → samma hash, oavsett plattform, Postgres-version eller ordning på INSERT.

### Schema-evolution

- Gammal schema-version kan läsas med nyare kod utan data-tappande migration.
- Nya scheman valideras mot bakåtkompatibilitets-regler före adoption.
- `schema_hash` i metadata detekterar tyst schema-swap.

### GDPR-gallring

- Gallring av bilage-bytes bevarar hash-kedjan intakt.
- Metadata + `innehålls_hash` överlever gallring.
- Systemhanterad gallring triggar på rätt tidpunkt enligt ärendelivscykel.
- Styrelsebeslut-kopplad extraordinär gallring kräver föregående `STYRELSEBESLUT` med motivering.

### Addenda-kedjan

- Addendum-händelser ändrar aldrig originalepokens hash-kedja.
- Addendum-kedjans egen `prev_addendum_hash` är korrekt.
- Alla fem legitima addendum-typer har giltig grund-validering.

## Verifikations-tröskel före implementation

MVP-kod påbörjas inte förrän följande är dokumenterat:

- [x] Minst 3 samfälligheter-stadgar jämförda mot stadge-modulen, variationen kartlagd — **10 föreningar avprickade apr 2026**, se [verification-reports/samfalligheter-2026-04.md](verification-reports/samfalligheter-2026-04.md)
- [ ] Alla 10 typscenarier genomgångna med luckor dokumenterade
- [ ] Juridisk mappning till LFS + Anläggningslagen + (för LEF) LEF genomförd
- [ ] Alla hot-mönster från hotbilden konflikt-simulerade minst en gång
- [ ] Minst en extern sakkunnig läsning (advokat, erfaren samfällighetsföreträdare, eller liknande)

## Metodiska principer

- **Verkliga fall före hypotetiska.** Användarens egen samfällighet är startpunkt, inte ett konstruerat exempel.
- **Dokumentera även positiva utfall.** *"Vi testade X-scenariot mot modellen och den höll"* är värdefull evidens — inte bara brister.
- **Vid misslyckande: ändra modellen, inte verkligheten.** En samfällighet som inte passar modellen indikerar modell-brist, inte samfällighets-brist.
- **Iterativ konvergens.** Verifikationscyklar tills modellens claim om *"denna typ av förening kan vi stödja"* är trovärdigt — inte bara hoppat på.
- **Öppen dokumentation.** Verifikationsresultaten (inklusive identifierade luckor) är offentligt open source — bidrar till förtroendet hos potentiella användare.

## Generalisering

Metodiken återanvänds för andra `AssociationType`:

- **LEF:** LEF-paragrafer dominerar Spår 3; stadgevariationen i Spår 1 blir större eftersom typen täcker många verksamheter. Spår 4 behöver hotmönster som är specifika för ekonomiska föreningar (insatsernas hantering, utdelning, varierande medlemsprövning).

Denna samfällighet-verifikation blir mall för LEF.
