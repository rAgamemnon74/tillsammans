# Styrelseroller

> Del av [Tillsammans-specifikation](../../tillsammans.md).

Styrelsen är föreningens verkställande organ — vald av stämman för att driva den dagliga governance-uppgiften mellan stämmorna. De roller som utgör styrelsen beskrivs här.

**Fristående stämma-valda organ** (revision och valberedning) är **inte** styrelseroller utan ligger i egna filer — se [oversight-roles.md](oversight-roles.md) och [nominating-committee.md](nominating-committee.md). Deras oberoende från styrelsen är konstitutionellt: en revisor som är styrelseledamot kan inte granska styrelsen meningsfullt, och en valberedningsledamot som föreslår sig själv till styrelsen nominerar i praktiken sig själv.

**Relaterade dokument:**
- [oversight-roles.md](oversight-roles.md) — revisor och revisorssuppleant (granskningsorgan)
- [nominating-committee.md](nominating-committee.md) — valberedning (beredningsorgan)
- [meeting-roles.md](meeting-roles.md) — flyktigt tillsatta mötesroller
- [other-roles.md](other-roles.md) — firmatecknare, utskottsledamot, ExternalAdministrator, bas-medlem
- [case-types.md](../case-types.md) — ärendetypernas livscykel refererar dessa roller i RBAC-tabeller
- [core-concepts.md](../core-concepts.md) — reservation och solidariskt ansvar gäller alla styrelseledamöter

## Principer

