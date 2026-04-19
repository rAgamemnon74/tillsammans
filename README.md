# Tillsammans

**Open source-plattform för föreningsdemokrati i Sverige** — samfälligheter, ekonomiska föreningar och föräldraföreningar.

Kärnfokus: *governance-sköld*. Tillsammans kodifierar demokratiska processer så att enskilda påverkansförsök, informella överenskommelser och asymmetrier mellan medlem och styrelse inte kan leda till beslut som avviker från majoritet och stadgar. Försäljningsargumentet är **beslutstrygghet** — inte fastighetsadministration, inte bokningssystem, inte bokföring.

## Status

**Specifikationsfas.** Ingen kod är skriven ännu. Hela specifikationen ligger under [`spec/`](spec/) och är levande dokument — öppna frågor är markerade `[ÖPPEN]` där de finns.

Ingångspunkten är [`tillsammans.md`](tillsammans.md) — innehållsregister med kort beskrivning av varje sektion.

## Varför ännu en föreningsplattform?

Den svenska föreningsmarknaden är delvis täckt (HSB/Riksbyggen för BRF, SportAdmin för idrottsklubbar), men tre stora kategorier saknar governance-bärigt stöd:

- **Samfällighetsföreningar** — gemensamhetsanläggningar med andelstal, anläggningslagen, ofta utsatta för "granne-politik" kring vägar och bryggor.
- **Föräldraföreningar** — klasskassor och skol-fonder där kassören bär ekonomisk misstro ensam.
- **Övriga ekonomiska föreningar** — LEF-styrda föreningar med compliance-krav men utan specialiserat verktyg.

Plattformen är uttryckligen **typ-agnostisk i kärnan** och **typ-anpassad i terminologi, defaults och onboarding**. Samma kod, tre editions: *Tillsammans för samfälligheter*, *Tillsammans för föräldraföreningar*, *Tillsammans för LEF*.

BRF är inte målgruppen — systerprojektet [Hemmet](https://github.com/rAgamemnon74/hemmet) täcker den domänen.

## Governance-principer

- **Stadgeförankring.** Varje beslut, varje utbetalning, varje ärende är kopplat till en stadgaparagraf. Kassören kan svara på *"finns ett giltigt beslut som täcker denna utgift, och ligger den inom stadgarnas ram?"*.
- **Transparens som default.** Beslutslogg, stadge-index, stämmoresultat och (anonymiserad) jäv-historik har publika läsvyer inom GDPR-ramarna. Medlemmen ser *hur* beslutet togs, inte bara utfallet.
- **Open source som granskningsbart löfte.** *"Läs koden om ni inte tror oss"* är ett starkare försvar än någon proprietär plattform kan erbjuda. Ingen central aktör kan bakvägen justera logiken utan att det syns.
- **Solidariskt ansvar synligt.** Varje ledamots individuella position (ja/nej/nedlagd/reservation) loggas per beslut. Passivt medhåll blir en aktiv handling.
- **God och samarbetsvillig ton i mallar.** Textbiblioteket för ärendetyper är konstruerat för att dämpa konflikt — saklig, konstruktiv, stadgehänvisande.

## Avgränsning

Tillsammans bygger **bara governance-kärnan**. Operativa behov skiljer sig för mycket per förening och löses med befintliga verktyg:

| Inom scope | Utanför scope |
|---|---|
| Möten, dagordning, protokoll | Bokningssystem (bastu, släp, lokal) |
| Beslutslogg, jäv, bordläggning | Mätarinsamling, vattenavläsning |
| Stadge-modul + beslut↔paragraf | Leverantörshantering, upphandling |
| Kassörsstöd (beslut→utbetalning→export) | Bokföring (SIE-export till externa system) |
| Medlemsregister, röstlängd, kallelser | Swish/betalningsinfrastruktur |
| Revisorsstöd, valberedning, audit-trail | Event, grupp-SMS, föreningens Facebook |
| Utskott, underfonder, huvudmanna-relation | Inventarielistor, utlåning |

## Editioner

| Edition | Primärrisk | Mest exponerad roll | Status |
|---|---|---|---|
| [Samfälligheter](spec/editions/samfallighet.md) | Governance-attacker (bryggkrig, vägpolitik) | Ordförande + kassör parallellt | Verifierad apr 2026 (10 föreningar) |
| [Föräldraföreningar](spec/editions/foraldraforening.md) | Ekonomisk misstro (klasskassa) | Kassör | Verifierad apr 2026 (12 föreningar) |
| Övriga ekonomiska föreningar (LEF) | Varierar | Beror på förening | Verifikation pågår |

## Teknik

**Specifikationen går före tekniken.** Stack väljs när domänmodellen är stabil — nuvarande riktning (apr 2026):

- **Distribution:** standalone single-binary som MVP. Styrelsemedlem startar verktyget lokalt, databasfil i samma mapp, revisor får kopia för egen granskning. Senare fas: iframe-embeds mot befintliga CMS, därefter hostad multi-tenant.
- **Databas:** Postgres som default; PGlite (Postgres→wasm) inbäddad i single-binary-läget — samma SQL-dialekt hela vägen.
- **Språk:** Svensk UI, svenska URL-sökvägar, all kod på engelska.

Exakta val (Go vs Rust/Tauri, frontend-ramverk) är öppna och diskuteras i [`spec/open-questions.md`](spec/open-questions.md).

## Läs specen

Börja med [`tillsammans.md`](tillsammans.md) — ladda sedan bara den sektion som är relevant för din fråga:

- Förstå uppdraget: [`spec/mission.md`](spec/mission.md)
- Vilka hot försvarar plattformen mot: [`spec/threats.md`](spec/threats.md)
- Arkitektur och scope: [`spec/architecture.md`](spec/architecture.md)
- Röstning, jäv, bordläggning: [`spec/core-concepts.md`](spec/core-concepts.md)
- Motioner, ärendeflöden, textmallar: [`spec/case-types.md`](spec/case-types.md)
- Domänmodell per föreningstyp: [`spec/domain-model.md`](spec/domain-model.md)
- Empirisk bakgrund: [`spec/verification-reports/`](spec/verification-reports/)

## Bidra

Projektet välkomnar bidrag inom flera ytor:

- **Specifikationskritik och juridisk granskning** — advokater, revisorer, lantmätare och aktiva föreningsfunktionärer som läser mot egen erfarenhet.
- **Verifikationsrapporter** — nya föreningstyper eller återverifikation av befintliga editions mot nya föreningar.
- **Stadgemallar** — Lantmäteriets normalstadgar och andra etablerade mallar som parametriserade data.
- **Textbibliotek** — välformulerade ärendemallar som bygger upp en god ton från start.
- **Kod** — när specifikationen är stabil nog för att påbörja implementation.

Fullständig bidragsguide kommer när community-sektionen i `mission.md` är färdig.

## Licens

MIT. Samma som [Hemmet](https://github.com/rAgamemnon74/hemmet). Öppen källkod är inte hygien här — det är själva governance-löftet.
