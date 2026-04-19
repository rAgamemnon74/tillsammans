# Uteslutningsärende

> Del av [Tillsammans-specifikation](../../tillsammans.md). Specialiserar [gemensam ärendehanterings-infrastruktur](../case-types.md#gemensam-infrastruktur-för-ärendehantering).

## Syfte

Det formella förfarandet för att utesluta en medlem ur föreningen. Uteslutning är **det starkast personrelaterade ärendet** i biblioteket — det vänder sig mot en namngiven individ, har juridiska konsekvenser och lever vidare i föreningens historia. Mekaniken är därför försedd med flera spärrar: obligatorisk varning + svarsfönster, dokumenterad grund, och rätten till stämmoöverklagan.

**Edition:** primärt LEF (samfällighet är sällan tillämpligt eftersom medlemskap följer fastigheten enligt LFS — uteslutning används bara vid grova stadgebrott som påverkar samfälligheten).

## Primär initiator

`BOARD_*` genom styrelsebeslut. Vid utebliven avgift kan stadgan föreskriva automatik — då initieras ärendet av systemet med stadge-paragrafen som auktoritetsgrund, men styrelsen måste fortfarande bekräfta innan uteslutningen verkställs.

Enskild medlem kan **inte** initiera uteslutning av annan medlem (allmänt ärende-kanalen finns för klagomål; styrelsen avgör om grund för uteslutning föreligger).

## Obligatoriska fält

- **Berörd medlem** — `Medlem`-referens.
- **Stadgaförankring** — vilken stadgaparagraf åberopas (typiskt LEF 3:7 eller motsvarande stadge-paragraf).
- **Saklig grund** — strukturerad redogörelse av handlandet eller underlåtenheten.
- **Varningens innehåll** — när och hur varning meddelats.
- **Bilagor** — bevisning, korrespondens, ev. läkarintyg/personliga handlingar (registreras som **egna `BILAGA_INLÄMNAD`-händelser** med strängare visibility, se [granskningslogg.md#visibility--strikt-inheritance-väg-a](../granskningslogg.md#visibility--strikt-inheritance-väg-a)).

## Livscykel

Specialiserad med obligatoriska spärrar:

```
INITIERAT
  → VARNING (styrelsen beslutar om varning + delgivning till medlemmen)
    → SVARSFÖNSTER (medlemmen har stadge-bunden tid att svara/rätta sig)
      → BEREDNING (styrelsen bedömer svaret)
        → STYRELSEBESLUT
          · UTESLUTNING_BESLUTAD
            → STÄMMOPRÖVNING (om medlemmen begär — stadge-/lagrätt enligt LEF 3:7)
              → AVGJORT_AV_STÄMMA · UTESLUTNING_FASTSTÄLLD / UPPHÄVD
            → AVGJORT (om ingen prövning begärs eller fristen passerats)
          · INGEN_ÅTGÄRD (varningen räckte; ärendet avslutas)

Sidospår:
  → ÅTERKALLAT (styrelsen drar tillbaka; medlemmen informeras)
  → AVSKRIVET (medlemmen lämnar frivilligt under processen)
```

`VARNING` och `SVARSFÖNSTER` är **inte hoppbara**. Ett utesluttings-beslut utan dem är formellt angripbart.

## RBAC per övergång

| Övergång | Roller |
|---|---|
| skapa INITIERAT | `BOARD_*` genom styrelsebeslut (eller automatik vid utebliven avgift) |
| INITIERAT → VARNING | `BOARD_*` med stadgehänvisning |
| SVARSFÖNSTER (read) | berörd medlem + `BOARD_*` + `AUDITOR` |
| BEREDNING → UTESLUTNING_BESLUTAD | `BOARD_*` genom styrelsebeslut (kvorum + stadge-grund obligatoriskt) |
| → STÄMMOPRÖVNING | berörd medlem (begäran inom stadge-frist) |
| AVGJORT | `BOARD_SECRETARY` verkställer |

## Tidsregler

- **Varningen** delges medlemmen genom dokumenterad kanal (registrerad e-post, brev, BankID-bekräftad mottagning).
- **Svarsfönster:** stadge-bundet, default 30 dagar — får aldrig vara kortare än stadgans minimum.
- **Stämmoprövning-fönster:** stadge-bundet, default 1 månad efter beslutet enligt allmän föreningspraxis.
- **Verkställighet** sker tidigast efter stämmoprövning-fönstrets utgång eller stämmans bekräftande beslut.

## Relationer

- Uteslutningsärende ↔ `Medlem` (1..1, central).
- Uteslutningsärende ↔ `STYRELSEBESLUT` (varningsbeslut + uteslutningsbeslut).
- Uteslutningsärende ↔ `Stadgaparagraf` (uteslutningsgrund).
- Uteslutningsärende ↔ ev. `STÄMMOBESLUT` vid prövning.
- Uteslutningsärende ↔ `MEDLEMSKAP_UTESLUTET`-händelse vid verkställighet.
- Uteslutningsärende ↔ ev. tidigare allmänt ärende-klagomål som ledde fram till ärendet.

## Publicering

**Personärende — avgränsad insyn enligt [mission.md](../mission.md) och [case-types.md#principer-för-ärendetyperna](../case-types.md#principer-för-ärendetyperna) princip 2.**

| Status | Publik | Medlem | Styrelse | Berörd |
|---|---|---|---|---|
| INITIERAT | — | — | läs | — (ej delgivet ännu) |
| VARNING | — | — | läs | **läs** + svarsmöjlighet |
| SVARSFÖNSTER | — | — | läs | läs + skriv (svar) |
| BEREDNING | — | — | läs | läs (sitt svar visas i loggen) |
| UTESLUTNING_BESLUTAD | — | — | läs | **läs** + besvärshänvisning |
| STÄMMOPRÖVNING | — | (agendapost utan personnamn)¹ | läs | läs |
| AVGJORT | (intern statistik) | — | läs | läs |

¹ Stämman behandlar uteslutningsfrågan, men kallelsen anger sakfrågan utan att exponera medlemsnamn för bredare publik. Berörda medlemmar deltar i röstningen som vanligt.

`AUDITOR` har permanent läsrätt på allt enligt [oversight-roles.md](../roles/oversight-roles.md#revisor).

## Notifiering

| Övergång | Notifiering till |
|---|---|
| VARNING | Berörd medlem (formell delgivning), styrelsen, revisorn |
| SVARSFÖNSTER (utgår) | Berörd medlem (påminnelse 7 dagar före), styrelsen |
| UTESLUTNING_BESLUTAD | Berörd medlem (formell delgivning + besvärshänvisning), styrelsen, revisorn |
| STÄMMOPRÖVNING | Stämman (via kallelse), berörd medlem |
| AVGJORT | Berörd medlem, styrelsen, kassören (avsluta avgifter), revisorn |

Föreningen kan kräva (via stadge-konfiguration) att uteslutnings-relaterade notifieringar dessutom skickas via brev — rättssäkerheten kräver dokumenterat mottagande, se [case-types.md#distribution-och-notifiering](../case-types.md#distribution-och-notifiering).

## Textbibliotek

Tonen är saklig, proportionerlig och **aldrig anklagande**. Mallarna är konstruktiva — varningen ger en väg ut.

- **Varning** — *"Hej [namn], styrelsen har vid sitt möte den DD/MM noterat följande förhållande: [saklig redogörelse]. Enligt §X i stadgarna kan detta utgöra grund för uteslutning. Du har möjlighet att rätta förhållandet eller lämna förklaring senast DD/MM. Vi vill betona att syftet med varningen är att ge dig möjlighet att fortsätta som medlem — inte att inleda uteslutning."*
- **Mottagningsbekräftelse av svar** — *"Tack för ditt svar. Styrelsen kommer att behandla det vid sitt möte den DD/MM. Du får besked därefter."*
- **Beslut: ingen åtgärd** — *"Styrelsen har efter bedömning av ditt svar och åtgärder beslutat att avsluta ärendet utan vidare åtgärd. Tack för din samverkan."*
- **Uteslutningsbeslut** — *"Efter prövning vid styrelsemöte den DD/MM har styrelsen beslutat att utesluta dig som medlem enligt §X. Skäl: [saklig hänvisning]. Du har rätt att begära att frågan prövas av föreningsstämman; begäran lämnas senast DD/MM via [kanal]. Beslutet verkställs efter denna fristens utgång eller efter stämmans bekräftande beslut."*
- **Stämmoprövnings-bekräftelse** — *"Din begäran om stämmoprövning av uteslutningsbeslutet är registrerad. Frågan kommer att tas upp på [ordinarie/extra stämma DD/MM]. Du har rätt att närvara, yttra dig och rösta i frågan."*

## GDPR-hänsyn

**Högsta känslighetsklass.** Uteslutningsärenden behandlar potentiellt känsliga personuppgifter (ekonomi, hälsa, beteende, konflikter).

- Bilagor med särskilt känsligt innehåll (läkarintyg, anklagelser) registreras som egna händelser med strängare visibility (`PERSON + BOARD_CHAIR + AUDITOR`).
- Uteslutningsbeslutets fulltext bevaras som del av föreningens formella historik (governance-bevisvärde) — gallras inte.
- Personuppgifter i bilagor gallras enligt särskild regel: 5 år efter beslut för dokumentation, därefter nullas innehåll men hashar och beskrivningar bevaras (se [granskningslogg.md#gdpr-gallring](../granskningslogg.md#gdpr-gallring)).
- Berörd medlems läsrätt på sitt eget ärende är permanent (oavsett `UTESLUTEN`-status).

## Hot och försvar

Se [threats.md](../threats.md):

- **Egenintressen och jäv:** styrelseledamot med personlig konflikt med berörd medlem måste avstå från beslutet → obligatorisk jäv-deklaration ([core-concepts.md#jäv](../core-concepts.md#jäv-conflict-of-interest)).
- **Asymmetrier:** medlem med juridisk kompetens kan utnyttja formaliabrister → mallarna är skarpt formaliarena (varning, svarsfönster, besvärshänvisning), beslutet är processuellt svårt att angripa om mallen följs.
- **Maktkoncentration / "rensa ut" politiska motståndare:** stämmoöverklagan + revisor-läsrätt + publik statistik gör mönster över tid synliga.
- **Personangrepp under sken av stadgebrott:** stadgehänvisning är obligatorisk; revisor granskar grunden löpande.

## Öppna frågor

- `[ÖPPEN]` Vad händer med ekonomiska skulder vid uteslutning (utebliven avgift)? Avgår de eller kvarstår de civilrättsligt? Förslag: kvarstår — uteslutning är medlemsstatus, inte skuldförsoning. Kassören eskalerar till inkasso enligt vanlig rutin.
- `[ÖPPEN]` Får en utesluten medlem söka inträde igen senare? Stadgan kan reglera. Förslag: tillåt ny ansökan men flagga historiken för styrelsen vid `MEDLEMSKAP_ANSÖKAN_INLÄMNAD` — inte automatisk avslag.
- `[ÖPPEN]` Tröskel för automatisk uteslutning vid utebliven avgift: stadge-bundet, men hur länge ska systemet vänta innan det föreslår initiering? Förslag: 90 dagar efter förfallodatum är default, stadgan kan justera.
- `[ÖPPEN]` Anonymisering vid stämmobehandling — ska medlemmens namn döljas i kallelsen, eller är ärendets natur (uteslutning) så formellt att namn ändå framgår? Förslag: namnet framgår i kallelsen (svensk föreningspraxis), men inte i publika vyer post-stämma.
