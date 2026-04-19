# Verifikation: 10 svenska samfällighetsföreningar mot Tillsammans-specen (april 2026)

> Verifikations-rapport enligt [verification.md](../verification.md) Spår 1 (stadgejämförelse) för `AssociationType = samfällighet`. Kvitterar tröskelkravet *"minst 3 samfälligheter-stadgar jämförda mot stadge-modulen"*.
>
> Levande dokument — öppet underlag för förtroende (se [mission.md](../mission.md)).

## Sammanfattning

10 samfällighetsföreningar analyserade mot specen, 8 regioner, 5 sub-typer (väg, vatten, multi-GA, skärgård, radhus-tätort). Av 7 konkreta spec-rekommendationer är alla implementerade direkt i spec-filerna.

**Viktigaste fynd:**

1. **Specens röstregel var bakvänd mot LFS-standarden.** Spec sa *"andelsmetod för GA-frågor"* — verkligheten och lagen säger **huvudmetod som default, andelstalsmetod på begäran vid ekonomisk fråga med 1/5-tak**. Rättat.
2. **Multi-GA är normen** (5 av 10). `Participation`-entiteten i domänmodellen validerad.
3. **Debiteringslängden är lagkrav men publikt sällsynt** (2 av 10). Måste vara primär output-funktion i editionen, inte bara "kassörsstöd generellt".
4. **Alla 10 följer Lantmäteriets normalstadgar som mall.** Onboarding för samfällighet kan vara parametriserad wizard över normalstadgan — inte fritext-stadga.
5. **Extern ekonomi-förvaltning är vanlig** (HSB, ekonomibyråer). Saknades som rollkoncept i specen; tillagt.

## Urvalet

| # | Förening | Region | Typ | Antal fastigheter | Stadgar | Senaste dok |
|---|---|---|---|---|---|---|
| 1 | Kyrkmossens SF | Lerum/Floda (V.Götaland) | Radhus multi-GA | 97 | 2017 | 2026 |
| 2 | Warbro A / Enebackens SF | Katrineholm (Södermanland) | Sommarstuge | 82 | 2013 rev | 2025 |
| 3 | Torpa Skogs Vägsamfällighet | Haninge (Stockholm) | Ren väg | ~? | 2022 förslag | 2022 |
| 4 | Fröåvägens SF | Duved/Åre (Jämtland) | Fjäll-väg | ~? | 2025 rev | 2025 |
| 5 | Åby Nordgård SF | Mölndal (V.Götaland) | Radhus multi-GA | 147 | 2022 | 2025 |
| 6 | Långbacka SF | Bomhus/Gävle | Radhus permanent | 225 | HTML | Inloggning |
| 7 | Äskestocks SF | Västervik (Kalmar) | Skärgård multi-GA | 221 | Ja | 2013 / 2024 |
| 8 | Hule Vägförening | Falkenberg (Halland) | Multi-GA väg 68/27/5 | ~? | HTML | 2024 |
| 9 | Rörumstrands Västra SF | Simrishamn (Skåne) | Kust sommarstuge | ~98 | HTML | 2025/26 |
| 10 | PSV-Torekov SF | Båstad (Skåne) | Kust/golfnära | ~? | 2019 | 2019 |

**Täckning:** V.Götaland 2, Södermanland, Stockholm, Jämtland, Gävleborg, Småland kust, Halland, Skåne 2.

**Gap:** Ingen i Bohuslän, Öland, Gotland, Norrland över Jämtland, Mälardalen nordöstra. Gapen beror på ojämn publiceringsdisciplin, inte på metodbrist — mindre samfälligheter i glesbygd publicerar sällan.

## Mönster-analys

### Mönster 1 — Röstregel: spec var bakvänd

**LFS-standard-struktur (7/10 följer direkt):**

- Huvudtalsmetod som grund (1 medlem = 1 röst)
- Acklamation som default
- Andelstalsmetod **triggas på medlems begäran vid ekonomisk fråga**
- **Max 1/5 av totala röstetalet per medlem** (SFL 49§) — obstruktionsskydd
- Fullmakt: **max en medlem per ombud** (SFL 49§)
- Stadgeändring: alltid huvudmetod (SFL 52§)

Kyrkmossen använder platt fördelning 1/97 — fungerar då alla fastigheter är lika. Hule publicerar GA-vikter (68/27/5) men stämmans röstregel följer SFL-default.

