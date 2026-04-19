# Kärnkoncept

> Del av [Tillsammans-specifikation](../tillsammans.md).

## Governance-sköld: "Ni lovade"-flödet

Algoritm för att hantera en medlem som hänvisar till informella "löften" som saknar spår i beslutsloggen:

1. Medlem påstår att styrelsen lovat något → styrelsen söker i beslutsloggen → *inget sådant löfte finns*.
2. Styrelsen bemöter inte med egen auktoritet utan hänvisar till systemet: *"vi kan inte agera på rykten, likabehandlingsprincipen"*.
3. Om medlemmen står på sig: **bordlägg till årsstämma** — flytta frågan från styrelserummet till majoritetsbeslut.
4. Systemet bygger automatiskt underlagspaketet: historik, tidigare beslut, styrelsens yttrande, motion från medlemmen.
5. Stämman avgör. Beslutet är svårt att angripa juridiskt om formalian följts.

Nyckelinsikt: styrelsen flyttar ansvaret från personlig konflikt till kollektivt majoritetsbeslut. Detta är den viktigaste "överlevnadsfunktionen" för svenska lekmannastyrelser — UI ska ha en tydlig *Lyft till stämman*-genväg.

## Röstning

**Generella röstmetoder:**

- **Huvudmetod** — en enhet (medlem/fastighet), en röst.
- **Andelsmetod** — viktad efter andelstal för den specifika gemensamhetsanläggningen som frågan rör.
- Valet görs **per beslutstyp** (inte per förening). Vägfråga kan använda väg-GA:s andelstal; stadgeändring kan kräva huvudmetod.

**Defaults per `AssociationType`:**

