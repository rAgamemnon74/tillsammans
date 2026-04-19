# Extrastämma — kallelsebegäran

> Del av [Tillsammans-specifikation](../../tillsammans.md). Specialiserar [gemensam ärendehanterings-infrastruktur](../case-types.md#gemensam-infrastruktur-för-ärendehantering).

## Syfte

Begäran om att kalla extra föreningsstämma. Initieras av medlemsgrupp (X% per stadga, LEF 6:13 / LFS 47§), styrelsen själv, eller revisor. **Ärendet är ett *skyldighets-ärende*** — styrelsen kan inte avslå en formenligt giltig begäran från medlemmar eller revisor; den måste verkställas. Mekaniken försvarar minoritetens möjlighet att lyfta brådskande frågor mellan ordinarie stämmor.

**Edition:** bägge.

## Primär initiator

- **Medlemsgrupp:** stadge-/lagbunden andel av medlemmarna (LEF 6:13: en tiondel; LFS 47§: en femtedel; stadgan kan föreskriva lägre tröskel).
- **Styrelsen:** genom styrelsebeslut.
- **Revisor:** ensam (rätt enligt LEF 8:8 / LFS 47§).

## Obligatoriska fält

- **Initiator-typ** — `MEDLEMSGRUPP` / `STYRELSE` / `REVISOR`.
- **Begärande aktör(er)** — vid medlemsgrupp: lista av minst kravantal medlemmar med angiven status `AKTIV`.
- **Skäl för extrastämma** — strukturerad redogörelse av vilken/vilka frågor som ska behandlas.
- **Föreslagen agenda** — minst en agendapunkt med yrkande eller frågeformulering.
- **Stadgaförankring** — paragrafen som ger initiatorn rätten (LEF 6:13 / LFS 47§ + stadge-paragraf).
- **Brådskandegrad** — `BRÅDSKANDE` / `NORMAL` (påverkar styrelsens svarstid och ev. kallelseplanering).

## Livscykel

Specialiserad — skyldighets-fokuserad:

```
BEGÄRD (initiator lämnar in begäran)
  → FORMALIAKONTROLL (systemet + sekreterare validerar tröskelvärde, status AKTIV, stadgaförankring)
    → AVVISAD_PÅ_FORMALIA (saknas tröskel, inaktiva initiatorer, eller stadgebrott)
      → KOMPLETTERAS (initiatorerna kan komplettera och återkomma)
    → GILTIG (begäran godkänns formellt — styrelsen MÅSTE kalla)
      → KALLELSE_FÖRBEREDS (sekreterare/styrelse arbetar med dagordning)
        → MÖTE_KALLAT (kallelse skickad — `MÖTE_KALLAT`-händelse)
          → MÖTE_GENOMFÖRT
            → AVSLUTAT
      → SKYLDIGHETSBROTT_PÅMINNELSE (om styrelsen inte kallar inom stadge-bunden tid)
        → ESKALERAT_TILL_REVISOR (revisor kan kalla själv enligt LEF 8:8)
        → ESKALERAT_TILL_LÄNSSTYRELSEN (sista utvägen — myndighet kan tvinga kallelse)

Sidospår:
  → ÅTERKALLAD (initiatorerna drar tillbaka — alla begärande måste samtycka)
```

`GILTIG` är en *konstaterande* status — styrelsen kan inte avslå, bara verkställa eller bryta mot stadgan.

## RBAC per övergång

| Övergång | Roller |
|---|---|
| skapa BEGÄRD | `MEMBER` (medlemsgrupp), `BOARD_*` (genom beslut), `AUDITOR` |
| BEGÄRD → FORMALIAKONTROLL | automatik |
| FORMALIAKONTROLL → AVVISAD_PÅ_FORMALIA | `BOARD_SECRETARY` (med saklig motivering) |
| FORMALIAKONTROLL → GILTIG | automatik vid uppfyllt tröskelvärde |
| GILTIG → KALLELSE_FÖRBEREDS | `BOARD_*` (sekreterare driver) |
| KALLELSE_FÖRBEREDS → MÖTE_KALLAT | `BOARD_SECRETARY` |
| GILTIG → SKYLDIGHETSBROTT_PÅMINNELSE | automatik vid passerad stadge-tid |
| SKYLDIGHETSBROTT → ESKALERAT_TILL_REVISOR | `AUDITOR` (självaktivering) |
| BEGÄRD/GILTIG → ÅTERKALLAD | samtliga begärande aktörer (full enighet) |

## Tidsregler

- **Tröskelvärden:** LEF 6:13: 1/10 av rösteberättigade (eller stadge-lägre); LFS 47§: 1/5 (eller stadge-lägre). Systemet hämtar tröskeln från stadgekonfiguration.
- **Styrelsens svarstid:** stadge-bunden, default 14 dagar från `GILTIG` att börja kallelse-arbete.
- **Kallelse senast:** stadge-bunden — typiskt 4 veckor från `GILTIG` enligt LEF 6:14, 1 vecka enligt LFS 49§.
- **Kallelsetid till stämman:** kortare än ordinarie stämma (LFS: 1 vecka; LEF: stadge-bart, typiskt 2 veckor).
- **Skyldighetsbrott-tröskel:** stadge-bunden + lagminimum, default 30 dagar utan kallelse → `SKYLDIGHETSBROTT_PÅMINNELSE`.

## Relationer

- Extrastämma-begäran ↔ `MÖTE_KALLAT`-händelse (1..1 vid genomförande).
- Extrastämma-begäran ↔ `Stämma`-entitet (skapas vid kallelse).
- Extrastämma-begäran ↔ initierande motioner / ärenden som ska behandlas.
- Extrastämma-begäran ↔ `Stadgaparagraf` (initiator-rätten).
- Extrastämma-begäran ↔ ev. tidigare extrastämma-begäran (mönster över tid).

## Publicering

| Status | Publik | Medlem | Styrelse | Revisor | Initiatorer |
|---|---|---|---|---|---|
| BEGÄRD | — | — | läs | läs | läs |
| FORMALIAKONTROLL | — | — | läs | läs | läs |
| AVVISAD_PÅ_FORMALIA | — | — | läs (motivering) | läs | läs (saklig) |
| GILTIG | — | (informativ¹) | läs | läs | läs |
| MÖTE_KALLAT | **läs** (kallelse) | **läs** | läs | läs | läs |
| SKYLDIGHETSBROTT_PÅMINNELSE | — | **läs** (transparens om brott) | läs | läs | läs |
| ESKALERAT | (publik vid revisorns kallelse) | läs | läs | läs | läs |

¹ Vid `GILTIG` informeras medlemmar att extra stämma kommer kallas, även om datum saknas — transparens om föreningens governance-arbete.

## Notifiering

| Övergång | Notifiering till |
|---|---|
| BEGÄRD | Initiatorerna (kvittens), styrelsen (ny begäran) |
| GILTIG | Styrelsen (skyldighet att kalla), revisorn (informativ), initiatorerna |
| AVVISAD_PÅ_FORMALIA | Initiatorerna (saklig grund + komplettering-förslag) |
| MÖTE_KALLAT | Alla medlemmar (kallelse), revisorn |
| SKYLDIGHETSBROTT_PÅMINNELSE | Styrelsen (varning), revisorn, alla medlemmar (transparens), initiatorerna |
| ESKALERAT_TILL_REVISOR | Revisorn (förvärvar kallelserätt), styrelsen, initiatorerna |

## Textbibliotek

- **Inlämningskvitto till medlemsgrupp** — *"Tack — er begäran om extra stämma är registrerad. Vi kontrollerar att tröskeln (X% av medlemmarna enligt §Y) är uppfylld inom kort. När begäran är formellt giltig kommer styrelsen kalla till extra stämma enligt stadgans tidsfrister."*
- **Bekräftelse av giltighet** — *"Begäran är formellt giltig. Styrelsen kommer kalla till extra stämma. Datum meddelas senast DD/MM."*
- **Avvisning på formalia** — *"Begäran kan tyvärr inte tas vidare i nuvarande form eftersom [saklig grund — t.ex. tröskeln 1/5 ej uppnådd; X medlemmar saknas]. Vid komplettering kan begäran återlämnas. Vi listar gärna vilka aktiva medlemmar som ännu ej skrivit under: [länk till hjälp]."* — aldrig ren avvisning utan väg framåt.
- **Skyldighetsbrott-påminnelse till styrelsen** — *"Det har gått 30 dagar sedan begäran om extra stämma blev formellt giltig. Enligt stadgans §X ska styrelsen ha kallat senast nu. Saknad kallelse innebär att revisor kan kalla själv enligt LEF 8:8 / LFS 47§. Vänligen agera snarast eller meddela skäl för dröjsmål."*
- **Eskalering till revisor** — *"Hej revisor, styrelsen har inte kallat till extra stämma trots formellt giltig begäran från [initiator]. Du har enligt LEF 8:8 / LFS 47§ rätt att kalla själv. Underlag och medlemslista finns på [länk]."*

## GDPR-hänsyn

Vid medlemsgrupp-begäran är initiatorernas namn synliga för styrelse och revisor (deras roll som begärande är spårbar). Publik vy visar antal initiatorer och tröskel-uppfyllnad — inte personnamn. Begäran och dess avgörande bevaras långsiktigt som governance-historik.

## Hot och försvar

Se [threats.md](../threats.md):

- **Styrelsen "tappar bort" begäran:** automatisk skyldighetsbrott-flagga vid passerad stadge-tid + revisor-eskaleringsväg.
- **Taktisk extrastämma-spam:** tröskelvärdet är skydd; AVVISAD_PÅ_FORMALIA-mallen är konstruktiv (kompletteringsväg) snarare än blockering.
- **Asymmetrier:** medlemsgrupp utan resurser kan begära extrastämma utan juridisk hjälp — formaliakontrollen gör jobbet, mallarna är språkligt enkla.
- **Påtryckningar mot enskilda begärande:** initiatorernas namn skyddas i publik vy (transparens om antal, inte identitet).
- **Skyldighetsbrott:** transparens via publika varningsflaggor + revisorns parallella kallelserätt + sista utvägens länsstyrelsemekanism.

## Öppna frågor

- `[ÖPPEN]` Får medlemmar dra tillbaka sina underskrifter individuellt efter `GILTIG`? Förslag: nej — när begäran är formellt giltig är den styrelsens skyldighet; återkallelse kräver att samtliga undertecknande aktivt drar tillbaka.
- `[ÖPPEN]` Vid `BRÅDSKANDE` brådskandegrad: vad innebär det operativt? Förslag: påminnelse-cykeln komprimeras (7+3+0 dagar istället för 30+7+0); inget annat ändras juridiskt.
- `[ÖPPEN]` Får styrelsen lägga till egna ärenden på den extrastämma som initieras av medlemsgrupp? Förslag: ja, men de begärda ärendena måste finnas på agendan; styrelsens tilläggsärenden tydligt markerade som sådana.
- `[ÖPPEN]` Hur länge är `SKYLDIGHETSBROTT_PÅMINNELSE` synlig publikt? Förslag: tills `MÖTE_KALLAT` eller `ESKALERAT_TILL_REVISOR`. Mönster över flera år (upprepad skyldighetsbrott) kvarstår i föreningens historiska vy.
- `[ÖPPEN]` Tröskelvärde i stadgan kan vara lägre än lagminimum (LEF/LFS) — eller? Lagstiftningen tillåter typiskt sänkning men inte höjning. Stadge-modulen ska validera vid stadgeändring.