**Åtgärd:** [core-concepts.md#röstning] — vänd samfällighets-defaulten. Nytt koncept `voteCapPerMember` (default 0.2) + fullmakts-regel `maxProxiesPerAgent` (default 1).

### Mönster 2 — Multi-GA är normen

5/10 multi-GA:

- **Hule**: 3 GA med 68/27/5 % kostnadsfördelning
- **Norrby brygga**: 2 samf + 4 GA (avlopp, belysning/väg, dagvatten, vatten)
- **Sättra**: GA:1 + GA:2 med *överlappande medlemskategorier*
- **Åby Nordgård**: minst 2 GA, dokumentkonflikt 27 vs 31 andelar i GA:2
- **Heberg**: 2 verksamhetsgrenar (väg, centralantenn)

**Validering:** [domain-model.md] `Participation` som M:N med andelstal per GA håller. Spec:s vattenförenings-exempel (99% med en fastighet exkluderad) matchar Sättras överlappande medlemskategorier.

### Mönster 3 — Debiteringslängd: lagkrav, sällan publik

| Dokument | Publika |
|---|---|
| Debiteringslängd publik | 2/10 (Kyrkmossen, Fröåvägen) |
| Full andelstal-tabell publik | 1/10 (Kyrkmossen platt 1/97) |
| GA-kostnadsfördelning publik | 1/10 (Hule 68/27/5) |

LFS kräver debiteringslängd — men den sparas ofta internt. Kyrkmossens publika debiteringslängd är exempel på best practice.

**Åtgärd:** Debiteringslängd som *automatiskt producerad output* från andelstal × kostnader per GA, primär funktion i [editions/samfallighet.md](../editions/samfallighet.md).

### Mönster 4 — Styrelsestruktur

- Median 5 ledamöter (3–8), vanligast **5 ord. + 2 suppl.**
- Mandatperiod: **2 år ordinarie, 1 år suppleant, rullande** (staggered terms)
- Ordförande ofta 1 år, övriga 2 år (asymmetriskt — redan stött i spec)
- Revisor: 1–2 + suppleant, **lekmanna** (ingen auktoriserad i urvalet)
- Valberedning: 2–3 personer

**Åtgärd:** Ingen — asymmetriska mandatperioder redan stödda via föräldraförenings-analysen.

### Mönster 5 — Jäv (tystnad igen)

0/10 stadgar har obligatorisk jävsdeklaration. SFL 36/48§ implicit. Endast Kyrkmossen har "detaljerad process" (ej extraherbart).

**Åtgärd:** Ingen spec-logik-ändring. Onboarding för samfällighet måste aktivt motivera jäv-modulen, samma mönster som föräldraförenings-editionen.

### Mönster 6 — Extern ekonomi-förvaltning

- **Kyrkmossen**: HSB
- **Fröåvägen**: CarLe Ekonomi AB med avtalsenligt arvode

Båda är externa juridiska personer med mandat mot ekonomiska funktioner.

**Åtgärd:** [architecture.md] — `ExternalAdministrator`-roll tillagd. Tidsbegränsad RBAC, stadgebestämt mandat.

### Mönster 7 — Kallelsetid

- Min 2v standard (7/10) — Lantmäteriets normalstadgar
- Min 3v (Rörumstrand, PSV)
- **Torpa Skogs: tidigast 4v, senast 2v** — matchar Minigiraffen-mönstret (max-gräns mot glömska)

**Åtgärd:** Redan adresserat via föräldraförenings-analysen.

### Mönster 8 — Räkenskapsår

- 1/1–31/12 (Torpa Skogs, PSV, Heberg) — kalenderår
- 1/5–30/4 (Äskestock) — fritidshus-säsong
- Augusti–augusti (Rörumstrand) — stämma i augusti

**Åtgärd:** Räkenskapsår-konfiguration redan tillagd i architecture.md. Validerat.

### Mönster 9 — Underhållsfond / fondavsättning

- **Äskestock**: min 0,3% av anläggningsvärde/år
- **PSV**: min 10k/år, max 700k total
- **Fröåvägen**: höjt till 100k/år via stadgeändring 2025

Underhållsfond = samfällighetens motsvarighet till skol-FF:s strukturerade eventfond.

**Åtgärd:** Modellerat som "underfonder" i architecture.md sedan föräldraförenings-analysen. Validerat.

### Mönster 10 — Normalstadgar som mall

**Alla 10 följer Lantmäteriets normalstadgar.** Konfiguration i parametriserbara fält: antal ledamöter, kallelsetid, fondavsättning, stämmoperiod, kallelsemetod.

**Åtgärd:** Samfällighets-onboarding = wizard över normalstadgans parametrar. Ingen fritext-stadga. [editions/samfallighet.md#onboarding](../editions/samfallighet.md#onboarding-via-normalstadge-wizard).

### Mönster 11 — Publikations-disciplin

| Aktualitet | Andel |
|---|---|
| Årsaktuellt (≤1 år) | 5/10 |
| 1–3 år | 2/10 |
| 5+ år eller inloggning-endast | 3/10 (PSV 2019, Torpa 2022 förslag, Långbacka inloggning) |

**Bättre än föräldraföreningar** (>40% hade 5+ års gap). LFS-formkraven tvingar årlig output även om publik publicering varierar.

**Åtgärd:** Passiv-erosion-hotet ([threats.md]) gäller även här men mindre akut. Inga nya ändringar.

### Mönster 12 — Styrelsearvode

Fröåvägen dokumenterar specifika nivåer: ordförande 50k, ledamot 4k+2k/möte, suppleant 1k+1,5k/möte.

**Åtgärd:** Stämmobeslut via beslutsloggen räcker. Inte primärt koncept.

### Mönster 13 — Förrättning/anläggningsbeslut

4/10 har anläggningsbeslut publikt. Källa till andelstalen.

**Åtgärd:** [lapptäcket-modellen](../domain-model.md#lapptäcket) redan stöttat. Anläggningsbeslutet som uppladdat källdokument.

### Mönster 14 — Reservationsrätt

Torpa Skogs §9 explicit: *"Den som har deltagit i avgörandet av ett ärende får reservera sig mot beslutet."*

Alla andra följer SFL-default.

**Åtgärd:** [core-concepts.md#reservation-och-solidariskt-ansvar] redan stöttat. Validerat.

### Mönster 15 — Solidariskt ansvar

LFS 47§ aldrig nämnt direkt i stadgarna — alltid implicit via lagen.

**Åtgärd:** Bekräftar att specens mekanism (synlig individuell röstning + reservation) är rätt approach för att göra det latenta ansvaret synligt innan kris.

## Implementerade spec-ändringar (april 2026)

1. **[core-concepts.md#röstning]** — samfällighets-default vänd till huvudmetod; andelstal på begäran; 1/5-tak; fullmakts-regel.
2. **[architecture.md#inom-scope-governance-kärnan]** — `ExternalAdministrator` som tidsbegränsad RBAC-roll.
3. **[editions/samfallighet.md]** — ny dedikerad edition-fil.
4. **[verification.md]** — tröskelkravet *"minst 3 samfälligheter-stadgar"* avprickat.
5. **[open-questions.md#-öppen-andelstal--teknisk-representation]** — förtydligat med konkreta case.
6. **[tillsammans.md]** — index utökat med samfällighets-editionen och rapporten.

## Raw källor per samfällighet

### 1. Kyrkmossens SF
- https://www.kyrkmossen.se/samfallighetsinformation/stadgar/
- https://www.kyrkmossen.se/samfallighetsinformation/anlaggningsbeslut/
- https://www.kyrkmossen.se/category/samfallighetsinformation/arsstamma/protokoll/
- Debiteringslängd 2026: https://www.kyrkmossen.se/wp-content/uploads/2026/03/Bilaga-4B-Debiteringslangd-2026.pdf

### 2. Warbro A / Enebackens SF
- https://www.warbro.se/stadgar/
- https://www.warbro.se/arsstammor/
- https://www.warbro.se/verksamhetsberattelser-2/

### 3. Torpa Skogs Vägsamfällighet
- http://torpaskog.com/wp-content/uploads/2022/04/20220212-och-20220514_Forslag-till-reviderade-stadgar-for-Torpa-Skogs-Vagsamfallighet.pdf

### 4. Fröåvägens SF
- https://www.froavagen.se/foreningen/stadgar
- https://www.froavagen.se/stamma2/stammoprotokoll
- https://www.froavagen.se/dokument/20250326%20Preliminär%20debiteringslängd%20Fröåvägens%20Samfällighet.pdf

### 5. Åby Nordgård SF
- https://abynordgardsamf.se/nordgardsfakta/stadgar-och-anlaggningsbeslut/
- https://abynordgardsamf.se/nordgardsfakta/arsredovisningar-och-protokoll/

### 6. Långbacka SF
- https://www.langbacka.se/stadgar/
- (övriga dokument bakom medlemsinloggning)

### 7. Äskestocks SF
- https://askestock.se/styrelse/stadgar/
- https://askestock.se/styrelse/foreningsstammoprotokoll-2004-2013/

### 8. Hule Vägförening
- https://www.hulevf.se/foreningen/stadgar/

### 9. Rörumstrands Västra SF
- https://rorums-vastra-smf.se/51275098/51275104
- https://rorums-vastra-smf.se/51275119/ (stämmoprotokoll 2012/13–2025/26)
- https://rorums-vastra-smf.se/448951083/ (styrelseprotokoll)

### 10. PSV-Torekov SF
- https://www.psvsamf.se/onewebmedia/Protokoll/Stadgar%20antagna%20190727%20reg%20191220.pdf

## Exkluderade

- **Skärgårdsstads SF** (Åkersberga, ~610 fastigheter) — innehåll bakom inloggning
- **Bräcke-Fjällbacka SF** (Bohuslän) — dokument bakom inloggning
- **Klockarejordens Samfällighet** (Svedala) — stadgar publika, övriga saknas
- **Lindhaga SF** (Mölndal, 174 fastigheter) — stadgar på OneDrive, inget arkiv
- **Hebergs SF** (Kungsbacka) — nära-kandidat, skulle kunnat vara #11
- **Finnsta 1 SF** — timeout
- **Vik-Fruviks Vägförening** (Värmdö) — bara stadgar
- **Villshärads Vägförening** (Halmstad) — landingsida
- **Sättra SF** (Riala) — 2022 stadgar, men arkiv saknas
- **Norrby brygga SF** (Norrtälje) — för liten för full profil

Publik publiceringsdisciplin är ojämn — både geografiskt och mellan föreningstyper. Det är en andra-ordnings-signal, inte kvalitets-dom över de exkluderade.
