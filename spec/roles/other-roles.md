# Övriga roller

> Del av [Tillsammans-specifikation](../../tillsammans.md).

Detta är rollerna utanför den stående styrelsen och mötets egna roller. De flesta är **attribut** — flaggor eller medlemskap som läggs ovanpå en befintlig roll (styrelseledamot eller medlem) — snarare än egna rollidentiteter. Undantag: `ExternalAdministrator` (icke-vald extern part) och bas-rollen `Medlem` (inneboende av föreningstillhörighet).

**Relaterade dokument:**
- [board-roles.md](board-roles.md) — styrelseroller
- [oversight-roles.md](oversight-roles.md) — revisor och revisorssuppleant
- [nominating-committee.md](nominating-committee.md) — valberedning
- [meeting-roles.md](meeting-roles.md) — ephemerala mötesroller
- [architecture.md](../architecture.md) — utskott, underfonder, huvudmanna-relation, `ExternalAdministrator`-konceptet
- [editions/foraldraforening.md](../editions/foraldraforening.md) — klassrepresentant och utskott i skol-FF-kontext

## Principer

1. **Attribut framför rolexplosion.** En person som är kassör, firmatecknare och sammankallande i eventkommittén har tre attribut på en kassör-roll — inte tre distinkta roller.
2. **Lag- eller stadgestyrd grund krävs för förstklassig status.** Firmatecknare är lagligt definierat begrepp men följer styrelseledamot (ingen särskild tilldelning i stämman). Kommittéledamot är stämma-vald men inom en avgränsad kontext (utskottet). ExternalAdministrator är avtalstyrd och icke-vald.
3. **Attribut bär egen audit.** Även om det är flaggor loggas tilldelning och upphörande — en firmatecknare som slutade 2024 ska synas i historiken om någon granskar en signatur från den tiden.

## Firmatecknare

