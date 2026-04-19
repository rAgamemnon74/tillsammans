# Domänmodell, medlemsmodell och lapptäcket

> Del av [Tillsammans-specifikation](../tillsammans.md).

## Domänmodell

### Förslag: fastighetsorienterad entitetsmodell

```
User ↔ Ownership ↔ PropertyUnit ↔ Participation ↔ GA (Asset) ↔ Association
         (share%)    (andelstal)
```

- `User` — auth, identitet, närvaro, signaturer, jävsdeklarationer
- `PropertyUnit` — fastigheten, bär olika deltagande i olika GA
- `Ownership` — User ↔ PropertyUnit (flera ägare per fastighet, flera fastigheter per person)
- `Association` — föreningen (samfällighet, föräldraförening, LEF)
- `GA` (Asset) — gemensamhetsanläggning (väg, vatten, brygga, allmänning)
- `Participation` — PropertyUnit ↔ GA med andelstal och rättighetstyp

### Verkligt exempel (användarens egen samfällighet)

| Anläggning | Medlemsomfång | Logik |
|---|---|---|
| Vägförening | 100% av tomtägare | Fast andelstal per fastighet |
| Allmänning (basskötsel) | 100% av tomtägare | Alla bidrar till allmännt underhåll |
| Stora bryggan | Delmängd, delrätter | M-N: flera fastigheter delar på en brygga |
| Enskild brygga 4 | 1 fastighet | Exklusiv rätt, ofta baserad på servitut |
| Vattenförening | 99% — en fastighet exkluderad | Särfall måste kunna modelleras som undantag |

Slutsats: `Participation` är central — det är inte en flagga, det är en relation med andelstal, rättighetstyp och ev. referens till källdokument.

## Medlemsmodell per föreningstyp

Medlemsregistret ser fundamentalt olika ut för de tre målgrupperna — formalisnivån och livscykeln skiljer sig.

### Samfälligheter — fastighetsanknutet, informellare än BRF

- Medlemskap *följer fastigheten* — vid försäljning byts medlem automatiskt, ingen prövning (till skillnad från BRF).
- Registret är en **adress- och ägarhistorik**, inte en godkännande-motor.
- Röriga mellantillstånd är normen, inte undantaget: dödsbon som lever i åratal, arvskiften, samägande mellan sambor/syskon, överlåtelser som inte kommit till föreningens kännedom.
- Kassören behöver kunna fakturera rätt part även när formell ägare är oklar — fältet `faktureringsmottagare` måste kunna avvika från `ägare`.
- Uppdatering sker ofta *reaktivt* (vid stämman, vid faktureringsomgång) snarare än realtid.

### Föräldraföreningar — vårdnadshavare, minimal barndata

Datastruktur: `Parent (medlem) → Child → Class → Association`.

- **Medlem = vårdnadshavaren**, inte eleven. Vårdnadshavaren är myndig och avtalsbehörig — eleven är bärare av klasstillhörighet.
- **Minimera barnuppgifter:** förnamn + klassbeteckning räcker. Inget personnummer, ingen födelsedatum, ingen övrig data. GDPR-principen "dataminimering" är särskilt viktig för minderåriga.
- `Class` kan bära sin egen klasskassa som delmängd av föreningens ekonomi — kassören ser pott per klass.
- **Onboarding via självregistrering**, inte integration med skolregister:
  - Skolan är personuppgiftsansvarig för elevdata och har varken rätt eller praktisk möjlighet att dela register.
  - Skolans system (Infomentor, SchoolSoft, Vklass, Unikum) är fragmenterade utan tredje parts-API.
  - Rektorns roll är på sin höjd att distribuera registreringslänk — inte att skicka data.
  - Länk + BankID → vårdnadshavaren lägger till sina barn själv.

### Övriga LEF — formell medlemsprövning

- Styrelsen prövar inträde enligt stadgarna (LEF 4:2).
- Formell in/ut-logg med datum, skäl.
- Närmast BRF-modellen i struktur, men utan lägenhetskopplingen.

## Medlemskapslivscykel — avslut och tvister

Föreningar utan automatisk avregistrerings-trigger (främst föräldraföreningar, men även samfälligheter vid ägarbyten som ej rapporterats) möter ett gemensamt problem: **ex-medlemmar som fortsätter hävda medlemsrätt**. Föräldrar vars barn lämnat skolan, ex-ägare som flyttat, dödsbon vars arvskifte genomförts — alla kan ligga kvar i registret i åratal om systemet bygger på passivt medlemskap.

Strukturell lösning: **medlemskap är aktivt, inte passivt.**

### Statusmodell

- `ACTIVE` — full rösträtt, insyn och tillgång.
- `LAPSED` — mjukt upphörande. Ingen rösträtt, ingen tillgång till aktuell data. Kan reaktiveras via ny bekräftelse (om kriterierna uppfylls).
- `EXCLUDED` — hårt upphörande via styrelsebeslut med dokumenterat skäl. Bordläggs till stämman om medlemmen bestrider. Reaktivering kräver nytt stämmobeslut.

### Triggers för `LAPSED`-övergång

Tre alternativa mekanismer stöds — föreningen väljer en (eller kombinerar) via stadgebeslut:

