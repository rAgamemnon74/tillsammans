# Revisionsroller

> Del av [Tillsammans-specifikation](../../tillsammans.md).

Revisorn är föreningens formella granskningsorgan — vald av stämman, **fristående från styrelsen**, med uppdraget att granska styrelsens förvaltning och räkenskaper för det gångna räkenskapsåret. Rollen är lagstyrd (LEF 8, LFS 47§, revisorslagen för auktoriserad revisor) och får inte sammanblandas med styrelseroller.

En revisor kan per definition inte vara styrelseledamot i samma förening — det skulle göra granskningen meningslös. Systemet hindrar dubbel-tilldelning styrelse/revisor.

**Relaterade dokument:**
- [board-roles.md](board-roles.md) — styrelserollerna som revisorn granskar
- [nominating-committee.md](nominating-committee.md) — det andra fristående stämma-valda organet
- [case-types.md](../case-types.md) — revisorsfråga och revisionsberättelse som ärendetyper
- [core-concepts.md](../core-concepts.md) — jäv, reservation, solidariskt ansvar

## Principer

1. **Oberoende är konstitutionellt.** Revisorn får inte vara styrelseledamot, anställd av föreningen eller närstående till styrelsen på sätt som påverkar opartiskheten. Systemet hindrar dubbel-tilldelning styrelse/revisor och ber om aktiv deklaration av kända närståendesamband.
2. **Löpande läsrätt, inte punkt-granskning.** Revisorn ska kunna följa räkenskapsår och styrelsearbete i realtid — inte bara granska i efterhand. Systemet ger revisorn permanent läsrätt från dag ett av räkenskapsåret.
3. **Ingen skrivrätt utöver egna dokument.** Revisorn skriver i sina egna entiteter (revisorsfrågor, revisionsberättelse) — aldrig i styrelsens beslut, protokoll eller ekonomidata. Observationer görs synliga via egen kanal, inte via datamanipulation.
4. **Uppdraget *är* skyddsmekanismen.** Att göra revisorns arbete möjligt utan att den behöver be om åtkomst är själva värdet. Om revisorn måste eskalera för att få se något — då har systemet misslyckats.

## Revisor

- **Stödnivå:** primär.
- **Lag- och stadge-grund:** LEF 8 (ekonomiska föreningar), LFS 47§ (samfälligheter), revisorslagen (för auktoriserad revisor).
- **Ansvar:** Granska räkenskaper och styrelsens förvaltning för det gångna räkenskapsåret. Avlämna revisionsberättelse till stämman med till- eller avstyrkan av ansvarsfrihet för styrelsen.
- **RBAC-kärna:** Permanent läsrätt — all ekonomi, alla styrelsebeslut, alla jävsdeklarationer, hela audit-trail, alla utläggsärenden, dokumentarkivet, kallelsematerial. Ingen skrivrätt utöver egna revisorsfrågor och revisionsberättelse ([case-types.md](../case-types.md)).
- **Edition-avvikelser:**
  - **Samfällighet:** lekmanna-revisor dominerar (0 auktoriserade i 10 observerade, se [verification-reports/samfalligheter-2026-04.md](../verification-reports/samfalligheter-2026-04.md)).
  - **LEF:** auktoriserad revisor kan krävas över vissa tröskelvärden för omsättning, tillgångar eller anställda.
  - **Föräldraförening:** typiskt lekmanna, ofta en förälder som inte är styrelseledamot. Backatorp §9 föreskriver *"fritt tillträde till föreningens protokoll, korrespondens och räkenskaper närhelst de så påkallar"* — systemet förverkligar det som default.
- **Hot att skydda:** Uppdraget *är* skyddsmekanismen. Systemet får inte skapa friktion som skrymmer granskningen; data ska finnas där löpande.

## Revisorssuppleant

- **Stödnivå:** primär.
- **Lag- och stadge-grund:** Föreningsstadgar; LEF 8 där stadgan föreskriver.
- **Ansvar:** Ersätter ordinarie revisor vid förhinder eller frånvaro.
- **RBAC-kärna:** Samma som revisor — läsrätten finns löpande så att suppleanten kan vara informerad när rollen aktiveras. Revisionsberättelsen skrivs av den som faktiskt agerar revisor under räkenskapsårets avslutande.
- **Edition-avvikelser:** Inga strukturella. Föreningar med auktoriserad revisor har oftast ingen suppleant eftersom revisionsbyrån själv hanterar resurssäkringen.
- **Hot att skydda:** Kontinuitet vid revisorsbyte eller frånvaro. Att suppleantens läsrätt är löpande gör övergången gnisselfri.

## Revisorns arbetsyta

Revisorn har en egen vy i systemet, åtskild från styrelsens arbetsyta:

- **Översikt** — räkenskapsåret, kritiska datum (kallelse, stämma, revisionsberättelsens deadline).
- **Beslutslogg** — kronologisk + sökbar per stadgaparagraf och per ekonomisk konsekvens.
- **Ekonomi** — kassaflöde, utläggsärenden, underfonder, debiteringslängd (samfällighet).
- **Jäv-historik** — anonymiserad i medlemmens publika vy men fullständig för revisorn, eftersom granskningen kräver namn.
- **Revisorsfrågor** — egen inkorg där styrelsen svarar; all kommunikation loggad.
- **Revisionsberättelse-utkast** — arbetsyta för utkast; slutversion signeras och distribueras inför stämman.

## Byte av revisor mid-år

Stämman kan entlediga en revisor och välja en ny (LEF 8:9). Systemet hanterar övergången:

- Den entledigade revisorns *skrivrätt* upphör vid beslutstillfället.
- Den nya revisorns rättigheter aktiveras från entledigande-datumet.
- **Den entledigade revisorns läsrätt på data från sin mandatperiod kvarstår** eftersom den kan behöva svara på stämmans frågor. Teknisk detalj: `RoleAssignment.readAccessEndsAt` skiljs från `RoleAssignment.mandateEndsAt`.

## Öppna frågor

- **Närståendekontroller.** Systemet vet inte automatiskt om en revisor är släkt eller nära affärspartner till en styrelseledamot. Förslag: begär aktiv deklaration vid revisors-tilldelning, jämförbart med jävsdeklaration.
- **Auktoriserad revisor och revisionsbyrå.** När en auktoriserad revisor representerar en byrå blir rollen knuten till byrån, inte enbart individen — handläggare kan byta. Modelleras som `ExternalAudit`-organisation med aktiva representanter, analogt med `ExternalAdministrator` ([other-roles.md#externaladministrator](other-roles.md#externaladministrator)). `[ÖPPEN]`.
- **Revisionsberättelse-signering.** BankID-krav för signering? Kopplas till [open-questions.md#öppen-autentisering](../open-questions.md).
- **Revisor från annan förening.** Flera skol-FF har revisor från moderföreningen eller kooperativet. Hur modelleras revisor som tillhör en extern entitet? Förslag: tillåt extern revisor men kräv att kontaktuppgifter finns i systemet för kallelse-flöden.