1. **Lag och stadga styr — inte teknik.** En roll är primär om lag eller stadgar behandlar den som distinkt (eget ansvar, egen jävsläge, egen signeringsrätt). Annars ligger den som attribut på en annan roll.
2. **Typ-agnostisk kärna.** Rollnycklarna är identiska mellan `AssociationType`. Skillnader i vardag (debiteringslängd, servitut-hantering, insats-administration) sker via data och i18n — inte via typ-specifika rollnycklar. Brott mot detta är ett arkitekturläckage; se [architecture.md](../architecture.md).
3. **RBAC är minimalt explicit.** Varje roll har en kärn-permission-uppsättning som speglar dess lagstyrda ansvar. Finare behörigheter byggs av sammansättningar — inte av fler roller.
4. **Solidariskt ansvar är synligt per ledamot.** Varje beslut loggar individuell position (ja/nej/nedlagd/reservation/frånvarande). Se [core-concepts.md#reservation-och-solidariskt-ansvar](../core-concepts.md#reservation-och-solidariskt-ansvar).
5. **Vakanser är synliga.** Att en roll är vakant ska vara synligt i styrelsevyn, inte gömt.
6. **Mandatperiod är data, inte hårdkodat.** En roll kan väljas på 1 år, 2 år, asymmetriskt eller rullande. Roll-definitionen säger inte *hur länge*; stadgemodulen gör det.

## Per-roll-struktur

Varje roll beskrivs med:
- **Stödnivå:** primär / attribut / ej stödd
- **Lag- och stadge-grund:** vilka lagrum och stadge-paragrafer som namnger rollen
- **Ansvar:** kärnuppdraget
- **RBAC-kärna:** vad rollen får göra i systemet, minimalistiskt formulerat
- **Edition-avvikelser:** typ-specifika nyanser (utan att bryta agnostik i nyckeln)
- **Hot att skydda:** från [threats.md](../threats.md) — vad rollen särskilt exponeras för

## Ordförande

- **Stödnivå:** primär.
- **Lag- och stadge-grund:** LEF 6:17, LFS 36§, föreningsstadgar. Nämns alltid explicit.
- **Ansvar:** Leder styrelsen, kallar till möten, håller ordning på dagordningen, är oftast firmatecknare (se [other-roles.md#firmatecknare](other-roles.md)), representerar föreningen externt, har utslagsröst vid lika röstetal om stadgan så föreskriver.
- **RBAC-kärna:** Kalla till styrelsemöte; godkänna dagordning; flytta ärenden till bordläggning eller stämma; signera kallelser; bekräfta protokoll efter justerare; utöva utslagsröst vid lika röstetal; se alla styrelsens arbetsytor.
- **Edition-avvikelser:** I samfällighet primärrollen mot sakägartvister ([audience.md](../audience.md)). I LEF varierar; sätts vid onboarding.
- **Hot att skydda:** Asymmetrier i förberedelse/uttryck ([threats.md](../threats.md)), informella anspråk och påtryckningar mellan möten, solidariskt ansvar som aktualiseras vid rättsprocess.

## Vice ordförande

- **Stödnivå:** primär. `[ÖPPEN]` möjlig degradering till attribut på ledamot — se nedan.
- **Lag- och stadge-grund:** Stadgar namnger den explicit i de flesta observerade föreningar.
- **Ansvar:** Ersätter ordförande vid frånvaro. I vissa föreningar permanent signeringspart jämte ordförande.
- **RBAC-kärna:** Läs allt som ordförande ser löpande. Skriv-rättigheter aktiveras när ordförande är frånvarande eller jävig — systemet markerar "agerar som ordförande" i audit-trail.
- **Edition-avvikelser:** Inga strukturella skillnader.
- **Hot att skydda:** Samma som ordförande när den agerar som sådan.
- **`[ÖPPEN]`:** Är vice ordförande en egen rollnyckel eller ett attribut *"deputy: true"* på en `BOARD_MEMBER`? Argument för egen: stadgar namnger rollen, signeringsläge skiftar mellan de två. Argument för attribut: i mindre föreningar är det bara en internt utpekad ledamot. Förslag: egen rollnyckel, eftersom jäv-kedjan (vad händer om både ordf och vice ordf är jäviga?) behöver kunna skilja dem.

## Sekreterare

- **Stödnivå:** primär.
- **Lag- och stadge-grund:** Föreningsstadgar.
- **Ansvar:** Protokollföring vid styrelsemöten, hantera kallelseutskick för styrelsen, arkivhantering.
- **RBAC-kärna:** Skapa och redigera protokollutkast; publicera protokoll när justerare godkänt; hantera kallelseutskick; skriva i dokumentarkivet; se styrelsens arbetsyta fullt ut.
- **Edition-avvikelser:** Inga strukturella. Workflow kan skifta om föreningen anlitar `ExternalAdministrator` för kallelsehantering.
- **Hot att skydda:** Historierevision — protokoll är det primära bevisvärdet över tid; en sekreterare med slarviga rutiner öppnar erosions-ytan ([threats.md](../threats.md)).

## Kassör

- **Stödnivå:** primär.
- **Lag- och stadge-grund:** Bokföringslagen (där föreningen är bokföringsskyldig), föreningsstadgar.
- **Ansvar:** Ekonomiförvaltning, kopplingen beslut → utbetalning, bokföringsunderlag, debiteringslängd (samfällighet — lagkrav enligt LFS).
- **RBAC-kärna:** Godkänna utläggsärenden ([case-types.md](../case-types.md)); se alla beslut med ekonomisk konsekvens; generera debiteringslängd / budget-rapporter; exportera till bokföringssystem; skriva i kassaflödesvyn; läs alla styrelsebeslut.
- **Edition-avvikelser:** I samfällighet bär kassören den juridiskt snåriga debiteringslängden ([editions/samfallighet.md](../editions/samfallighet.md)). I LEF är bokföringslagens krav mest uttalade.
- **Hot att skydda:** Juridisk komplexitet vid debiteringslängden (samfällighet), jäv vid utläggsgodkännande till egen räkning. Kassör kan inte godkänna sitt eget utlägg — se analys-reglerna ([analysis-rules.md](../analysis-rules.md) grupp 5).

## Ledamot

- **Stödnivå:** primär.
- **Lag- och stadge-grund:** LEF 6, LFS 36§. Generisk styrelseledamot utan särskild tilldelad post.
- **Ansvar:** Delta i styrelsebeslut, solidariskt ansvar (LFS 47§, LEF 8:12).
- **RBAC-kärna:** Rösträtt vid styrelsemöten; reservationsrätt med motivering ([core-concepts.md](../core-concepts.md)); läs hela styrelsens arbetsyta; skapa motion-yttranden och deltaga i beredning.
- **Edition-avvikelser:** Inga strukturella. Mandatperiod varierar per stadga (1 år, 2 år, rullande).
- **Hot att skydda:** Passivt medhåll — att flyta med utan att aktivt reservera sig. Systemets personliga exponerings-vy ([core-concepts.md#reservation-och-solidariskt-ansvar](../core-concepts.md#reservation-och-solidariskt-ansvar)) gör det synligt för ledamoten själv.

## Suppleant

- **Stödnivå:** primär.
- **Lag- och stadge-grund:** Föreningsstadgar; LEF 6:14 (om stadgarna tillåter).
- **Ansvar:** Ersätter ordinarie ledamot vid frånvaro. Närvaro- och yttranderätt kan ges i stadgarna även utan rösträtt.
- **RBAC-kärna:** Läsrätt löpande på styrelsens arbetsyta. Rösträtt aktiveras automatiskt när ordinarie är frånvarande eller jävig; systemet markerar "agerar som ordinarie X" i audit-trail.
- **Edition-avvikelser:** Föreningar skiljer sig i om suppleanten alltid får yttra sig eller bara vid ersättning. Stadge-inställning.
- **Hot att skydda:** Samma som ordinarie när den agerar som sådan.

## Vakansvy

Varje styrelseroll har ett vakansläge: *"rollen existerar i stadgan men är obesatt"*. Styrelsevyn visar alla roller stadgan kräver och vilka som är vakanta — inte bara dem som någon innehar. Detta är förutsättningen för att valberedningen (se [nominating-committee.md](nominating-committee.md)) och stämman ska kunna se obemannade poster.

Vakans är inte samma sak som "roll existerar inte". Om stadgan inte föreskriver en vice ordförande ska rollen inte ens synas. Om den föreskrivs och ingen är vald → synlig som vakans.

## RBAC — sammanfattning

| Roll | Styrelsemöte (röst) | Ekonomi (skriv) | Stämma-kallelse | Protokoll (skriv) | Audit (läs) |
|---|---|---|---|---|---|
| Ordförande | ✓ | läs | ✓ | bekräftar | ✓ |
| Vice ordförande | ✓ | läs | aktiveras vid frånvaro | aktiveras vid frånvaro | ✓ |
| Sekreterare | ✓ | läs | verkställer | ✓ | ✓ |
| Kassör | ✓ | ✓ | läs | läs | ✓ |
| Ledamot | ✓ | läs | läs | läs | ✓ |
| Suppleant | aktiveras vid frånvaro | läs | läs | läs | ✓ |

Denna tabell är en orientering, inte en fullständig permission-modell. Kolumn-detaljer framgår av respektive rolls kärn-RBAC ovan.

## Vad som *inte* är en styrelseroll

- **Revisor, revisorssuppleant** → [oversight-roles.md](oversight-roles.md). Fristående granskningsorgan; kan per definition inte vara styrelseledamot.
- **Valberedningsledamot, valberedningens sammankallande** → [nominating-committee.md](nominating-committee.md). Fristående beredningsorgan; stämma-valda men organisatoriskt åtskilda från styrelsen.
- **Mötesordförande, mötessekreterare, protokolljusterare, rösträknare** → [meeting-roles.md](meeting-roles.md). Väljs per stämma och kan sammanfalla med styrelseroller men är formellt separata.
- **Firmatecknare** → attribut på en eller flera styrelseledamöter; [other-roles.md#firmatecknare](other-roles.md#firmatecknare). Inte egen post även om stadgan säger *"firma tecknas av ordförande och kassör var för sig"*.
- **Utskotts-/kommittéordförande** → attribut inom ett utskott; [other-roles.md#utskotts-och-kommittéledamot](other-roles.md#utskotts-och-kommittéledamot).

## Läsrätt efter mandatet

Tidigare styrelseledamöter behåller **permanent läsrätt** till de epoker de var aktiva under — deras juridiska ansvar sträcker sig bortom mandatet (klandertalan, skadeståndsfråga, upphävd ansvarsfrihet kan aktualiseras år senare). Den personliga exponerings-vyn (egna röster, reservationer, jävsdeklarationer) kvarstår permanent som juridisk dokumentation.

Principen är tvärgående över alla förtroendevalda roller och beskrivs i [föreningen.md#historisk-läsrätt-efter-mandatet](../föreningen.md#historisk-läsrätt-efter-mandatet).

## Öppna frågor

Utöver de `[ÖPPEN]` som markerats per roll:

- **Vice-typer vid frånvaro** — ska systemet föreslå suppleant/vice automatiskt när ordinarie markeras frånvarande, eller ska aktivering vara manuell? Förslag: manuell aktivering loggad i protokollet, så att *"har suppleanten röstat"* alltid är ett explicit beslut.
- **Flera roller på samma person** — en kassör kan också vara ledamot (är det automatiskt?); en styrelseledamot kan även vara firmatecknare. Ska systemet tillåta rollsammanslagning (en person, flera rolltillstånd)? Förslag: roller är distinkta tilldelningar; en person har uppsättningen `{kassör, ledamot}` och rösträtt från *ledamot*-rollen, inte dubbel-röst.
- **Mandatperiod som data** — är start- och slutdatum per roll eller per person-i-roll? Förslag: per tilldelning (`RoleAssignment` med `startAt`/`endAt`), där rollen själv är tidlös.
- **Kombinationsförbud mellan styrelse och granskning/valberedning** — systemet ska förhindra att samma person tilldelas både styrelseledamot och revisor samtidigt (konstitutionellt oförenligt). Valberedning är mjukare — tekniskt tillåtet men flaggas vid tilldelning; självnominering flaggas vid förslagspublicering. Se [oversight-roles.md#principer](oversight-roles.md) och [nominating-committee.md#principer](nominating-committee.md).