**1. Årlig reconfirmation (tidsbaserat).** Medlemskap löper 1 år i taget och kräver aktiv bekräftelse av medlemskapskriteriet.

- **Föräldraförening:** *"Jag är vårdnadshavare till [barn] i klass [Y] på [skola] per [datum]."*
- **Samfällighet:** *"Jag är ägare till fastighet [X] per [datum]."* (eller dödsbo-representant, se Actor-modell).
- **LEF:** enligt stadgarnas medlemskriterium.

Uteblir bekräftelsen → automatisk övergång till `LAPSED`.

**2. Extern händelse (fakta-baserat).** `LAPSED` triggas när ett externt faktum ändras — t.ex. barnet ej längre inskriven i skolan, fastigheten såld enligt ägarbytesrapport. Empiriskt mönster: **Backatorpsskolans FF §2** (*"Medlemskapet upphör då medlemmen ej längre är vårdnadshavare till barn som är inskrivna på skolan"*). Enklare än årlig reconfirmation — ingen blankett behövs — men förutsätter att det externa faktumet är synligt för systemet (manuell uppdatering per termin eller integration).

**3. Utebliven avgift (betalnings-baserat).** För föreningar med årsavgift (Nya Elementar-mönstret). Utebliven betalning vid påminnelse → `LAPSED`. Avgiften fungerar de facto som årlig aktivering.

Föreningar kombinerar ofta: t.ex. extern händelse (automatisk avregistrering när barnet slutar) + reconfirmation vid varje ägarbyte. Se [editions/foraldraforening.md#val-av-trigger-för-lapsed](editions/foraldraforening.md#val-av-trigger-för-lapsed).

Bevisbördan flyttas oavsett trigger: ingen behöver bevisa att någon *inte* är medlem. Medlemmen — eller det externa faktumet — bevisar att de *är*.

### Tvister — "Jag är fortfarande medlem"

Samma mönster som "Ni lovade"-flödet:

1. Styrelsen slår upp i registret — person är `LAPSED` eller `EXCLUDED`.
2. Hänvisar till stadgarna och reconfirmation-policy, inte till egen auktoritet.
3. Om tvistigt: bordlägg till stämman — kollektivt majoritetsbeslut.
4. Röstlängden för den stämman genereras från `ACTIVE`-snapshot vid kallelsetillfället. Personen röstar inte om sitt eget medlemskap.

### Historisk insyn bevaras

- Ex-medlem får fortsatt läsa protokoll, beslut och ekonomi *för den period de var medlem*.
- Transparens bakåt avväpnar "ni gömmer information för mig"-argumentet utan att ge framåtriktat inflytande.
- GDPR-balans: personliga uppgifter om andra nuvarande medlemmar döljs enligt samma regler som gäller aktiva medlemmar.

### Ärlig begränsning

Systemet kan inte verifiera reconfirmation mot externa register (skol-integration är avvisad, fastighetsregister är opålitligt). En moraliskt flexibel medlem kan ljuga i sin bekräftelse. Men:

- Lögnen är **signerad och tidsstämplad**.
- När den avslöjas (andra medlemmar påpekar, annan kontext) har styrelsen dokumenterat underlag för `EXCLUDED` — och eventuellt för civilrättsliga åtgärder om ekonomisk skada uppstått.
- Kostnaden för att ljuga blir reell istället för gratis.

### `[ÖPPEN]` Actor-modell vs. User

Dödsbon och företag som äger fastighet är verkliga fall. Två vägar:

- **MVP-pragmatiskt:** User = auth-identitet. `Ownership` har ett fritextfält `ownerLabel` ("Dödsboet efter Karl Svensson", "AB Exempel") och ett `representedBy: User` om juridisk person/dödsbo har utsedd företrädare.
- **Fullt abstraherat:** `Actor` = `Individual | LegalEntity | Estate`, kopplas till `Holding` i `Property`. Renare men mer komplexitet upfront.

Förslag: starta pragmatiskt, abstrahera när verkliga behov tvingar fram det.

## Lapptäcket — etablering och dokumentkonflikter

När en förening onboardas kliver de in i en juridisk arkeologi: Lantmäteridata, gåvobrev, domar, servitut, kommundata, "urminnes hävd". Dessa källor kan motsäga varandra.

**Designprinciper:**

1. **Ingen automatisk import från officiella register.** Data i fastighetsregistret är ofta felaktig för äldre samfälligheter (`s:1`, `s:2` etc.). Föreningen bygger själv upp sin digitala tvilling.
2. **Flera källor kan existera parallellt.** Systemet representerar *vad olika dokument säger*, inte *vad som är sant*. Sanningen framkommer av att styrelsen/revisorn/stämman väger källorna.
3. **Konflikt-flagg.** När uppgifter från olika källor motsäger varandra (t.ex. andelstal summerar inte till 1.0, eller servitut i gåvobrev saknas i förrättning) flaggas ärendet för styrelsens genomgång.
4. **Hävd hanteras som bevisbarhet, inte som sanning.** Medlemmar kan ladda upp foton, brev, vittnesmål. Systemet validerar inte — det exponerar.
