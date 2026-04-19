# Verifikation: 12 svenska föräldraföreningar mot Tillsammans-specen (april 2026)

> Verifikations-rapport enligt [verification.md](../verification.md) Spår 1 (stadgejämförelse) för `AssociationType = föräldraförening`.
>
> Levande dokument — öppet underlag för förtroende (se [mission.md](../mission.md)). Resultaten matas tillbaka in i specen.

## Sammanfattning

12 föräldraföreningar (skol-FF och föräldrakooperativ-förskolor) analyserade mot specen. Urvalet spänner 6 regioner och 5 skoltyper. Av 7 konkreta spec-rekommendationer är 6 implementerade direkt i spec-filerna (se slutet av denna rapport); en (arbetsplikt för kooperativ) har lyfts till öppen fråga.

**Viktigaste fynd:**

1. **Vår röstmodell "1 barn splittat" är unik bland 12 observerade** — ingen använder den.
2. **Jäv-deklarationsplikt saknas helt i publicerade stadgar** — vår obligatoriska jäv-modul är en feature, inte imitation.
3. **Publikations-gap på 5+ år i >40% av urvalet** — det nya hotet "passiv erosion" är empiriskt validerat.
4. **Föräldrakooperativ-förskolor (LEF) dominerar "publikt kompletta stadgar"-kategorin** — deras formkrav tvingar fram disciplin som skol-FF saknar.

## Urvalet

| # | Förening | Region | Typ | Stadgar publika | Senaste publika dok |
|---|---|---|---|---|---|
| 1 | Skol-FF A | Stockholm | Fristående grundsk+gy | §4 citerat i vb | 2020 |
| 2 | Rudolf Steinerskolan GBG FF | Göteborg | Waldorf grundsk | Ja (HTML) | 2012 (stadgar) |
| 3 | Nya Elementar FF | Bromma/Stockholm | Kommunal F-9 | Ja | 2023 (protokoll) |
| 4 | KG Parents | Kungsholmen/Stockholm | Kommunalt gymnasium | Ja (bilingualt) | 2018 (stadgar) |
| 5 | Karl Johansskolans FF | Majorna/Göteborg | Kommunal F-6 | I arkiv 2018 | 2018 |
| 6 | HAFF (Lunds Waldorf) | Lund | Waldorf | Bakom friktion | Okänt |
| 7 | Kottarna | Torsby/Värmland | Kooperativ förskola | Ja (HTML) | Odaterat |
| 8 | Minigiraffen | Linköping/Östergötland | Kooperativ förskola | Ja (2020 PDF) | 2024 (vb) |
| 9 | Barnlåten | V. Frölunda/Göteborg | Kooperativ förskola | Ja (HTML) | 2024 (organisation) |
| 10 | Solen | Östersund/Jämtland | Kooperativ förskola | Nej | — |
| 11 | Backatorpsskolans FF | Hisings Backa/Göteborg | Friskola F-6 | Ja (2020 PDF) | 2020 (stadgar) |
| 12 | Montessoriföreningen Svanen | Älta/Nacka | Kooperativ Montessori | Ja (2016 PDF) | 2016 (stadgar) |

**Täckning:** Stockholm, Göteborg, Värmland, Östergötland, Jämtland, Lund. Föreningskategorier: kommunala skolor 3/12; fristående skolor (inkl. Waldorf) 4/12; föräldrakooperativ-förskolor 5/12.

**Gap:** Skåne (utanför Waldorf-HAFF) hittades inget med publika stadgar. Malmö, Uppsala, Umeå, Norrland ospublicerat. Detta är en observation om Sveriges föreningskultur, inte om metoden.

## Mönster-analys

### Mönster 1 — Medlemskapsmodell

Tre distinkta modeller observerade:

| Modell | Föreningar |
|---|---|
| Automatiskt vid barn på skolan, gratis | Skol-FF A, KJFF, Rudolf Steiner GBG, HAFF |
| Avgiftsbaserat (explicit årlig aktivering) | Nya Elementar, KG Parents, Backatorp |
| Ansökan + insats (kooperativ, LEF) | Kottarna, Minigiraffen, Barnlåten, Svanen |

