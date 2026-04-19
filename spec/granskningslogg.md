# Granskningslogg

> Del av [Tillsammans-specifikation](../tillsammans.md).

Granskningsloggen är föreningens kontinuerliga, räkenskapsårs-indelade händelselogg — det register som styrelser och myndigheter är vana vid att kalla "protokolls-, besluts- och revisionsunderlag". Tillsammans adderar tekniska integritetsmekanismer (hash-kedja, extern tidsstämpling, distribuerade kopior) så att loggen inte bara finns — den går att **bevisa obruten**.

**Relaterade dokument:**
- [case-types.md](case-types.md) — ärendetypernas livscykel manifesteras som händelser i loggen
- [core-concepts.md](core-concepts.md) — röstning, jäv, reservation och bordläggning är samtliga händelsetyper
- [roles/*](roles/) — vem som får skriva och läsa vilka händelsetyper
- [analysis-rules.md](analysis-rules.md) — grupp 2 (historierevision) och grupp 3 (passiv erosion) adresseras av just denna logg
- [architecture.md](architecture.md) — loggen är arkitekturens centrala datastruktur
- [mission.md](mission.md) — transparens och granskningsbarhet som grundlöfte

## Grundprincip — en logg, inte flera

Det finns **en** append-only händelselogg per förening. "Audit-loggen", "beslutsloggen", "jäv-historiken", "kassaflödet", "protokollet" är inte separata system — de är olika *vyer* (projektioner) över samma underliggande ström av händelser.

Detta ger tre konkreta fördelar:

1. **En hash-kedja att hålla intakt** — ingen rekonciliering mellan parallella loggar.
2. **En förankringspunkt per tidsintervall** — ankerhashen till OpenTimestamps representerar *hela* föreningens governance-tillstånd vid den tidpunkten.
3. **En enhetlig rättelse-modell** — alla rättelser, oavsett vad de rör, följer samma mönster: ny händelse som refererar originalhändelsen.

Aktuellt tillstånd av en entitet (en motion, en roll, en medlem, ett utlägg) är alltid en **projektion** av händelseströmmen. Händelseströmmen är sanningen; vyer är härledda.

## Räkenskapsår som epoker

Loggen är indelad i **epoker**, där varje epok motsvarar ett räkenskapsår. Räkenskapsåret är den gräns både svensk föreningsrätt och bokföringsstandarder känner igen — stämman "stänger" det när ansvarsfrihet beviljas.

En epok har tre livsfaser:

| Fas | Händelse | Innebörd |
|---|---|---|
| **Öppning** | `EPOK_ÖPPNAD` | Genesis-post för året. Refererar föregående epoks sigill, vilket skapar kontinuitet mellan åren. |
| **Löpande** | alla governance-händelser | Normal drift. Händelser skrivs enligt integritetslagerna nedan. |
| **Sigill** | `EPOK_FÖRSEGLAD` | Stämman har beviljat eller avslagit ansvarsfrihet. Epoken är read-only därefter. |

### Sigill-varianter

Alla sigill avslutar epoken tekniskt, men flaggar ansvarsläget olika:

| Sigill-typ | När | Konsekvens |
|---|---|---|
| `ANSVARSFRIHET_BEVILJAD` | Normalfallet | Ren kvittens. |
| `ANSVARSFRIHET_AVSLAGEN` | Stämman avslår | Epoken stängd, civilrättslig process utanför systemet. Flagga "obesutt ansvarsläge" synlig. |
| `ANSVARSFRIHET_UPPSKJUTEN` | Uppskjuten till extra stämma | Epoken är tekniskt *inte* stängd. Ingen ny epok öppnas förrän extra stämman satt sigill. |
| `STÄMMA_UTEBLEV` | Stadgebrott — stämma hölls aldrig | Sigill sätts först när stämman faktiskt hålls. Systemet flaggar stadgebrottet löpande. |
| `RÄKENSKAPSÅR_ÄNDRAT` | Stadgeändring mid-år | Övergångsepok (förlängd/förkortad) med metadata som förklarar bytet. |

### Kontinuitet mellan epoker

Varje `EPOK_ÖPPNAD` refererar den föregående `EPOK_FÖRSEGLAD`:s tip-hash. Kedjan bryts aldrig mellan åren — den är en obruten sekvens som bara *ceremoniellt* delas in i räkenskapsår.

## Transaktionstyper (indikativ lista)

Listan växer allteftersom specen utvecklas. Den är inte uttömmande här, men täcker alla händelser relevanta för governance-kärnan.

**Epok-struktur:**
- `EPOK_ÖPPNAD`, `EPOK_FÖRSEGLAD`

**Beslut:**
- `STYRELSEBESLUT`, `STÄMMOBESLUT`, `UTSKOTTSBESLUT`

**Ärenden:**
- `ÄRENDE_INLÄMNAT`, `ÄRENDE_STATUS_ÄNDRAT`, `ÄRENDE_AVGJORT`, `ÄRENDE_ÅTERKALLAT`, `ÄRENDE_GRUPPERAT`

**Röstning:**
- `RÖST_AVGIVEN`, `RÖSTRÄKNING_GENOMFÖRD`, `JÄV_DEKLARERAT`, `RESERVATION_INGIVEN`

**Möten:**
- `MÖTE_KALLAT`, `KALLELSE_UTSKICKAD`, `MÖTE_ÖPPNAT`, `MÖTE_AVSLUTAT`, `PROTOKOLL_UPPRÄTTAT`, `PROTOKOLL_JUSTERAT`

**Ekonomi:**
- `UTLÄGG_INLÄMNAT`, `UTLÄGG_GODKÄNT`, `UTLÄGG_AVSLAGET`, `BETALNING_EXPORTERAD`, `FAKTURA_UTSTÄLLD`, `DEBITERINGSLÄNGD_FASTSTÄLLD`

**Stadgar:**
- `STADGEVERSION_ANTAGEN`, `STADGETOLKNING_REGISTRERAD`

**Medlemskap:**
- `MEDLEMSKAP_AKTIVERAT`, `MEDLEMSKAP_VILANDE` (LAPSED), `MEDLEMSKAP_UTESLUTET`, `MEDLEMSKAP_BEKRÄFTAT` (reconfirmation)

**Roller:**
- `ROLL_TILLDELAD`, `ROLL_AVSAGD`, `MANDATPERIOD_LÖPT_UT`, `FIRMATECKNINGSRÄTT_ÄNDRAD`

**Granskning:**
- `REVISORSFRÅGA_STÄLLD`, `REVISORSSVAR_AVLÄMNAT`, `REVISIONSBERÄTTELSE_AVLÄMNAD`, `INTEGRITETSKONTROLL_UTFÖRD`

**Dokument:**
- `DOKUMENT_BIFOGAT`, `DOKUMENT_ERSATT` (pekar på nytt dokument; originalet bevaras)

**Addenda (retroaktiva tillägg till stängd epok):**
- `TILLÄGG_SKATTERÄTTSLIG`, `TILLÄGG_DOMSTOLSAVGÖRANDE`, `TILLÄGG_VÄSENTLIGT_FEL`, `TILLÄGG_STÄMMOUPPHÄVANDE`, `TILLÄGG_KLANDERTALAN`, `TILLÄGG_ÅTERTAGEN_KLANDERTALAN`

**Förankring:**
- `OTS_PROOF_SPARAD`, `TIP_HASH_PUBLICERAD`

Varje typ har ett eget JSON-schema som validerar `payload`-fältet. Schemata versioneras — om en typ utvecklas lagras schema-version i payloaden så att gamla händelser kan läsas utan modernisering.

## Vyer (projektioner)

Byggs som filter eller aggregationer över händelseströmmen. Exempel:

| Vy | Filter |
|---|---|
| Beslutslogg | `type IN (STYRELSEBESLUT, STÄMMOBESLUT, UTSKOTTSBESLUT)` + tillhörande `RÖST_AVGIVEN`, `JÄV_DEKLARERAT`, `RESERVATION_INGIVEN` |
| Jäv-historik | `type = JÄV_DEKLARERAT` |
| Kassaflöde | `type LIKE 'UTLÄGG_*' OR 'BETALNING_*' OR 'FAKTURA_*'` |
| Motionshistorik | `type LIKE 'ÄRENDE_*' AND aggregate_type = MOTION` |
| Revisorns vy | allt inom epok, kronologiskt, med granskningslins |
| Medlemmens publika vy | allt med `visibility = PUBLIC` — GDPR-filtrerat |
| Anslagstavlan | `KALLELSE_UTSKICKAD` + refererade besluts |

Aktuellt tillstånd av en aggregat (t.ex. en motion) beräknas genom att replay-a alla händelser med samma `aggregate_id`. För prestanda kan systemet cache:a projektioner (materialized views) — men de är aldrig auktoritativa. Vid tveksamhet är kedjan sanningen.

## Integritet — sex skyddslager

Loggen skyddas genom lager som tillsammans gör manipulation synlig även om den skedde. Obligatoriskt i kärnan: L1–L3 och L5. Opt-in per förening: L4 och L6.

### L1 — Appkonvention (obligatorisk)
Domänkoden erbjuder bara `INSERT`. Inga `UPDATE`/`DELETE`-endpoints mot granskningsloggen. Rättelser är alltid nya händelser som refererar originalhändelsen.

### L2 — DB-tvingad immutabilitet (obligatorisk)
Postgres/PGlite-triggers blockerar `UPDATE` och `DELETE`. `REVOKE` på samma operationer för alla icke-administrativa roller. En DBA kan tekniskt bypassa via `ALTER TABLE` — men bara på bekostnad av att bryta efterföljande integritetslager.

### L3 — Hash-kedja (obligatorisk)
Varje händelse har:
- `prev_hash` — föregående händelses `row_hash`
- `row_hash = SHA-256(prev_hash || canonical_payload_json || type || aggregate_id || sequence)`

En ändring någonstans i historiken bryter kedjan från den punkten. En granskare kan beräkna tip-hashen och jämföra mot förväntad.

### L4 — Extern förankring (opt-in, rekommenderad default: OpenTimestamps)
Periodiskt (dagligen eller vid varje sigill) publiceras kedjans tip-hash till en extern, svåråterskriven infrastruktur. Föreningen väljer vid onboarding:

| Metod | Kostnad | Juridisk tyngd |
|---|---|---|
| **OpenTimestamps** (Bitcoin-förankrat via aggregator) — default | 0 kr | Stark: verifierbar mot Bitcoin-blockchain |
| RFC 3161 TSA (svensk leverantör, t.ex. Svea Tid) | några hundralappar/år | Starkast juridiskt inom eIDAS |
| Sigsum.org (öppen transparens-logg) | 0 kr | Stark, ekosystem-vänlig |
| E-post tip-hash till medlemslistan | 0 kr | Svag men distribuerad |

Systemet schemalägger anchor-körning; fail-soft vid extern nedgång (retry, ingen hård failure).

### L5 — Distribuerade kopior (obligatorisk, utfallsprodukt)
Single-binary-distributionen ger det gratis: revisorn håller en kopia av db-filen. Extra medlemmar kan också. Hash-kedjan gör att diff mellan kopior omedelbart avslöjar divergens.

### L6 — Publik publicering av tip-hash (opt-in, rekommenderad)
Tip-hash skrivs ut i varje protokoll, på anslagstavlan, i årsredovisningens bilagor, i kallelser. Låg kostnad, hög symbolisk effekt — och en framtida rättsprocess kan visa *"så här såg kedjan ut vid stämman 2027-03-15"*.

## Rättelser under löpande epok

Fel upptäcks. En post registrerades med fel belopp, fel datum, fel stavning. Inom den *öppna* epoken:

- Ingen tyst edit tillåts (L1 + L2).
- Användaren registrerar en **rättelse-post** — egen händelse med typ som återspeglar ursprunglig typ + fält `rättar` som pekar på originalhändelsens `event_id`.
- Motivering är obligatoriskt fält. Stadgeanknytning där relevant.
- Vyerna visar korrigerad uppgift som default, med *"Rättad 2026-09-14, motivering: ..."*-markör. Originalet visas vid hovring eller fullständig granskning.

Systemet sparar båda versionerna evigt. Rättelser är aldrig destruktiva.

## Retroaktiva tillägg till stängd epok (addenda)

Efter `EPOK_FÖRSEGLAD` är epoken read-only. Men svensk rätt och bokföringsstandard erkänner att stängda år kan behöva korrigeras. Lösningen är **addenda** — nya händelser i en parallell kedja knuten till den stängda epoken.

### De fem legitima vägarna

| Typ | Grund | Triggas av |
|---|---|---|
| `TILLÄGG_SKATTERÄTTSLIG` | Skatteverket omtaxerar | Myndighetsbeslut |
| `TILLÄGG_DOMSTOLSAVGÖRANDE` | Dom fastställer annan faktisk situation | Domstol |
| `TILLÄGG_VÄSENTLIGT_FEL` | K2/K3 kräver retroaktiv rättelse vid väsentligt fel | Styrelsen, granskat av revisor |
| `TILLÄGG_STÄMMOUPPHÄVANDE` | Senare stämma upptäcker dolda förhållanden, upphäver tidigare ansvarsfrihet | Ny stämma, inom preskriptionstid (LEF 7:54) |
| `TILLÄGG_KLANDERTALAN` | Klandertalan bifallen — tidigare stämmobeslut upphävt | Domstol |

Plus `TILLÄGG_ÅTERTAGEN_KLANDERTALAN` för fall där en påbörjad klandertalan läggs ner.

### Addenda-kedjans mekanik

Varje addendum är en egen post i en parallell kedja knuten till den stängda epoken:

- **Referens** till originalhändelsen som korrigeras eller påverkas (`event_id` inom epoken).
- **Grund** som strukturerad data: myndighetsbeslut-dnr, domsreferens, stämmo-protokollsreferens.
- **Behandlingsspår** — styrelsens beslut att registrera tillägget + revisorns kommentar.
- **Egen hash-kedja** — addenda-kedjan har `prev_hash` mot föregående addendum för samma epok. Bryter aldrig originalepokens hash-kedja.
- **Dubbelt förankrad** — både i addendum-kedjan och via OpenTimestamps, vid registreringen.

### Vad addenda *inte* gör

- Ändrar aldrig original-händelsens `row_hash`.
- Räknar aldrig om originalepokens tip-hash.
- Påverkar aldrig föregående epok-sigill.

Årsredovisningens presentation visar efteråt: *"Ursprunglig siffra 45 000 kr (fastställd 2026-05-14); retroaktiv rättelse 47 500 kr (2026-09-03, Skatteverket-beslut 123-456)."* Granskaren ser alltid bägge.

## Praktiskt flöde genom ett räkenskapsår

Antagande: föreningen har aktiverat L4 (OpenTimestamps) vid onboarding; L1–L3 och L5 är obligatoriska; L6 är på.

### Installation

- Kassören kör Tillsammans-binären för första gången. Onboarding-wizard aktiverar OpenTimestamps.
- Genesis-post skrivs: `EPOK_ÖPPNAD` för räkenskapsår 2026. Genesis-hash visas för kassören som ska arkivera den i onboarding-protokollet.
- Db-filen skapas lokalt. Revisor-onboarding instruerar om db-kopian.

### Styrelsemedlems förberedelse

- Sekreteraren skapar dagordningsutkast — draft-mode, ingen kedja.
- Vid `KALLELSE_UTSKICKAD` är det commit: alla beredda händelser skrivs till kedjan som event-kärna. Ny tip-hash. Kallelsen skickas med integritetskoden inbäddad.

### Medlemsaktivitet

- Medlem läser publika vyer via webb-embed eller publicerat medlemspaket.
- Medlem kontrollerar att publicerad tip-hash matchar den live. Om hen vill verifiera extern förankring kan hen ladda ner OTS-proof — systemet visar hur, men de flesta utnyttjar det aldrig. Möjligheten *är* försvaret.
- Medlem lämnar motion → `ÄRENDE_INLÄMNAT` → bekräftelse med ny tip-hash.

### Mötesgenomförande

- Styrelsemöte: varje beslut är `STYRELSEBESLUT` + `RÖST_AVGIVEN` per ledamot + eventuella `JÄV_DEKLARERAT` + `RESERVATION_INGIVEN`.
- Stämma: mötesordförande förkunnar tip-hash vid öppnande (matchar kallelsens) och vid avslut (ny). Båda skrivs i protokollet.

### Efterarbete

- Inom 24h kör OTS-workern. Aggregatorn poolar dygnets hashar i en Bitcoin-transaktion. Proof sparas lokalt när det landat.
- Protokoll cirkuleras till justerare → `PROTOKOLL_JUSTERAT` när signerat.
- Eventuella rättelser registreras som rättelse-poster, aldrig som tyst edit.

### Revisorsgranskning och epok-sigill

- Revisorn öppnar sin kopia av db-filen i granskningsläge. Systemet kör automatiskt integritetskontroll:
  - Hash-kedjan intakt från genesis till tip.
  - Alla OTS-ankare verifierade mot Bitcoin.
  - Publicerade tip-hashar matchar faktiska kedjepunkter.
  - Rättelse- och addendum-poster har grund och motivering.
- Revisorn skriver `REVISIONSBERÄTTELSE_AVLÄMNAD` med till- eller avstyrkan.
- Stämman beviljar (eller avslår) ansvarsfrihet → `EPOK_FÖRSEGLAD` med sigill-variant. OTS-förankring vid sigilltillfället.
- Ny epok öppnas samma ögonblick med `EPOK_ÖPPNAD` som refererar sigillet.

## Datastruktur — referensmodell

```
granskningslogg_händelse
  event_id           UUIDv7 (klient-genererad för idempotens)
  epoch_id           FK till epok
  sequence           monotont inom epok
  type               transaktionstyp (enum)
  aggregate_id       vilken entitet händelsen rör (motion, roll, utlägg, etc.)
  aggregate_type     motsvarande typ-diskriminator
  actor_id           vem som utförde
  at                 tidpunkt (server-verifierad)
  payload            JSON (validerat mot typens schema)
  schema_version     payload-schemats version
  visibility         PUBLIC / MEMBER / BOARD / AUDITOR / PERSON
  prev_hash          32 byte
  row_hash           32 byte
  rättar             optional FK till event_id (om rättelse-post)

granskningslogg_epok
  epoch_id
  fiscal_year_start, fiscal_year_end
  genesis_event_id
  genesis_hash
  seal_event_id      (null om ej förseglad)
  seal_variant       ANSVARSFRIHET_BEVILJAD / ... (null om ej förseglad)
  tip_hash_at_seal   (null om ej förseglad)
  ots_proof          (null om L4 ej aktivt eller ej förankrat)

granskningslogg_tillägg
  addendum_id        UUIDv7
  epoch_id           den stängda epok tillägget rör
  type               TILLÄGG_*
  references_event_id  vilken originalhändelse som berörs
  grund              strukturerad referens (dnr, domsref, protokollsref)
  behandling         styrelsebeslutets/revisorns referens
  payload            JSON (validerat mot typens schema)
  prev_addendum_hash 32 byte (kedja per epok)
  addendum_hash      32 byte
  ots_proof          (null om L4 ej aktivt eller ej förankrat)
```

Detta är en referensmodell, inte färdig datamodell-spec. Implementations-detaljer (index, partitionering, sekvensgenerering) avgörs i [domain-model.md](domain-model.md) när den skrivs ut.

### Prestanda-överväganden

- **Snapshots är cache, inte sanning.** Materialized views får byggas för frekventa vyer, men kan när som helst rebuild-as från kedjan.
- **Merkle-träd inom epok** är ett alternativ vid stora volymer — tillåter partiell verifiering utan full replay. Platt hash-kedja räcker i MVP; överväg Merkle vid klagomål.
- **OTS-aggregator** hanterar miljontals hashar per transaktion; föreningens volym är triviell för den.

### Idempotens

Client-genererat `event_id` (UUIDv7) + unique-constraint garanterar att retry efter nätverksfel inte skapar dubletter. Servern validerar `prev_hash` mot aktuellt tip innan insert; vid mismatch returneras fel.

## Edition-hänsyn

- **Samfällighet.** Debiteringslängden (lagkrav enligt LFS) är egen händelsetyp `DEBITERINGSLÄNGD_FASTSTÄLLD` vid varje stämma. Dess hash inkluderas i sigillets payload.
- **Föräldraförening.** Brutet räkenskapsår (1/7–30/6) för skol-FF som följer läsår. Epokens datum styrs av stadgans räkenskapsår-inställning.
- **LEF.** Bolagsverket-inlämning av årsredovisning loggas som `DOKUMENT_BIFOGAT` med metadata + kvittensreferens, men är inte del av epok-sigillet — den är nästa-epok-händelse.

## Gränsdragningar — vad granskningsloggen inte är

- **Inte operativ data.** Mätvärden, bokningar, inventarier, skaderapporter, störningsanmälan — hanteras i externa verktyg ([architecture.md#utanför-scope](architecture.md)). Om ett beslut fattats om dem logas *beslutet*, inte den operativa datamängden.
- **Inte styrelsens interna utkast.** Draft-mode är friktionsfri; loggen börjar vid publicering. En lista där styrelsen brainstormar möjliga leverantörer ingår inte.
- **Inte fri-textarkiv.** Dokumentarkivet refereras av händelser (`DOKUMENT_BIFOGAT`) men är inte del av händelseloggen. Dokument versioneras separat; händelser är pekare.
- **Inte chattloggar eller informell kommunikation.** E-post och muntliga samtal mellan ledamöter är utanför systemet. Det som händer i systemet (anslagstavla, kommentarer på motioner) loggas; resten är utanför scope.

## Öppna frågor

- **Merkle-träd vs. platt kedja.** MVP kör platt; när skalas det till träd? Förslag: när en driftad förening har >50 000 händelser per räkenskapsår.
- **Snapshot-strategi vid epok-sigill.** Ska systemet automatiskt generera en kanoniserad läsbar "årsbok" (PDF/HTML) vid varje sigill som kan arkiveras oberoende? Förslag: ja, som del av sigill-processen. `[ÖPPEN]` exakt format.
- **Rättelse-motiveringens stringens.** Ska motiveringen vara fri text eller kategoriserad (typo / räknefel / uppdaterad källdata / annat)? Förslag: obligatorisk kategorisering + valfri fri text.
- **Grund för addendum.** Ska `grund`-fältet kräva uppladdat dokument (myndighetsbeslut-kopia, doms-PDF) eller räcker referens? Förslag: dokument obligatoriskt för de tre myndighets-/domstolsbaserade typerna; referens räcker för `TILLÄGG_VÄSENTLIGT_FEL` och `TILLÄGG_STÄMMOUPPHÄVANDE`.
- **OTS-aggregatorns livslängd.** OpenTimestamps-aggregatorn är en publik tjänst utan SLA. Om den försvinner — hur migreras befintliga proofs? Förslag: systemet stöder flera L4-metoder parallellt; aktivering av en ny metod genererar proofs framåt, befintliga proofs förblir verifierbara mot Bitcoin så länge den kedjan finns.
- **Revisorns roll vid addenda.** Innevarande revisor granskar alla addenda till alla stängda epoker, oavsett vem som var revisor då. Behöver vi särskild mekanism för att tidigare revisor ska kunna kommentera? Förslag: läsrätt bevaras per tidigare revisors mandatperiod (enligt [oversight-roles.md#byte-av-revisor-mid-år](roles/oversight-roles.md)); de kan skriva `REVISORSSVAR_AVLÄMNAT` på egna gamla frågor men inte på nya.
- **BankID-signering.** När BankID-stöd läggs till aktualiseras signering av sigill, protokoll och addenda. Kopplas till [open-questions.md#öppen-autentisering](open-questions.md).