- **Stödnivå:** attribut på en eller flera styrelseledamöter.
- **Lag- och stadge-grund:** LEF 6:29, LFS 37§, föreningsstadgar.
- **Ansvar:** Teckna föreningens firma — binda föreningen juridiskt gentemot tredje part genom sin underskrift.
- **Hur stödjs:** `RoleAssignment` får en `signingAuthority`-flagga med typ: `each-separately` (*"var för sig"*), `two-together` (*"två i förening"*), `all-together` (*"gemensamt"*). Default enligt stadgan.
- **Typiska mönster:** Ordförande och kassör var för sig (vanligast i mindre föreningar). Ordförande och en ledamot gemensamt (större föreningar för säkerhet). Hela styrelsen gemensamt (mycket stora eller riskkänsliga avtal).
- **Hot att skydda:** Avsteg från stadgan — en enskild firmatecknare som ingår avtal utanför stadgans ram binder föreningen juridiskt men utsätter sig själv för skadeståndsansvar. Systemet dokumenterar; det validerar inte mot externa avtal.
- **`[ÖPPEN]`:** Ska systemet kräva digital signering via BankID för firmatecknings-handlingar? Kopplas till [open-questions.md#öppen-autentisering](../open-questions.md). Förslag: stödja men inte kräva — många föreningar tecknar fortfarande papper.

## Utskotts- och kommittéledamot

- **Stödnivå:** attribut via medlemskap i en `Committee`-entitet ([architecture.md](../architecture.md), [editions/foraldraforening.md#utskott](../editions/foraldraforening.md#utskott)).
- **Lag- och stadge-grund:** Stadge-bestämd delegering. Utskottets mandat, takbelopp, rapporteringsflöde definieras i det stämmobeslut som inrättade utskottet.
- **Ansvar:** Beslut inom utskottets mandat utan att behöva vänta på styrelsens möte. Rapportera till styrelsen via namngiven kontaktperson.
- **Hur stödjs:** En `Committee` har medlemmar, ett mandat (belopp, scope, tidsperiod), en kontaktperson och en rapporteringsskyldighet. Utskotts-beslut går in i samma beslutslogg som styrelsebeslut men märks med utskottets namn.
- **Typiska mönster:** Föräldraförenings eventkommitté, samfällighets underhållsgrupp, kooperativ fixgrupp.
- **Hot att skydda:** Mandatläckage — utskott tar beslut utanför sitt tilldelade scope. Systemet spärrar utläggsgodkännanden över takbeloppet och flaggar beslut utanför scope för styrelsens granskning.

### Utskottets ordförande / sammankallande

- **Stödnivå:** attribut inom ett utskott (`committeeRole: chair` eller `convener`).
- **Ansvar:** Samma som vanlig utskottsmedlem plus ansvaret att kalla till möten och lämna rapporten till styrelsen.
- **`[ÖPPEN]`:** Ska utskotts-ordförandens signering räknas som firmateckning *för utskottsbeslut inom mandatet*? Förslag: ja, inom takbeloppet; över takbeloppet krävs styrelsens firmatecknare.

## Klassrepresentant

- **Stödnivå:** attribut på en medlem (föräldraförenings-specifikt).
- **Lag- och stadge-grund:** Stadgar i vissa föräldraföreningar; tradition i andra. *Ej* styrelsepost.
- **Ansvar:** Kommunikationsansvar mellan föräldragruppen i en specifik klass och styrelsen / skolan. Ingen beslutsmandat.
- **Hur stödjs:** `Membership` har en `classRepresentativeFor: ClassId[]`-flagga. En medlem kan vara klassrepresentant för flera klasser (ovanligt men möjligt när syskon är utspridda).
- **Typiska mönster:** Två per klass (primär + backup) är vanligt. Vald vid klass-föräldramöte, inte vid föreningsstämma — men stadgan kan föreskriva att valet protokollförs hos styrelsen.
- **Edition-avvikelser:** Detta attribut finns bara i föräldraförening-editionen; andra editions har `classRepresentativeFor = null` alltid.
- **Hot att skydda:** Ingen formell jäv-exponering, men klassrepresentanten är ibland kanalen genom vilken föräldrar framför önskemål till styrelsen. Dokumentation av *vad som förmedlats* ligger hos styrelsen via ärendena ([case-types.md](../case-types.md)), inte hos klassrepresentanten.

## Kontaktperson för underfond

- **Stödnivå:** attribut på en ledamot eller medlem, knutet till en specifik `Fund`.
- **Lag- och stadge-grund:** Stadge-bestämd vid fondens inrättande.
- **Ansvar:** Hantera ansökningar mot fonden enligt fondens beslutsgång. Rapportera löpande status till styrelsen.
- **Hur stödjs:** `Fund.contactPersonId` pekar på en `RoleAssignment` eller `Membership`. Kontaktpersonen har skrivrätt på fondens beslutsflöde inom fondens mandat; styrelsen läser och granskar.
- **Typiska mönster:** Skol-FF:s strukturerade eventfond har en kontaktperson, typiskt kassören eller en utsedd ledamot. Samfällighets underhållsfond har oftast styrelsen som kollektiv beslutsfattare — ingen enskild kontaktperson.
- **Hot att skydda:** Konfidentialitet — elevfond-mönstret (*"pengar synliga, mottagare ej"*) fångas av `ConfidentialBudget`-flaggan ([editions/foraldraforening.md#underfonder](../editions/foraldraforening.md#underfonder)). Kontaktpersonen ser mottagarinformation; övriga styrelseledamöter ser bara belopp.

## ExternalAdministrator

- **Stödnivå:** förstklassig (egen rollnyckel, ej attribut).
- **Lag- och stadge-grund:** Avtalstyrd. Stadgarna kan tillåta/förbjuda, men själva rollen är ett externt avtal — inte ett val på stämman.
- **Ansvar:** Sköter delar av ekonomiförvaltningen enligt avtalets scope — bokföring, fakturering, årsredovisning eller hela ekonomin. **Inget beslutsmandat.**
- **Hur stödjs:** Tidsbegränsat `RoleAssignment` med `scope: FINANCIAL_*` och `validUntil: <avtalets slutdatum>`. Läs/skriv mot ekonomiska vyer enligt scope; läs övrigt; skriv inga beslut.
- **Typiska mönster:** Samfällighet med HSB som förvaltare, samfällighet med lokal ekonomibyrå, föräldrakooperativ som outsourcar löneadministration.
- **Edition-avvikelser:** Empiriskt vanligt i samfällighet (Kyrkmossen→HSB, Fröåvägen→CarLe Ekonomi). Förekommer också i skol-FF utan egen kassör. Sällsynt i kooperativ-förskolor (de har oftast egen kassör ur föräldragruppen).
- **Hot att skydda:** Uppdraget *är* skyddsmekanismen i ena riktningen (extern kompetens kompletterar lekmannakassören). I andra riktningen är hotet att styrelsen tappar greppet om ekonomin — systemet måste visa kassörs-vyerna för styrelsen även när `ExternalAdministrator` gör det dagliga arbetet.
- **Redan beskriven i:** [architecture.md](../architecture.md) och [editions/samfallighet.md#extern-ekonomi-förvaltning](../editions/samfallighet.md#extern-ekonomi-förvaltning).

## Medlem (bas-roll)

- **Stödnivå:** förstklassig — alla registrerade i föreningen har denna roll.
- **Lag- och stadge-grund:** LEF 6:3 (rösträtt på stämma), LFS 36§ (sakägare i samfällighet), föreningsstadgar.
- **Ansvar:** Rösta på stämma, följa stadgarna, betala avgifter där relevant.
- **RBAC-kärna:** Rösträtt vid stämma (enligt röstmodell, se [core-concepts.md#röstning](../core-concepts.md#röstning)); motionsrätt ([case-types.md](../case-types.md)); läsrätt på publicerade beslut, stadgar, protokoll och anslagstavla.
- **Terminologi per edition (via i18n):**
  - Samfällighet: **tomtägare** eller **sakägare**
  - Föräldraförening: **vårdnadshavare** (eller **förälder** i vardagligt UI)
  - LEF (annan): **medlem**
- **Livscykel:** `ACTIVE` / `LAPSED` / `EXCLUDED` — se [domain-model.md#medlemskapslivscykel](../domain-model.md#medlemskapslivscykel).
- **Edition-avvikelser:** Se respektive edition-fil. Samfällighet har `PropertyUnit` som bärare; föräldraförening har `Parent`/`Child` som nyckelentiteter; LEF har `Member` direkt.
- **Hot att skydda:** Asymmetrier i förberedelse/uttryck, informella anspråk ([threats.md](../threats.md)) — bas-rollens rättigheter är det som stämman i slutändan försvarar via stadgarna. Systemets transparens-defaults ger medlemmen lika mycket insyn som styrelsen vill medge ([mission.md](../mission.md)).

### Undervarianter — representation

En medlem kan företrädas av någon annan i specifika situationer:

- **`representedBy: Actor`** — dödsbo, omyndig, godman-/förvaltarärende. Stadge-styrt vem som kan representera.
- **Ombud via fullmakt** — per stämma, skriftligt, registreras i röstlängden.
- **Samägd fastighet (samfällighet)** — ägarna enas internt eller splittar röstvikten enligt `UNIFIED`/`PROPORTIONAL` ([core-concepts.md#samägda-fastigheter](../core-concepts.md#samägda-fastigheter)).

Representation är inte en egen roll utan ett tillstånd på medlemskapet.

## Relationsmatris — attributöverlägg

| Attribut | Bärs av | Aktivitetsyta |
|---|---|---|
| Firmatecknare | Styrelseledamot | Signering av föreningens avtal |
| Utskottsledamot | Medlem (ofta styrelseledamot) | Utskottets eget beslutsflöde |
| Utskotts-ordförande | Utskottsledamot | Utskottets kallelse och rapport |
| Klassrepresentant | Medlem (vårdnadshavare) | Klass-gruppens kommunikation |
| Fond-kontaktperson | Ledamot eller medlem | Fondens beslutsflöde |

Attributöverlägg skapas genom tilldelning, inte genom stämmans val — undantag för utskottsledamöter som väljs på stämman men inom utskottets kontext.

## Vad som uttryckligen *inte* finns

Roller som andra plattformar (särskilt Hemmet för BRF) har men som Tillsammans medvetet saknar:

- **`BOARD_PROPERTY_MGR`** — fastighetsansvarig som egen styrelsepost. I Tillsammans är fastighet/anläggning operativt scope ([architecture.md](../architecture.md)) → externt verktyg. Om en förening vill utse en person som särskilt fastighetsansvarig blir det ett utskott eller ett attribut, inte egen rollnyckel.
- **`BOARD_ENVIRONMENT`, `BOARD_EVENTS`** — miljö-/evenemangsansvariga. Samma skäl: operativt.
- **Admin som distinkt roll.** Hemmet har `ADMIN` separat från styrelseroller för teknisk konfiguration. I Tillsammans ligger teknisk konfiguration hos styrelsen själva — det finns ingen extern "föreningsadmin" som inte är förtroendevald.
- **Resident / Boende.** Hemmet skiljer medlem (andelsägare) från boende (kan vara hyresgäst). Tillsammans har inte boende-begreppet — om föreningen har hyresgäster hanteras de utanför governance-scopet.

## Öppna frågor

- **Utskotts-vs-kommitté-terminologi.** Svenska föreningar använder båda; är de synonymer i UI eller ska vi skilja? Förslag: välj en term (*utskott*) för UI-konsekvens, men tillåt föreningen skriva *"Eventkommittén"* som egennamn.
- **Klassrepresentant som egen entitet vs. attribut.** Om framtida edition (idrottsförening) behöver lagrepresentant med liknande mönster — generaliserar vi? Förslag: håll det som attribut tills ett andra användningsfall dyker upp.
- **ExternalAdministrator med flera personer.** HSB skickar typiskt en handläggare men kan byta. Ska `ExternalAdministrator` vara en organisation med tillhörande personer, eller en individuell person? Förslag: organisation med aktiva representanter, så att byte av handläggare inte kräver nytt avtal i systemet.
