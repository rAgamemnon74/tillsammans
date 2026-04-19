# Målgrupp

> Del av [Tillsammans-specifikation](../tillsammans.md).

Tillsammans är **rollstöd för förtroendevalda** i svenska föreningar. Den gemensamma nämnaren är *rollerna* — ordförande, kassör, sekreterare, revisor, valberedning — inte föreningstypen. Domänen (samfällighet / föräldraförening / LEF) är en konfiguration ovanpå en gemensam governance-kärna.

**Primärt:**

- **Samfällighetsföreningar** — gemensamhetsanläggningar (GA), andelstal, anläggningslagen. Förvaltar väg, vatten, bryggor, allmänningar. Primärt objekt: **fastigheten**. Starkt fit: unik andelsmatematik + stöd vid sakägartvister med asymmetriska resurser.
- **Föräldraföreningar** — klasskassor, skolfond-insamlingar. Starkt fit för *kassörsrollen*: stadgar och styrelsebeslut ligger till grund för utbetalningar. Ordförande och kassör är de primära användarna — inte medlemmen. Insynen här är inte skydd mot extern påtryckning utan hanterar den interna ekonomiska misstron — skyddar kassören och lugnar föräldragruppen.
- **Övriga ekonomiska föreningar** — lagen om ekonomiska föreningar. Samma rollstöd; compliance (KU, Bolagsverket) blir viktigare här.

**Uttryckligen bortvalt:**

- **BRF** — Hemmets domän.
- **Idrottsklubbar** — mättad marknad (SportAdmin, Svenska Lag). LOK-stöd, matchschema och närvaroregistrering är inte vår kamp.

## Primärexponerad roll per typ

Vilken roll är mest utsatt för de governance-hot Tillsammans försvarar mot? Svaret skiljer sig — och det styr vilka användarvyer som prioriteras per edition.

| Föreningstyp | Primärhot | Mest exponerad roll |
|---|---|---|
| Föräldraförening | Ekonomisk misstro (swish, klasskassa, *"vem snodde pengarna"*) | **Kassör** |
| Samfällighet | Governance-asymmetrier (sakägartvister, bryggkrig, vägpolitik) | **Ordförande** |
| LEF | Varierar enligt verksamhet; oftast ordförande, ibland kassör | Beror på förening |

**Viktig nyans — samfällighetens kassör är inte långt efter ordförande.** Debiteringslängder (andelstal × olika GA × varierande kostnader, lagreglerat dokument enligt LFS) är ofta den juridiskt snårigaste tekniska bördan i svenskt föreningsliv. Till skillnad från BRF (där HSB/Riksbyggen tar ekonomiförvaltning) står samfällighetens kassör oftast helt ensam med denna matematiska-juridiska uppgift. Komplexiteten är en annan typ än föräldraförenings-kassörens (matematisk/juridisk vs. social), men lika tung.

## Implikation för MVP-roadmap per edition

- **Tillsammans för samfälligheter:** ordförandens dashboard (governance-skölden) och kassörens debiteringsstöd byggs parallellt — båda är primära, ingen sekundär.
- **Tillsammans för föräldraföreningar:** kassörens vy är tydligt primär, ordförandens administrativa stöd kan följa efteråt.
- **Tillsammans för LEF:** beror på förening; default = ordförande-centrerad, konfigureras vid onboarding.