- **Samfällighet:** **huvudmetod som grundregel enligt SFL 49§** (1 medlem = 1 röst). Vid ekonomisk fråga kan medlem begära andelstalsmetod — då tillämpas `Participation.andelstal` för den frågan, med tak `voteCapPerMember = 0.2` (ingen medlem får rösta med mer än 1/5 av totala röstetalet, SFL 49§). Stadgeändring och andra LFS-tvingade frågor använder alltid huvudmetod (SFL 52§). Se [editions/samfallighet.md#röstmodeller](editions/samfallighet.md#röstmodeller) för empirisk bakgrund (10 föreningar jämförda 2026-04).
- **LEF:** huvudmetod enligt LEF 6:3, andelsmetod endast om stadgarna uttryckligen säger det.

**Gemensamma röststrukturer (gäller alla `AssociationType`):**

- `voteCapPerMember` — tak för andelstal-röster per medlem (default 0.2 för samfällighet enligt SFL 49§; ej satt för övriga). Skyddar mot att enskild stor fastighetsägare dominerar beslutet.
- `maxProxiesPerAgent` — max antal medlemmar ett ombud får företräda (default 1 enligt SFL 49§ för samfällighet; varierar för övriga typer). Fullmakt är alltid skriftlig och registreras före stämman.

### Samfällighet — andelstal per fastighet

Grundenhet är `PropertyUnit` med `Participation.andelstal` per relevant GA.

Vid flera ägare (`Ownership`) stödjer systemet två modeller; stadgarna avgör vilken som gäller per förening (`Association.coOwnershipVotingRule`, ändras endast via stadgeändring):

- **`UNIFIED` (enad-röst, default):** fastigheten avger en samlad röst med fulla andelstalet. Ägarna enas internt om ståndpunkt. Om de inte enas avges ingen röst. Enklare, vanligast i praxis.
- **`PROPORTIONAL` (splittning):** varje `Ownership.sharePercent` splittar andelstalet. Två syskon som äger 50/50 en fastighet med andelstal 0.025 → varje syskon röstar 0.0125. Används när stadgarna explicit föreskriver detta; sällsyntare men verkligt fall.

### Röstlängd — snapshot per fråga

- Genereras vid kallelsetillfället från `ACTIVE`-medlemmar + relevant GA-medlemskap (för samfälligheter).
- Är **fråge-specifik** i samfälligheter — olika GA har olika medlemsomfång (se [Domänmodell](domain-model.md#verkligt-exempel-användarens-egen-samfällighet): vattenföreningen med en exkluderad fastighet).
- Fullmakter registreras före stämman och lagras tillsammans med röstlängden.
- Dödsbon hanteras via Actor-modellens `representedBy`-fält.

### Röstlängdsetablering — processen vid stämmans öppnande

Röstlängden är ett snapshot vid kallelse, men **slutgiltig etablering sker när stämman öppnas** — det är då oklarheter redas ut innan röstning påbörjas:

- **Närvarokontroll** — vilka är faktiskt närvarande (fysiskt/digitalt)?
- **Fullmaktskontroll** — presenterade fullmakter granskas och loggas.
- **Representationsfrågor** — vem röstar för samägd fastighet / dödsbo / juridisk person?
- **Mandatverifiering för juridisk person** — representantens bemyndigande (Bolagsverket-utdrag, fullmakt, styrelseprotokoll, delegationsordning) granskas av mötesordförande eller särskilt utsedd person. Mandat som är för gamla, oklara eller tvistiga kan föranleda att mötesordföranden begär nytt dokument eller att representationen inte godkänns för just denna stämma. Vid *"i förening"*-mandat krävs närvaro av samtliga representanter för att rösten ska kunna avges. Se [medlemskap.md#juridisk-person-som-medlem](medlemskap.md#juridisk-person-som-medlem).

**Principen: Tillsammans löser inga civilrättsliga tvister — föreningen ska kunna komma vidare.** Oklara fall dokumenteras (vem hävdade vad, varför), rösten för den fastigheten avges inte den stämman, parterna hänvisas till rätt forum (domstol, Lantmäteri). Sedan kör stämman vidare på de klara fallen.

Detta är värdefullt: utan denna princip kan en enskild tvist låsa hela stämman. *"Kom vidare"* är ett försvar mot obstruktion.

## Jäv (conflict of interest)

- Obligatorisk jävsdeklaration vid varje ekonomiskt beslut.
- Jäv loggas med kontext: vem, anledning, kvarstående deltagare (before/after).
- Revisorn ser jäv-historik löpande.

## Reservation och solidariskt ansvar

Styrelsens **solidariska ansvar** (LFS 47 §, LEF 8:12) betyder att alla ledamöter formellt är lika ansvariga för beslut. I praktiken känns detta sällan reellt förrän rättsliga processer startar — och då vaknar ledamöter plötsligt upp och söker skydda sitt egenintresse, ofta på kollektivets bekostnad. Ordföranden har oftast burit konflikten ensam dessförinnan.

Det är ett eget sorts hot: passiv ledamot som inte engagerar sig i governance-disciplinen förrän det bränner. Jäv-reglerna fångar inte detta eftersom det inte handlar om intressekoppling — det handlar om upplevelsemässigt avstånd till det kollektiva ansvaret.

**Mekanism: synlig, enkel reservation + personlig exponerings-vy.**

Reservation i protokollet är den juridiska friskrivningen från medansvar för ett specifikt beslut. Den finns redan i lagen — problemet är att den sällan används, eftersom analoga styrelser gör reservation krångligt, socialt tyst och ofta osynligt. När systemet gör reservation **synlig, enkel och kostnadsfri**, vänds logiken: passivt medhåll blir en aktiv handling. *"Jag valde att inte reservera mig → jag står bakom."*

Varje beslut loggar ledamöternas individuella positioner:

- **JA** / **NEJ** / **Nedlagd röst**
- **Reservation med motivering** (den formella friskrivningen)
- **Frånvaro**

Övriga ledamöter och revisorn ser positionerna i loggen → social ansvarighet blir synlig.

**Personlig exponerings-vy.** Varje ledamot har en privat vy som visar sitt eget spår: beslut deltagit i, röster avgivit, reservationer gjort, jäv deklarerat. Inte offentlig — den är personlig och tjänar som påminnelse om att ansvaret är reellt *idag*, inte bara vid kris. Motsvarar frågan: *"Vad skulle en framtida motpart hitta om mig i systemet?"*

**Styrelseonboarding vid nyval.** Nyvald ledamot får en kort introduktion till vad solidariskt ansvar innebär, hur reservation används, och hur den personliga exponerings-vyn hjälper. Syftet är inte att skrämma — det är att göra pliktkänslan tidig och faktisk.

Denna mekanism är inte bara teknisk. Den är **kulturell**: systemet gör skillnaden mellan "flyta med" och "aktivt stå bakom" synlig i varje beslut.

## Bordläggning

- Lågt tröskelvärde — ska kännas som en trygg utväg, inte en eftergift.
- Status `BORDLAGD_TILL_STAMMA` gör ärendet automatiskt till en agendapunkt vid nästa stämma.

## Kallelsemodell (myndighetskrav) `[ÖPPEN scope]`

Lagen om ekonomiska föreningar har tvingande krav på kallelsetider och underlag. Systemet ska:

- Känna till stadgarnas krav som både **min-gräns** (typiskt 2v före stämma, 1v för extra stämma) **och max-gräns** (t.ex. 4-6v). Max-gränsen skyddar inte bara mot för tidig kallelse utan mot "glömska" — kallelse 3 månader i förväg och medlem missar mötet helt. Empiriskt validerat för samfälligheter (Torpa Skogs *"tidigast 4 och senast 2 veckor"*, se [verification-reports/samfalligheter-2026-04.md#mönster-7--kallelsetid](verification-reports/samfalligheter-2026-04.md#mönster-7--kallelsetid)).
- Räkna baklänges från mötesdatum och räkna fram sista dag för kallelse, sista dag för tillgängligt underlag.
- Logga att kallelsen "anslagits" digitalt — bevisvärde om någon hävdar att de inte kallats.

`[ÖPPEN]`: är KU-rapportering (kontrolluppgifter till Skatteverket) och Bolagsverket-flöden del av MVP eller fas 2? Min bedömning: fas 2. MVP fokuserar på governance.

## Anslagstavla

- Rollbaserad moderation (styrelse + medlemmar kan skriva, roller styr vad).
- Syfte: intern kommunikation utan algoritmisk brus — alternativ till föreningens Facebook-grupp.
- Sitter ihop med beslutsloggen: styrelsebeslut som berör alla kan "postas" direkt till tavlan med länk till underlaget.