**Spec-påverkan:** Ingen kör *årlig skriftlig reconfirmation* som [domain-model.md#medlemskapslivscykel] föreslår. Men **Backatorp §2** har empiriskt validerad automatik-modell: *"Medlemskapet upphör då medlemmen ej längre är vårdnadshavare till barn som är inskrivna på skolan."* Externt faktum (barn-inskrivningsstatus) triggar `LAPSED` — enklare än årlig reconfirmation, täcker samma svaghet.

### Mönster 2 — Röstregel (största spec-avvikelsen)

| Regel | Föreningar |
|---|---|
| 1 medlem = 1 röst | Rudolf Steiner GBG, Nya Elementar, Backatorp, Minigiraffen |
| 1 familj = 1 röst | KG Parents, Minigiraffen (§12) |
| 2 föräldrar = 2 röster (båda är medlemmar) | Svanen |
| 1 närvarande förälder = 1 röst | Skol-FF A (styrdokument), Barnlåten |
| 1 barn splittat mellan vårdnadshavare | **Ingen** |

**Spec-påverkan:** Vår "1 barn = 1 röst splittad" är unik — finns inte i någon observerad svensk föräldraförenings praxis. Degraderad till stadge-alternativ; default bör vara "1 medlem = 1 röst" eller "1 familj = 1 röst." LEF 6:3 är kompatibelt utan tillägg.

### Mönster 3 — Jäv (tystnad är fyndet)

**Generell jävsdeklarationsplikt:** 0 av 12.

**Strukturella jäv-regler finns dock:**
- **Rudolf Steiner GBG**: personal får ej sitta i FF-styrelsen.
- **Barnlåten §5**: *"Förälder som är anställd av kooperativet äger ej rösträtt i frågor som rör personal."*

**Svanen** har uteslutningsklausul men ingen löpande deklaration.

**Spec-påverkan:** Vår obligatoriska jäv-modul ligger utanför dominerande praxis. Onboarding-berättelsen måste *motivera* den aktivt — inte anta att den är etablerad standard. Själva spec-logiken är oförändrad.

### Mönster 4 — Kallelsetid

2 veckor dominerar (8 av 12). 1 vecka för extra stämma är standard. **Minigiraffen §17** har en *max*-gräns (6 veckor) — ovanligt, skyddar mot att kallelsen kommer så tidigt att medlemmar glömmer.

**Spec-påverkan:** Stöd både min- och max-gräns på kallelsetid.

### Mönster 5 — Mandatperiod och valberedning

- 1 år dominerar (Rudolf Steiner GBG, KG Parents, Svanen, Barnlåten).
- 2 år i skol-FF med rotation (Skol-FF A, Backatorp övriga, Minigiraffen).
- Asymmetriskt (Kottarna: ordf 2 år, övriga 1-2) — ovanligt men finns.
- Valberedning är obligatorisk i alla granskade stadgar.
- **Nya Elementar 2025/2026: valberedningsposten är vakant.** Formellt kravet lever, bemanningen saknas.

**Spec-påverkan:** Stadge-modul måste stödja både symmetriska och asymmetriska mandatperioder per roll. Vakansrapportering behövs i styrelsevyn.

### Mönster 6 — Ekonomi-governance

- **Styrdokument för fondfördelning publikt:** bara Skol-FF A (strukturerad eventfond).
- **Backatorp §9**: revisor har *"fritt tillträde till föreningens protokoll, korrespondens och räkenskaper närhelst de så påkallar"* — löpande granskningsrätt inskriven.
- **Minigiraffen §14**: strukturerat revisionsfönster — revisor får årsredovisning senast 2 v före stämma, medlem får revisionsberättelse senast 1 v före.
- **Brutet räkenskapsår** (1/7–30/6) hos KG Parents och Svanen — följer läsåret.

**Spec-påverkan:** Räkenskapsår-konfiguration (kalender/brutet/läsår) måste vara stadge-inställning, inte hårdkodad.

### Mönster 7 — Klasskassor

- **Explicit separat bankkonto per klass**: HAFF Lund.
- **Implicit klasskassa-hantering**: Skol-FF A, Nya Elementar, Minigiraffen.
- **Ingen klasskassestruktur**: kooperativ-förskolor (inga klasser).

**Spec-påverkan:** Abstraktionen "Class-kassa" är rätt. Separat bankkonto är implementationsdetalj — stöd, men kräv inte.

### Mönster 8 — Publikations-disciplin (passiv erosion)

| Dokument-aktualitet | Andel |
|---|---|
| ≤1 år gamla | 2/12 (Minigiraffen, Barnlåten-omslag) |
| 1-3 år | 2/12 |
| 5+ år / inaktivt | 5/12 (Skol-FF A, KJFF, KG Parents stadgar, Rudolf Steiner GBG 2012, Svanen stadgar 2016) |
| Okänt/inget publicerat | 3/12 (HAFF, Kottarna, Solen) |

**Över 40% av urvalet har 5+ års publikations-gap.** KJFF är det tydligaste fallet — föreningen är aktiv 2026 (Facebook live), men webbpublicering upphörde 2018.

**Spec-påverkan:** Publicering måste vara *biprodukt* av systemet — inte en separat manuell rutin som dör med styrelsebyten.

### Mönster 9 — Uteslutning och medlemskapstvister

- **Minigiraffen §6**: uteslutning → barnet avstängs (koppling medlemskap↔tjänst).
- **Svanen §8**: uteslutning med 1 månads överklagan till stämman — matchar spec `EXCLUDED` exakt.
- **Kottarna**: överklagan inom 1 månad.

**Spec-påverkan:** [domain-model.md#tvister]-modellen är empiriskt stödd. Minigiraffen-mönstret är edge case — ej bygg for.

### Mönster 10 — Rollövergripande representationsmandat

- **Backatorp §5**: FF utser ledamot till Stiftelsen Backatorpsskolans styrelse.
- **Skol-FF A**: stämman och valberedningen utser 5 ledamöter i huvudmannaskolans styrelse.

Föreningen är ofta **formell governance-intressent mot huvudmannen** (stiftelse, ab).

**Spec-påverkan:** Relationen förening↔huvudman är värd förstklassigt koncept.

### Mönster 11 — Arbetsplikt

| Förening | Plikt |
|---|---|
| Svanen | 20 dagar/år (max) |
| Minigiraffen | 15-17 dagar/år |
| Barnlåten | 8-12 städdagar/termin + storstädning |
| Kottarna | Arbetsplikt i stadgarna |

Föräldrakooperativ har **kvantifierad arbetsplikt**. Skol-FF har generisk "engagemang" utan kvantifiering.

**Spec-påverkan:** Lucka. Öppen fråga: ska arbetsplikt modelleras i scope, eller ligga utanför?

### Mönster 12 — Stadgetolkning vs. stadgeändring

- **Skol-FF A 2019**: stämman tolkade "två suppleanter" som *minst* två.
- **Minigiraffen §17**: stadgeändring = 2/3 på två följande stämmor, inklusive fullmaktsröster.
- **Backatorp §10**: 3/4 på två följande årsmöten, minst 6 månaders mellanrum.
- **Svanen §19**: upplösning = enhälligt eller 2/3 på två följande stämmor.

**Spec-påverkan:** Stadgeändringströsklar är stadge-inställning (2/3, 3/4, osv), inte hårdkodad regel.

## Implementerade spec-ändringar (april 2026)

1. **[core-concepts.md#röstning]** — "1 barn splittat" degraderad till stadge-alternativ; default är "1 medlem = 1 röst."
2. **[core-concepts.md#kallelsemodell]** — max-gräns på kallelsetid explicitgjord.
3. **[domain-model.md#medlemskapslivscykel]** — `LAPSED`-trigger kan vara externt faktum (Backatorp-modellen).
4. **[architecture.md]** — utskott, underfonder, huvudmanna-relation, räkenskapsår-konfiguration listade som förstklassiga koncept.
5. **[verification.md] Spår 2** — nytt scenario 11 "onboarding med gammal stadga."
6. **[editions/foraldraforening.md]** — ny dedikerad edition-fil för föräldraförening.
7. **[open-questions.md]** — ny öppen fråga: arbetsplikt-modell i scope eller ej?

## Raw källor per förening

### 1. Skol-FF A
- Källor ej publikt länkade (anonymiserad post; dokument i arbetsarkivet).
- Underlag: styrdokument strukturerad eventfond v1.1 (2018), årsmötesprotokoll 2019, verksamhetsberättelse 19/20, årsredovisning 19/20.

### 2. Rudolf Steinerskolan Göteborg FF
- https://www.rudolfsteinerskolan.nu/waldorf-stadgar-foraldraforening
- https://docplayer.se/1691466-Stadgar-for-foraldraforeningen-vid-rudolf-steinerskolan-i-goteborg.html
- Stadgar antagna 2012-03-05

### 3. Nya Elementar FF
- https://www.ff-nyaelementar.se/om/stadgar/
- https://www.ff-nyaelementar.se/om/arsmotesprotokoll/
- Årsmötesprotokoll-arkiv 2004–2023

### 4. KG Parents (Kungsholmens Gymnasium)
- https://www.kgparents.se/
- https://kgparents.se/stadgar-statutes/
- Stadgar 2018-10-06, bilingualt

### 5. Karl Johansskolans FF (KJFF)
- https://kjff.se/kategorier/protokoll/
- Stadgar i årsmöteshandlingar 2018-05-14; senaste publika protokoll 2018

### 6. HAFF (Hardeberga/Lunds Waldorf)
- https://www.lundswaldorf.se/foraldraforeningen
- https://lundswaldorfforaldrar.se/hantering-av-klasskassor/

### 7. Kottarna (Torsby)
- https://kottarna.nu/styrelsen/stadgar-for-foraldrakooperativet-kottarna/

### 8. Minigiraffen (Linköping)
- https://minigiraffen.se/onewebmedia/Stadgar_Minigiraffen_2020.pdf
- https://minigiraffen.se/f-r-ldrakooperativet/styrelsen/verksamhetsber-ttelse

### 9. Barnlåten (V. Frölunda)
- https://barnlaten.se/stadgar.html
- https://barnlaten.se/docs/Detta är Barnlåten.pdf (reviderad aug 2024)

### 10. Solen (Östersund)
- https://www.fksolen.se/

### 11. Backatorpsskolans FF
- https://backatorpsskolan.se/om-oss/foraldraforening
- Stadgar fastställda 2020-11-16

### 12. Montessoriföreningen Svanen (Älta/Nacka)
- https://forskolansvanen.se/wp-content/upLoads/STADGAR-MONTESSORIFORENINGEN-SVANEN-20160420.pdf
- Stadgar antagna 1989, reviderade 2001, 2013, 2016

## Exkluderade från urvalet

- **Samskolan Göteborg** — Sveriges äldsta FF (1908), men stadgar ej publikt.
- **Malmaskolans FF (Uppsala)** — landing page, inga dokument publikt.
- **Nälstaskolans FF (Stockholm)** — bloggformat, inga stadgar.
- **Myråsskolans FF (Borås)** — bara på Borås Stads kommunsida.
- **Umeå Waldorfskolas FF** — ingen egen webbplats, dokumentmässigt tunt.
- **Katedralskolans FF (Linköping)** — kommunsida 404.
- **Föräldrakooperativet Lönnen (Göteborg)** — webbplats utan stadgar/protokoll.
- **Förskolan Framtiden (Lund)** — ingen publik stadgar/protokoll.
- **Förskolan Vilda Väster (Malmö)** — ingen publik årsredovisning.

Detta säger *inte* att dessa föreningar är dåligt skötta — det säger att publiceringsdisciplinen är ojämn. En andra-ordnings-signal.
